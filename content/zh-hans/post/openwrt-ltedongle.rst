---
title: "OpenWRT 使用 qmi 实现 4G 访问"
tags: ["OpenWRT", "LEDE", "qmi"]
date: 2017-06-29T22:24:21+08:00
lastmod: 2018-09-16T13:19:27+8:00
---

UPDATE:
1. 更新为最新的 OpenWRT
2. 更新 OpenWRT 官方文档的地址

参考文档：

1. `How To use LTE modem in QMI mode for WAN connection`_


我使用的设备的 GL.iNet GL-MiFi，自带的 LTE 模块是 QUECTEL EC20-CE，虽然官方提供了定制的 OpenWRT 系统，但是由于是使用的 :code:`comgt`，似乎也只支持到 3G，然后从没信号的地方出来，必须重启才能够上网，所以我就换了最新版的原版 OpenWRT（UPDATE：官方固件也开始使用 qmi 了）。

1. 安装必要的包
    .. code-block:: shell

        opkg update
        opkg install usb-modeswitch kmod-mii kmod-usb-net kmod-usb-wdm kmod-usb-net-qmi-wwan uqmi

    如果需要通过 Luci 管理的话，还得安装 :code:`luci-proto-qmi`

2. 配置网络，修改 :code:`/etc/config/network` 文件在末尾添加
    .. code-block::

        config interface 'modem'
            option ifname 'wwan0'
            option proto 'qmi'
            option device '/dev/cdc-wdm0'
            option apn '3gnet'

    这里的「apn」根据自己的运营商填写不同的内容，如联通的 3gnet、移动的 cmnet、电信的 ctnet，「pincode」也需要根据自己的设置来填写，如果没有开启，可以直接删除

3. 配置防火墙，修改 :code:`/etc/config/firewall` 将 modem 加入到 wan 中
    .. code-block::

        config zone
            option name wan
            ···
            list network 'modem'
            ···

4. 配置 led 灯，修改 :code:`/etc/config/system`
    .. code-block::

        config led 'led_3gnet'
            ···
            option dev 'wwan0'

3. 重启，开机之后，如果没有什么问题，就可以上网啦～

.. _`How To use LTE modem in QMI mode for WAN connection`: https://openwrt.org/docs/guide-user/network/wan/wwan/ltedongle
