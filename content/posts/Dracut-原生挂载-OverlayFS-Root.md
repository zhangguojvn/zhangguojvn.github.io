---
title: "Dracut 原生挂载 OverlayFS Root"
date: 2022-05-12T23:40:37+08:00
draft: true
tags: ['Dracut','OverlayFS']
categories: ['Linux']
---

## ArchLinux

目前发现在特定的 Dracut 配置下可以成功挂载 OverlayFS 的跟系统。原因不明。因为我主要使用 Gentoo 系统，希望有缘人继续深入研究。

```bash
/etc/dracut.conf
----------
hostonly="yes"
use_fstab="yes"
add_drivers+=" squashfs overlay "
install_items+=" /sbin/lvm /etc/fstab.sys "
add_fstab="/etc/fstab.sys"
```
该为挂载配置，/dev/VolGroup/ovkerlay 为 overlayFS 的 upperdir, /dev/VolGroup/rootfs 为 OverlayFS 的 lowerdir, 并且要重复讲 root 挂载至 /sysroot。
```
/etc/fstab.sys
----------
/dev/VolGroup/ovkerlay /run/overlay ext4 defaults 0 0
/dev/VolGroup/rootfs /run/rootfs squashfs defaults 0 0
overlay / overlay lowerdir=/run/rootfs,upperdir=/run/overlay/upperdir,workdir=/run/overlay/workdir,x-systemd.requires-mounts-for=/run/overlay,x-systemd.requires-mounts-for=/run/rootfs 0 1
overlay /sysroot overlay lowerdir=/run/rootfs,upperdir=/run/overlay/upperdir,workdir=/run/overlay/workdir,x-systemd.requires-mounts-for=/run/overlay,x-systemd.requires-mounts-for=/run/rootfs 0 1
```