---
layout: post
title:  "Testing in Python"
date:   2024-03-25 00:00:00 -0700
categories: python
---

## Overall Structure

I structure small projects with an entrypoint named after the project itself,
or simply `app.py`, a directory called `lib/` that contains modules used in
the entrypoint, and a director called `test/` containing tests. Inside the
project root, and both `lib/` and `test/` directories, are empty `__init__.py`
files so that the Python interpreter can load modules from the `lib/` directory
in the `test/` directory.

Below is the typical shape of a small application.

```
project
├── README.md
├── .gitignore
├── __init__.py
├── lib
│   ├── __init__.py
│   ├── adapters.py
│   ├── core.py
│   └── interfaces.py
├── app.config.yaml
├── app.py
├── pyproject.toml
├── requirements.txt
├── test
│   ├── __init__.py
│   ├── data
│   ├── conftest.py
│   └── test.py
└── venv
    └── ...
```


## Testing Modules

Inside `test/test.py` I'll import modules from the sibling directory `lib/` as

{% highlight python %}
from ..lib import (
    adapters,
    core,
    interfaces,
)
{% endhighlight %}


## Organizing Fixtures

I keep all test fixtures in `test/conftest.py` which *magically* does not need
to be imported into each test script.

When a fixture needs to be cleaned up after a test, I yield the fixture instead
of returning it.

{% highlight python %}
import os
import pathlib
import pytest

from ..lib import (
    adapters,
)

@pytest.fixture
def formation_adapter():
    '''Yield a formation adapter and then destroy it.'''
    path = pathlib.Path('test/data/test.duckdb')
    path = str(path.absolute())
    fmn_adapter = adapters.FormationDuckDBAdapter(path)
    fmn_adapter.create_formation_table()
    yield fmn_adapter
    os.remove(path)
{% endhighlight %}


## Running Tests with Coverage

I run scripts from the root of the project using [coverage](https://coverage.readthedocs.io/), [pytest](https://docs.pytest.org/), and [toml](https://docs.python.org/3/library/tomllib.html).
Coverage configuration goes into `pyproject.toml` as,

{% highlight toml %}
[tool.coverage.run]
omit = ["test/*", "*/__init__.py"]
{% endhighlight %}

To run the tests I execute,

{% highlight console %}
coverage run -m pytest test -v
coverate report -m
{% endhighlight %}
