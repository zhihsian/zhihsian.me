---
title: "OpenWRT 使用 smstools3 接收短信并转发到 Telegram"
tags: ["OpenWRT", "LEDE", "SMS", "Telegram"]
date: 2017-06-29T22:44:57+08:00
lastmod: 2018-09-16T13:34:34+8:00
---

上一篇说了如何使用 OpenWRT 来配置 4G 上网，这篇就要说一下接收短信啦～

1. 安装完 OpenWRT 之后，默认是看不到 :code:`/dev/ttyUSB*` 设备的。所以需要安装几个 modules
    按照 `How To use LTE modem in QMI mode for WAN connection`_ 的说法只需要执行下面的步骤

    .. code-block:: shell

        opkg update
        opkg install kmod-usb-serial-option kmod-usb-serial kmod-usb-serial-wwan kmod-usb-serial-qualcomm

    虽然文档里没说要安装 :code:`kmod-usb-serial-qualcomm`，但是我的设备（QUECTEL EC20-CE）必须安装 :code:`kmod-usb-serial-qualcomm`。

    重启，:code:`/dev/ttyUSB*` 都出来啦～

2. 安装 smstools3 和脚本需要的软件
    .. code-block:: shell

        opkg install smstools3 ca-bundle curl iconv


3. 修改配置文件 /etc/smsd.conf
    1. 将 device 的值改为正确的接受 AT命令 的设备地址，我的是 /dev/ttyUSB2
    2. 将 baudrate 的值改为 115200。指定串行通讯速率，单位 bits/s，如果 115200 不行，可以试试 19200 和 9600
    3. 在全局设置最下面（[GSM1] 上面）添加一行 :code:`eventhandler = /usr/local/bin/pushsms`，意思是，在接收到短信的时候调用这个脚本

4. 创建 /usr/local/bin/pushsms 文件
    .. code-block:: shell

        #!/bin/sh

        token='xxxxxxxx'
        chat_id='xxxxxxxx'

        if [ "$1" == "RECEIVED" ]; then
          from=$(grep "From:" $2 | awk -F ': ' '{printf $2}')
          sent=$(grep "Sent:" $2 | awk -F ': ' '{printf $2}')
          received=$(grep "Received:" $2 | awk -F ': ' '{printf $2}')
          alphabet=$(grep "Alphabet:" $2 | awk -F ': ' '{printf $2}')

          if [ "$alphabet" = "UCS2" ]; then
            content=$(sed -e '1,/^$/ d' < "$2" | iconv -f UNICODEBIG -t UTF-8)
          else
            content=$(sed -e '1,/^$/ d' < "$2")
          fi

          text=$(cat <<EOF
        发件人： ${from}
        发件时间： ${sent}
        收件时间： ${received}

        ${content}
        EOF
        )

          curl -d "chat_id=${chat_id}&text=${text}" -X POST "https://api.telegram.org/bot${token}/sendMessage"
        fi

    其中 token 和 chat_id，按照自己的情况修改

5. 给 /usr/local/bin/pushsms 添加可执行权限 :code:`sudo chmod +x /usr/local/bin/pushsms`

.. _`How To use LTE modem in QMI mode for WAN connection`: https://wiki.openwrt.org/doc/recipes/ltedongle
