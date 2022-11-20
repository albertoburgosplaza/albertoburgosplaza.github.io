---
layout: post
title:  "Using pyenv with Docker"
date:   2022-11-20 10:00:00 +0100
categories: pyenv python docker
---
Sometimes you need to use a different Python version than the one that comes with a container's image. For example, if you're using the image `ubuntu:18.04` the Python version will be `3.6.9`. Let's check it.

1. Using the following Dockerfile:
```dockerfile
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y python3
```

2. Building and checking the Python version:

```shell
$ docker build -t testing_pyenv .
```

```shell
$ docker run testing_pyenv python3 --version
Python 3.6.9
```

How to address easily the setup of a different Python version? Using **pyenv**. The following is an example of installing Python `3.11.0` on the Ubuntu 18.04 image.

1. Using the following Dockerfile:

```dockerfile
FROM ubuntu:18.04

RUN apt-get update
RUN apt-get install -y \
    build-essential \
    gcc \
    cmake \ 
    curl \
    git \
    libssl-dev \
    libffi-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    liblzma-dev \
    libncurses-dev

RUN curl https://pyenv.run | bash
RUN echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
RUN echo 'command -v pyenv >/dev/null \
    || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
RUN echo 'eval "$(pyenv init -)"' >> ~/.bashrc

ENV HOME="/root"
ENV PYENV_ROOT="${HOME}/.pyenv"
ENV PATH="${PYENV_ROOT}/shims:${PYENV_ROOT}/bin:${PATH}"

ENV PYTHON_VERSION=3.11.0
RUN pyenv install ${PYTHON_VERSION}
RUN pyenv global ${PYTHON_VERSION}
```

2. Building and checking the Python version:

```shell
$ docker build -t testing_pyenv .
```

```shell
$ docker run testing_pyenv python3 --version
Python 3.11.0
```

You can find this code at [https://github.com/albertoburgosplaza/testing_pyenv](https://github.com/albertoburgosplaza/testing_pyenv).