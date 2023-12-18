+++
title = "Python 3.11 changes"
author = ["KK"]
date = 2023-12-10T15:24:00+08:00
lastmod = 2023-12-18T21:52:10+08:00
tags = ["Python"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

In [[Packaging] Support Python 3.11 by bebound · Pull Request #26923 · Azure/azure-cli (github.com)](https://github.com/Azure/azure-cli/pull/26923) , I bumped azure-cli to use Python 3.11. We've bump the dependency in other PRs, I thought it should be a small PR, but in the end, a lot of changes are made.


## `args.getargspec` {#args-dot-getargspec}

`getargspec` is dropped in 3.11. You can easily replaced it with [`getfullargspec`](https://docs.python.org/3/library/inspect.html#inspect.getfullargspec) . It returns `FullArgSpec(args, varargs, varkw, defaults, kwonlyargs, kwonlydefaults, annotations)` instead of `ArgSpec(args, varargs, keywords, defaults)` So `args, _, kw, _ = inspect.getargspec(fn)` can be replaced by `args, _, kw, *_ = inspect.getfullargspec(fn)` However, `getfullargspec` is retained primarily for use in code that needs to maintain compatibility with the Python 2 `inspect` module API.

> Note that [`signature()`](https://docs.python.org/3.11/library/inspect.html#inspect.signature) and [Signature Object](https://docs.python.org/3.11/library/inspect.html#inspect-signature-object) provide the recommended API for callable introspection, and support additional behaviours (like positional-only arguments) that are sometimes encountered in extension module APIs. This function is retained primarily for use in code that needs to maintain compatibility with the Python 2 `inspect` module API. --[inspect --- Inspect live objects --- Python 3.11.4 documentation](https://docs.python.org/3.11/library/inspect.html#inspect.getfullargspec)

The modern `signature` function provides the similar result but needs more modification:

```python
import inspect
def testfunc(a, /, b=1, c=2, *args, kk, **kwargs):
    pass

print(inspect.getfullargspec(testfunc))
print(inspect.signature(testfunc).parameters)

for i, j in inspect.signature(testfunc).parameters.items():
    print(i, type(i), j, type(j), j.kind)

args, _, kw, *_ = inspect.getfullargspec(testfunc)
print(args, kw)

from inspect import Parameter

parameters = inspect.signature(testfunc).parameters
args = [k for k, v in parameters.items() if v.kind in {Parameter.POSITIONAL_OR_KEYWORD, Parameter.POSITIONAL_ONLY}]
kw = next(iter([k for k, v in parameters.items() if v.kind == Parameter.VAR_KEYWORD]), None)
print(args, kw)
```

```nil
FullArgSpec(args=['a', 'b', 'c'], varargs='args', varkw='kwargs', defaults=(1, 2), kwonlyargs=['kk'], kwonlydefaults=None, annotations={})
OrderedDict([('a', <Parameter "a">), ('b', <Parameter "b=1">), ('c', <Parameter "c=2">), ('args', <Parameter "*args">), ('kk', <Parameter "kk">), ('kwargs', <Parameter "**kwargs">)])

a <class 'str'> a <class 'inspect.Parameter'> POSITIONAL_ONLY
b <class 'str'> b=1 <class 'inspect.Parameter'> POSITIONAL_OR_KEYWORD
c <class 'str'> c=2 <class 'inspect.Parameter'> POSITIONAL_OR_KEYWORD
args <class 'str'> *args <class 'inspect.Parameter'> VAR_POSITIONAL
kk <class 'str'> kk <class 'inspect.Parameter'> KEYWORD_ONLY
kwargs <class 'str'> **kwargs <class 'inspect.Parameter'> VAR_KEYWORD

['a', 'b', 'c'] kwargs
['a', 'b', 'c'] kwargs
```


## Enum `__format__` change {#enum-format-change}

There is some custom classes in azure-cli, which makes `Foo.BAR=`'bar'`. In 3.11, the [[https://docs.python.org/3/whatsnew/3.11.html#enum][Enum]] =__format__()` changes, it returns the enum and member name (ex: `Color.RED`). (The `__str__` method is the same as Python 3.10)

> Changed [`Enum.__format__()`](https://docs.python.org/3/library/enum.html#enum.Enum.__format__) (the default for [`format()`](https://docs.python.org/3/library/functions.html#format), [`str.format()`](https://docs.python.org/3/library/stdtypes.html#str.format) and [f-string](https://docs.python.org/3/glossary.html#term-f-string)s) to always produce the same result as `Enum.__str__()`: for enums inheriting from [`ReprEnum`](https://docs.python.org/3/library/enum.html#enum.ReprEnum) it will be the member's value; for all other enums it will be the enum and member name (e.g. `Color.RED`). --[What's New In Python 3.11](https://docs.python.org/3/whatsnew/3.11.html#enum)

```python
from enum import Enum
class Foo(str, Enum):
    BAR = "bar"

# Python 3.10
f"{Foo.BAR}"  # > bar
str(Foo.BAR)  # > Foo.BAR

# Python 3.11
f"{Foo.BAR}"  # > Foo.BAR
str(Foo.BAR)  # > Foo.BAR
```

The standard way to replace Foo class is [StrEnum](https://docs.python.org/3.11/library/enum.html#enum.StrEnum)

```python
class Foo(StrEnum):
    BAR = "bar"

# Python 3.11
f"{Foo.BAR}"  # > bar
```

If you also use `Bar(int, Enum)`, you can replace it with [ReprEnum](https://docs.python.org/3.11/library/enum.html#enum.ReprEnum): `Bar(int, ReprEnum)`.


## `unittest.Mock` {#unittest-dot-mock}

The `unittest` module replace `unittest.mock._importer` with `pkgutil.resolve_name` in [bpo-44686 replace unittest.mock._importer with pkgutil.resolve_name by graingert · Pull Request #18544 · python/cpython (github.com)](https://github.com/python/cpython/pull/18544), which also introduces some changes.

Previously, it use `__import__` to import the patch target, which does not check the module name. But `pkgutil.resolve_name` will check name first, thus `mock.patch` fails if the target is not a valid Python module name. For example, this statement fails in 3.11:

```python
@mock.patch('azure.cli.command_modules.vm.aaz.2020_09_01_hybrid.network.vnet.List', _mock_network_client_with_existing_vnet_location)
```

as `2020_09_01_hybrid` is not a valid variable name in Python.

```python
            _NAME_PATTERN = re.compile(f'^(?P<pkg>{dotted_words})'
                                       f'(?P<cln>:(?P<obj>{dotted_words})?)?$',
                                       re.UNICODE)

        m = _NAME_PATTERN.match(name)
        if not m:
>           raise ValueError(f'invalid format: {name!r}')
E           ValueError: invalid format: 'azure.cli.command_modules.vm.aaz.2020_09_01_hybrid.network.vnet'
```

As a workaround, `mock.patch.object` works.

```python
vnet = import_module('azure.cli.command_modules.vm.aaz.2018_03_01_hybrid.network.vnet')
with mock.patch.object(vnet, 'List', _mock_network_client_with_existing_vnet):
```

The ultimate solution is fix module name.


## `argparse.ArgumentError` {#argparse-dot-argumenterror}

[bpo-39716: Raise on conflicting subparser names. by anntzer · Pull Request #18605 · python/cpython (github.com)](https://github.com/python/cpython/pull/18605) Raise an ArgumentError when the same subparser name is added twice.

```python
import argparse

parser = argparse.ArgumentParser()
t = parser.add_subparsers()
t.add_parser('a')
t.add_parser('a')
```

The above code works on 3.10 but raises this error in 3.11:

```nil
Traceback (most recent call last):
  File "C:\Users\kk\Developer\azure-cli\p.py", line 6, in <module>
    t.add_parser('a')
  File "C:\Program Files\WindowsApps\PythonSoftwareFoundation.Python.3.11_3.11.1264.0_x64__qbz5n2kfra8p0\Lib\argparse.py", line 1192, in add_parser
    raise ArgumentError(self, _('conflicting subparser: %s') % name)
argparse.ArgumentError: argument {a}: conflicting subparser: a
```


## Ref {#ref}

-   [Enum with `str` or `int` Mixin Breaking Change in Python 3.11 (pecar.me)](https://blog.pecar.me/python-enum)
-   [Enum with `str` or `int` Mixin Breaking Change in Python 3.11 · Issue #100458 · python/cpython (github.com)](https://github.com/python/cpython/issues/100458)
-   [What's New In Python 3.11 --- Python 3.11.4 documentation](https://docs.python.org/3/whatsnew/3.11.html)
