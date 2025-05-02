---
link: https://fabianlee.org/2024/01/10/ubuntu-pyenv-for-managing-multiple-python-versions-and-environments/
site: "Fabian Lee : Software Engineer"
excerpt: "Keeping the Ubuntu system-level Python version and modules independent
  from those desired at each project level is a difficult task best managed by a
  purpose-built tool. There are many solutions in the Python ecosystem, but one
  that stands out for simplicity is pyenv and pyenv-virtualenv. pyenv allows you
  to install and switch between different versions ... Ubuntu: pyenv for
  managing multiple Python versions and environments"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/env
  - slurp/independent
  - slurp/isolated
  - slurp/penv
  - slurp/pyenv
  - slurp/virtualenv
slurped: 2025-03-08T08:03
title: "Ubuntu: pyenv for managing multiple Python versions and environments"
---

![](https://fabianlee.org/wp-content/uploads/2016/07/ubuntu.png)Keeping the Ubuntu system-level Python version and modules independent from those desired at each project level is a difficult task best managed by a purpose-built tool.

There are many solutions in the Python ecosystem, but one that stands out for simplicity is [pyenv and pyenv-virtualenv](https://realpython.com/intro-to-pyenv/).

pyenv allows you to install and switch between different versions of Python, while pyenv-virtualenv provides isolation of pip modules, for independence between projects.

## Install pyenv

Because pyenv builds various Python versions directly from source, you first need the Ubuntu OS level apt dependencies.

sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl

Then install pyenv using its installation script.

curl https://pyenv.run | bash

## Configure Bash settings

To persist the proper shell settings, add the following lines to ~/.bashrc

export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

reload with new settings.

exec "$SHELL"

## Multiple Python versions with pyenv

Use pyenv to install two different versions of Python.

pyenv install --list | grep 3\.12

# install 2 different versions
pyenv install 3.12.0
pyenv install 3.12.1

# set global Python version as default
pyenv versions; pyenv global 3.12.1; pyenv versions

# validate where python and pip are pointed to
which python; which python3
which pip; which pip3

# validate newest version of python and pip are in use
python --version; python3 --version
pip --version; pip3 --version

# make sure we are using latest version of pip globally
pip3 install --upgrade pip

## Distinct virtualenv for pip module isolation

Use [virtualenv](https://realpython.com/intro-to-pyenv/) to create isolated environments that can be used for specific projects.

# create virtualenv for two different versions of Python
pyenv virtualenv 3.12.0 pytest3120
pyenv virtualenv 3.12.1 pytest3121

# list virtualenv available
pyenv virtualenvs

Test pip module isolated to pytest3120 virtualenv.

# enter virtualenv
pyenv activate pytest3120
python --version
pip list
pip install python-string-utils
pip list

# test usage of module
python -c "import string_utils; print(string_utils.is_string('hello'))"

# leave virtualenv
pyenv deactivate

Test pip module isolated to pytest3121 virtualenv.

# enter virtualenv
pyenv activate pytest3121
python --version
pip list
pip install simple_env
pip list

# test usage of module
foo=BAR python -c "import simple_env as se; print(se.get('foo'))"

# leave virtualenv
pyenv deactivate

## Auto-loading virtualenv

Because of this line added to ~/.bashrc earlier, we also have the ability to auto-activate a virtualenv simply by changing into a python project directory.

eval "$(pyenv virtualenv-init -)"

Here we auto-load the virtualenv named “pytest3120”.

# create python project directory
mkdir pytest3120 && cd $_

# set virtualenv to use
pyenv local pytest3120
# special file now created, which triggers virtualenv
cat .python-version

# show local pyenv settings
pyenv versions
pyenv which python; pyenv which pip
python --version ; pip --version

# test, should be 3.12.0 version
echo -en '#!/usr/bin/env python\nimport sys\nprint(sys.version)' > main.py
python main.py

cd ..

And here we auto-load the virtualenv named “pytest3121”.

# create python project directory
mkdir pytest3121 && cd $_

# set virtualenv to use
pyenv local pytest3121
# special file now created, which triggers virtualenv
cat .python-version

# show local pyenv settings
pyenv versions
pyenv which python; pyenv which pip
python --version ; pip --version

# test, should be 3.12.1 version
echo -en '#!/usr/bin/env python\nimport sys\nprint(sys.version)' > main.py
python main.py

cd ..

REFERENCES

[realpython.com, usage of pyenv](https://realpython.com/intro-to-pyenv/)

[github pyenv](https://github.com/pyenv/pyenv)

[github pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)

[Python build dependencies on Ubuntu](https://realpython.com/intro-to-pyenv/)