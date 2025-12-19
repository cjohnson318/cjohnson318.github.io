---
layout: post
title: "Simple Textual App"
date: 2025-11-12 00:00:00 -0700
categories: tech
tags: python textual
---

This is a simple [Textual](https://textual.textualize.io/) app that involves an
input textbox, and an output scrollable container, basically a terminal. I
added get, set, list, delete functionality for Python's builtin `dbm` module,
which provides a persistent key-value store.

![Textual Example](/assets/images/textual-example.png)


## Code

{% highlight python %}
from textual.app import App
from textual.widgets import (
    Header,
    Footer,
    Input,
    Button,
    Static,
)
from textual.containers import (
    ScrollableContainer,
    Horizontal,
)

import dbm

DBM_FILE_NAME = 'data.dbm'

class DbmWrapper(App):
    """A Textual app to display 'Hello, World!'."""

    def on_mount(self):
        self.theme = "solarized-light"
        self.user_input.focus()

    def compose(self):
        self.user_input = Input(name="user_input")
        self.compute_button = Button("Compute", id='compute')
        yield Header()
        with Horizontal():
            yield self.compute_button
            yield self.user_input
        with ScrollableContainer(id="output-container"):
            yield Static(id="output")
        yield Footer()

    def clear_input(self):
        self.user_input.value = ''

    def update_output(self, data):
        output = self.query_one('#output', Static)
        output_container = self.query_one('#output-container', ScrollableContainer)
        output.update(data)
        output_container.scroll_end(animate=True)

    def compute(self):
        data = self.user_input.value
        if data == 'kv list':
            with dbm.open(DBM_FILE_NAME, 'c') as db:
                result = 'KV STORE\n'
                for key, val in db.items():
                    result += f'  - {key}: {val}\n'
                self.update_output(result)
        elif data.startswith('kv get '):
            key = data.split('kv get ')[1]
            with dbm.open(DBM_FILE_NAME, 'c') as db:
                if key in db:
                    self.update_output(str(db[key]))
        elif data.startswith('kv del '):
            key = data.split('kv del ')[1]
            with dbm.open(DBM_FILE_NAME, 'c') as db:
                if key in db:
                    del db[key]
        elif data.startswith('kv '):
            data = data.split('kv ')[1]
            key, val = data.split('::')
            with dbm.open(DBM_FILE_NAME, 'c') as db:
                db[key] = val
        self.clear_input()    

    def on_button_pressed(self, event: Button.Pressed):
        if event.button.id == 'compute':
            self.compute()

    def on_input_submitted(self, message: Input.Submitted):
        self.compute()
        message.input.clear()
        message.input.focus()

if __name__ == "__main__":
    app = DbmWrapper()
    app.run()

{% endhighlight %}


## Resources

  - [remusao blog](https://remusao.github.io/posts/python-dbm-module.html)
  - [HN comment section](https://news.ycombinator.com/item?id=32849592)
