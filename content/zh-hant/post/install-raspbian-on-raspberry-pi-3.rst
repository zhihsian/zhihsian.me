---
title: "樹莓派 3 raspbian（debian） 安裝過程"
tags: ["Raspberry Pi", "Debian", "Raspbian", "樹莓派"]
date: 2017-04-24T00:56:53+08:00
---

前幾天京東滿 199-100，就順便買了個樹莓派 3，記錄下安裝過程

1. 從 `<https://mirrors.tuna.tsinghua.edu.cn/raspbian/images/>`_ 下載 raspbian

1. 通過 dd 將鏡像烤入 CF 卡
    .. code-block:: shell

        sudo dd bs=1M if=rpi_pisces_r3.img of=/dev/sdb && sync

1. 將 CF 卡插入樹莓派中，插上鍵盤和 hdmi 線，接通電源

1. 使用用戶名 pi 和密碼 raspberry 登錄

1. 修改密碼
    .. code-block:: shell

        passwd

1. 設置鍵盤佈局
    1. 先 :code:`export XKBLAYOUT="us"` 讓鍵盤可以正常使用
    2. 然後修改 :code:`/etc/default/keyboard` 中的 :code:`XKBLAYOUT` 爲 :code:`us`

1. 設置 wifi
    1. 確認 /etc/network/interfaces 已經有以下配置（自動調用 wpa_supplicant，我使用的當前版本 pisces 自帶）
        .. code-block::

            allow-hotplug wlan0
            iface wlan0 inet manual
                wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

    2. 將 /etc/wpa_supplicant/wpa_supplicant.conf 加入 wifi 信息即可
        .. code-block::

            wpa_passphrase "SSID" "password" | sudo tee -a /etc/wpa_supplicant/wpa_supplicant.conf

    3. 重啓網絡
        .. code-block:: shell

            sudo systemctl restart networking

1. 修改 apt 源，將 /etc/apt/sources.list 中的 :code:`http://mirrordirector.raspbian.org/raspbian/` 替換成 :code:`https://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/`

1. 修改時區
    .. code-block:: shell

        sudo dpkg-reconfigure tzdata

1. 修改 locales
    .. code-block:: shell

        sudo dpkg-reconfigure locales

1. 使 ssh 連接的時候，能夠使用客戶端發送過來的 LANG 以及 locals
    1. 確認 /etc/ssh/sshd_config 中有 :code:`AcceptEnv` 且後面至少有 :code:`LANG LC_*`

    2. 修改 /etc/pam.d 下面的 sshd、sudo、su 等文件，將其中類似於下面的行註釋掉
        .. code-block::

            session    required     pam_env.so user_readenv=1 envfile=/etc/default/locale

1. 關機或者重啓的時候，ssh 自動斷開
    1. 確認 /etc/ssh/sshd_config 中有 :code:`UsePAM yes`
    2. 安裝 libpam-systemd
        .. code-block:: shell

            sudo apt install libpam-systemd

1. 讓 sshd 自啓動
    .. code-block:: shell

        sudo systemctl enable ssh

1. 更新
    .. code-block:: shell

        sudo apt update && sudo apt dist-upgrade

1. 安裝 zsh，然後讓 pi 和 root 全部使用 zsh
    .. code-block:: shell

        sudo apt install zsh
        chsh -s /bin/zsh
        sudo chsh -s /bin/zsh

1. 安裝 prezto
