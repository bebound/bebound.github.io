+++
title = "Namespace Package in Python"
date = 2025-08-10T18:04:00+08:00
lastmod = 2025-08-25T21:44:40+08:00
tags = ["Python"]
categories = ["Programming"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Recently, there is a [GitHub issue](https://github.com/Azure/azure-cli/issues/31843#issuecomment-3125269740) about namespace package in Azure CLI. I think it is a good time to write down the knowledge about namespace package.


## What is Namespace Package {#what-is-namespace-package}

If several packages share the same root folder, then the root folder is a namespace package. `subpackageA` and `subpackageb` can be installed separately, even in different Python path, but they can be imported as importing a single package: `import root`.


## How to create namespace Package {#how-to-create-namespace-package}

There are three ways to create namespace package in Python, you can find the details in [Packaging namespace packages](https://packaging.python.org/en/latest/guides/packaging-namespace-packages/).


### Native namespace packages {#native-namespace-packages}

Don't need to create `__init__.py` in root folder, and use `include = ["mynamespace.subpackage_a"]` in `pyproject.toml`.

After installation, the root folder does not contain `__init__.py`, so Python treats it as an implicit because of [PEP 420](https://peps.python.org/pep-0420/).


### Legacy method: pkgutil-style namespace packages {#legacy-method-pkgutil-style-namespace-packages}

The only reason to use this method is to support Python2. You need to create this `__init__.py` in root folder:

```python
__path__ = __import__('pkgutil').extend_path(__path__, __name__)
```

After installation, the `__init__.py` is also kept in the root folder.


### Legacy method: pkg_resource-style namespace packages {#legacy-method-pkg-resource-style-namespace-packages}

This method relies on `setuptools`. After Python 3.12, `setuptools` is not installed by default, and now it's also deprecated in `setuptools`. Currently, it shows this warning:

```text
UserWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html. The pkg_resources package is slated for removal as early as 2025-11-30. Refrain from using this package or pin to Setuptools<81.
```

So I don't recommend using this method.

You need to create this `__init__.py` in root folder, and declare the namespace package in `setup.py`: `namespace_packages=['mynamespace']`.

```python
__import__('pkg_resources').declare_namespace(__name__)
```

After installation, the `__init__.py` is not kept in the root folder.

If you forget to declare the namespace package in `setup.py`, `__init__.py` is kept in the root folder, and the root folder become a normal package. You will get trouble if you install subpackages in different Python path.


## Difference between these three styles {#difference-between-these-three-styles}

The main difference is whether there is a `__init__.py` in the root folder after installation. If there is, then it is a imported as regular package, otherwise it is a namespace package. `pkgutil` style will include a `__init__.py` in the root folder.

Another difference is that if it's a namespace package, then if you change the `sys.path`, the namespace package will be updated. If it's a regular package, then it will not be updated. For example, if you update the `sys.path` and there is new subpackage in the added path, the namespace package will be updated to include the new subpackages. But if it's a regular package, it will not be updated, and you will get `ModuleNotFoundError` when you try to import the new subpackages.

I encountered this issue in the development of Azure CLI. `azure-cli` and `azure-cli-core` still use the `pkgutls` style namespace package. After installing them in editable mode, the extensions' `azure-xxxx` dependency fails to load. Because there is a `__init__.py` in source code, and `azure` folder is treated as a regular package. The extension dependencies are added to the `sys.path` after `azure` is imported so the new `azure=xxx` package is ignored. (Maybe deleting `azure` from `sys.moudule` and importing again can fix this but I haven't tried this.)


## Mix use of different style namespace packages {#mix-use-of-different-style-namespace-packages}

You should avoid mixing different style namespace packages, but if you're working with legacy code, you may encounter this situation. (It get even worse that some packages' packaging is wrong. For example, they forget to declare the namespace package in `setup.py` in pkg_resource-style namespace packages, then the root folder becomes a normal package.)

Let's see the Python's import mechanism first:

> If `<directory>/foo/__init__.py` is found, a regular package is If `<directory>/foo/__init__.py` is found, a regular package is imported and returned.
>
> If not, but `<directory>/foo.{py,pyc,so,pyd}` is found, a module is imported and returned. The exact list of extension varies by platform and whether the -O flag is specified. The list here is representative.
>
> If not, but `<directory>/foo` is found and is a directory, it is recorded and the scan continues with the next directory in the parent path.
> Otherwise the scan continues with the next directory in the parent path.
>
> --[PEP 420 Specification](https://peps.python.org/pep-0420/#specification)

As you can see, if the `__init__.py` is found, the process stops and a regular package is imported. When you use the `pkgutil` style, it will also returns a normal package, even though it has tries to import the packages from all `sys.path`.

For example, if you use the [pypa sample-namespace-packages](https://github.com/pypa/sample-namespace-packages) to test the namespace package. In the `pkgutil` style package, `import example_pkg` returns a regular package: `<module 'example_pkg' from '/Users/kk/Developer/sample-namespace-packages/pkgutil/pkg_a/example_pkg/__init__.py'>`. In other two styles, it's a namespace package: `<<module 'example_pkg' (namespace) from ['/Users/kk/Developer/sample-namespace-packages/native/pkg_a/example_pkg']>`.

So, it's okay to mix these three styles, as all subpackages should be imported. But if some invalid package (not following these three styles, only a normal package) is also installed, the import might be interrupted by the invalid package.

If you use the native style or pkg_resource style, Python only returns the normal package. The namespace package is ignored. That's why in the [#31843 issue](https://github.com/Azure/azure-cli/issues/31843), the `azure.cli` is found.

If a normal package and the pkgutil style namespace package are installed, if the pkgutil package loads first, then all subpackages are imported. If the normal package loads first, then only the normal package will be returned.

In conclusion, if you want to use namespace package, then use the native styles and make sure each subpackages are packaged correctly. Otherwise, you may encounter some module not found error.
