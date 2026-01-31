+++
title = "Modern pip build process (–-use-pep517)"
date = 2024-11-24T20:49:00+08:00
lastmod = 2025-08-10T18:44:06+08:00
tags = ["Python", "pip"]
categories = ["Programming"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Nowadays, `pyproject.toml` becomes the standard configuration file for packaging. Compare with the old `setup.py`, it adds two feature pep517 and pep518.

[pep517](https://peps.python.org/pep-0517/) defines two hooks: `build_wheel` and `build_sdist`, which is required to build the package from source. Each build backend must implement these two hooks. It makes it possible to create other build backend such as `flit` or `poetry`.

```toml
[build-system]
# Defined by PEP 518:
requires = ["flit"]
# Defined by this PEP:
build-backend = "local_backend"
backend-path = ["backend"]
```

Besides `setuptools`, there are some other build back-end such as `hatchling` and `flit`. You can find the example here: [Python Packaging Uer Guide - Choosing a build backend](https://packaging.python.org/en/latest/tutorials/packaging-projects/#choosing-a-build-backend)

[pep518](https://peps.python.org/pep-0518/) defines the format of `pyproject.toml`, where you can specify you build dependencies, ensuring the necessary tools will be installed when building project. For example:

```toml
[build-system]
requires = ["setuptools ~= 58.0", "cython ~= 0.29.0"]
```


## Is `setup.py` deprecated {#is-setup-dot-py-deprecated}

According to [python packaging doc](https://packaging.python.org/en/latest/discussions/setup-py-deprecated/), it is still the valid configuration file for `setuptools`, but use `setup.py` in command is depracetd:

| Deprecated                  | Replacement      |
|-----------------------------|------------------|
| python setup.py install     | pip install .    |
| python setup.py develop     | pip install -e . |
| python setup.py sdist       | python -m build  |
| python setup.py bdist_wheel | python -m build  |

[Build](https://build.pypa.io/en/stable/) is a pep 517 compatible build fontend, it calls build backend to generate the source and wheel distribution. It's the recommended way to build the package.


## When `--use-pep517` is activated? {#when-use-pep517-is-activated}

If the source distribution contains `pyproject.toml`, `pip` will use `pep517` to build the package.

If the current env does not have `` setuptools` `` or `wheel`, `pip` will use `pep517` to build the package: [pip source code](https://github.com/pypa/pip/blob/ec5faeac4ef6bae97df0e779566ceb2b0de89d3f/src/pip/_internal/pyproject.py#L107)

You can also force `pip` to use `pep517` with `--use-pep517`, or disable it and use the legacy behavior with `--no-use-pep517`.


## What's the difference between `--use-pep517` and legacy behavior? {#what-s-the-difference-between-use-pep517-and-legacy-behavior}

This is a typical log which uses `--use-pep517`:

```text
  root@427314aff523:/# pip install requests --no-binary :all: --no-deps
Collecting requests
  Using cached requests-2.32.3.tar.gz (131 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... done
Building wheels for collected packages: requests
  Building wheel for requests (pyproject.toml) ... done
  Created wheel for requests: filename=requests-2.32.3-py3-none-any.whl size=64922 sha256=9ee1e853d3d86a8b484cf10c2920601befe81bfad4bd0c3319274b67143ac266
```

This is the one which uses the legacy behavior:

```text
  Processing d:\a\_work\1\s\src\azure-cli
  Preparing metadata (setup.py): started
  Preparing metadata (setup.py): finished with status 'done'
Building wheels for collected packages: azure-cli
  Building wheel for azure-cli (setup.py): started
  Building wheel for azure-cli (setup.py): finished with status 'done'
```

The main difference is that `--use-pep517` will create a temp build env and build the package in it. The build env is totally fresh, it has to install the build dependencies such as `setuptools` first, then call the backend to build the wheel pacakge. Finally, it will install the wheel package.

In legacy behavior, `pip` use the current env's `pip`, `setuptools` and `wheel` to build the package with `python setup.py bdist_wheel`, then install the wheel package.


## A weird issue related to `--use-pep517` {#a-weird-issue-related-to-use-pep517}

When bump Python 3.12 in Azure CLI, the `get-pip.py` does not install `setuptools` by default, as well as `wheel`. So the `pip` tries to use `pep517` to build azure-cli.

However, the runner agent is using Python 3.12.6 and the embedded Python is 3.12.7. They have a compatibility issue. In the build env, the `python -m pip` fails with code 57005, because it tries to load modules in 3.12.6. I have to use `python -Im pip` to install the package. However, in the build env, the `-I` param is not honored, the command still fails. I've created an [issue](https://github.com/pypa/pip/issues/13023) in pip repo. The workaround is so complicated, so I have to install `setuptools` and `wheel` to let `pip` use the legacy behavior, which use the env's `pip` to build the package.

The details can be found in the [PR](https://github.com/Azure/azure-cli/pull/29887#discussion_r1802648918)


### Ref: {#ref}

-   [pip build_env.py source code](https://github.com/pypa/pip/blob/ec5faeac4ef6bae97df0e779566ceb2b0de89d3f/src/pip/_internal/build_env.py#L227-L248)
-   [original issue when bump Python 3.12](https://github.com/Azure/azure-cli/pull/29887#discussion_r1802648918)
-   [pip build process for pyproject.toml](https://pip.pypa.io/en/latest/reference/build-system/pyproject-toml/)
-   [PEP 517 and 518 in Plain English](https://chadsmith-software.medium.com/pep-517-and-518-in-plain-english-47208ca8b7a6)
-   [Writing your pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/)
