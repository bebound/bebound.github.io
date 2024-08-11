+++
title = "sys.path in Python"
author = ["KK"]
date = 2024-08-11T15:56:00+08:00
lastmod = 2024-08-11T16:09:22+08:00
tags = ["Python"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Here is the process how `sys.path` is set in Python, with some parts omitted.


## Python Command Line Arguments {#python-command-line-arguments}

By default, as initialized upon program startup, a potentially unsafe path is prepended to `sys.path`:

`python -m`: prepend the current working directory.

`python script.py`: prepend the script’s directory. If it’s a symbolic link, resolve symbolic links.

`python -c` and python (REPL): prepend an empty string, which means the current working directory.

You can remove these path with `-P` param.


## `PYTHONPATH` {#pythonpath}

If this environment variable is set, the folders in it will be added to `sys.path`. The folders are separated by colons on Unix and semicolons on Windows.


## `prefix` and `exec_prefix` {#prefix-and-exec-prefix}

These two variable define the standard Python modules and extension modules. Python has a specific path to search depends on the OS. The start point is Python executable path, which is called `home` (the symbolic links are followed).

Once `home` is determined, the `prefix` directory is found by looking for `pythongmajorversionminorversion.zip`. For example, `python312.zip`. On Windows, the zip package is in the same directory as the Python executable. On Unix, it is in `/lib` folder. If it is not found, on Windows, it will looks for `Lib\os.py`. On Unix, it will look for `lib/python3.12/os.py`.

On macOS, the `home` is `/opt/homebrew/opt/python@3.12/Frameworks/Python.framework/Versions/3.12/bin/python3.12`. The `prefix` is `/opt/homebrew/opt/python@3.12/Frameworks/Python.framework/Versions/3.12`, because `lib/python3.12/os.py` is there.

On Windows, the `exec_prefix` is the same as `prefix`. But on other OS, `exec_prefix` is determined by `lib/python3.xx/lib-dynload`. On my mac, it's still `/opt/homebrew/opt/python@3.12/Frameworks/Python.framework/Versions/3.12`.

`lib/python312.zip`, `lib/python3.12` and `lib/python3.12/lib-dynload` are added into `sys.path`.


## `site` module {#site-module}

This module is automatically called during Python startup, which tries to append the `site-packages` folder into `sys.path`. It can be disabled with `-S` option.

Finding `site-packages` folder is easy. It can be guessed by `prefix` and `exec_prefix`. These two path is head, and the tail part is `lib/site-packages` on Windows or `lib/pythonX.Y/site-packages` on \*nix. For each of the head-tail combinations, it add the path into `sys.path` if it exists.


### .pth files {#dot-pth-files}

If a `name.pth` file exits in the `site-packages` folder, its content are additional items to be added into `sys.path`. Each line is a relative path.

The site module also tries to add `USER_SITE` folder into `sys.path`. Default value is `~/.local/lib/pythonX.Y/site-packages` for UNIX and non-framework macOS builds, `~/Library/Python/X.Y/lib/python/site-packages` for macOS framework builds, and `%APPDATA%\Python\PythonXY\site-packages` on Windows.


## Example {#example}

We can use \`python3 -m site\` to quickly check `sys.path` and user site. Here is the output on my mac:

```nil
sys.path = [
    '{current folder}',
    '/opt/homebrew/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python312.zip',
    '/opt/homebrew/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python3.12',
    '/opt/homebrew/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python3.12/lib-dynload',
    '/opt/homebrew/lib/python3.12/site-packages',
    '/opt/homebrew/opt/python@3.12/Frameworks/Python.framework/Versions/3.12/lib/python3.12/site-packages',
]
USER_BASE: '/Users/kk/Library/Python/3.12' (doesn't exist)
USER_SITE: '/Users/kk/Library/Python/3.12/lib/python/site-packages' (doesn't exist)
ENABLE_USER_SITE: True
```

The path is slightly different from what the document states. The `site-packages` folder is not the same as `prefix`. I guess that because Homebrew creates lots of symbol link. `python3` is `/opt/homebrew/bin/python3` -&gt; `opt/homebrew/Cellar/python@3.12/3.12.4/bin/python3` -&gt; `/opt/homebrew/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/bin/python3`.

```nil
>>> import sys
>>> sys.prefix
'/opt/homebrew/opt/python@3.12/Frameworks/Python.framework/Versions/3.12'
>>> sys.exec_prefix
'/opt/homebrew/opt/python@3.12/Frameworks/Python.framework/Versions/3.12'
```

Here is the output on my Ubuntu server:

```nil
sys.path = [
    '{current folder}',
    '/usr/lib/python38.zip',
    '/usr/lib/python3.8',
    '/usr/lib/python3.8/lib-dynload',
    '/home/kk/.local/lib/python3.8/site-packages',
    '/usr/local/lib/python3.8/dist-packages',
    '/usr/local/lib/python3.8/dist-packages/cloud_init-20.1-py3.8.egg',
    '/usr/lib/python3/dist-packages',
]
USER_BASE: '/home/kk/.local' (exists)
USER_SITE: '/home/kk/.local/lib/python3.8/site-packages' (exists)
ENABLE_USER_SITE: True
```

Ref:

-   [sys — System-specific parameters and functions](https://docs.python.org/3/library/sys.html#sys.path)
-   [The initialization of the sys.path module search path](https://docs.python.org/3/library/sys_path_init.html)
-   [\_pth files](https://docs.python.org/3/library/sys_path_init.html#pth-files)
