---
title: "GL-MiFi OpenWRT LAN 口不插網線導致 dnsmasq 啓動失敗"
tags: ["OpenWRT", "LEDE", "MIFI"]
date: 2018-09-10T23:09:34+08:00
draft: true
---


.. code-block::

    config interface 'lan'
        option type 'bridge'
        option ifname 'eth1'
