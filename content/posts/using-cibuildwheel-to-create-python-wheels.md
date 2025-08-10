+++
title = "Using cibuildwheel to Create Python Wheels"
date = 2020-07-29T22:53:00+08:00
lastmod = 2025-08-10T18:44:05+08:00
tags = ["Python"]
categories = ["Programming"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Have you ever tried to install `MySQL-python`? It contains the C code and need to compile the code while install the package. You have to follow the steps in this articles: [Install MySQL and MySQLClient(Python) in MacOS](https://ruddra.com/install-mysqlclient-macos/). Things get worse if you are using Windows.

Luckily, as new distribution format **Wheel** has been published in [PEP 427](https://www.python.org/dev/peps/pep-0427/).

> The wheel binary package format frees installers from having to know about the build system, saves time by amortizing compile time over many installations, and removes the need to install a build system in the target environment.

Installation of wheels does not require a compiler on system and is much faster.

[Cibuildwheel](https://github.com/joerick/cibuildwheel) is a very useful tool for building wheels. It can run on many CI server (GitHub Actions, Travis , Azure Pipelines etc) and build wheels across many platforms.


## Usage {#usage}

You need to create a configuration file for the CI server, you can read the [examples](https://github.com/joerick/cibuildwheel/tree/master/examples) and [documents](https://cibuildwheel.readthedocs.io/en/stable/options/).

For example, GitHub Actions can use this configuration file:

```yml
name: Build

on: [push, pull_request]

jobs:
  build_wheels:
    name: Build wheels on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-18.04, windows-latest, macos-latest]

    steps:
      - uses: actions/checkout@v2

      - uses: actions/setup-python@v2
        name: Install Python
        with:
          python-version: '3.7'

      - name: Install cibuildwheel
        run: |
          python -m pip install cibuildwheel==1.5.5
      - name: Install Visual C++ for Python 2.7
        if: runner.os == 'Windows'
        run: |
          choco install vcpython27 -f -y
      - name: Build wheels
        run: |
          python -m cibuildwheel --output-dir wheelhouse
      - uses: actions/upload-artifact@v2
        with:
          path: ./wheelhouse/*.whl
```


## Useful Options {#useful-options}

These options can be applied by setting environment variables.


### [CIBW_BUILD / CIBW_SKIP](https://cibuildwheel.readthedocs.io/en/stable/options/#build-skip) {#cibw-build-cibw-skip}

Use this options to filter the Python versions to build.

Example:

```nil
# Only build on Python 3.6
CIBW_BUILD: cp36-*

# Skip building on Python 2.7 on the Mac
CIBW_SKIP: cp27-macosx_x86_64

# Skip building on Python 3.8 on the Mac
CIBW_SKIP: cp38-macosx_x86_64
```


### [CIBW_BEFORE_BUILD](https://cibuildwheel.readthedocs.io/en/stable/options/#before-build) {#cibw-before-build}

Execute the shell command before wheel building.


## Upload to PyPI {#upload-to-pypi}

Now you can download `wheelhouse.zip` from `Actions` panel on GitHub, and unzip it to dist folder. Then manually publish it by `rm -rf dist && python setup.py sdist && twine upload dist/*`. You can get more detailed guide from this article: [Packaging Python Projects](https://packaging.python.org/tutorials/packaging-projects/).

This process can also be done automatically by using CI configuration file. You can find the example configuration files from [official repo](https://github.com/joerick/cibuildwheel/blob/master/examples/).


## Ref {#ref}

1.  [Building Python Platform Wheels for Packages with Binary Extensions](https://gertjanvandenburg.com/blog/wheels/)
2.  [How to include external library with python wheel package](https://stackoverflow.com/questions/23916186/how-to-include-external-library-with-python-wheel-package)
3.  [cibuildwheel](https://github.com/joerick/cibuildwheel)
