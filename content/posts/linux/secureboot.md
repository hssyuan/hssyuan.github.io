---
title: "在Debian+自定义内核上启用安全启动"
date: 2024-09-02T21:48:27+08:00
draft: false
tags: ["linux"]
---

## 注意：

Debian/Fedora/Ubuntu 官方支持了安全启动，因此属于‘使用已签署的引导加载程序’，支持的方式为shim

根据Debian wiki对安全启动的描述中可以得知，Debian自带了EFI系统签名因此无需手动注册，但是由dkms加载的内核模块没有经过MOK的注册，因此需要手动注册：

```shell
sudo mokutil --import /var/lib/dkms/mok.pub #由MOK注册，执行完需要重启
```

接下来，由于使用的是自定义内核，所以内核也需要**手动签名**后由MOK注册：

```shell
#创建一个机器所有者密钥（MOK）：
openssl req -newkey rsa:4096 -nodes -keyout MOK.key -new -x509 -sha256 -days 3650 -subj "/CN=my Machine Owner Key/" -out MOK.crt
openssl x509 -outform DER -in MOK.crt -out MOK.cer

#手动签署你的引导加载程序（名为grubx64.efi）以及内核：
sbsign --key MOK.key --cert MOK.crt --output /boot/vmlinuz-linux /boot/vmlinuz-linux
sbsign --key MOK.key --cert MOK.crt --output esp/EFI/BOOT/grubx64.efi esp/EFI/BOOT/grubx64.efi

sudo mokutil --import MOK.cer #由MOK注册，执行完需要重启
```

接下来创建initramfs-tools的钩子：

```sh
#!/bin/sh

set -e

PREREQ=""

# 这个函数会在 initramfs-tools 处理时被调用
case $1 in
    prereqs)
        # 返回依赖项
        echo "$PREREQ"
        exit 0
        ;;
    *)
        # 处理脚本
        ;;
esac

# 定义证书和密钥的路径
cert="/path/to/your/certificate.crt"
key="/path/to/your/private.key"

# 扫描 /boot 目录下以 vmlinuz 开头的文件
for kernel in /boot/vmlinuz*; do
    if [ -f "$kernel" ]; then
        # 检查签名
        if ! sbverify --cert "$cert" "$kernel" &>/dev/null; then
            echo "Signing kernel: $kernel"
            # 执行签名
            sbsign --key "$key" --cert "$cert" --output "$kernel" "$kernel"
        else
            echo "Kernel already signed: $kernel"
        fi
    fi
done
```
