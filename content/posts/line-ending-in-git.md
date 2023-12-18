+++
title = "Line Ending in Git"
author = ["KK"]
date = 2023-10-21T15:40:00+08:00
lastmod = 2023-12-18T21:38:39+08:00
tags = ["Git"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

When working on a project with multiple developers, the line ending can be troublesome. This article will explain how to configure line ending in Git.


## Basic configuration {#basic-configuration}

The line ending on Windows is `CRLF`, on Linux is `LF`. To prevent the line ending issue, we can set `core.autocrlf` to `true` on Windows to let git convert `CRLF` to `LF` when commit, and convert `LF` to `CRLF` when checkout. It is automatically configured if you install git on Windows.

[Configuring Git to handle line endings - GitHub Docs](https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings?platform=windows)

```powershell
# Configure Git to ensure line endings in files you checkout are correct for Windows.
# For compatibility, line endings are converted to Unix style when you commit files.
$ git config --global core.autocrlf true
```


## Advanced configuration {#advanced-configuration}

You can also use `.gitattributes` to control the line ending in each repository. The `.gitattributes` file is a text file that tells Git how to handle files in the repository. You can specify the line ending of each file type in this file.


### Auto convert line ending {#auto-convert-line-ending}

With `* text=auto`, Git handles the files in whatever way it thinks is best. This is a good default option.

Use `*.c text` to explicitly declare a file as a text file, so this file is always normalized and converted to native line endings on checkout.

Use `*.png binary` to explicitly declare a file as binary, so Git does not convert it.  (`binray` is an alias for `-text -diff`)


### Force conversion when checkout: {#force-conversion-when-checkout}

You can use `eol` to force conversion when checkout. The following config enforces bat files to be converted to `CRLF` when checkout even on Mac and Linux.

```text
* text=auto
*.bat eol=crlf
```

This is the result of `git ls-files --eol` on Windows and Linux:

```text
git ls-files --eol src/azure-cli/az.bat
i/lf    w/crlf  attr/text=auto eol=crlf src/azure-cli/az.bat
```

`i` means the index, `w` means the working tree, `attr` means the attribute used when checking out or committing.

You can set `eof` to `crlf` or `lf`. If it's not specified, the line ending will be determined by `core.autocrlf` or `core.eol`. If `text` is set but neither of those variables are set, then the default value is `crlf` on Windows and `lf` on Linux and Mac.


### Refresh setting {#refresh-setting}

If you change the `.gitattributes` file, you need to run the following command to refresh the working tree.

```shell
# Please commit the .gitattributes changes before run this command.
git rm -rf --cached .
git reset --hard HEAD
```


## Extra {#extra}

1.  Line endings in tarball also follows the `.gitattributes`. It's identical to Git checkout on Linux machine.
2.  The `.gitattributes` settings will only affect new commits. If you want to change the line ending of the files that already in the Git index after changing line ending settings, you can use `git add --renormalize .` to force Git to refresh all tracked files. For example, if the bat file has been add as `crlf` in Git index and then you set it as `text` in `.gitattributes`. Running this command asks Git change it to `lf` in index.


## Ref {#ref}

-   [gitattributes - Defining attributes per path](https://www.git-scm.com/docs/gitattributes#_eol)
-   [Configuring Git to handle line endings - GitHub Docs](https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings?platform=windows)
-   [{CI} Enforce LF-only line endings in git #27137 · Azure/azure-cli (github.com)](https://github.com/Azure/azure-cli/pull/27137)
-   [Git - git-add Documentation --renormalize](https://git-scm.com/docs/git-add#Documentation/git-add.txt---renormalize)
