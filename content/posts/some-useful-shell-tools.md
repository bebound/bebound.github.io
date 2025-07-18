+++
title = "Some Useful Shell Tools"
date = 2017-05-07T15:34:00+08:00
lastmod = 2025-07-18T19:07:22+08:00
tags = ["Shell"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Here are some shell tools I use, which can boost your productivity. [Mordern-unix](https://github.com/johnalanwoods/maintained-modern-unix) is a great repo that list lots of modern unix tools.


## [Prezto](https://github.com/sorin-ionescu/prezto) {#prezto}

A zsh configuration framework. Provides auto completion, prompt theme and lots of modules to work with other useful tools. I extremely love the `agnoster` theme.

{{< figure src="/images/shell_agnoster.png" class="image-size-s" >}}


## [Fasd](https://github.com/clvv/fasd) {#fasd}

Help you to navigate between folders and launch application.

Here are the official usage example:

```nil
v def conf       =>     vim /some/awkward/path/to/type/default.conf
j abc            =>     cd /hell/of/a/awkward/path/to/get/to/abcdef
m movie          =>     mplayer /whatever/whatever/whatever/awesome_movie.mp4
o eng paper      =>     xdg-open /you/dont/remember/where/english_paper.pdf
vim `f rc lo`    =>     vim /etc/rc.local
vim `f rc conf`  =>     vim /etc/rc.conf
```


## [pt](https://github.com/monochromegane/the_platinum_searcher) {#pt}

A fast code search tool similar to `ack`.


## [fzf](https://github.com/junegunn/fzf) {#fzf}

A great fuzzy finder, it can also integrate with vim by [fzf.vim](https://github.com/junegunn/fzf.vim)

{{< figure src="/images/shell_fzf.gif" class="image-size-s" >}}


## [thefuck](https://github.com/nvbn/thefuck) {#thefuck}

Magnificent app which corrects your previous console command.

{{< figure src="/images/shell_thefuck.gif" class="image-size-s" >}}


## [tldr](https://github.com/tldr-pages/tldr) {#tldr}

More concise and user-friendly man pages. (This screenshot uses [powerlevel10k](https://github.com/romkatv/powerlevel10k) theme)

{{< figure src="/images/shell_tldr.png" >}}


## [ripgrep](https://github.com/BurntSushi/ripgrep) {#ripgrep}

Another search tool. Use `rg -.` to include hidden files.


## [fd](https://github.com/sharkdp/fd) {#fd}

A user-friendly alternative to `find`. Ignore hidden files and gitignore file by default.

For example: `fd -H 'flac$'` search all files ends with flac.


## [bat](https://github.com/sharkdp/bat) {#bat}

Similar to `cat` with syntax highlighting and git integration.

{{< figure src="/images/shell_bat.png" >}}


## [Zim](https://github.com/zimfw/zimfw) {#zim}

A fast Zsh framework. You can use OMZ plugin like this:

```shell
export ZSH_CACHE_DIR=~/.cache

zmodule ohmyzsh/ohmyzsh --use degit --source 'plugins/fasd/fasd.plugin.zsh'
```

---

-   update 18/11/19
    Add tldr
    [powerlevel10k](https://github.com/romkatv/powerlevel10k) theme is a fancy ZSH theme

-   update 29/12/21
    Add rg, bat, fd
-   update 06/01/22
    Add Zim
-   update 01/04/24
    Add maintained-modern-unix repo
