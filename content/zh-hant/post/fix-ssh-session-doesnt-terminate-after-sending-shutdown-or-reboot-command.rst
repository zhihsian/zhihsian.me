---
title: "解決 debian、ubuntu 關機／重啓時 ssh 不斷開"
tags: ["Debian", "Ubuntu", "SSH"]
date: 2017-04-24T00:52:00+08:00
---

以前用 debian 或者 ubuntu server 的時候經常遇到關機或者重啓的時候 ssh 不斷開，當時都是直接關掉 terminal，這次懶癌稍微好了點，就查了查，順便做個備忘

1. 確認 /etc/ssh/sshd_config 中有 :code:`UsePAM yes`
2. 安裝 libpam-systemd
    .. code-block:: shell

        sudo apt install libpam-systemd
