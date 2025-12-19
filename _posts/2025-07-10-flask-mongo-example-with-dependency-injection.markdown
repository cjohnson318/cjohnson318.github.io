---
layout: post
title: "Simple Flask Server with MongoDB"
date: 2025-07-10 00:00:00 -0700
categories: tech
tags: python mongo
---

The goal of this blog post is to write a simple server in Flask that uses
dependency injection to support testing. Here, the server is in `src/app.py`,
and the routes supported by the server are in `src/routes.py`. Both `app` and
`routes` reference `src/db.py` which contains some type aliases, and functions
that take `Config` classes that return database primitives: a client and a
database object.

On the testing side, `test/conftest.py` holds a lot of setup code controlled by
`Config` objects, while `test/test.py` imports from `test/conftest.py` and uses
those resources automagically.


## App Organization

I've split my application into a `src/` directory holding production code, and
a `test/` directory holding testing code. Ideally, you could then build a staged
`Dockerfile` that builds one environment for the test environment, and another
for the production environment, but that's beyond the scope of this post right
now.

```
.
├── Dockerfile
├── README.md
├── docker-compose.yaml
├── pyproject.toml
├── src
│   ├── __init__.py
│   ├── app.py
│   ├── config.py
│   ├── db.py
│   ├── requirements.txt
│   └── routes.py
├── test
│   ├── __init__.py
│   ├── conftest.py
│   └── test.py
└── uv.lock
```


## Dependency Management

I used `uv` for setting up my environment, but within the `Dockerfile`, I've
used the built-in `pip` and `requirements.txt` pattern.

To convert from `uv` to to `pip`/`requiremtents.txt`, run the following

{% highlight console %}
uv pip compile pyproject.toml -o requirements.txt
{% endhighlight %}

And then replace the `requirements.txt` in the `src/` directory.


## Docker Compose

For now, we're creating a simple server and mounting the `src` and `test`
directories.

{% highlight yaml %}
services:

  mongodb:
    image: mongo:latest
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: user
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongo_data:/data/db

  app:
    build: .
    container_name: app
    ports:
      - "5000:5000"
    depends_on:
      - mongodb
    volumes:
      - ./src:/app/src
      - ./test:/app/test

volumes:
  mongo_data:
{% endhighlight %}


## Dockerfile

The main thing here is that we're using `gunicorn` instead of `python app.py`.

{% highlight dockerfile %}
FROM python:3.13-slim-bullseye AS build

ENV PYTHONPATH=/app:$PYTHONPATH

WORKDIR /app/src

COPY ./src /app/src
COPY ./test /app/test

RUN pip install --upgrade pip
RUN pip install -r requirements.txt

EXPOSE 5000

ENTRYPOINT ["gunicorn"]
CMD ["-k", "gevent", "-w", "3", "--bind", "0.0.0.0:5000", "app:app"]
{% endhighlight %}


## Config

These classes are used to hold configuration options for the production and
test environments. This pattern uses a Python class with class variables.

{% highlight python %}
class Config:
    pass

class RunConfig(Config):
    MONGO_HOST = 'mongodb'
    MONGO_PORT = 27017
    MONGO_DBNAME = 'my_database'
    MONGO_URI = f"mongodb://user:password@{MONGO_HOST}:{MONGO_PORT}"

class TestConfig(Config):
    MONGO_HOST = 'mongodb'
    MONGO_PORT = 27017
    MONGO_DBNAME = 'my_database_test'
    MONGO_URI = f"mongodb://user:password@{MONGO_HOST}:{MONGO_PORT}"
{% endhighlight %}


## Flask App

Here, we're defining a function that takes a database object and returns a
Flask app. Inside the `get_app` function we create the app object, and register
a `Blueprint` of routes, by calling a function that takes a database object and
returns a routes object. This way, `get_app` acts like a factory: it takes a
database connection, and returns an app that uses that database.

In production, gunicorn takes the `app` object and passes HTTP requests to it.
In test, we pass a test database to the app, and pass fake HTTP requests to it
through its test client object.

{% highlight python %}
from flask import (
    Flask,
)
from src.db import (
    MongoDb,
    get_mongo_client,
    get_mongo_db,
)

from src.config import RunConfig
from src.routes import keyval_bp_factory

def get_app(db: MongoDb) -> (Flask):
    app = Flask(__name__)
    app.register_blueprint(keyval_bp_factory(db))
    return app

mongo_client = get_mongo_client(RunConfig)   
mongo_db = get_mongo_db(RunConfig, mongo_client)
app = get_app(mongo_db)
{% endhighlight %}


## Database Configuration

This is pretty simple, we're defining some simple aliases using the new builtin
`type` functionality in Python 3.10. and we're creating wrappers for pymongo
so that we can quickly and easily configure a client and a database object using
our `Config` objects.

{% highlight python %}
import pymongo

from src.config import Config

type MongoClient = pymongo.synchronous.mongo_client.MongoClient
type MongoDb = pymongo.synchronous.database.Database

def get_mongo_client(config: Config) -> MongoClient:
    client: MongoClient = pymongo.MongoClient(config.MONGO_URI)
    return client

def get_mongo_db(config: Config, client: MongoClient) -> MongoDb:
    db: MongoDb = client[config.MONGO_DBNAME]
    return db
{% endhighlight %}


## Flask App Routes

These routes supprt a simple key value store. You can add items, and you can get
all of the items back.

{% highlight python %}
from flask import (
    Blueprint,
    jsonify,
    request,
)
from src.db import (
    MongoDb,
)

def keyval_bp_factory(mongo_db:MongoDb) -> Blueprint:

    keyval_bp = Blueprint("keyval_bp", __name__)

    @keyval_bp.route('/')
    def hello_world():
        return 'Hello from Flask with tMongoDB!'

    @keyval_bp.route('/add', methods=['POST'])
    def add_item():
        data = request.get_json()
        if not data:
            return jsonify({"error": "Data is missing"}), 400
        if "key" not in data:
            return jsonify({"error": "Key attribute is missing in data"}), 400
        item = {
            "key": data['key'],
            "val": data.get('val', 'EMPTY'),
        }
        result = mongo_db["mycollection"].insert_one(item)
        return jsonify({"message": "Item added", "id": str(result.inserted_id)}), 201

    @keyval_bp.route('/items', methods=['GET'])
    def get_items():
        items = []
        for item in mongo_db["mycollection"].find({}, {'_id': 0}):  # Exclude _id from response
            items.append(item)
        return jsonify(items)

    return keyval_bp
{% endhighlight %}


## Testing

This is the `conftest.py` file that holds the pytest fixtures used by the test.

{% highlight python %}
import pytest

from flask import Flask

import src.app as app
import src.config

CONFIG = src.config.TestConfig

@pytest.fixture(scope='session')
def mongo_client():
    client = app.get_mongo_client(CONFIG)
    yield client
    client.close()

@pytest.fixture(scope='session')
def mongo_db(mongo_client):
    db = app.get_mongo_db(CONFIG, mongo_client)
    yield db
    db.client.drop_database(CONFIG.MONGO_DBNAME)

@pytest.fixture(scope='function')
def flask_app(mongo_db) -> Flask:
    """
    Provides a Flask application instance configured for testing.
    The app uses the test database provided by the `mongo_db` fixture.
    """
    aut = app.get_app(mongo_db) # aut is App Under Test
    aut.config['TESTING'] = True # Enable Flask's testing mode
    yield aut 

@pytest.fixture(scope='function')
def flask_client(flask_app: Flask):
    """
    Provides a test client for the Flask application.
    This client can be used to simulate HTTP requests to the app.
    """
    with flask_app.test_client() as client:
        yield client

{% endhighlight %}

Here is the test code

{% highlight python %}
DATA_ITEM = {
    "key": "phone",
    "val": "iphone",
}

def test_api_root(flask_client):
    resp = flask_client.get("/")
    assert resp.status_code == 200

def test_api_add(flask_client):
    resp = flask_client.post(
        "/add",
        json=DATA_ITEM,
        content_type="application/json",
    )
    assert resp.status_code == 201

def test_api_list(flask_client):
    resp = flask_client.get("/items")
    assert resp.status_code == 200
    assert resp.json == [DATA_ITEM] 
{% endhighlight %}


## Tooling

Now we can run the app with

{% highlight console %}
docker compose up --build
{% endhighlight %}

And we can test locally with

{% highlight console %}
docker exec -it app pytest ../test/test.py -vv
{% endhighlight %}


## Extending Functionality

From here, we can use a similar pattern to add a connections to Redis or
Postgres, and still have a relatively simple way access those databases in the
test environment. The trick of the whole thing is to not hard-code the app and
the database together, but to return the database and the app from simple
factory functions that use a single standardized configuration object, either a
custom class, or a built-in data structure, like a dictionary.

