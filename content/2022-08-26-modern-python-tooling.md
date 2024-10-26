Title: Modern Python Tooling
Date: 2022-08-31 12:00:00
Author: Ryan McKay
Tags: python
Summary: Modern Python tooling for managing dependencies, testing, and linting


Modern Python development involves managing dependencies, and testing and linting your code.
I'm going to review a few of the related tools for that I've had success with.

To help demonstrate some of the concepts, I'm going to do some work building a [bowling scorer](https://kata-log.rocks/bowling-game-kata).

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
3.10.6 (set by /Users/ryanmckay/projects/py-bowling/.python-version)

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
$ poetry new --src py-bowling
Created package py_bowling in py-bowling
$ tree py-bowling
py-bowling
├── README.md
├── pyproject.toml
├── src
│   └── py_bowling
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
$ cd py-bowling
$ poetry install
Creating virtualenv py-bowling in /Users/ryanmckay/projects/py-bowling/.venv
Updating dependencies
Resolving dependencies... (0.1s)

Writing lock file

Installing the current project: py-bowling (0.1.0)
```

We can activate the virtual env with:
``` bash
$ poetry shell
Spawning shell within /Users/ryanmckay/projects/py-bowling/.venv
py-bowling . /Users/ryanmckay/projects/py-bowling/.venv/bin/activate

$ which python
/Users/ryanmckay/projects/py-bowling/.venv/bin/python

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
Let's add a quick red test that shows off fixtures and informative assertion failures:
``` python
import pytest
from py_bowling.bowling_game import BowlingGame

@pytest.fixture
def game():
    return BowlingGame()

def test_first_roll_score(game):
    # EXPECT
    assert game.roll(10) == 10

```

and implementation
```python
class BowlingGame:
    score: int = 0

    def roll(self, pins: int) -> int:
        return self.score
```

test output looks like:
```
$ pytest
================================= test session starts =================================
platform darwin -- Python 3.10.6, pytest-8.3.3, pluggy-1.5.0
rootdir: /Users/ryanmckay/projects/py-bowling
configfile: pyproject.toml
collected 1 item

tests/test_bowling_score.py F                                                   [100%]

====================================== FAILURES =======================================
________________________________ test_first_roll_score ________________________________

    def test_first_roll_score():
        # GIVEN
        game = BowlingGame()

        # EXPECT
>       assert game.roll(10) == 10
E       assert 0 == 10
E        +  where 0 = roll(10)
E        +    where roll = <py_bowling.bowling_game.BowlingGame object at 0x102395d50>.roll

tests/test_bowling_score.py:9: AssertionError
=============================== short test summary info ===============================
FAILED tests/test_bowling_score.py::test_first_roll_score - assert 0 == 10
================================== 1 failed in 0.02s ==================================
```