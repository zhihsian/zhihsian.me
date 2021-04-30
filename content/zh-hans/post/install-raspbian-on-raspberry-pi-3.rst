---
title: "树莓派 3 raspbian（debian） 安装过程"
tags: ["Raspberry Pi", "Debian", "Raspbian", "树莓派"]
date: 2017-04-24T00:56:53+08:00
---

前几天京东满 199-100，就顺便买了个树莓派 3，记录下安装过程

1. 从 `<https://mirrors.tuna.tsinghua.edu.cn/raspbian/images/>`_ 下载 raspbian

1. 通过 dd 将镜像烤入 CF 卡
    .. code-block:: shell

        sudo dd bs=1M if=rpi_pisces_r3.img of=/dev/sdb && sync

1. 将 CF 卡插入树莓派中，插上键盘和 hdmi 线，接通电源

1. 使用用户名 pi 和密码 raspberry 登录

1. 修改密码
    .. code-block:: shell

        passwd

1. 设置键盘布局
    1. 先 :code:`export XKBLAYOUT="us"` 让键盘可以正常使用
    2. 然后修改 :code:`/etc/default/keyboard` 中的 :code:`XKBLAYOUT` 为 :code:`us`

1. 设置 wifi
    1. 确认 /etc/network/interfaces 已经有以下配置（自动调用 wpa_supplicant，我使用的当前版本 pisces 自带）
        .. code-block::

            allow-hotplug wlan0
            iface wlan0 inet manual
                wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

    2. 将 /etc/wpa_supplicant/wpa_supplicant.conf 加入 wifi 信息即可
        .. code-block::

            wpa_passphrase "SSID" "password" | sudo tee -a /etc/wpa_supplicant/wpa_supplicant.conf

    3. 重启网络
        .. code-block:: shell

            sudo systemctl restart networking

1. 修改 apt 源，将 /etc/apt/sources.list 中的 :code:`http://mirrordirector.raspbian.org/raspbian/` 替换成 :code:`https://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/`

1. 修改时区
    .. code-block:: shell

        sudo dpkg-reconfigure tzdata

1. 修改 locales
    .. code-block:: shell

        sudo dpkg-reconfigure locales

1. 使 ssh 连接的时候，能够使用客户端发送过来的 LANG 以及 locals
    1. 确认 /etc/ssh/sshd_config 中有 :code:`AcceptEnv` 且后面至少有 :code:`LANG LC_*`

    2. 修改 /etc/pam.d 下面的 sshd、sudo、su 等文件，将其中类似于下面的行注释掉
        .. code-block::

            session    required     pam_env.so user_readenv=1 envfile=/etc/default/locale

1. 关机或者重启的时候，ssh 自动断开
    1. 确认 /etc/ssh/sshd_config 中有 :code:`UsePAM yes`
    2. 安装 libpam-systemd
        .. code-block:: shell

            sudo apt install libpam-systemd

1. 让 sshd 自启动
    .. code-block:: shell

        sudo systemctl enable ssh

1. 更新
    .. code-block:: shell

        sudo apt update && sudo apt dist-upgrade

1. 安装 zsh，然后让 pi 和 root 全部使用 zsh
    .. code-block:: shell

        sudo apt install zsh
        chsh -s /bin/zsh
        sudo chsh -s /bin/zsh

1. 安装 prezto
