---
title: "訪客 WiFi"
tags: ["OpenWRT", "LEDE", "MIFI"]
date: 2018-09-10T23:09:34+08:00
draft: true
---

Windows 10 1809 的 install.wim 已經超過 4G 了。而 BootCamp 在製作 Windows 10 啓動分區的時候，會使用 FAT32，直接導致複製 install.wim 的時候報錯。並且，Mac 目前的固件也不支持直接從 NTFS 引導系統。

1. U盤分两個分區，一個 NTFS，一個 FAT32（200M即可）
1. 將 iso 鏡像內容複製到 NTFS 分區中
1. 將 Grub2 裝入 FAT32 分區，chainloader NTFS 中的 /EFI/Boot/bootx64.efi
1. 從 macOS 的 BootCamp 中下載 WindowsSupport
1. 將 WindowsSupport 中的內容複製進 NTFS 分區根目錄中
