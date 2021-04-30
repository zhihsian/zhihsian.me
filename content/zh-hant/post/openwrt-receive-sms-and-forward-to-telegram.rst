---
title: "OpenWRT 使用 smstools3 接收短信並轉發到 Telegram"
tags: ["OpenWRT", "LEDE", "SMS", "Telegram"]
date: 2017-06-29T22:44:57+08:00
lastmod: 2018-09-16T13:34:34+8:00
---

上一篇說了如何使用 OpenWRT 來配置 4G 上網，這篇就要說一下接收短信啦～

1. 安裝完 OpenWRT 之後，默認是看不到 :code:`/dev/ttyUSB*` 設備的。所以需要安裝幾個 modules
    按照 `How To use LTE modem in QMI mode for WAN connection`_ 的說法只需要執行下面的步驟

    .. code-block:: shell

        opkg update
        opkg install kmod-usb-serial-option kmod-usb-serial kmod-usb-serial-wwan kmod-usb-serial-qualcomm

    雖然文檔裏沒說要安裝 :code:`kmod-usb-serial-qualcomm`，但是我的設備（QUECTEL EC20-CE）必須安裝 :code:`kmod-usb-serial-qualcomm`。

    重啓，:code:`/dev/ttyUSB*` 都出來啦～

2. 安裝 smstools3 和腳本需要的軟件
    .. code-block:: shell

        opkg install smstools3 ca-bundle curl iconv


3. 修改配置文件 /etc/smsd.conf
    1. 將 device 的值改爲正確的接受 AT命令 的設備地址，我的是 /dev/ttyUSB2
    2. 將 baudrate 的值改爲 115200。指定串行通訊速率，單位 bits/s，如果 115200 不行，可以試試 19200 和 9600
    3. 在全局設置最下面（[GSM1] 上面）添加一行 :code:`eventhandler = /usr/local/bin/pushsms`，意思是，在接收到短信的時候調用這個腳本

4. 創建 /usr/local/bin/pushsms 文件
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
        發件人： ${from}
        發件時間： ${sent}
        收件時間： ${received}

        ${content}
        EOF
        )

          curl -d "chat_id=${chat_id}&text=${text}" -X POST "https://api.telegram.org/bot${token}/sendMessage"
        fi

    其中 token 和 chat_id，按照自己的情況修改

5. 給 /usr/local/bin/pushsms 添加可執行權限 :code:`sudo chmod +x /usr/local/bin/pushsms`

.. _`How To use LTE modem in QMI mode for WAN connection`: https://wiki.openwrt.org/doc/recipes/ltedongle
