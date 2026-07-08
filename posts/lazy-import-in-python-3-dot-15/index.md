# Lazy Import in Python 3.15


Python 3.15 will be release in October 2026, and lazy import is my favorite feature. In general, we often import some modules in the beginning of the script, but we just use them in some functions. When performance matters, the global import will slow down the startup time. With lazy import, we can defer the import until the module is actually used.


## Basic usage {#basic-usage}

```python
lazy import json
lazy from pathlib import Path

print("Starting up...")  # json and pathlib not loaded yet

data = json.loads('{"key": "value"}')  # json loads here
p = Path(".")  # pathlib loads here
```

The syntax is pertty easy, just add the `lazy` keyword before `import` or `from`. After this, the module will be loaded when it's first used. But you can't use `lazy` in star import or `future` import. For example, `lazy from module import *` and `from __future__ import xxx` are not allowed.

Please note that the lazy import only works in module level, so you can't use it inside function, class body or try/except/finally block.


## How to enable lazy import globally {#how-to-enable-lazy-import-globally}

Besides adding `lazy` keyword in every import statement, you can use any of the following ways to enable lazy import globally:

-   Start python with `-X lazy_imports=all` option
-   Set `PYTHONLAZYIMPORTS=all` environment variable
-   Call `sys.set_lazy_imports('all')` explicitly in your code

All of these methods set the lazy import mode to `all`, which means all top-level imports become potentially lazy.

As you know, some modules may have side effects when imported. If you want to key them import eagerly, you can use the `sys.set_lazy_imports_filter()` [import filter](https://docs.python.org/3.15/library/sys.html#sys.set_lazy_imports_filter).

```python
import sys

def my_filter(importer, name, fromlist):
    # Don't lazily import modules known to have side effects
    if name in {'problematic_module', 'another_module'}:
        return False  # Import eagerly
    return True  # Allow lazy import

sys.set_lazy_imports_filter(my_filter)
```


## Compatibility with existing code {#compatibility-with-existing-code}

For prior versions of Python, you can use `PYTHONLAZYIMPORTS=all` to enable lazy import for 3.15+. But if you only want to use lazy import for some modules, you can define the `__lazy_modules__` variable. When defined at module scope, any regular import statement in that module whose target module name appears in this container is treated as a lazy import, as if the lazy keyword had been used.

```python
__lazy_modules__ = ["json", "pathlib"]

import json     # lazy
import os       # still eager
```


## Ref {#ref}

-   [PEP 810 – Explicit lazy imports](https://peps.python.org/pep-0810/)
-   [What's New In Python 3.15](https://docs.python.org/3.15/whatsnew/3.15.html#pep-810-explicit-lazy-imports)
-   [module.\__lazy_modules\_\_](https://docs.python.org/3.15/reference/datamodel.html#module.__lazy_modules__)


---

> Author: KK  
> URL: https://fromkk.com/posts/lazy-import-in-python-3-dot-15/  

