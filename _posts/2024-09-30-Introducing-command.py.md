---
layout: post
title:  "Introducing command.py - a flexible template for Python CLIs using the standard library"
date:   2024-09-30 16:00:00 -0400
categories: ["python", "python3", "windows", "mac", "linux", "CLI", "argparse", "logging", "standard library"]
---

## Introducing `command.py`
*A Flexible CLI Template using just the standard library*

I often need to create little CLI scripts, and I usually reach for Python for this task. Now there are many easy ways to make sensible CLIs with Python such as [click](https://click.palletsprojects.com/en/8.1.x/), [cliff](https://github.com/openstack/cliff) and others.

However, reaching for third party libraries, while frequently necessary, can lead to problems later on as you continue to support your utility. Sometimes libraries become unmaintained or introduce incompatibilities and leave you with extra work updating all of your scripts.

Luckily, Python's standard library contains everything you need to have a sensible, featureful CLI and has a pretty strong history of maintaining backwards compatibility. 

In this article, we will explore the template I made to kickstart simple CLI scripts using the Python standard library.

## The code

The template (at of the time of writing) is shown below, I am pasting it here in its entirety and I will refer to pieces of the code throughout this article.

*NOTE*: You can find the latest version of this template in the repository [here](https://github.com/notesofcliff/python_cli_template), please feel free to open issues or submit pull requests. 

```python
"""This script is meant to serve as an example and a
template for writing programs with a Command Line
Interface.

Features Include:
    * -vvv style logging verbosity arguments
    * Option to output logs to a file in addition to stderr
    * Sensible Return Codes
    * Intuitive code organization
    * Input file that defaults to stdin
    * Output file that defaults to stdout 

This code is released under the GPL v3.

Feel free to copy this file `command.py` for the scaffolding
of the script and the file `test_command.py` for the tests.
"""
import sys
import pathlib
import logging
import argparse

LOG_LEVELS = [
    'CRITICAL',
    'FATAL',
    'ERROR',
    'WARNING',
    'INFO',
    'DEBUG',
]

RETURN_CODES = {
    "SUCCESS": 0,
    "UNHANDLED_EXCEPTION": 1,
}

def parse_args(argv):
    """This function is responsible for parsing the command line
    arguments as well as any required post-processing.
    """
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "source",
        type=argparse.FileType("r"),
        default="-",
        nargs="?",
        help="The input file. Defaults to stdin",
    )
    parser.add_argument(
        "destination",
        type=argparse.FileType("w"),
        default="-",
        nargs="?",
        help="The output file. Defaults to stdout.",
    )
    parser.add_argument(
        "-v", "--verbose",
        action="count",
        default=0,
        help="If specified, increases logging verbosity. "
             "Can be specified multiple times (ie. -vvv)",
    )
    parser.add_argument(
        "-l", "--log-file",
        type=pathlib.Path,
        help="If provided, it must be the path to a file. The "
             "given file will be used for logging. NOTE: This file"
             "will be overwritten."
    )
    args = parser.parse_args(argv)
    args.verbose = LOG_LEVELS[min(args.verbose, len(LOG_LEVELS)-1)]
    return args

def configure_logging(level, filename=None):
    """This function configures the Python logging system
    based on the level and filename provided.
    """
    handlers = [
        logging.StreamHandler(
            stream=sys.stderr,
        ),
    ]
    if filename is not None:
        handlers.append(
            logging.FileHandler(
                filename,
                mode="w",
            )
        )

    logging.basicConfig(
        level=level,
        handlers=handlers,
    )

def _main(args):
    """This function is responsible for executing the application logic.
    """
    log = logging.getLogger(__name__)
    for line in args.source:
        log.debug(f"Found line: {line}")
        bytes_written = args.destination.write(line)
        log.debug(f"Wrote {bytes_written} bytes to {args.destination}")
    log.info(f"Exiting successfully")
    return RETURN_CODES["SUCCESS"]


def main(argv):
    """This method is executed when this script is called
    from the command line.

    This function is responsible for the following:

    * parse command line arguments
    * Configure logging based on arguments
    * Call the function `_main` which should hold the application logic
    * return the return code as determined by function `_main`
    """
    log = logging.getLogger(__name__)
    try:
        args = parse_args(argv)
    except Exception as exception:
        log.critical(f"An unhandled exception has occurred: {exception}")
        return RETURN_CODES["UNHANDLED_EXCEPTION"]
    configure_logging(args.verbose, args.log_file)
    log = logging.getLogger(__name__)
    log.debug(f"Received args: {args}")
    try:
        return_value = _main(args)
    except Exception as exception:
        log.critical(f"An unhandled exception has occurred: {exception}")
        return_value = RETURN_CODES["UNHANDLED_EXCEPTION"]
    return return_value

if __name__ == "__main__":
    try:
        rc = main(sys.argv[1:])
    except:
        rc = RETURN_CODES["UNHANDLED_EXCEPTION"]
    finally:
        logging.shutdown()
    sys.exit(rc)
```

## What the Script does

As a template, there was little point in writing complex application logic, but I still wanted to show a use-case for context, so I decided to give the script the task to take a source file and writes its contents to a destination file. A simple echo program.

## The Execution

When executed, the interpreter will run from top-to-bottom defining the global variables and functions until it encounters the line `if __name__ == "__main__":`. That's where our application logic begins.

Inside the `if __name__ == "__main__":` block, we execute the `main` function in a try...except block. Any exceptions that are caught cause a generic return code. Finally, we shutdown the logging and exit with our return code which was either returned by main or assigned the generic error code `UNHANDLED_EXCEPTION`.

The `main` function handles the following tasks:

* parse the command line arguments
* Configure logging based on user-provided arguments
* Call the function `_main`
* return the return code as returned by function `_main`

The `_main` function is where the bulk of our application logic will live. In our case, the `_main` function logs some information and copies the contents of source to destination.

That's the basic execution flow, let's look next at some specifics.

### Input and Output Files

One of the things that makes `command.py` so flexible is its input and output options. By default, the input is `stdin` and the output is `stdout`. However, users can also specify a file path to read or write to files on disk.

Let's look at where those arguments are defined:

```python
...
parser.add_argument(
    "source",
    type=argparse.FileType("r"),
    default="-",
    nargs="?",
    help="The input file. Defaults to stdin",
)
parser.add_argument(
    "destination",
    type=argparse.FileType("w"),
    default="-",
    nargs="?",
    help="The output file. Defaults to stdout.",
)
...
```

As you can see in the code above, we're creating two arguments:

* **source**: This argument represents the input file(s) for our program. We're using argparse.FileType("r") to specify that this argument should accept a file path that opens for reading ("r"). The default="-" means that if no input file is provided, it will default to the standard input (stdin).
* **destination**: This argument represents the output file(s) for our program. We're using argparse.FileType("w") to specify that this argument should accept a file path that opens for writing ("w"). The default="-" means that if no output file is provided, it will default to the standard output (stdout).

## Logging Made Easy (for the user)

One of the key features that sets `command.py` apart is how flexible the logging is on the user's end.

With a simple `-v` flag, you can control the verbosity of your logs. What's clever about this is that you can specify `-v` multiple times to increase the logging level with each occurrence. For example, the default is `CRITICAL` and `-v` would change the log level to `FATAL`, `-vvvv` would output info messages, `-vvvvv` would output debug messages. This pattern makes it easy to fine-tune the logging at runtime to suit your needs.

Let's look at how that works:

```python
LOG_LEVELS = [
    'CRITICAL',
    'FATAL',
    'ERROR',
    'WARNING',
    'INFO',
    'DEBUG',
]

def parse_args(argv):
    parser = argparse.ArgumentParser()
    ...
    parser.add_argument(
        "-v", "--verbose",
        action="count",
        default=0,
        help="If specified, increases logging verbosity. "
             "Can be specified multiple times (ie. -vvv)",
    )
    ...
    args = parser.parse_args(argv)
    args.verbose = LOG_LEVELS[min(args.verbose, len(LOG_LEVELS)-1)]
```

The first part of the verbosity option is the `action="count""` which causes the `ArgumentParser` to count the number of times the option was specified at the command line.

Next, let's examine where the real magic happens. At the top of the script, in the `global` scope, we define a constant called `LOG_LEVELS`.

`LOG_LEVELSis` is used in the semi-confusing line `args.verbose = LOG_LEVELS[min(args.verbose, len(LOG_LEVELS)-1)]`. Here we are pulling out an item from `LOG_LEVELS` based on the count of `-v`s present in the arguments. The weird part is the call to `min` inside of the square brackets (ie. `[]`), this is needed in case someone specifies too many `-v`s in which case we will select the most verbose option available (ie. `DEBUG`)

The next piece of logging configuration exposed to the user is the ability to specify a file to write logs to using the `--log-file` option. Logs default to going to `stderr`, but if a user provides a log file, logs will be written to the log file in addition to `stderr`. This flexibility makes it easy for your users to customize their logging to suit their own needs.

**Error Codes, Simplified**

When it comes to error handling, `command.py` uses a simple global dictionary called `RETURN_CODES` to keep track of all possible return codes. This makes it easy to add new error codes or modify existing ones without having to modify the underlying code.

In order to use the `RETURN_CODES` dict, simply return the relevant return code from the dict, like we do in `_main` in the line `return RETURN_CODES["SUCCESS"]`.

**Free and Open-Source**

Finally, we want to emphasize that `command.py` is free to use and is released under the GPL v3. This means that you can modify and distribute the code as you see fit, without worrying about licensing restrictions.

We hope you find `command.py` to be a valuable tool in your development workflow. Whether you're building a new CLI tool or just need a template to get started, this script is designed to make your life easier. So go ahead, give it a try, and see what you can create!

