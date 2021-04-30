---
title: "macOS 连接不支持 ipsec 的 L2TP VPN 并手动设置路由"
tags: ["macOS", "L2TP", "IPSec"]
date: 2017-05-18T12:42:14+08:00
---

不知道这篇会不会导致我的博客消失

基本上所有买的 Mac 的朋友都问过我 Mac 如何连接 VPN 这个问题（俺现在的公司就是使用的不支持 ipsec 的 L2TP 服务器，而刚好有个离职的兄弟的新公司给他配了 Macbook Pro（表示羡慕），而新公司也刚好是使用的不支持 ipsec 的 L2TP）。。。其实这个问题直接 Google 关键词「mac l2tp without ipsec」就能找到答案啊

所以我还是写出来，到时候有新的使用 Mac 的人 的话，直接甩给他我的博客地址，还能增加点访问量。

1. 创建文件 :code:`/etc/ppp/options` （需要 sudo），文件内容
    .. code-block::

        plugin L2TP.ppp
        l2tpnoipsec

2. 添加 VPN，注意不要勾选「通过 VPN 连接传输所有流量」
3. 连接 VPN
4. Terminal 运行 :code:`netstat -rn`，能找出一条类似于这样的结果
    .. code-block::

        Destination        Gateway            Flags        Refs      Use   Netif Expire
        10.10.200.1        10.10.200.2        UH              1        0    ppp0

    说明连接 VPN 的接口名是 ppp0；连接上 VPN 之后，远端服务器的 IP 地址是 10.10.200.1，VPN 服务器分配给电脑的 IP 地址是 10.10.200.2。然后，因为要操作的公司服务器 IP 段是 10.10.15/24，所以需要把这个网段加入路由表，让他们都走 VPN。

5. 创建 :code:`/etc/ppp/ip-up` 文件，他会在每次连接 VPN 的时候被调用，内容
    .. code-block:: shell

        #!/bin/bash
        #
        # Script which handles the routing issues as necessary for pppd
        # Only the link to Newman requires this handling.
        #
        # When the ppp link comes up, this script is called with the following
        # parameters
        #   $1  the interface name used by pppd (e.g. ppp3)
        #   $2  the tty device name
        #   $3  the tty device speed
        #   $4  the local IP address for the interface
        #   $5  the remote IP address
        #   $6  the parameter specified by the 'ipparam' option to pppd
        #

        case "$5" in
        '10.10.200.1')
            /sbin/route add -net 10.10.15/24 -interface ppp0
            ;;
        *)
        esac

    这个脚本意思是，当远端服务器 IP 地址是 10.10.200.1 的时候，添加路由使 10.10.15/24 这个网段全部走 ppp0 设备。
    关于 $1、$2... 都是啥，脚本上面的注释里面也有说，这里就不再赘述

6. 给 :code:`/etc/ppp/ip-up` 可执行权限：
    .. code-block:: shell

        sudo chmod +x /etc/ppp/ip-up
