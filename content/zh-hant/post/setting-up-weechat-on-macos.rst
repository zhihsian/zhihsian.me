---
title: "macOS 設置 weechat"
tags: ["macOS", "IRC", "WeeChat"]
date: 2017-03-08T13:28:42+08:00
---

純屬備忘

1. 安裝 weechat
    .. code-block:: shell

        brew install weechat

2. 打開 weechat
    .. code-block:: shell

        weechat

3. 設置證書路徑
    默認的 `/etc/ssl/certs/ca-certificates.crt` 並不存在，所以需要手動設置下證書路徑，可以使用 macOS 自帶的 `/etc/ssl/cert.pem` 或者通過 brew 安裝的 openssl 的 `/usr/local/etc/openssl/cert.pem`
    .. code-block::

        /set weechat.network.gnutls_ca_file "/usr/local/etc/openssl/cert.pem"

4. 設置默認暱稱，逗號分割按優先級排序的多個，當前面的被佔用時，嘗試時候後面的
    .. code-block::

        /set irc.server_default.nicks "zijung,zijungch"

4. 添加 freenode 服務器
    .. code-block::

        /server add freenode chat.freenode.net/6697 -ssl

5. 設置 freenode 的 SASL 驗證信息
    .. code-block::

        /set irc.server.freenode.sasl_username "zijung"
        /set irc.server.freenode.sasl_password "**********"

6. 進入 freenode
    .. code-block::

        /connect freenode

7. 進入 channel
    .. code-block::

        /join #archlinux-cn
