---
title: "macOS 連接不支持 ipsec 的 L2TP VPN 並手動設置路由"
tags: ["macOS", "L2TP", "IPSec"]
date: 2017-05-18T12:42:14+08:00
---

不知道這篇會不會導致我的博客消失

基本上所有買的 Mac 的朋友都問過我 Mac 如何連接 VPN 這個問題（俺現在的公司就是使用的不支持 ipsec 的 L2TP 服務器，而剛好有個離職的兄弟的新公司給他配了 Macbook Pro（表示羨慕），而新公司也剛好是使用的不支持 ipsec 的 L2TP）。。。其實這個問題直接 Google 關鍵詞「mac l2tp without ipsec」就能找到答案啊

所以我還是寫出來，到時候有新的使用 Mac 的人 的話，直接甩給他我的博客地址，還能增加點訪問量。

1. 創建文件 :code:`/etc/ppp/options` （需要 sudo），文件內容
    .. code-block::

        plugin L2TP.ppp
        l2tpnoipsec

2. 添加 VPN，注意不要勾選「通過 VPN 連接傳輸所有流量」
3. 連接 VPN
4. Terminal 運行 :code:`netstat -rn`，能找出一條類似於這樣的結果
    .. code-block::

        Destination        Gateway            Flags        Refs      Use   Netif Expire
        10.10.200.1        10.10.200.2        UH              1        0    ppp0

    說明連接 VPN 的接口名是 ppp0；連接上 VPN 之後，遠端服務器的 IP 地址是 10.10.200.1，VPN 服務器分配給電腦的 IP 地址是 10.10.200.2。然後，因爲要操作的公司服務器 IP 段是 10.10.15/24，所以需要把這個網段加入路由錶，讓他們都走 VPN。

5. 創建 :code:`/etc/ppp/ip-up` 文件，他會在每次連接 VPN 的時候被調用，內容
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

    這個腳本意思是，當遠端服務器 IP 地址是 10.10.200.1 的時候，添加路由使 10.10.15/24 這個網段全部走 ppp0 設備。
    關於 $1、$2... 都是啥，腳本上面的註釋裏面也有說，這裏就不再贅述

6. 給 :code:`/etc/ppp/ip-up` 可執行權限：
    .. code-block:: shell

        sudo chmod +x /etc/ppp/ip-up
