---
title: "macOS 自動掛載 Kerberized NFS"
cover: "/static/images/cover.jpg"
tags: ["macOS", "NFS"]
date: 2017-11-22T20:55:41+08:00
draft: true
---

每次開機需要重新運行 kinit
fstab 不會自動掛載
需要 root 權限手動創建 /Volumes/nfs 文件夾
shell shebang 不能 setuid
