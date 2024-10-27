Title: Modern Python Tooling
Date: 2022-08-31 12:00:00
Author: Ryan McKay
Tags: python
Summary: Modern Python tooling for managing dependencies, testing, and linting


Modern Python development involves managing dependencies, and testing and linting your code.
I'm going to review a few of the related tools for that I've had success with.

To help demonstrate some of the concepts, I'm going to do some work on the [gilded rose kata](https://kata-log.rocks/gilded-rose-kata).

# Python Version Management
Python projects destined for deployment should pin a specific version of Python, just like any other dependency, so that you get repeatable behavior in every environment, from local dev to production. 
At some point you're going to need multiple versions of Python on hand, either because you want to test a new version without getting rid of the old version, or because you work in multiple projects that require different versions.

![pyenv logo]({static}/images/pyenv-logo.png "pyenv")

[`pyenv`](https://github.com/pyenv/pyenv) is a tool for installing and switching between different versions of Python.
## pyenv Installation
``` bash
brew update
brew install pyenv
pyenv init
# Load pyenv automatically by appending
# the following to
# ~/.zprofile (for login shells)
# and ~/.zshrc (for interactive shells) :

export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

# Restart your shell for the changes to take effect.
```

## Python version installation and use
``` bash
# List available versions
$ pyenv install -l
...
  3.10.0
  3.10-dev
  3.10.1
  3.10.2
  3.10.3
  3.10.4
  3.10.5
  3.10.6
...

# Install version
$ pyenv install 3.10.6
python-build: use openssl from homebrew
python-build: use readline from homebrew
Downloading Python-3.10.6.tar.xz...
-> https://www.python.org/ftp/python/3.10.6/Python-3.10.6.tar.xz
Installing Python-3.10.6...
python-build: use readline from homebrew
python-build: use zlib from xcode sdk
Installed Python-3.10.6 to /Users/ryanmckay/.pyenv/versions/3.10.6

# Use version globally
$ pyenv global 3.10.6

# Use version locally
$ pyenv local 3.10.6

$ pyenv version
3.10.6 (set by /Users/ryanmckay/projects/gilded-rose-kata/.python-version)

$ cat ./.python-version
3.10.6

$ python --version
Python 3.10.6
```

# Python dependency management
For this I like to use [`poetry`](https://python-poetry.org/), for a few reasons:  

* Integrated virtual environement support
* Dependency groups - Easy separation between production dependencies and dev/test/etc dependencies
* Lock file - specify desired versions with wildcards and inequalities, but still get repeatable builds with exactly the same versions
* IDE support - PyCharm and VScode
* Python version spec - constrain acceptable Python version just like any other dependency

![poetry logo]({static}/images/poetry-logo-origami.png "poetry")

## Poetry Installation
``` bash
$ curl -sSL https://install.python-poetry.org | python3 -
```
By default, poetry will create new virtual environments under {cache-dir}/virtualenvs, 
but I prefer to have the virtual env for a project stored in the project directory.
So after installation, I configure poetry to do that:
``` bash
$ poetry config virtualenvs.in-project true
```

## Project Initialization
``` bash
$ poetry new --src gilded-rose-kata
Created package gilded_rose_kata in gilded-rose-kata
$ tree gilded-rose-kata
gilded-rose-kata
├── README.md
├── pyproject.toml
├── src
│   └── gilded_rose_kata
│       └── __init__.py
└── tests
    └── __init__.py
```
This created the basic project structure, including a `pyproject.toml` file, 
which is where project dependencies are declared, and also where various tools (like pytest, pylint, etc) can be configured.

We don't have any dependencies yet, but we can go ahead and create a virtual environment and a lockfile. 
The lockfile is good for making sure that every install gets exactly the same versions of dependencies.
Because of this property, we can use it for dependency caching in our CI job later.

``` bash
$ cd gilded-rose-kata
$ poetry install
Creating virtualenv gilded-rose-kata in /Users/ryanmckay/projects/gilded-rose-kata/.venv
Updating dependencies
Resolving dependencies... (0.1s)

Writing lock file

Installing the current project: gilded-rose-kata (0.1.0)
```

We can activate the virtual env with:
``` bash
$ poetry shell
Spawning shell within /Users/ryanmckay/projects/gilded-rose-kata/.venv
gilded-rose-kata . /Users/ryanmckay/projects/gilded-rose-kata/.venv/bin/activate

$ which python
/Users/ryanmckay/projects/gilded-rose-kata/.venv/bin/python

$ python --version
Python 3.10.6
```

One feature of poetry that I really like is 
``` bash
poetry install --sync
```
This will synchronize the dependencies installed in the virtual env with the current state of the poetry.lock file (including removing/downgrading dependencies if necessary). 
This is very useful when you are jumping around between branches that might have different dependencies specified.

# Testing
The two main testing libraries for Python are unittest (built in to Python) and pytest. I like pytest for a few reasons:

* Informative assertion failures
* Easy, robust fixture management
* Lots of useful plugins
* Low explicit dependence on test framework
    * No inheriting from Testcase
    * No self.assertEqual, etc
* Supports existing UnitTest-based tests

![pytest logo]({static}/images/pytest-200.png "pytest")

## Pytest Plugins and Built-ins
There are almost 1500 [registered plugins](https://docs.pytest.org/en/stable/reference/plugin_list.html) for pytest. 
Here are a few plugins (and built-ins) I have found useful:

* [--durations](https://docs.pytest.org/en/stable/example/simple.html#profiling-test-duration) - pytest has built-in support for measuring and reporting test durations. 
I use this to identify the slowest-running tests, in order to target them for speeding up.
[pytest-durations](https://pypi.org/project/pytest-durations/) is a plugin that gives more detailed information broken down into fixture creation, setup, run, and teardown.
* [caplog](https://docs.pytest.org/en/stable/how-to/logging.html#caplog-fixture)/[pytest-loguru](https://pypi.org/project/pytest-loguru/) - caplog is useful for making assertions about logging activity in the code under test.
I like to use [loguru](https://loguru.readthedocs.io/en/stable/) for logging with context, and the pytest-loguru plugin makes those logs available to caplog.
* [pytest-asyncio](https://pypi.org/project/pytest-asyncio/) - convenient for testing async code
* [pytest-cov](https://pypi.org/project/pytest-cov/) - capture coverage report from test run. 
This can be configured to fail if coverage is under a specified %.
* [pytest-datadir](https://pypi.org/project/pytest-datadir/) - pytest plugin for test data directories and files. 
I mostly use datadir to hold input files or expected output files I want to compare against. 
pytest-datadir creates a pristine copy of the datadir for each test run, so you can write output files to it for post-hoc analysis, without interfering with other tests.  
It cleans up these copies automatically after a while.
* [pytest-rerunfailures](https://pypi.org/project/pytest-rerunfailures/) - mark specific tests as flaky and specify a retry policy. Obviously it is better to root cause and remove the flakiness, but sometimes you need a quick fix.
* [pytest-timeout](https://pypi.org/project/pytest-timeout/) - use this to kill tests that take too long.  The threshold can be set globally or on specific tests.

## Installation
```bash
poetry add --group dev pytest
```
Poetry lets you group dependencies.  This is handy for keeping your dev dependencies out of your production deployable.

## Test
Starting with the existing [gilded rose implementation](https://github.com/emilybache/GildedRose-Refactoring-Kata/blob/main/python/gilded_rose.py)
(with some minor modifications to show off pytest fixtures), let's add a quick red test:
``` python
import pytest
from gilded_rose_kata.gilded_rose import GildedRose, Item

@pytest.fixture
def rose():
    return GildedRose()

def test_update_quality(rose):
    # GIVEN
    initial_quality = 6
    item = Item(name="Conjured Mana Cake", sell_in=3, quality=initial_quality)

    # EXPECT quality should have degraded by 2
    assert rose.update_quality(item).quality == 4
```
Notice how easy it is to set up reusable test fixtures. 
If any test wants to use the `rose` fixture, it just has to add the `rose` parameter.
Test output looks like:
``` bash
$ pytest
================================== test session starts =================================
platform darwin -- Python 3.10.6, pytest-8.3.3, pluggy-1.5.0
rootdir: /Users/ryanmckay/projects/gilded-rose-kata
configfile: pyproject.toml
collected 1 item

tests/test_gilded_rose.py F                                                      [100%]

======================================= FAILURES =======================================
__________________________________ test_update_quality _________________________________

rose = <gilded_rose_kata.gilded_rose.GildedRose object at 0x105655f00>

    def test_update_quality(rose):
        # GIVEN
        initial_quality = 6
        item = Item(name="Conjured Mana Cake", sell_in=3, quality=initial_quality)

        # EXPECT quality should have degraded by 2
>       assert rose.update_quality(item).quality == 4
E       assert 5 == 4
E        +  where 5 = Conjured Mana Cake, 2, 5.quality
E        +    where Conjured Mana Cake, 2, 5 = update_quality(Conjured Mana Cake, 3, 6)
E        +      where update_quality = <gilded_rose_kata.gilded_rose.GildedRose object at 0x105655f00>.update_quality

tests/test_gilded_rose.py:15: AssertionError
================================ short test summary info ===============================
FAILED tests/test_gilded_rose.py::test_update_quality - assert 5 == 4
=================================== 1 failed in 0.01s ==================================
```
I really like this detailed assertion failure output. 
I spent most of my career working with Java, where you needed something like [Spock]({static}/docker-java-example-part-2-spring-web.html) to get output like this.

# Pre-commit
[Pre-commit](https://pre-commit.com/) is a tool for running a variety of static checks as a git precommit hook. 
Two of my favorites for Python are [black](https://github.com/psf/black) and [pylint](https://github.com/pylint-dev/pylint).

![pre-commit logo]({static}/images/pre-commit-logo.png "pre-commit")

## Installation

with `.pre-commit-config.yaml` configured as:
``` yaml
repos:
-   repo: https://github.com/psf/black
    rev: 22.6.0
    hooks:
    -   id: black
-   repo: https://github.com/pylint-dev/pylint
    rev: 'v2.15.0'  # Replace with the latest version of pylint
    hooks:
    -   id: pylint
        args: [
            '--disable=C0114,C0116', # Add any pylint disables you want here
        ]
```

then install
``` bash
$ poetry add --group dev pre-commit
$ pre-commit install-hooks
[INFO] Initializing environment for https://github.com/psf/black.
[INFO] Initializing environment for https://github.com/pylint-dev/pylint.
[INFO] Installing environment for https://github.com/psf/black.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
[INFO] Installing environment for https://github.com/pylint-dev/pylint.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
$ pre-commit install
pre-commit installed at .git/hooks/pre-commit
```

## Run
The git precommit hook runs against modified files only, but we can run the hooks against all the code:
``` bash
$ pre-commit run --all-files
black....................................................................Failed
- hook id: black
- files were modified by this hook

reformatted tests/test_gilded_rose.py
reformatted src/gilded_rose_kata/gilded_rose.py

All done! ✨ 🍰 ✨
2 files reformatted, 2 files left unchanged.

pylint...................................................................Failed
- hook id: pylint
- exit code: 30

************* Module gilded_rose_kata.gilded_rose
src/gilded_rose_kata/gilded_rose.py:4:0: C0115: Missing class docstring (missing-class-docstring)
src/gilded_rose_kata/gilded_rose.py:11:15: C0209: Formatting a regular string which could be a f-string (consider-using-f-string)
src/gilded_rose_kata/gilded_rose.py:4:0: R0903: Too few public methods (1/2) (too-few-public-methods)
src/gilded_rose_kata/gilded_rose.py:14:0: C0115: Missing class docstring (missing-class-docstring)
src/gilded_rose_kata/gilded_rose.py:14:0: R0205: Class 'GildedRose' inherits from object, can be safely removed from bases in python3 (useless-object-inheritance)
src/gilded_rose_kata/gilded_rose.py:19:12: R1714: Consider merging these comparisons with 'in' by using 'updated_item.name not in ('Aged Brie', 'Backstage passes to a TAFKAL80ETC concert')'. Use a set instead if elements are hashable. (consider-using-in)
src/gilded_rose_kata/gilded_rose.py:15:4: R0912: Too many branches (17/12) (too-many-branches)
src/gilded_rose_kata/gilded_rose.py:14:0: R0903: Too few public methods (1/2) (too-few-public-methods)
************* Module tests.test_gilded_rose
tests/test_gilded_rose.py:1:0: E0401: Unable to import 'pytest' (import-error)
tests/test_gilded_rose.py:10:24: W0621: Redefining name 'rose' from outer scope (line 6) (redefined-outer-name)

------------------------------------------------------------------
Your code has been rated at 6.74/10 (previous run: 6.74/10, +0.00)
```
Both hooks failed.  If that had happened during `git commit`, it would have aborted the commit.
The `black` hook fixes python formatting, so all we need to do is commit the modified files.
I like this approach for running black, because you can see what it did.

A pylint score of `6.74` is pretty low.  By default it fails with anything less than 10/10.
You can set the failing score threshold with `--fail-under`, 
which is convenient for setting a linting floor when you are adding linting to an existing project.