---
title: "OpenWRT 添加訪客 WiFi，並使用 nodogspash 進行 Web 驗證"
tags: ["OpenWRT", "LEDE", "nodogsplash", "Guest WiFi"]
date: 2018-09-10T23:09:34+08:00
---

參考文檔：

1. `OpenWrt Project: Configure a guest WLAN`_


發現我的 WiFi 被人把密碼共享出去了，所以我決定，建一個訪客 WiFi，以後只給別人訪客 WiFi 的密碼，並且加上 Web 驗證（連上來也別想上網）


1. 添加 vlan，修改 :code:`/etc/config/network`，複製一次 lan，然後改名爲 lan_guest，並且將 ifname 改爲 eth1.2（看情況修改），然後加一個 switch vlan（如果不支持也可以不添加），只 target CPU
    .. code-block::

        config interface 'lan_guest'
            option type 'bridge'
            option ifname 'eth1.2'
            option proto 'static'
            option ipaddr '192.168.255.1'
            option netmask '255.255.255.0'

        config switch_vlan
            option device 'switch0'
            option vlan '2'
            option ports '0t'

2. 配置防火牆，修改 :code:`/etc/config/firewall`，添加下面幾行
    .. code-block::

        # Guest start
        config zone
            option name         lan_guest
            list   network      'lan_guest'
            option input        REJECT
            option output       ACCEPT
            option forward      REJECT

        config forwarding
            option src          lan_guest
            option dest         wan

        config rule
            option name         Allow-Guest-DHCP
            option src          lan_guest
            option proto        udp
            option src_port     68
            option dest_port    67
            option target       ACCEPT

        config rule
            option name         Allow-Guest-DNS
            option src          lan_guest
            option proto        udp
            option dest_port    53
            option target       ACCEPT
        # Guest end

3. 配置 WiFi，修改 :code:`/etc/config/wireless`，添加下面幾行
    .. code-block::

        config wifi-iface 'guest_radio0'
            option device 'radio0'
            option network 'lan_guest'
            option mode 'ap'
            option ssid 'NO NAME'
            option encryption 'psk2+ccmp'
            option key '0123456789'
            option isolate '1'

4. 配置 DHCP，修改 :code:`/etc/config/dhcp`，添加下面幾行
    .. code-block::

        config dhcp lan_guest
            option interface    lan_guest
            option start     100
            option limit    5
            option leasetime    1h
            option dhcp_option    '6,199.29.29.29'

5. 配置 nodogsplash，修改 :code:`/etc/config/nodogsplash`
    1. 設置綁定的 interface，這裏使用「br-」加 vlan 的 interface 名稱
        .. code-block::

            option gatewayinterface 'br-lan_guest'

    2. 註釋掉所有 :code:`authenticated_users` 的規則，因爲已驗證用戶的規則如果爲空，會 fallback 到自帶防火牆（我覺得上面配置的自帶防火牆的規則剛剛好）
        .. code-block::

            #list authenticated_users 'allow all'

    3. 開啓 :code:`preauthenticated_users` 的 53 端口，讓未驗證用戶可以解析域名
        .. code-block::

            list preauthenticated_users 'allow tcp port 53'
            list preauthenticated_users 'allow udp port 53'

    4. :code:`users_to_router` 僅允許 53（DNS）、67（DHCP） 端口，註釋掉其他端口
        .. code-block::

            #list users_to_router 'allow tcp port 22'
            #list users_to_router 'allow tcp port 23'
            list users_to_router 'allow tcp port 53'
            list users_to_router 'allow udp port 53'
            list users_to_router 'allow udp port 67'
            #list users_to_router 'allow tcp port 80'

.. _`OpenWrt Project: Configure a guest WLAN`: https://openwrt.org/docs/guide-user/network/wifi/guestwifi/guest-wlan
