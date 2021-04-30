---
title: "macOS 自动挂载 Kerberized NFS"
cover: "/static/images/cover.jpg"
tags: ["macOS", "NFS"]
date: 2017-11-22T20:55:41+08:00
draft: true
---

每次开机需要重新运行 kinit
fstab 不会自动挂载
需要 root 权限手动创建 /Volumes/nfs 文件夹
shell shebang 不能 setuid
