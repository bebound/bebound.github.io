+++
title = "Preview LaTeX in Org Mode with Emacs in MacOS"
date = 2019-05-12T20:26:00+08:00
lastmod = 2025-07-18T19:07:22+08:00
tags = ["Emacs", "Org Mode", "LaTeX"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

## Using the right Emacs Version {#using-the-right-emacs-version}

I failed to preview LaTeX with `emacs-plus`. If you have installed `d12frosted/emacs-plus`, uninstall it and use `emacs-mac`.

```nil
brew tap railwaycat/emacsmacport
brew install emacs-mac
```

If you like the fancy spacemacs icon, install it with cask: `brew cask install emacs-mac-spacemacs-icon`


## Install Tex {#install-tex}

-   Download and install BasicTeX.pkg [here](http://www.tug.org/mactex/morepackages.html).
-   Add `/Library/TeX/texbin` to PATH.
-   Install `dvisvgm` by `sudo tlmgr update --self && sudo tlmgr install dvisvgm collection-fontsrecommended`


## Emacs settings {#emacs-settings}

-   Add TeX related bin to path: `(setenv "PATH" (concat (getenv "PATH") ":/Library/TeX/texbin"))`
-   Tell Org Mode to create svg images: `(setq org-latex-create-formula-image-program 'dvisvgm)`

Now you can see the rendered LaTeX equation by calling `org-preview-latex-fragment` or using shortcut `,Tx`.

If you want to load LaTeX previews automatically at startup, add this at the beginning of org file: `#+STARTUP: latexpreview`.

---

-   update 31-07-19

    `_` and `...` are not displayed in Emacs, as some fonts are missing. `tlmgr install collection-fontsrecommended` should fix this.

    If `Org Preview Latex` buffer output warn `processing of PostScript specials is disabled (Ghostscript not found)`, run `brew install ghostscript`.
