---
title: "如何通过 mobile provision 重新签名 ipa 并安装到手机"
tags: ["ipa", "Xcode"]
date: 2016-09-18T22:48:59+08:00
---

##########
参考资料：
##########

1. `resign .ipa with new CFBundleIdentifier and certificate`_
2. `How to Resign an iOS App`_

######
步骤：
######

1. 通过 xcode 申请 mobile provision

2. 解压 ipa 文件
    .. code-block:: shell

        unzip "$ipa_path"

3. 删除原有签名
    .. code-block:: shell

        rm -rf Payload/Application.app/_CodeSignature

4. 修改 Bundle ID 为 mobile provision 中的 bundle ID
    .. code-block:: shell

        /usr/libexec/PlistBuddy Payload/Application.app/Info.plist
        Set :CFBundleIdentifier com.mycompany.newbundleidentifier
        save
        quit

5. 复制 mobile provision 文件
    .. code-block:: shell

        cp "$mobile_provision_path" Payload/Application.app/embedded.mobileprovision

6. 添加 provision.plist
    .. code-block:: shell

        security cms -D -i Payload/Application.app/embedded.mobileprovision > provision.plist
        /usr/libexec/PlistBuddy -x -c 'Print :Entitlements' provision.plist > entitlements.plist

7. 查找证书名称
    .. code-block:: shell

        security find-identity -p codesigning

8. 重新签名
    .. code-block:: shell

        codesign -f -s "$identiti_name" --entitlements entitlements.plist Payload/Application.app

9. 打包
    .. code-block:: shell

        zip -qr resigned.ipa Payload/

10. 最后，用 xcode 安装到手机


.. _resign .ipa with new CFBundleIdentifier and certificate: https://coderwall.com/p/qwqpnw/resign-ipa-with-new-cfbundleidentifier-and-certificate
.. _How to Resign an iOS App: https://stackoverflow.com/a/37172815
