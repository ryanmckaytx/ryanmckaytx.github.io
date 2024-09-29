Title: Modern Python Tooling
Date: 2022-08-31 12:00:00
Author: Ryan McKay
Tags: python
Summary: Modern professional software engineering tries to pull a lot of devops processes to the left - from the continuous delivery pipeline into the local development environment. 


Modern professional software engineering tries to pull a lot of devops processes to the left - from the continuous delivery pipeline into the local development environment. 
You want to catch as many issues in local dev as possible, which means you want to be able to run many/most/all of the same checks locally.
You want to minimize differences between environments so that new issues don't crop up in later stages.
I'm going to review a few of the related tools for Python development that I've had success with.

To help demonstrate some of the concepts, I'm going to do some work building a [bowling scorer](https://kata-log.rocks/bowling-game-kata).

## Python Version Management
Python projects destined for deployment should pin a specific version of Python, just like any other dependency, so that you get repeatable behavior in every environment, from local dev to production. 
At some point you're going to need multiple versions of Python on hand, either because you want to test a new version without getting rid of the old version, or because you work in multiple projects that require different versions.

![pyenv logo]({static}/images/pyenv-logo.png "pyenv")

[`pyenv`](https://github.com/pyenv/pyenv) is a tool for installing and switching between different versions of Python.
### pyenv Installation
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

### Python version installation and use
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

## Python dependency management
For this I like to use `poetry`, for a few reasons:  

* Integrated virtual environement support
* Dependency groups - Easy separation between production dependencies and dev/test/etc dependencies
* Lock file - specify desired versions with wildcards and inequalities, but still get repeatable builds with exactly the same versions
* IDE support - PyCharm and VScode
* Python version spec - constrain acceptable Python version just like any other dependency

![poetry logo]({static}/images/poetry-logo-origami.png "poetry")

### Poetry Installation
``` bash
$ curl -sSL https://install.python-poetry.org | python3 -
```
By default, poetry will create new virtual environments under {cache-dir}/virtualenvs, 
but I prefer to have the virtual env for a project stored in the project directory.
So after installation, I configure poetry to do that:
``` bash
$ poetry config virtualenvs.in-project true
```

### Project Initialization
``` bash
$ poetry init

This command will guide you through creating your pyproject.toml config.

Package name [py-bowling]:
Version [0.1.0]:
Description []:
Author [Ryan McKay <ryan.michael.mckay@gmail.com>, n to skip]:
License []:
Compatible Python versions [^3.10]:

Would you like to define your main dependencies interactively? (yes/no) [yes] no
Would you like to define your development dependencies interactively? (yes/no) [yes] no
Generated file

[tool.poetry]
name = "py-bowling"
version = "0.1.0"
description = ""
authors = ["Ryan McKay <ryan.michael.mckay@gmail.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.10"


[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"


Do you confirm generation? (yes/no) [yes]
```

This created a `pyproject.toml` file, which where project dependencies are declared, and also where various tools (like pytest, pylint, etc) can be configured.

We don't have any dependencies yet, but we can go ahead and create virtual environment and a lockfile. 
The lockfile is good for making sure that every install gets exactly the same versions of dependencies.
Because of this property, we can use it for dependency caching in our CI job later.
``` bash
$ poetry install
Updating dependencies
Resolving dependencies... (0.1s)

Writing lock file

/Users/ryanmckay/projects/py-bowling/py_bowling does not contain any element

$ cat poetry.lock
package = []

[metadata]
lock-version = "1.1"
python-versions = "^3.10"
content-hash = "53f2eabc9c26446fbcc00d348c47878e118afc2054778c3c803a0a8028af27d9"

[metadata.files]
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