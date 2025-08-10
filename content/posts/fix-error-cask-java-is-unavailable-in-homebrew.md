+++
title = "Fix Error: Cask 'java' is unavailable in Homebrew"
date = 2021-03-07T00:10:00+08:00
lastmod = 2025-08-10T18:44:06+08:00
tags = ["Homebrew"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

After update brew to latest version, when calling `cask` related command, it always outputs `Error: Cask 'java' is unavailable: No Cask with this name exists.`, such as `brew list --cask`. However, the `brew` command works.

After doing some research, I found [Java has been moved to homebrew/core](https://github.com/Homebrew/homebrew-cask/pull/72284). This makes sense now. I installed java by cask, but it's not available now and cask throw this error. If I uninstall java from cask, the error should disappear.

This is not easy as cask is broken. Finally, I found this issue: [brew cask upgrade fails with "No Cask with this name exists"](https://github.com/Homebrew/homebrew-cask/issues/72562). After running `rm -rf "$(brew --prefix)/Caskroom/java`, cask is back.
