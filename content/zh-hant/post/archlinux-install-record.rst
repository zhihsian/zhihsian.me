---
title: "ArchLinux 安裝記錄"
tags: ["Arch Linux"]
date: 2017-08-05T22:14:15+08:00
---

裝了很多次 Arch Linux 了，然而一直沒有記錄過，這次記錄下。前面的製作啓動盤略過。

1. 連接 WiFi
    1. 啓動設備
        .. code-block:: shell

            ip link set [無線網卡設備名] up

    2. 連接 Wifi
        .. code-block:: shell

            wpa_passphrase [SSID] [密碼] > /etc/wpa_supplicant/wpa.conf
            wpa_supplicant -B -i [無線網卡設備名] -c /etc/wpa_supplicant/wpa.conf

    3. DHCP
        .. code-block:: shell

            dhcpcd [無線網卡設備名]

2. 開啓 SSH
    1. 設置密碼
        .. code-block:: shell

            passwd

    2. 開啓 SSH 服務
        .. code-block:: shell

            systemctl start sshd

3. 分區並過載
    1. 分區
        .. code-block:: shell

            fdisk [硬盤設備文件路徑]

    2. 格式化
        1. 格式化 EFI 分區
            .. code-block:: shell

                mkfs.fat -F32 [EFI 分區設備文件路徑]

        2. 格式化跟目錄
            .. code-block:: shell

                mkfs.xfs -f [根目錄分區設備文件路徑]

        3. 創建 swap 分區
            .. code-block:: shell

                mkswap [swap 分區設備文件路徑]

    3. 掛載目錄
        1. 掛載跟目錄
            .. code-block:: shell

                mount [根目錄分區設備文件路徑] /mnt

        2. 創建 EFI 分區掛載文件夾並過載
            .. code-block:: shell

                mkdir /mnt/boot
                mount [EFI 分區設備文件路徑] /mnt/boot

        3. 開啓 swap
            .. code-block:: shell

                swapon [swap 分區設備文件路徑]

4. 修改 pacman 源
    .. code-block:: shell

        vim /etc/pacman.d/mirrorlist

5. 安裝基本程序
    .. code-block:: shell

        pacstrap -i /mnt base base-devel intel-ucode dkms vim sudo zsh openssh git 

6. 生成 fstab
    .. code-block:: shell

        genfstab -U /mnt >> /mnt/etc/fstab

7. 進入 chroot 環境，開始安裝軟件
    1. 進入 chroot
        .. code-block:: shell

            arch-chroot /mnt /bin/zsh

    2. 設置主機名
        .. code-block:: shell

            echo [主機名] >> /etc/hostname

    3. 設置時區
        .. code-block:: shell

            ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

    4. 設置語言
        1. 生成 local
            1. 修改 /etc/locale.gen
                .. code-block:: shell

                    vim /etc/locale.gen
                    // 取消 en_US.UTF-8 UTF-8、zh_TW.UTF-8 UTF-8、zh_HK.UTF-8 UTF-8、zh_CN.UTF-8 UTF-8 前的註釋

            2. 生成 local
                .. code-block:: shell

                    locale-gen

        2. 設置語言
            1. 設置默認語言
                .. code-block:: shell

                    echo LANG=zh_TW.UTF-8 >> /etc/locale.conf
                    echo LANGUAGE=zh_TW:zh_HK:zh_CN:zh >> /etc/locale.conf

    5. 添加用戶，並禁止 root 登錄
        1. 添加用戶
            .. code-block:: shell

                useradd [用戶名] -c "[全名]" -m -G wheel -s /bin/zsh

        2. 設置密碼
            .. code-block:: shell

                passwd [用戶名]

        3. 禁用 root 密碼
            .. code-block:: shell

                passwd -l root

        4. 設置 root 的 shell 爲 zsh
            .. code-block:: shell

                chsh -s /bin/zsh

        5. 修改 sudoer，允許 wheel 組使用 sudo
            .. code-block:: shell

                visudo

    6. 安裝 systemd-boot 引導
        1. 安裝 systemd-boot 到 EFI 分區
            .. code-block:: shell

                bootctl install

        2. 獲取根目錄所在分區的 uuid
            .. code-block:: shell

                blkid [根目錄分區設備文件路徑]

        3. 修改配置文件

    7. 安裝字體
        .. code-block:: shell

            pacman -S wqy-microhei wqy-zenhei ttf-arphic-ukai ttf-arphic-uming wqy-bitmapfont ttf-dejavu ttf-droid ttf-liberation ttf-bitstream-vera noto-fonts noto-fonts-cjk noto-fonts-emoji adobe-source-code-pro-fonts

    8. 安裝軟件
        1. 安裝 fcitx-rime 輸入法
            .. code-block:: shell

                pacman -S fcitx-rime fcitx-im fcitx-configtool
                echo 'export GTK_IM_MODULE=fcitx' >> /home/[用戶名]/.xprofile
                echo 'export QT_IM_MODULE=fcitx' >> /home/[用戶名]/.xprofile
                echo 'export XMODIFIERS=@im=fcitx' >> /home/[用戶名]/.xprofile
                chown [用戶名]:[用戶名] /home/[用戶名]/.xprofile

        2. 安裝藍牙、alsa
            .. code-block:: shell

                pacman -S bluez bluez-utils bluedevil alsa-utils alsa-plugins

        3. 安裝 KDE
            .. code-block:: shell

                pacman -S xorg sddm plasma-desktop kde-applications kde-l10n-zh_cn kde-l10n-zh_tw kwallet-pam
                sddm --example-config > /etc/sddm.conf
                vim /etc/sddm.conf
                vim /etc/pam.d/sddm

        4. 設置開機自啓動服務
            .. code-block:: shell

                systemctl enable systemd-timesyncd
                systemctl enable NetworkManager
                systemctl enable bluetooth
                systemctl enable sddm
