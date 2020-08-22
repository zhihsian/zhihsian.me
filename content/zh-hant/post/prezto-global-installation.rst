---
title: "全局安裝 Prezto"
tags: ["prezto"]
date: 2016-09-13T21:30:52+08:00
---

由於我經常會在 root 和普通用戶之間切換，然後又不想每個用戶都裝一次 prezto。所以這時候，全局安裝 prezto 就可以辣～
然而，從 prezto 的代碼上來看，它並不支持這種方式，因爲它裏面的路徑，都是以 :code:`${ZDOTDIR:-$HOME}/` 開頭的，這就決定了，如果自定義了 prezto 安裝目錄的話，它會把所有的緩存都放在 :code:`$ZDOTDIR` 目錄下面。導致 root 與普通用戶的緩存之類的都在一起了。然後，權限問題就出來了～

所以，修改起來很簡單～

1. 因爲要全局安裝 prezto，所以，如果路徑中還出現類似於 :code:`${ZDOTDIR:-$HOME}/.zprezto` 的就不好啦～
    1. 定義一個變量 :code:`PREZTO='/usr/local/share/zprezto'`
    2. 把代碼中的 :code:`${ZDOTDIR:-$HOME}/.zprezto` 都替換成 :code:`$PREZTO`

2. 剩下的，帶有 :code:`${ZDOTDIR:-$HOME}` 路徑的，就是 :code:`.zcompcache`、:code:`.zhistory`、:code:`.zcompdump`、:code:`.zprofile` 幾個文件辣，這幾個文件直接放 :code:`$HOME` 下面就好了，所以直接批量替換 :code:`${ZDOTDIR:-$HOME}` 爲 :code:`$HOME`

3. 最後，要把 :code:`PREZTO='/usr/local/share/zprezto'` 放在 :code:`runcoms/zshrc` 最上面，就可以工作辣～

奧，得貼下 GayHub 的地址：`<https://github.com/zijung/prezto>`_

安裝的時候按照 README 來就行了，把 :code:`PREZTO="$HOME"/.prezto` 改為想要安裝的全局路徑
