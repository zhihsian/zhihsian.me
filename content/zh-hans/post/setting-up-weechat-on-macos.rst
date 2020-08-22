---
title: "macOS 设置 weechat"
tags: ["macOS", "IRC", "WeeChat"]
date: 2017-03-08T13:28:42+08:00
---

纯属备忘

1. 安装 weechat
    .. code-block:: shell

        brew install weechat

2. 打开 weechat
    .. code-block:: shell

        weechat

3. 设置证书路径
    默认的 `/etc/ssl/certs/ca-certificates.crt` 并不存在，所以需要手动设置下证书路径，可以使用 macOS 自带的 `/etc/ssl/cert.pem` 或者通过 brew 安装的 openssl 的 `/usr/local/etc/openssl/cert.pem`
    .. code-block::

        /set weechat.network.gnutls_ca_file "/usr/local/etc/openssl/cert.pem"

4. 设置默认暱称，逗号分割按优先级排序的多个，当前面的被占用时，尝试时候后面的
    .. code-block::

        /set irc.server_default.nicks "zijung,zijungch"

4. 添加 freenode 服务器
    .. code-block::

        /server add freenode chat.freenode.net/6697 -ssl

5. 设置 freenode 的 SASL 验证信息
    .. code-block::

        /set irc.server.freenode.sasl_username "zijung"
        /set irc.server.freenode.sasl_password "**********"

6. 进入 freenode
    .. code-block::

        /connect freenode

7. 进入 channel
    .. code-block::

        /join #archlinux-cn
