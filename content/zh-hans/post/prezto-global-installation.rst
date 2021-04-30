---
title: "全局安装 Prezto"
tags: ["prezto"]
date: 2016-09-13T21:30:52+08:00
---

由于我经常会在 root 和普通用户之间切换，然后又不想每个用户都装一次 prezto。所以这时候，全局安装 prezto 就可以辣～
然而，从 prezto 的代码上来看，它并不支持这种方式，因为它里面的路径，都是以 :code:`${ZDOTDIR:-$HOME}/` 开头的，这就决定了，如果自定义了 prezto 安装目录的话，它会把所有的缓存都放在 :code:`$ZDOTDIR` 目录下面。导致 root 与普通用户的缓存之类的都在一起了。然后，权限问题就出来了～

所以，修改起来很简单～

1. 因为要全局安装 prezto，所以，如果路径中还出现类似于 :code:`${ZDOTDIR:-$HOME}/.zprezto` 的就不好啦～
    1. 定义一个变量 :code:`PREZTO='/usr/local/share/zprezto'`
    2. 把代码中的 :code:`${ZDOTDIR:-$HOME}/.zprezto` 都替换成 :code:`$PREZTO`

2. 剩下的，带有 :code:`${ZDOTDIR:-$HOME}` 路径的，就是 :code:`.zcompcache`、:code:`.zhistory`、:code:`.zcompdump`、:code:`.zprofile` 几个文件辣，这几个文件直接放 :code:`$HOME` 下面就好了，所以直接批量替换 :code:`${ZDOTDIR:-$HOME}` 为 :code:`$HOME`

3. 最后，要把 :code:`PREZTO='/usr/local/share/zprezto'` 放在 :code:`runcoms/zshrc` 最上面，就可以工作辣～

奥，得贴下 GayHub 的地址：`<https://github.com/zijung/prezto>`_

安装的时候按照 README 来就行了，把 :code:`PREZTO="$HOME"/.prezto` 改为想要安装的全局路径
