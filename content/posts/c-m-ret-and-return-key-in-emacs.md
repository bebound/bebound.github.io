+++
title = "C-m, RET and Return Key in Emacs"
date = 2020-04-11T21:23:00+08:00
lastmod = 2025-07-18T19:07:22+08:00
tags = ["Emacs"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

I use Emacs to write blog. In the recent update, I found `M-RET` no longer behave as leader key in org mode, but behave as `org-meta-return`. And even more strange is that in other mode, it behave as leader key. And `M-RET` also works in terminal in org mode. In GUI, pressing `C-M-m` can trigger leader key.

SO I opened this [issue](https://github.com/syl20bnr/spacemacs/issues/13374), with the help of these friends, the issue has been fixed. Here is the cause of the bug.

In Emacs, `RET` is not a key in keyboard, it's a logical key). Emacs bind `RET` to `C-m` in source code. In terminal, `<Enter>` and `C-m` both send `<CR>` (ASCII 13) character, so `<Enter>` / `<Return>` key is equal to `RET`. In GUI, pressing `<Enter>` / `<Return>` key actually sends `<return>` to Emacs, and Emacs automatically translate `<return>` to `RET`.

{{< figure src="/images/emacs.png" >}}

This can be proved: type `SPC h d k <Enter>` in spacemacs, it will output `RET (translated from <return>) runs the command org-open-at-point, which is an
interactive compiled Lisp function in ‘org.el’.`

Pressing `C-m` or `<Enter>` key usually given the same result, but you can also bind these with two different command. Take `M-RET` as example. If only `<M-return>` is bind, the `M-RET` is unbinded. If only `M-RET` is binded, then `M-return` is implicitly also bind to same command as `M-RET`.

In org mode [scr](https://github.com/bzg/org-mode/blob/093e65ecc74767fb6452f5b9cf13abc4c2f44917/lisp/org-keys.el#L468-L469):

```elisp
(org-defkey org-mode-map (kbd "M-<return>") #'org-meta-return)
(org-defkey org-mode-map (kbd "M-RET") #'org-meta-return)
```

These two keys were binded to `org-meta-return`.

The unfixed Spacemacs configuration file binds `C-M-m` as `dotspacemacs-major-mode-emacs-leader-key`.

In GUI, the `<Enter>` key will send `<return>` to Emacs. Org mode has explicitly bind `M-<return>` to `org-meta-return`, so `org-meta-return` is triggered. In other mode, the `M-<return>` key binding is not defined, so `<return>` will translate to `RET`, then trigger leader key.

In the fixed version, `dotspacemacs-major-mode-emacs-leader-key` bind to `M-<return>` in GUI, and this override org mode's binding. Finally meta return becomes leader key again.


## Ref {#ref}

1.  [M-RET no longer org mode prefix in GUI](https://github.com/syl20bnr/spacemacs/issues/13374)
2.  [Difference between the physical “RET” key and the command 'newline in the minibuffer](https://emacs.stackexchange.com/questions/14943/difference-between-the-physical-ret-key-and-the-command-newline-in-the-minibu)
3.  [Emacs中的 return, RET, Enter, Ctrl-m解析](http://www.zhangley.com/article/emacs-ret/)
4.  [How to turn off alternative Enter with Ctrl+M in Linux](https://stackoverflow.com/questions/2298811/how-to-turn-off-alternative-enter-with-ctrlm-in-linux/)
