---
layout: post
title:  "Introducing command.py - a flexible template for Python CLIs"
date:   2024-09-30 16:00:00 -0400
categories: ["python", "python3", "windows", "mac", "linux", "CLI", "argparse", "logging"]
---

**Introducing `command.py`: A Flexible CLI Template for Your Next Project**

As a developer, you're always on the lookout for tools that make your life easier. A well-designed Command Line Interface (CLI) can be a game-changer, allowing you to focus on the things that matter most – writing code, not wrestling with command-line flags. That's why we've put together `command.py`, a flexible CLI template that includes some useful linux-inspired patterns.

*NOTE*: You can find the code in the repository [here](https://github.com/notesofcliff/python_cli_template), please feel free to open issues or submit pull requests. 

**Logging Made Easy**

One of the key features that sets `command.py` apart is its logging system. With a simple `-v` flag, you can control the verbosity of your logs. What's clever about this is that you can specify `-v` multiple times to increase the logging level with each occurrence. For example, `-vvvv` would output info messages, `-vvvvv` would output debug messages, and so on. This pattern makes it easy to fine-tune your logging to suit your needs.

But that's not all – users can also specify a file to write logs to using the `--log-file` option. Logs default to going to `stderr`, but if a user provides a log file, logs will still be written to `stderr` as well as the logfile. This flexibility makes it easy to customize your logging to suit your needs.

**Error Codes, Simplified**

When it comes to error handling, `command.py` uses a simple global dictionary called `RETURN_CODES` to keep track of all possible return codes. This makes it easy to add new error codes or modify existing ones without having to modify the underlying code. With `RETURN_CODES`, you can quickly and easily handle errors in a consistent and predictable way.

**Input and Output Options**

One of the things that makes `command.py` so flexible is its input and output options. By default, the input is `stdin` and the output is `stdout`. However, users can also specify a file path to read or write to files on disk. This is where the magic of Linux happens – if a user passes a dash `-` as a file, it's treated as one of the standard files (i.e., `stdin` or `stdout`). For example, `python command.py - < input.txt` would read from `input.txt`, while `python command.py - > output.txt` would write to `output.txt`.

**Free and Open-Source**

Finally, we want to emphasize that `command.py` is free to use and is released under the GPL v3. This means that you can modify and distribute the code as you see fit, without worrying about licensing restrictions.

We hope you find `command.py` to be a valuable tool in your development workflow. Whether you're building a new CLI tool or just need a template to get started, this script is designed to make your life easier. So go ahead, give it a try, and see what you can create!