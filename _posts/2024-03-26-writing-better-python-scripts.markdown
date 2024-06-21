---
layout: post
title: 'Writing Better Python Scripts'
date: 2024-03-26 00:00:00 -0700
tags: python
---

The great thing about expressive, interpreted, dynamic languages like Python and Ruby is that one can write a working script extremely quickly. However, as scripts grow larger and larger, they become more difficult to maintain. One way to make script maintenance easier is to follow best practices, and optimize for readability and usage from the very beginning.

Optimizing for readability involves providing type hints for function signatures, and helpful doctstrings. Optimizing for usage involves helpful error messages, progress bars, adjustable verbosity, refactoring prompts for user input into command line arguments, and moving magic strings and numbers into constants at the top.

## Typing

Even though Python uses duck typing and ignores type hints, they can be invaluable to you when you read your code later. I never regret adding type type hints to my function signatures. Documentation for using type hints in Python can be found [here](https://docs.python.org/3/library/typing.html).

## Docstrings

You should also include docstrings in your functions so that you just add a question mark to your function and see its docstring in the REPL environment later, or an IDE if you use one. This saves you time trying to find that function in source code later. Google and NumPy have separately developed guides for docstrings. You can find out more about those guides [here](https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html).

## Imports

Organizing import statements alphabetically helps users quickly discover what a script is importing. Sometimes it is helpful to further break imports out into three, sorted lists: built-in modules, third-party modules, and in-house modules. I sometimes list third-party licenses also.

## Command Line Arguments

The first thing people usually do is `import sys`, and then use `sys.argv` to gather and parse command line arguments. This is okay when there's one command line argument, but it's usually better to use an argument parser. I like to use the argparse module since it is built in to modern Python distributions, but there are other options like [docopt](http://docopt.org/) and [click](https://click.palletsprojects.com/).

I like to use a YAML/TOML file to configure a script if it requires more than one or two arguments. This way you can document the arguments you've used, you can easily "replay" a script, and you can add helpful metadata to the config file for future-you by using the comment functionality provided by YAML/TOML.

## Logging

Another common thing people do is put print statements everywhere. This is another quick and easy solution that doesn't scale well. For one thing, you cannot control verbosity without commenting out or deleting print statements. With the logging module, you can set the level of logging you would like the report, or you can allow the user to do so through a command line argument. You can also have multiple loggers in one script sending different information to different logs.

## Error Messages

When writing exceptions, resist the urge to raise a `RuntimeError` for every issue. Make sure the exception matches the actual problem, and provide a helpful message. Remember that you can always create your own custom exceptions for special cases.

## Magic Numbers and Strings

A magic number, or magic string, is any literal number or string hanging out in the middle of your code is not assigned to a variable. The is a problem because it makes code more difficult to debug and refactor. When numbers and strings are defined by a variable, you only need to make one edit to change all of the instances of that variable.

Refactor all hard-coded magic strings and magic numbers into variables at the top of your script, under the imports, and don't change them in the middle of your script. You shouldn't have to spend time debugging errors caused by these random numbers and strings.

## No User Interaction at Runtime

It seems like it's a good idea to request user input in the middle of a script, but it generally never is. Scripts that require user-input at runtime cannot be automated easily. Users that forget a script requires extra input at runtime will be annoyed and frustrated when they come back from getting a coffee, and the script is sitting there waiting for more input. Scripts that do not require user input at runtime can easily be called by other scripts, in other languages.

However, if your script needs to provide user input to another, external script during the other script's runtime, then you can solve that problem with [pexpect](https://pexpect.readthedocs.io/en/stable/). For example, if you'd like to write a script to automate another script that requires user to input a username or port number at some point, then you can use pexpect to monitor that script and supply the username or port number when it's needed.

## Environment Paths

Don't hard-code paths like `C:\\some\path\` or `/some/path`. Instead, use Python's [os](https://docs.python.org/3/library/os.html) or [pathlib](https://docs.python.org/3/library/pathlib.html) module to build paths that will be appropriate for the environment that your script will run in. For example, `os.path.join('some', 'path')` will render correctly whether it is run in a Windows environment, or a Linux/Mac environment.

## Virtual Environments

One great way to avoid "it works on my machine" problems is to use a virtual environment, and track requirements in a `requirements.txt` file in the root of your project directory. There are a number of exciting solutions for this including [poetry](https://python-poetry.org/). I tend to use the built in [venv](https://docs.python.org/3/library/venv.html) module, because I favor built in solutions over external tools whenever possible.

## SOLID Principles

I like to separate the functional core of the business logic from external dependencies as much as possible. This makes it much easier to perform testing, and plug in different external dependencies later. I use an entrypoint script, usually called `app.py` or named after the project itself, where I consolidate the specification of external dependencies and inject those dependencies into the functional core of the application. Then, as dependencies change, as long as everything adheres to the same protocols, the core logic should not have to change.

## Testing

You should absolutely, always test things. I like to use [pytest](https://docs.pytest.org/) and [coverage](https://coverage.readthedocs.io/) to run tests. I keep my coverage configuration in `pyproject.toml` under the `[tool.coverage.run]` section. I talk  more about testing [here](https://cjohnson318.github.io/2024/03/25/testing-in-python.html). The better you separate concerns according to SOLID principles, the easier it is to write tests.

## Versioning

If you're distributing code, then it should be versioned. One easy way to do this is to track the version as a string using a `_version.py` file in the root of your project.

{% highlight python %}
# _version.py
__version__ = '1.2.3'
{% endhighlight %}

You should tag versions using git in order to easily retrieve older versions as needed. (More information about using git tags [here](http://localhost:4000/2024/03/25/git-tagging.html).)

For an extra layer of sanity, include a `CHANGELOG` in your project. This is different from just copy/pasting your git log. Maintaining a changelog describes what was added, changed, deprecated, removed, or fixed at in every version. It is intended to be consumed by humans and answer questions like, "Are we still doing X in version Y?". The site [keepachangelog.com](https://keepachangelog.com/) is the best resource on this topic.

## Bonus: Debugging

Before Python 3.7 you needed to use `import pdb; pdb.set_trace()` now you can
simply say `breakpoint()`. This will open a debugging environment. You can use
`p` to print a variable, `n` to go to the next line, `s` to step into an 
instruction, `c` to go to the next breakpoint, and `q` to quit. These are the
most basic and intuitive commands. More information can be found [here](https://docs.python.org/3/library/pdb.html).

## Bonus: Windows Executables

I use [PyInstaller](https://pyinstaller.org/en/stable/) for building binary executable EXEs for Windows out of Python projects. The downside is that this solution does not support cross compilation, so you still need a Windows machine in order to build the executable.

## Bonus: Iterate Over Large Files

A common mistake is to write `open('data.csv', 'r').readlines()`. This is a quick and easy solution to read every line of a file into memory. However, if a file is too large to fit into memory, then this will not work. Instead, you should read each line of a file, individually, since these will generally fit into memory. See the code below for an example.

## Bonus: Progress Bars

For long running scripts, you can include a progress bar. I have used tqdm and progressbar2. Below is an example of using tqdm.

## Example

This is an example script with the boilerplate for setting up logging to the console, command line argument parsing, a progress bar, and a for loop to iterate over the contents of an arbitrarily large text file without reading all of it into memory at once.

{% highlight python %}
#!/usr/bin/python3
# app.py

# built-in modules
import argparse
import logging
import os
import pdb

# third-party modules
import tqdm # MIT

# in-house modules
import lib.business_logic

# constants
LOG_FORMAT = '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
LOG_LEVEL = None

class FileNotFoundError(Exception):
    pass

if __name__ == '__main__':

    # configure progressbar (before logging)
    progressbar.streams.wrap_stderr()

    # configure arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('csv', type=str, help='Name of CSV file.')
    parser.add_argument(
        'output_dir',
        nargs='?',
        type=str,
        help='Name of output directory', default='out'
    )
    parser.add_argument('-v', '--verbose', action='store_true')

    # manage arguments
    args = parser.parse_args()
    if args.verbose:
        LOG_LEVEL = logging.DEBUG
    else:
        LOG_LEVEL = logging.INFO

    # configure logging
    logger = logging.getLogger(__file__)
    logger.setLevel(LOG_LEVEL)
    formatter = logging.Formatter(LOG_FORMAT)
    console_handler = logging.StreamHandler()
    console_handler.setLevel(LOG_LEVEL)
    console_handler.setFormatter(formatter)
    logger.addHandler(console_handler)

    # guard statements
    if os.path.isfile(args.csv):
        logger.info(f'Found CSV {args.csv}.')
    else:
        logger.error(f'File {args.csv} not found.')
        raise FileNotFoundError(f'Could not find {args.csv}')

    # business logic imported from another module
    with tqdm.tqdm(total=os.path.getsize(args.csv)) as pbar:
        with open(args.csv, 'r') as fh:
            # read lines of a file one at a time, instead of all at once
            for line in fh:
                lib.business_logic.operation(line)
                pbar.update(len(line))

    logger.debug('Done.')
{% endhighlight %}
