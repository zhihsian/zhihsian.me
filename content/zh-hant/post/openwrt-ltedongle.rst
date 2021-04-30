---
title: "OpenWRT 使用 qmi 實現 4G 訪問"
tags: ["OpenWRT", "LEDE", "qmi"]
date: 2017-06-29T22:24:21+08:00
lastmod: 2018-09-16T13:19:27+8:00
---

UPDATE:
1. 更新爲最新的 OpenWRT
2. 更新 OpenWRT 官方文檔的地址

參考文檔：

1. `How To use LTE modem in QMI mode for WAN connection`_


我使用的設備的 GL.iNet GL-MiFi，自帶的 LTE 模塊是 QUECTEL EC20-CE，雖然官方提供了定製的 OpenWRT 系統，但是由於是使用的 :code:`comgt`，似乎也只支持到 3G，然後從沒信號的地方出來，必須重啓才能夠上網，所以我就換了最新版的原版 OpenWRT（UPDATE：官方固件也開始使用 qmi 了）。

1. 安裝必要的包
    .. code-block:: shell

        opkg update
        opkg install usb-modeswitch kmod-mii kmod-usb-net kmod-usb-wdm kmod-usb-net-qmi-wwan uqmi

    如果需要通過 Luci 管理的話，還得安裝 :code:`luci-proto-qmi`

2. 配置網絡，修改 :code:`/etc/config/network` 文件在末尾添加
    .. code-block::

        config interface 'modem'
            option ifname 'wwan0'
            option proto 'qmi'
            option device '/dev/cdc-wdm0'
            option apn '3gnet'

    這裏的「apn」根據自己的運營商填寫不同的內容，如聯通的 3gnet、移動的 cmnet、電信的 ctnet，「pincode」也需要根據自己的設置來填寫，如果沒有開啓，可以直接刪除

3. 配置防火牆，修改 :code:`/etc/config/firewall` 將 modem 加入到 wan 中
    .. code-block::

        config zone
            option name wan
            ···
            list network 'modem'
            ···

4. 配置 led 燈，修改 :code:`/etc/config/system`
    .. code-block::

        config led 'led_3gnet'
            ···
            option dev 'wwan0'

3. 重啓，開機之後，如果沒有什麼問題，就可以上網啦～

.. _`How To use LTE modem in QMI mode for WAN connection`: https://openwrt.org/docs/guide-user/network/wan/wwan/ltedongle
