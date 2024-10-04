---
layout: post
title:  "Writing a simplified, cross-platform grep clone in Python"
date:   2024-10-02 16:00:00 -0400
categories: ["python", "python3", "windows", "mac", "linux", "CLI", "argparse", "standard library", "grep", "search", "text files"]
---

## Writing a simplified, cross-platform grep clone in Python

In this tutorial, we will explore how to write a simplified, cross-platform version of the `grep` command line utility.

If you don't know, `grep` is a posix utility present on nearly every version of Linux. `grep` takes a regular expression and an optional list of files, then it outputs every line of the files that match the regular expression.

If you don't know what regular expressions are, please stay tuned for a regular expression tutorial that I will have coming soon.

### Step 1: Setting up the basic structure

First, we are going to set up the basic structure of the program.

Take a look at the code, and then we will explore what it is doing:

```python
import sys

def main(argv):
    return 0

if __name__ == "__main__":
    sys.exit(main(sys.argv[1:]))
```

We import the `sys` module on the first line, which provides functions and variables for interacting with the operating system and the Python interpreter.

The `main` function is defined next. It is empty now save for the `return 0`, but we will soon add our logic here.

The `if __name__ == "__main__":` block checks if the script is being run directly (i.e., not being imported as a module by another script). If so, it calls the `main` function with the command-line arguments.

The `sys.argv[1:]` expression returns the command-line arguments passed to the script, excluding the script name itself.

### Step 2: Adding argument parsing

Next, we will add argument parsing to the program.

Take a look at the code, and then we will explore what it is doing:

```python
import sys
import argparse

def main(argv):
    parser = argparse.ArgumentParser(
        prog="grepy",
        description="A grep-inspired tool with different trade-offs",
    )
    parser.add_argument(
        "expression",
        type=str,
        help="The regular expression to search for",
    )
    parser.add_argument(
        "input_files",
        type=argparse.FileType("r"),
        nargs="*",
        default=tuple("-"),
        help="The file(s) to search"
    )
    args = parser.parse_args(argv)
    return 0

if __name__ == "__main__":
    sys.exit(main(sys.argv[1:]))
```

Here, we add `argparse` to the list of imports at the top, which provides a way to define the command-line interface of the program.

The `main` function is then updated to create an `ArgumentParser` object, which defines the available arguments.

In this case, we define two arguments: `expression` and `input_files`.

The `expression` argument is a string that specifies the regular expression to search for, and the `input_files` argument is a list of files to search in.

The `nargs="*"` argument specifies that `input_files` should be a list of files, and the `default=tuple("-")` argument specifies that if no files are provided, the program should search `stdin` (i.e., the `-` character is replaced `stdin`).

The value `"-"` is being passed to the `tuple` factory, I'm doing it like this because the `nargs="*"` will cause any arguments passed to be in a list, but if no arguments are passed the default value will be used. I chose `tuple` instead of `list` because `list`s are mutable and would be attached to the `ArgumentParser` instance which creates an oportunity for the value to change unexpectedly.

### Step 3: Adding loops to go through each line of the files

Next, we will add in the loops to go through each line of each of the files.

Take a look at the code, and then we will explore what it is doing:

```python
import sys
import argparse

def main(argv):
    parser = argparse.ArgumentParser(
        prog="grepy",
        description="A grep-inspired tool with different trade-offs",
    )
    parser.add_argument(
        "expression",
        type=str,
        help="The regular expression to search for",
    )
    parser.add_argument(
        "input_files",
        type=argparse.FileType("r"),
        nargs="*",
        default=tuple("-"),
        help="The file(s) to search"
    )
    args = parser.parse_args(argv)
    for input_file in args.input_files:
        for line in input_file:
            ...
    return 0

if __name__ == "__main__":
    sys.exit(main(sys.argv[1:]))
```

Here, we add loops to go through each line of the files in turn.

We import the `re` module, which provides regular expression matching operations.

The `main` function is then updated to iterate over the files and then to iterate through each line of the file.

### Step 4: Adding support for regular expression search

Next, we are going to add the magic of our script with the Python `re` module

Take a look at the code, and then we will explore what it is doing:

```python
# Add support for regular expression search
import re
import sys
import argparse

def main(argv):
    parser = argparse.ArgumentParser(
        prog="grepy",
        description="A grep-inspired tool with different trade-offs",
    )
    parser.add_argument(
        "expression",
        type=str,
        help="The regular expression to search for",
    )
    parser.add_argument(
        "input_files",
        type=argparse.FileType("r"),
        nargs="*",
        default=tuple("-"),
        help="The file(s) to search"
    )
    args = parser.parse_args(argv)
    expression = re.compile(args.expression)
    for input_file in args.input_files:
        for line in input_file:
            if expression.search(line):
                print(line.strip())
    return 0

if __name__ == "__main__":
    sys.exit(main(sys.argv[1:]))
```

The `re` module was added to the imports at the top.

Then `expression.search(line)` is used to see if the line matches the regular expression, and if it matches, the `print(line.strip())` statement prints the matched line.

### Step 5: Adding support for filenames and line numbers

There are two common options that I almost always end up passing to `grep`, so let's add them now.

Take a look at the code, and then we will explore what it is doing:

```python
import re
import sys
import argparse

def main(argv):
    parser = argparse.ArgumentParser(
        prog="grepy",
        description="A grep-inspired tool with different trade-offs",
    )
    parser.add_argument(
        "-n", "--line-number",
        action="store_true",
        help="Print the line number with the match",
    )
    parser.add_argument(
        "-H", "--with-filename",
        action="store_true",
        help="Print the filename with the match",
    )
    parser.add_argument(
        "expression",
        type=str,
        help="The regular expression to search for",
    )
    parser.add_argument(
        "input_files",
        type=argparse.FileType("r"),
        nargs="*",
        default=tuple("-"),
        help="The file(s) to search"
    )
    args = parser.parse_args(argv)
    expression = re.compile(args.expression)
    for input_file in args.input_files:
        for line_number, line in enumerate(input_file):
            if expression.search(line):
                message = line.strip()
                if args.line_number:
                    message = f"{line_number}: {message}"
                if args.with_filename:
                    message = f"{input_file.name}: {message}"
                print(message)
    return 0

if __name__ == "__main__":
    sys.exit(main(sys.argv[1:]))
```

Here, we add support for adding the filenames and line numbers to the output. 

The first change you will probably notice are the updates the `main` function to define two additional arguments: `--line-number` and `--with-filename`. I tried to add these options with the same names as are used in GNU grep.

The `--line-number` argument specifies whether to print the line number with the match, and the `--with-filename` argument specifies whether to print the filename with the match.

The `enumerate(input_file)` function returns an iterator over the lines of the input file, along with their line numbers so we can add them to the output.

The `if expression.search(line)` call checks if the line matches the regular expression, and the `print(message)` statement prints the matched line with the specified options.

## Conclusion

Thanks for taking the time to read this, I tried to make it as accessible and sensible as I could, but if you have any questions or concerns, please feel free to open an issue in the [Github repo](https://github.com/notesofcliff/notesofcliff.github.io/issues) for this blog.
