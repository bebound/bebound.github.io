+++
title = "Emacs Chinese-related Settings"
author = ["KK"]
date = 2022-02-17T01:27:00+08:00
lastmod = 2022-02-17T01:39:06+08:00
tags = ["Emacs"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

## Auto Switch Input Method in Evil {#auto-switch-input-method-in-evil}

This setting makes it possible to switch input method based on the context of cursor when entering insert mode.


### sis {#sis}

I'm using `sis` package with this configuration. You may need to install `macism` if you're not using `railwaycat/emacsmacport`. More settings can be found in [emacs-smart-input-source](https://github.com/laishulu/emacs-smart-input-source).

```elisp
(sis-ism-lazyman-config
  "com.apple.keylayout.US"
  "com.apple.inputmethod.SCIM.ITABC")

(sis-global-cursor-color-mode t)
(sis-global-respect-mode t)
(sis-global-context-mode t)
(sis-global-inline-mode t)
```


### fcitx {#fcitx}

You can also install [fcitx-remote for-osx](https://github.com/xcodebuild/fcitx-remote-for-osx) and use `cute-jumper/fcitx.el` to do so. As `homebrew` no longer support some build options, you need to follow the install instructions in the GitHub repository to build `fcitx`.


## Mono Chinese Font {#mono-chinese-font}

I use a 14pt English font and 16pt Chinese font, one Chinese character is the same width as two English characters. It can be set by adding this into Emacs configuration file.

```elisp
dotspacemacs-default-font '("Menlo"
                            :size 14.0
                            :weight normal
                            :width normal)

;; add into dotspacemacs/user-config()
(dolist (charset '(kana han symbol cjk-misc bopomofo))
   (set-fontset-font (frame-parameter nil 'font)
                     charset (font-spec :family "PingFang SC"
                                        :size 16)))
```

If you enable the `chinese` layer in Spacemacs, it provides a more convenient function:

```elisp
(spacemacs//set-monospaced-font   "Menlo" "PingFang SC" 14 16)
```

PS: [valign](https://github.com/casouri/valign) provides visual alignment for Org Mode and Markdown without changing fonts.

Ref:

1.  [Spacemacs - Chinese Layer](https://develop.spacemacs.org/layers/+intl/chinese/README.html)
2.  [Emacs 中文环境配置](https://blindwith.science/2019/07/443.html/)