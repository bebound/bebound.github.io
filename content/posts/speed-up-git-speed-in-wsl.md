+++
title = "Improve Git speed in WSL"
date = 2023-12-26T11:16:00+08:00
lastmod = 2025-07-18T19:07:22+08:00
tags = ["Git"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

The disk performance in WSL2 is poor, it takes a long time to run `git status` in a host's repo. Moreover, if you set a fancy shell prompt, it will take a long time to show the prompt. This article will introduce how to speed up Git in WSL2.


## How to speed up Git Command {#how-to-speed-up-git-command}

The performance of file system in WSL2 is poor, it takes a long time to run `git status` in a host's repo. The solution is to use `git.exe` in Windows folder. You can add this into your `bashrc`:

```bash
  function git() {
  if $(pwd -P | grep -q "^\/mnt\/c\/*"); then
    git.exe "$@"
  else
    command git "$@"
  fi
}
```


## How to speed up Shell Prompt {#how-to-speed-up-shell-prompt}

If you have configured a fancy shell prompt, powerlevel10k for example, it will automatically get the git status when you enter a git repo. It will take a long time to show the prompt inside a host's repo. You can accelerate it with two methods.
The first one is disable git status in prompt. You may edit the `.p10k.zsh` file and comment the `vcs` prompt element. Therefor, it will not get git status when enter a git repo. However, you can't see the git status though you are in WSL repo.

The second way is to disable untracked file check. You can run this command to disable it:

```bash
# stop checking for unstaged and staged changes
git config bash.showdirtystate false
# stop checking for untracked files
git config bash.showuntrackedfiles false
```

In this way, you can still see other git status such as branch name and staged files with a instant response.


## Ref {#ref}

-   [Git status and checkout very slow on some repository](https://github.com/romkatv/powerlevel10k/issues/1039)
-   [Faster git status under WSL2](https://markentier.tech/posts/2020/10/faster-git-under-wsl2/)
