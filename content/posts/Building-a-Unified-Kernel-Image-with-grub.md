---
title: "使用 Grub 构建统一内核映像"
date: 2022-06-25T12:40:51+08:00
tags: ['SecureBoot','Grub']
categories: ['Linux']
draft: false
---
[统一内核映像](https://systemd.io/BOOT_LOADER_SPECIFICATION/#type-2-efi-unified-kernel-images)是单个可执行文件，可以直接从 UEFI 固件启动，或者由引导加载程序自动获取，只需很少或没有配置。

## 准备 Grub 配置文件

此配置文件和普通 Grub 配置文件一样，一下只是适配我电脑的配置,默认情况下 Grub 的 root 为 (memdisk).

```
~/SecureBoot/grub.cfg
-----
menuentry 'Gentoo' --class gentoo --class gnu-linux --class gnu --class os{
    insmod all_video
    linux /vmlinuz root=/dev/sda1
    initrd /initramfs.img
}
```

## 构建 EFI 文件
使用 grub-mkstandalone 构建单一的内核映像, 将命令中的 vmlinuz-5.15.41-gentoo 以及 iniramfs-5.14.41-gentoo 替换为自己的内核及 initramfs.

```bash
grub-mkstandalone --direcotry /usr/lib/grub/x86_64-efi --format x86_64-efi --disable-shim-lock --output grub.efi /vmlinuz=/boot/vmlinuz-5.15.41-gentoo /initramfs.img=/boot/initramfs-5.14.41-gentoo /boot/grub/grub.cfg=~/SecureBoot/grub.cfg
```

## 签名 EFI 文件 (可选)
如果使用了安全启动，可以对 EFI 文件进行签名.
```bash
sbsign --key ~/keys/db.key --cert ~/keys/db.crt --output grub.efi grub.efi
```

## 安装 EFI 文件
将生成的 grub.efi 复制到 ESP 目录下. 默认情况下可以复制到 `esp/EFI/BOOT/BOOTX64.EFI`