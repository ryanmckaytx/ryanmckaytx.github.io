Title: Modern Python Tooling
Date: 2022-08-26 12:00:00
Author: Ryan McKay
Tags: python
Summary: Modern professional software engineering tries to pull a lot of devops processes to the left - from the continuous delivery pipeline into the local development environment. 


Modern professional software engineering tries to pull a lot of devops processes to the left - from the continuous delivery pipeline into the local development environment. 
You want to catch as many issues in local dev as possible, which means you want to be able to run many/most/all of the same checks locally.
You want to minimize differences between environments so that new issues don't crop up in later stages.
I'm going to review a few of the related tools for I've had success with for Python development.

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

