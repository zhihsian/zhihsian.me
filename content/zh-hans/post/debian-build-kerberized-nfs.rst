---
title: "Debian 搭建 Kerberized NFS"
tags: ["Debian", "NFS", "Kerberos"]
date: 2017-08-07T00:34:21+08:00
draft: true
---

总把移动硬盘带着上下班非常不方便，于是想着搭一个 NFS 服务，但是呢，NFS 又没有用户验证，也就是说，如果我放在公网的话，那就谁都可以上了。本来想 NFS over SSH 或者 NFS over OpenVPN 来着，不过最后找到了钦点的 Kerberos，那就用这个吧。

1. 首先，只安装 NFS，测试 NFS 没问题
    1. 安装
        .. code-block:: shell

            sudo apt install nfs-kernel-server

    2. 创建需要共享的文件夹
        .. code-block:: shell

            mkdir /srv/nfs

        因为我的 VPS 有俩硬盘，而第二块只有一个分区，且挂载在 /mnt/data 下面，既然是放数据的，自然是不放在系统盘比较好，所以，直接把 /mnt/data/nfs 捆绑（bind）过来，使用命令
        .. code-block:: shell

            sudo mount --bind /mnt/data/nfs /srv/nfs

    3. 修改 /etc/exports 文件，添加下面一行，然后保存
        .. code-block::

            /srv/nfs        *(rw,sync,fsid=0,crossmnt,insecure,no_subtree_check)

        其中 insecure 是因为，Mac 连接 NFS 的时候，默认不使用保留端口号<rt>reserved socket port number</rt>，也可以在 Mac 上挂载的的时候使用 resvport 选项（但是这样的话，就不能直接使用 Finder 或写入 /etc/fstab 之后，使用普通用户挂载了，因为 resvport 必须有 root 权限）

    4. 运行 exportfs 使对 /etc/exports 的修改生肖
        .. code-block:: shell

            sudo exportfs -rva

    5. 开启防火墙（这里使用的是 firewalld）
        .. code-block:: shell

            # 永久开启 NFS 服务
            sudo firewall-cmd --permanent --add-service=nfs
            # 刷新使配置生肖
            sudo firewallctl reload

    6. 本地挂载 NFS 测试
        * Linux 挂载
            .. code-block:: shell

                # 创建目录
                sudo mkdir /mnt/nfs
                sudo mount -t nfs4 nfs.zijung.me:/ /mnt/nfs

        * Mac 挂载
            .. code-block:: shell

                sudo mkdir /mnt/nfs
                sudo mount -t nfs -o vers=4 nfs.zijung.me:/ /mnt/nfs

2. 然后，开始安装 Kerberos，并让 NFS 使用 Kerberos 验证用户
    Kerberos 有两套实现，一个是 MIT 的，一个是 Heimdal 的，这里，我统一使用 MIT 的（Mac 自带的是 Heimdal，但是不影响使用，直接不能远程 kadmin 而已）

    1. 安装 Kerberos
        * 服务端（可以跟 NFS 装在一起，也可以不装在一起）
            .. code-block:: shell

                sudo apt install krb5-admin-server krb5-kdc

            安装的时候会询问两个选项：
            1. realm：默认会是你的域名的大写（如果设置主机名的时候设置了的话，比如我设置的主机名是 xxx.zijung.me，这里就是 ZIJUNG.ME 了），当然随便填也可以，区分大小写
            2. KDC 服务器地址，直接填 Kerberos 服务器所在的服务器的域名就好了
            3. 管理服务器地址<rt>admin server</rt>，同上

            Debian 上安装成功之后，会报错（krb5-kdc.service 启动失败，没关系，因为没有初始化数据库）

        * 客户端（如果 Kerberos 服务器没有跟 NFS 服务器放一起的话，NFS 服务器上也要装这个）
            .. code-block:: shell

                sudo apt install krb5-user

            这里安装的时候也会询问上面两个选项，跟服务端填一样的就好了，也可以安装好之后，把服务端的 /etc/krb5.conf 复制下来

    2. 在服务器上初始化 Kerberos 数据库
        下面的 ZIJUNG.ME 替换成自己的 realm 名称
        .. code-block:: shell

            sudo kdb5_util -r ZIJUNG.ME create -s

        然后重启 krb5-kdc.service
        .. code-block:: shell

            sudo systemctl restart krb5-kdc.service


    3. 生成 NFS 服务器的 
        很晚了，先睡觉
