---
layout: post
title: "Tools: UV and Marimo"
date: 2025-05-15 00:00:00 -0700
categories: tech
tags: python
---

I've been using some new Python tools lately, [uv](https://docs.astral.sh/uv/)
instead of venv+pip, and [marimo](https://docs.marimo.io/) instead of 
jupyterlab/ipynotebooks. I love jupyterlab, it's how I learned to program 15 
years ago, via jupyter notebooks, and ipython, but as the Python versions have
piled up over the years, and all the different solutions for versions and
environments have cost me hours and hours over the years, it's so nice to find
a tool like uv that Just Works, and a notebook environment like marimo, that
works with uv and also Just Works. Marimo has mroe features than you can shake 
a stick at, but it's dayenu feature is that you can create an environment and
run a notebook from that environment. I never have figured out how to do that
with jupyter--uv and marimo Just Work.


## uv

Installation: `brew install uv`. That's it.

Create project with: `uv init project-name`.

Run a project's entrypoint: `uv run main.py`.

Add a module: `uv add pandas`, or remove: `uv remove module`.

In this case, all of your top level modules are stored in a `pyproject.toml`.

If you need a `requirements.txt` file, then use the command `uv pip compile
pyproject.toml -o requirements.txt`. This will lock the dependencies specified
in your `pyproject.toml` and write them to the `requirements.txt` file.

If you need to pull from a specific branch from a GitHub, then you can do that
like this,

{% highlight console %}
uv add git+https://github.com/<username>/<project> --branch main
{% endhighlight %}


## marimo

You can use `uv` to install things globally using their "tool" subcommand:
`uv tool install tool-name`. I'll install `marimo` with it's recommended extras.
`uv tool install "marimo[recommended]"`

