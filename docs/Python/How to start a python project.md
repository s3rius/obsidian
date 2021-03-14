# How to start a project
You have several options. For example you can store a bunch of `.py` files in repo on github and be fully satisfied, or you can add the `setup.py` script and publish it on pypi, but the best option as I can see is to start and develop your project with poetry.

Why?

`Poetry` has many features. I'm here to tell you about them, and how to use it and what to do.


Let's create a project with poetry.
```console
$ poetry new new_proj
Created package new_proj in new_proj
$ cd new_proj
```

This command generates project with the following structure:
```
new_proj
├── new_proj
│   └── __init__.py
├── pyproject.toml
├── README.rst
└── tests
    ├── __init__.py
    └── test_new_proj.py
```

It's our brand new project with several stubs. There is nothing unusual. All the code you want to write will be placed in a directory with the same name as the project. `README.rst` is the default README. `pyproject.toml` contains meta-data about project, such as dependencies, description, extra installation options and more. You can read about `pyproject.toml` [here](https://python-poetry.org/docs/pyproject/).

## How to deal with poetry

All the information you need you can find in official documentation poetry. I'll tell you about basic stuff here.

### Dependency management

To add a dependency to your project you can simply run this:
```console
$ poetry add ${dependency}
```

This command finds the last convinient version of the dependency and writes it to the `pyproject.toml` and `poetry.lock`.

To add dependency only for development, such as linters or formatters,
you can add the `--dev` flag.

```console
$ poetry add ${dependency} --dev 
```

To remove dependency you simply need to replace `add` with `remove`.

Working example:
```console
$ poetry add loguru   
Using version ^0.5.3 for loguru  

Updating dependencies  
Resolving dependencies... (0.1s)  

Writing lock file
  
Package operations: 1 install, 0 updates, 0 removals  
 • Installing loguru (0.5.3)
$ poetry remove loguru
Updating dependencies
Resolving dependencies... (0.1s)

Writing lock file

Package operations: 0 installs, 0 updates, 1 removal  
 • Removing loguru (0.5.3)
```

### Running commands

With virtual environment you need to write something like this, to enter the shell:
```bash
source venv/bin/activate
```

But with poetry it's much easier. It creates and manages virtual envs on it's own. To run a single command you can call `run`. 

E.G.:
```console
$ poetry install black --dev
Using version ^20.8b1 for black   
 • Installing appdirs (1.4.4)  
 • Installing click (7.1.2)  
 • Installing mypy-extensions (0.4.3)  
 • Installing pathspec (0.8.1)  
 • Installing regex (2020.11.13)  
 • Installing toml (0.10.2)  
 • Installing typed-ast (1.4.2)  
 • Installing typing-extensions (3.7.4.3)  
 • Installing black (20.8b1)
$ poetry run black .
reformatted new_proj/new_proj/__init__.py
reformatted new_proj/tests/test_new_proj.py
All done! ✨ 🍰 ✨
2 files reformatted, 1 file left unchanged.
```

For executing multiple commands within the shell you can enter interactive shell session with `poetry shell`. It's similar to virtualenv's `source venv/bin/activate`.

```console
$ poetry shell
Spawning shell within /home/s3rius/.cache/pypoetry/virtualenvs/new-proj-eutP4v0O-py3.9
$ black .
reformatted new_proj/new_proj/__init__.py
reformatted new_proj/tests/test_new_proj.py
All done! ✨ 🍰 ✨
2 files reformatted, 1 file left unchanged.
```

### Versioning

With poetry you don't need to change package versions manually.

The Poetry has something for you.
```console
$ poetry version patch
Bumping version from 0.1.0 to 0.1.1
$ poetry version preminor
Bumping version from 0.1.1 to 0.2.0-alpha.0
$ poetry version minor   
Bumping version from 0.2.0-alpha.0 to 0.2.0
$ poetry version premajor
Bumping version from 0.2.0 to 1.0.0-alpha.0
$ poetry version major   
Bumping version from 1.0.0-alpha.0 to 1.0.0
```

## pyproject.toml

As mentioned before this file contains package's meta-information.

Example of `pyproject.toml`
```toml
[tool.poetry]
name = "new_proj"
version = "0.1.0"
description = "Test library for example"
readme = "README.rst"
homepage = "https://test_url.com/"
repository = "https://github.meme/"
authors = ["Pavel Kirilin <win10@list.ru>"]

[tool.poetry.dependencies]
python = "^3.8"

[tool.poetry.dev-dependencies]
pytest = "^6.1"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

To install all dependencies simply run:
```console
$ poetry install
```

This command creates virtual environment and installs dependencies from the `pyproject.toml` file. If you want install dependencies only for runtime you can add `--no-dev`.
## Packaging and publishing on pypi
It's really simple. 

Let's add a function in the project.

```python
# new_proj/main.py
def ab_problem(a: int, b: int) -> int:
    return a + b
```

```python
# new_proj/__init.py
from new_proj.main import ab_problem

__all__ = [
    'ab_problem'
]
```

Now you can build the project.

```console
$ poetry build
Building new_proj (0.1.0)
  - Building sdist
  - Built new_proj-0.1.0.tar.gz
  - Building wheel
  - Built new_proj-0.1.0-py3-none-any.whl
```

You can find ready for publication package in `dist/` folder.
Let's check if everything works.

```console
$ pip install "./dist/new_proj-0.1.0-py3-none-any.whl"  
Processing ./dist/new_proj-0.1.0-py3-none-any.whl
Installing collected packages: new-proj
Successfully installed new-proj-0.1.0

$ python
Python 3.9.1 (default, Feb  1 2021, 04:02:33) 
[GCC 10.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
$ from new_proj import ab_problem
$ ab_problem(1,33)
34
```

It works! Now we can use it.

If you want to publish the package you can run:
```console
$ poetry publish -u "user" -p "password"
```

More information you can find [here](https://python-poetry.org/docs/cli/#publish).

# Configuring project
Project without configuration is a bad thing. Let's configure development environment.

```console
$ poetry add \
$	flake8 \
$	black \
$	isort \
$	mypy \
$	pre-commit \
$	yesqa \
$	autoflake \
$	wemake-python-styleguide --dev
---> 100%
```

Now we need to add configuration files for linters and formatters.
This is my preffered configurations, you can change it as you wish.

`.mypy.ini` for types validation.
```ini
[mypy]
strict = True
ignore_missing_imports=True
allow_subclassing_any=True
allow_untyped_calls=True
pretty=True
show_error_codes=True
implicit_reexport=True
allow_untyped_decorators=True
```

`.isort.cfg` for import sorting.
```ini
[isort]
multi_line_output = 3
include_trailing_comma = true
use_parentheses = true
```

`.flake8` - Linter configuration. Biggest part of the document is ignore statements for errors that's not really an errors.
```ini
[flake8]
max-complexity = 6
inline-quotes = double
max-line-length = 88
extend-ignore = E203
docstring_style=sphinx

ignore =
  ; Found `f` string
  WPS305,
  ; Missing docstring in public module
  D100,
  ; Missing docstring in magic method
  D105,
  ; Missing docstring in __init__
  D107,
  ; Found class without a base class
  WPS306,
  ; Missing docstring in public nested class
  D106,
  ; First line should be in imperative mood
  D401,
  ; Found `__init__.py` module with logic
  WPS412,
  ; Found implicit string concatenation
  WPS326,
  ; Found string constant over-use
  WPS226,
  ; Found upper-case constant in a class
  WPS115,
  ; Found nested function
  WPS430,
  ; Found using `@staticmethod`
  WPS602,
  ; Found method without arguments
  WPS605,
  ; Found overused expression
  WPS204,
  ; Found too many module members
  WPS202,
  ; Found too high module cognitive complexity
  WPS232,
  ; line break before binary operator
  W503,
  ; Found module with too many imports
  WPS201,
  ; Found vague import that may cause confusion: X
  WPS347,
  ; Inline strong start-string without end-string.
  RST210,
  ; subprocess call with shell=True seems safe, but may be changed in the future.
  S602,
  ; Starting a process with a partial executable path.
  S607,
  ; Consider possible security implications associated with subprocess module.
  S404,
  ; Found nested class
  WPS431,
  ; Found wrong module name
  WPS100,
  ; Found too many methods
  WPS214,
  ; Found too long ``try`` body
  WPS229,
  ; Found function with too much cognitive complexity
  WPS231,
  
  ; all init files
  __init__.py:
  ; ignore not used imports
  F401,
  ; ignore import with wildcard
  F403,
  ; Found wrong metadata variable
  WPS410,

per-file-ignores =
  ; all tests
  test_*.py,tests.py,tests_*.py,*/tests/*:
  ; Use of assert detected
  S101,

exclude =
  ./.git,
  ./venv,
  ./cached_venv,
  ./var,
```

`.pre-commit-config.yaml` - Git hooks configuration. To perform all checks right before `git commit`.
```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.4.0
    hooks:
      - id: check-ast
      - id: trailing-whitespace
      - id: check-toml
      - id: end-of-file-fixer

  - repo: https://github.com/asottile/add-trailing-comma
    rev: v2.1.0
    hooks:
    -   id: add-trailing-comma

  - repo: local
    hooks:
      - id: black
        name: Format with Black
        entry: black
        language: system
        types: [python]

      - id: autoflake
        name: autoflake
        entry: autoflake
        language: system
        types: [ python ]
        args: [ --in-place, --remove-all-unused-imports, --remove-duplicate-keys ]

      - id: isort
        name: isort
        entry: isort
        language: system
        types: [ python ]

      - id: flake8
        name: Check with Flake8
        entry: flake8
        language: system
        pass_filenames: false
        types: [ python ]
        args: [--count, .]

      - id: mypy
        name: Validate types with MyPy
        entry: mypy
        language: system
        types: [ python ]

      - id: yesqa
        name: Remove usless noqa
        entry: yesqa
        language: system
        types: [ python ]

      - id: pytest
        name: pytest
        entry: pytest
        language: system
        pass_filenames: false
        types: [ python ]
```

Don't forget to add `.gitignore`. You can find it [here](https://github.com/github/gitignore/blob/master/Python.gitignore).

It' REALLY IMPORTANT for correct work of pre-commit.

Now we can install hooks.
```console
$ git init
Initialized empty Git repository in .git/
$ poetry shell
$ pre-commit install
pre-commit installed at .git/hooks/pre-commit
$ git commit
... # It will have a lot of errors.
```

Let's fix it.

At first we need to fix tests.

In `tests/test_new_proj.py` add this:
```python
from new_proj import ab_problem


def test_ab() -> None:
    """AB problecm success case."""
    assert ab_problem(1, 2) == 3
```

Add a description in `__init__` files.

`tests/__init__.py`:
```python
"""Tests for new_proj."""
```

`new_proj/__init__.py`:
```python
"""Project for solving ab problem."""
from new_proj.main import ab_problem

__all__ = [
    "ab_problem",
]
```

Fix the main file of a project.

`new_proj/main.py`:
```python
def ab_problem(first: int, second: int) -> int:
    """
    Solve AB problem.

    The function sums two integers.

    :param first: a argument.
    :param second: b argument.
    :returns: sum.
    """
    return first + second
```

Now you can commit without errors.

```console
$ git commit     
Check python ast................Passed
Trim Trailing Whitespace........Passed
Check Toml......................Passed
Fix End of Files................Passed
Add trailing commas.............Passed
Format with Black...............Passed
autoflake.......................Passed
isort...........................Passed
Check with Flake8...............Passed
Validate types with MyPy........Passed
Remove usless noqa..............Passed
pytest..........................Passed
```

Now you know how to create a masterpiece.
# Creating CLI tool
What if I want to create a CLI-tool?
It easier than you think.

Let's modify our main file.
```python
import argparse


def parse_args() -> argparse.Namespace:
    """
    Parse CLI arguments.

    :returns: parsed namespace.
    """
    parser = argparse.ArgumentParser()

    parser.add_argument("a", type=int)
    parser.add_argument("b", type=int)

    return parser.parse_args()


def ab_problem(first: int, second: int) -> int:
    """
    Solve AB problem.

    The function sums two integers.

    :param first: a argument.
    :param second: b argument.
    :returns: sum.
    """
    return first + second


def main() -> None:
    """Main function."""
    args = parse_args()
    print(ab_problem(args.a, args.b))  # noqa: WPS421

```

Now we need to change the `pyproject.toml`. Add the following section somewhere in `pyproject.toml`.

```toml
[tool.poetry.scripts]
ab_solver = "new_proj.main:main"
```

Now you can use the `ab_solver` in your terminal.
```console
$ poetry install
$ poetry shell
$ ab_solver 1 2
3
```

Do you want to install it globally in the system? Here you go.
```console
$ poetry build
Building new_proj (0.1.0)
  - Building sdist
  - Built new_proj-0.1.0.tar.gz
  - Building wheel
  - Built new_proj-0.1.0-py3-none-any.whl
$ pip install "./dist/new_proj-0.1.0-py3-none-any.whl"
Processing ./dist/new_proj-0.1.0-py3-none-any.whl
Installing collected packages: new-proj
Successfully installed new-proj-0.1.0
$ ab_solver 1 2                                     
3
```

If you publish your project in `pypi` or anywhere else. Users who install your program will get it as the cli-tool.