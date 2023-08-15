---
title: "自签名安全启动 Grub 设置"
date: 2022-04-20T03:50:51+08:00
tags: ['SecureBoot','Grub']
categories: ['Linux']
draft: false
description: 自签名安全启动记录
---

最近配置了自签名的安全启动。生成证书可以参考 [ArchWiki](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot)

因为启用安全启动grub会启动验证模式，会对所有从硬盘读取的文件进行签名验证，如果要从硬盘读取模块需要给所有的模块签名。为了避免麻烦可以生成 standalone 的 efi 文件，该文件包含一个 memdisk 其包含了所有的模块。并且我们没有使用垫片，可以关闭垫片验证程序( disable-shim-lock )，所以生成命令如下。

```bash
grub-mkstandalone --directroy /usr/lib/grub/x86_64-efi --format x86_64-efi --disable-shim-lock --output grub.efi /boot/grub/grub.cfg=grub-pre.cfg
```

其中 /boot/grub/grub.cfg=grub-pre.cfg 代表将 grub-pre.cfg 放入 memdisk 的 /boot/grub/grub.cfg 中。grub-pre.cfq 为grub启动后执行的第一个配置，负责基础的模块调用。内容如下。其中必须要加载 tpm 模块对内核进行验证。

```cfg
insmod part_gpt
insmod mdraid09
insmod mdraid1x
insmod ext2
insmod lvm
insmod luks
insmod tpm
cryptomount ....
set root=...
configfile /grub/grub.cfg
```

## 参考
- [Grub 帮助论坛](https://lists.gnu.org/archive/html/help-grub/2022-02/msg00001)
- [Gentoo 帮助论坛](https://forums.gentoo.org/viewtopic-p-8695263.html?sid=1bc60e6c7b4f00073bb4d39d5fee9993)
