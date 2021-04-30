---
title: "访客 WiFi"
tags: ["OpenWRT", "LEDE", "MIFI"]
date: 2018-09-10T23:09:34+08:00
draft: true
---

Windows 10 1809 的 install.wim 已经超过 4G 了。而 BootCamp 在制作 Windows 10 启动分区的时候，会使用 FAT32，直接导致复制 install.wim 的时候报错。并且，Mac 目前的固件也不支持直接从 NTFS 引导系统。

1. U盘分两个分区，一个 NTFS，一个 FAT32（200M即可）
1. 将 iso 镜像内容复制到 NTFS 分区中
1. 将 Grub2 装入 FAT32 分区，chainloader NTFS 中的 /EFI/Boot/bootx64.efi
1. 从 macOS 的 BootCamp 中下载 WindowsSupport
1. 将 WindowsSupport 中的内容复制进 NTFS 分区根目录中
