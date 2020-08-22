---
title: "ArchLinux 安装记录"
tags: ["Arch Linux"]
date: 2017-08-05T22:14:15+08:00
---

装了很多次 Arch Linux 了，然而一直没有记录过，这次记录下。前面的制作启动盘略过。

1. 连接 WiFi
    1. 启动设备
        .. code-block:: shell

            ip link set [无线网卡设备名] up

    2. 连接 Wifi
        .. code-block:: shell

            wpa_passphrase [SSID] [密码] > /etc/wpa_supplicant/wpa.conf
            wpa_supplicant -B -i [无线网卡设备名] -c /etc/wpa_supplicant/wpa.conf

    3. DHCP
        .. code-block:: shell

            dhcpcd [无线网卡设备名]

2. 开启 SSH
    1. 设置密码
        .. code-block:: shell

            passwd

    2. 开启 SSH 服务
        .. code-block:: shell

            systemctl start sshd

3. 分区并过载
    1. 分区
        .. code-block:: shell

            fdisk [硬盘设备文件路径]

    2. 格式化
        1. 格式化 EFI 分区
            .. code-block:: shell

                mkfs.fat -F32 [EFI 分区设备文件路径]

        2. 格式化跟目录
            .. code-block:: shell

                mkfs.xfs -f [根目录分区设备文件路径]

        3. 创建 swap 分区
            .. code-block:: shell

                mkswap [swap 分区设备文件路径]

    3. 挂载目录
        1. 挂载跟目录
            .. code-block:: shell

                mount [根目录分区设备文件路径] /mnt

        2. 创建 EFI 分区挂载文件夹并过载
            .. code-block:: shell

                mkdir /mnt/boot
                mount [EFI 分区设备文件路径] /mnt/boot

        3. 开启 swap
            .. code-block:: shell

                swapon [swap 分区设备文件路径]

4. 修改 pacman 源
    .. code-block:: shell

        vim /etc/pacman.d/mirrorlist

5. 安装基本程序
    .. code-block:: shell

        pacstrap -i /mnt base base-devel intel-ucode dkms vim sudo zsh openssh git 

6. 生成 fstab
    .. code-block:: shell

        genfstab -U /mnt >> /mnt/etc/fstab

7. 进入 chroot 环境，开始安装软件
    1. 进入 chroot
        .. code-block:: shell

            arch-chroot /mnt /bin/zsh

    2. 设置主机名
        .. code-block:: shell

            echo [主机名] >> /etc/hostname

    3. 设置时区
        .. code-block:: shell

            ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

    4. 设置语言
        1. 生成 local
            1. 修改 /etc/locale.gen
                .. code-block:: shell

                    vim /etc/locale.gen
                    // 取消 en_US.UTF-8 UTF-8、zh_TW.UTF-8 UTF-8、zh_HK.UTF-8 UTF-8、zh_CN.UTF-8 UTF-8 前的注释

            2. 生成 local
                .. code-block:: shell

                    locale-gen

        2. 设置语言
            1. 设置默认语言
                .. code-block:: shell

                    echo LANG=zh_TW.UTF-8 >> /etc/locale.conf
                    echo LANGUAGE=zh_TW:zh_HK:zh_CN:zh >> /etc/locale.conf

    5. 添加用户，并禁止 root 登录
        1. 添加用户
            .. code-block:: shell

                useradd [用户名] -c "[全名]" -m -G wheel -s /bin/zsh

        2. 设置密码
            .. code-block:: shell

                passwd [用户名]

        3. 禁用 root 密码
            .. code-block:: shell

                passwd -l root

        4. 设置 root 的 shell 为 zsh
            .. code-block:: shell

                chsh -s /bin/zsh

        5. 修改 sudoer，允许 wheel 组使用 sudo
            .. code-block:: shell

                visudo

    6. 安装 systemd-boot 引导
        1. 安装 systemd-boot 到 EFI 分区
            .. code-block:: shell

                bootctl install

        2. 获取根目录所在分区的 uuid
            .. code-block:: shell

                blkid [根目录分区设备文件路径]

        3. 修改配置文件

    7. 安装字体
        .. code-block:: shell

            pacman -S wqy-microhei wqy-zenhei ttf-arphic-ukai ttf-arphic-uming wqy-bitmapfont ttf-dejavu ttf-droid ttf-liberation ttf-bitstream-vera noto-fonts noto-fonts-cjk noto-fonts-emoji adobe-source-code-pro-fonts

    8. 安装软件
        1. 安装 fcitx-rime 输入法
            .. code-block:: shell

                pacman -S fcitx-rime fcitx-im fcitx-configtool
                echo 'export GTK_IM_MODULE=fcitx' >> /home/[用户名]/.xprofile
                echo 'export QT_IM_MODULE=fcitx' >> /home/[用户名]/.xprofile
                echo 'export XMODIFIERS=@im=fcitx' >> /home/[用户名]/.xprofile
                chown [用户名]:[用户名] /home/[用户名]/.xprofile

        2. 安装蓝牙、alsa
            .. code-block:: shell

                pacman -S bluez bluez-utils bluedevil alsa-utils alsa-plugins

        3. 安装 KDE
            .. code-block:: shell

                pacman -S xorg sddm plasma-desktop kde-applications kde-l10n-zh_cn kde-l10n-zh_tw kwallet-pam
                sddm --example-config > /etc/sddm.conf
                vim /etc/sddm.conf
                vim /etc/pam.d/sddm

        4. 设置开机自启动服务
            .. code-block:: shell

                systemctl enable systemd-timesyncd
                systemctl enable NetworkManager
                systemctl enable bluetooth
                systemctl enable sddm
