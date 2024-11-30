+++
title = "__import__ in Python"
author = ["KK"]
date = 2024-04-07T15:58:00+08:00
lastmod = 2024-11-30T20:36:57+08:00
tags = ["Python"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

It's known that Python's `import` statement is implemented by `__import__` function. In general, if we want to import a module dynamically, we can use `import_module` function, which is a wrapper around `__import__`.

> The most important difference between these two functions is that import_module() returns the specified package or module (e.g. pkg.mod), while __import__() returns the top-level package or module (e.g. pkg). -- <https://docs.python.org/3/library/importlib.html#importlib.import_module>

`import itertools` and `from requests import exceptions` can be translated to:

```python
import importlib

itertools = importlib.import_module('itertools')
exceptions = importlib.import_module('requests.exceptions')
```


## `__import__` {#import}

This is an advanced function that is not needed in everyday Python programming, unlike `importlib.import_module()`.

Here is an example of how `__import__` is called:

```python
  old_import = __import__

  def noisy_importer(name, globals=None, locals=None, fromlist=None, level=0):
      print(f'name: {name!r}')
      print(f'fromlist: {fromlist}')
      print(f'level: {level}')
      print('-' * 80)
      return old_import(name, locals, globals, fromlist, level)

  import builtins
  builtins.__import__ = noisy_importer

  print('import math')
  import math
  print('from math import sqrt')
  from math import sqrt

  >>>
  import math
  name: 'math'
  fromlist: None
  level: 0
  --------------------------------------------------------------------------------
  from math import sqrt
  name: 'math'
  fromlist: ('sqrt',)
  level: 0
  --------------------------------------------------------------------------------
```

As we mentioned earlier, the `__import__` returns the top level module.

For example, `requests=__import('requests.exceptions',globals(),locals(),[],0)`. If you want to get the submodule `exceptions`, you need to use `getattr`: `equests_exceptions=getattr(__import__('requests', globals(), locals(), [], 0), 'exceptions')`.

There is another tricky way to import the submodule: use a non-empty `fromlist`: `requests_exceptions = __import__('requests.exceptions', globals(), locals(), [None], 0)`.

Additionally, we can also set `fromlist` to specify the names of submodules that should be imported. The statement `from spam.ham import eggs, sausage as saus` can be translated to

```python
_temp = __import__('spam.ham', globals(), locals(), ['eggs', 'sausage'], 0)
eggs = _temp.eggs
saus = _temp.sausage
```


## Skip importing non-existing modules with `__import__` {#skip-importing-non-existing-modules-with-import}

This a use case of the `__import__` function. Some packages are missing, but we want to make sure that the code does not crash when importing them.

```python
import builtins
from unittest.mock import Mock
old_import = __import__

def skip_imports(name, globals=None, locals=None, fromlist=None, level=0):
    skip_list = {'urllib3', 'requests_oauthlib', 'cryptography'}
    if name in skip_list or any(name.startswith(f'{p}.') for p in skip_list):
        return Mock()
    else:
        return old_import(name, globals, locals, fromlist, level)

builtins.__import__ = skip_imports
```


### Ref: {#ref}

-   [Built-in Functions — Python 3.12.1 documentation](https://docs.python.org/3/library/functions.html#import__)
-   [Python - How to use the <span class="underline"><span class="underline">import</span></span> function to import a name from a submodule? - Stack Overflow](https://stackoverflow.com/questions/9806963/how-to-use-the-import-function-to-import-a-name-from-a-submodule)
