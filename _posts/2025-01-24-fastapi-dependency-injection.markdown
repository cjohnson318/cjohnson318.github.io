---
layout: post
title: "FastAPI Dependency Injection"
date: 2025-01-24 00:00:00 -0700
tags: python
---

This pattern combines the Ports and Adapters idea with FastAPI's dependency
injection API. In this example, I'm passing data from a "frontend", a dead
simple `index.html` file, through FastAPI, to a MongoDB instance. I want to
separate my data source from by logic as much as possible, within reason, so
I'm using the Ports and Adapters architecture for setting up my MongoDB
database. I'll collect all of my adapters in the adapters directory, since
MongoDB is a database, I'm putting it into an `adapters.database` module. The
idea is to have a `DatabaseInterface`, which is the "port" in this framework,
and a `MongoAdapter` that implements the `DatabaseInterface`. From FastAPI's
perspective, it's using the `DataBaseInterface` in the application code, except
for a block at the top of `main.py` that configures the `MongoAdapter`.

## Project Organization

This is the directory structure.

{% highlight console %}
project/
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── src/
│   │   ├── __init__.py
│   │   ├── adapters/
│   │   │   ├── __init__.py
│   │   │   └── database.py
│   │   └── main.py
│   ├── test/
│   │   └── __init__.py
│   └── venv/
├── .gitignore
├── README.markdown
├── docker-compose.yaml
└── index.html
{% endhighlight %}

## Docker Configuration

The `docker compose` configuration is minimal, just a `web` container for
FastAPI, and a `db` container for MongoDB.

{% highlight yaml %}
services:
  web:
    build: ./backend
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - MONGO_URI=mongodb://db:27017
  db:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
{% endhighlight %}

The Dockerfile responsible for the FastAPI server is also minimal.

{% highlight docker %}
FROM python:3.13

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY src/ .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
{% endhighlight %}

For completeness, this is the `requirements.txt`.

{% highlight console %}
annotated-types==0.7.0
anyio==4.8.0
click==8.1.8
dnspython==2.7.0
fastapi==0.115.7
h11==0.14.0
httptools==0.6.4
idna==3.10
pydantic==2.10.5
pydantic_core==2.27.2
pymongo==4.10.1
python-dotenv==1.0.1
PyYAML==6.0.2
sniffio==1.3.1
starlette==0.45.2
typing_extensions==4.12.2
uvicorn==0.34.0
uvloop==0.21.0
watchfiles==1.0.4
websockets==14.2
{% endhighlight %}

## FastAPI

This is the basic idea for the server. I use the `MongoAdapter` at the top of
file, and then the rest of the routes use a `db: DatabaseDependency`, unaware
of any other details. FastAPI's dependency injection API prefers that users use
the `typing.Annotated` type to wrap the type, in our case `DatabaseInterface`,
and the metadata, the `mongo_dependency()` function wrapped in FastAPI's
`Depends` object.

In this example, we're communicating with the client over websockets, and then
writing to mongo through the `DatabaseDependency` abstraction.

{% highlight python %}
# built-in imports
import json
import logging
from typing import Annotated
import os

# in-house imports
from adapters.database import DatabaseInterface, MongoAdapter

# 3rd-party imports
from fastapi import FastAPI, HTTPException, WebSocket, WebSocketDisconnect, Depends

app = FastAPI()

# logging configuration

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# db configuration

MONGO_URI = os.getenv('MONGO_URI', 'mongodb://localhost:27017')

async def mongo_dependency():
    db = MongoAdapter(MONGO_URI, "database", "items")
    try:
        yield db
    finally:
        db.close()

DatabaseDependency = Annotated[DatabaseInterface, Depends(mongo_dependency)]

# routes

@app.get("/")
async def root():
    return {"message": "hello"}

@app.post("/items")
async def create_item(item: dict, db: DatabaseDependency):
    try:
        result = db.create(item)
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Error creating item: {e}")

@app.get("/items")
async def read_items(db: DatabaseDependency):
    try:
        items = db.read()
        return items
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Error getting items: {e}")

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket, db: DatabaseDependency):
    await websocket.accept()  # Accept the connection
    logger.info(">>> Client connected")
    try:
        while True:
            data = await websocket.receive_text()  # Receive data from the client
            data = json.loads(data)
            logger.info(f">>> Received from client: {data}")
            item = {data.get("key"): data.get("value")}
            try:
                result = db.create(item)
                await websocket.send_text(f"Message received: {result}")
            except Exception as e:
                logger.error(f"Error creating item: {e}")
    except WebSocketDisconnect:
        logger.info(">>> Client disconnected")
{% endhighlight %}

Finally, in the `src/adapters/database.py` module I have this. The
`DatabaseInterface` defines the shape of what the client code can expect, and
the `MongoAdapter` implements that interface. When it is time to change the
database, then we would write another class implementing the interface.

{% highlight python %}
from pymongo import MongoClient

class DatabaseInterface:

    def create(self, item: dict):
        pass

    def read(self):
        pass

    def close(self):
        pass

class MongoAdapter(DatabaseInterface):

    def __init__(self, uri: str, database: str, collection: str):
        try:
            self.client = MongoClient(uri)
            self.db = self.client[database]
            self.collection = self.db[collection]
        except Exception as e:
            self.close()
            raise Exception(f"Error connecting to MongoDB: {e}")

    def create(self, item: dict):
        try:
            result = self.collection.insert_one(item)
            item["_id"] = str(result.inserted_id)
            return item
        except Exception as e:
            raise Exception(f"Error creating item: {e}")

    def read(self):
        try:
            items = list(self.collection.find({},{"_id":0})) # exclude the mongo _id
            for item in items:
                if "_id" in item:
                    item["_id"] = str(item["_id"])  # Convert ObjectId to string
            return items
        except Exception as e:
            raise Exception(f"Error getting items: {e}")
        
    def close(self):
        self.client.close()
{% endhighlight %}

## The Client

All this does is allow a user to submit a key value pair over a websocket, and
then see the result that comes back from the server.

{% highlight html %}
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket Example</title>
</head>
<body>
    <h1>WebSocket Test</h1>
    <input type="text" id="keyInput" placeholder="Enter key">
    <input type="text" id="valueInput" placeholder="Enter value">
    <button onclick="sendMessage()">Send</button>
    <div id="messages"></div>

    <script>
        const websocket = new WebSocket("ws://localhost:8000/ws"); // Replace with your server URL

        websocket.onopen = (event) => {
            console.log("WebSocket connection opened:", event);
            displayMessage("Connected to server!");
        };

        websocket.onmessage = (event) => {
            console.log("Received from server:", event.data);
            displayMessage("Server: " + event.data);
        };

        websocket.onclose = (event) => {
            console.log("WebSocket connection closed:", event);
            displayMessage("Disconnected from server!");
        };

        websocket.onerror = (error) => {
            console.error("WebSocket error:", error);
            displayMessage("Error: " + error);
        };

        function sendMessage() {
            const keyInput = document.getElementById("keyInput");
            const valueInput = document.getElementById("valueInput");
            if (keyInput.value && valueInput.value) {
                const packet = {
                    key: keyInput.value,
                    value: valueInput.value
                }
                websocket.send(JSON.stringify(packet));
                displayMessage(JSON.stringify(packet));
                messageInput.value = "";
            }
        }

        function displayMessage(message) {
            const messagesDiv = document.getElementById("messages");
            const messageElement = document.createElement("p");
            messageElement.textContent = message;
            messagesDiv.appendChild(messageElement);
        }
    </script>
</body>
</html>
{% endhighlight %}