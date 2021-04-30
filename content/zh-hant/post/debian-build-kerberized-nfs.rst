---
title: "Debian 搭建 Kerberized NFS"
tags: ["Debian", "NFS", "Kerberos"]
date: 2017-08-07T00:34:21+08:00
draft: true
---

總把移動硬盤帶着上下班非常不方便，於是想着搭一個 NFS 服務，但是呢，NFS 又沒有用戶驗證，也就是說，如果我放在公網的話，那就誰都可以上了。本來想 NFS over SSH 或者 NFS over OpenVPN 來着，不過最後找到了欽點的 Kerberos，那就用這個吧。

1. 首先，只安裝 NFS，測試 NFS 沒問題
    1. 安裝
        .. code-block:: shell

            sudo apt install nfs-kernel-server

    2. 創建需要共享的文件夾
        .. code-block:: shell

            mkdir /srv/nfs

        因爲我的 VPS 有倆硬盤，而第二塊只有一個分區，且掛載在 /mnt/data 下面，既然是放數據的，自然是不放在系統盤比較好，所以，直接把 /mnt/data/nfs 捆綁（bind）過來，使用命令
        .. code-block:: shell

            sudo mount --bind /mnt/data/nfs /srv/nfs

    3. 修改 /etc/exports 文件，添加下面一行，然後保存
        .. code-block::

            /srv/nfs        *(rw,sync,fsid=0,crossmnt,insecure,no_subtree_check)

        其中 insecure 是因爲，Mac 連接 NFS 的時候，默認不使用保留端口號<rt>reserved socket port number</rt>，也可以在 Mac 上掛載的的時候使用 resvport 選項（但是這樣的話，就不能直接使用 Finder 或寫入 /etc/fstab 之後，使用普通用戶掛載了，因爲 resvport 必須有 root 權限）

    4. 運行 exportfs 使對 /etc/exports 的修改生肖
        .. code-block:: shell

            sudo exportfs -rva

    5. 開啓防火牆（這裏使用的是 firewalld）
        .. code-block:: shell

            # 永久開啓 NFS 服務
            sudo firewall-cmd --permanent --add-service=nfs
            # 刷新使配置生肖
            sudo firewallctl reload

    6. 本地掛載 NFS 測試
        * Linux 掛載
            .. code-block:: shell

                # 創建目錄
                sudo mkdir /mnt/nfs
                sudo mount -t nfs4 nfs.zijung.me:/ /mnt/nfs

        * Mac 掛載
            .. code-block:: shell

                sudo mkdir /mnt/nfs
                sudo mount -t nfs -o vers=4 nfs.zijung.me:/ /mnt/nfs

2. 然後，開始安裝 Kerberos，並讓 NFS 使用 Kerberos 驗證用戶
    Kerberos 有兩套實現，一個是 MIT 的，一個是 Heimdal 的，這裏，我統一使用 MIT 的（Mac 自帶的是 Heimdal，但是不影響使用，直接不能遠程 kadmin 而已）

    1. 安裝 Kerberos
        * 服務端（可以跟 NFS 裝在一起，也可以不裝在一起）
            .. code-block:: shell

                sudo apt install krb5-admin-server krb5-kdc

            安裝的時候會詢問兩個選項：
            1. realm：默認會是你的域名的大寫（如果設置主機名的時候設置了的話，比如我設置的主機名是 xxx.zijung.me，這裏就是 ZIJUNG.ME 了），當然隨便填也可以，區分大小寫
            2. KDC 服務器地址，直接填 Kerberos 服務器所在的服務器的域名就好了
            3. 管理服務器地址<rt>admin server</rt>，同上

            Debian 上安裝成功之後，會報錯（krb5-kdc.service 啓動失敗，沒關係，因爲沒有初始化數據庫）

        * 客戶端（如果 Kerberos 服務器沒有跟 NFS 服務器放一起的話，NFS 服務器上也要裝這個）
            .. code-block:: shell

                sudo apt install krb5-user

            這裏安裝的時候也會詢問上面兩個選項，跟服務端填一樣的就好了，也可以安裝好之後，把服務端的 /etc/krb5.conf 複製下來

    2. 在服務器上初始化 Kerberos 數據庫
        下面的 ZIJUNG.ME 替換成自己的 realm 名稱
        .. code-block:: shell

            sudo kdb5_util -r ZIJUNG.ME create -s

        然後重啓 krb5-kdc.service
        .. code-block:: shell

            sudo systemctl restart krb5-kdc.service


    3. 生成 NFS 服務器的 
        很晚了，先睡覺
