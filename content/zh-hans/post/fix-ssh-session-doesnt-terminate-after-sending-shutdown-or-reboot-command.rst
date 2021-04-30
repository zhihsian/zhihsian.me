---
title: "解决 debian、ubuntu 关机／重启时 ssh 不断开"
tags: ["Debian", "Ubuntu", "SSH"]
date: 2017-04-24T00:52:00+08:00
---

以前用 debian 或者 ubuntu server 的时候经常遇到关机或者重启的时候 ssh 不断开，当时都是直接关掉 terminal，这次懒癌稍微好了点，就查了查，顺便做个备忘

1. 确认 /etc/ssh/sshd_config 中有 :code:`UsePAM yes`
2. 安装 libpam-systemd
    .. code-block:: shell

        sudo apt install libpam-systemd
