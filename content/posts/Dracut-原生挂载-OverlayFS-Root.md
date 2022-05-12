---
title: "Dracut 原生挂载 OverlayFS Root"
date: 2022-05-12T23:40:37+08:00
draft: false
tags: ['Dracut','OverlayFS']
categories: ['Linux']
---

## Dracut 配置

需要手动导入模块，不可以使用hostonly模式，即配置中`hostonly="no"`。

```bash
/etc/dracut.conf
----------
hostonly="no"
add_drivers+=" squashfs overlay "
add_dracutmodules+=" dmsquash-live "
```
因为关闭了 hostonly 模式，所以 lvm 等配置需要手动写入内核参数。

## 准备镜像

镜像可以放入可挂载文件系统，或直接烧录进分区。在此介绍放入挂载文件系统情况。

镜像内部至少需要 proc,sys,dev。其中 [90dmsquash-live](https://github.com/dracutdevs/dracut/blob/86bf2533d77762e823ad7a3e06a574522c1a90e3/modules.d/90dmsquash-live/dmsquash-live-root.sh#L301) 使用 /proc 检测是否为正确的系统格式。98dracut-systemd 使用 usable_root() 函数，[该函数](https://github.com/dracutdevs/dracut/blob/86bf2533d77762e823ad7a3e06a574522c1a90e3/modules.d/99base/dracut-lib.sh#L761)使用 /proc,/sys,/dev 检测是否和正确的系统格式。建议也创建 /run 文件夹。

镜像放在内核参数`<rd.live.dir>/<rd.live.squashimg>`。这两个参数默认值为 [LiveOS](https://github.com/dracutdevs/dracut/blob/86bf2533d77762e823ad7a3e06a574522c1a90e3/modules.d/90dmsquash-live/dmsquash-live-root.sh#L21) 和 [squashfs.img](https://github.com/dracutdevs/dracut/blob/86bf2533d77762e823ad7a3e06a574522c1a90e3/modules.d/90dmsquash-live/dmsquash-live-root.sh#L23) 即默认放入 LiveOS/squashfs.img 即可。

## OverlayFS 持久化分区准备

该分区需要在根目录下创建内核参数 `<rd.live.overlay>` 目录作为 upperdir。并且在上级目录创建 [ovlwork](https://github.com/dracutdevs/dracut/blob/86bf2533d77762e823ad7a3e06a574522c1a90e3/modules.d/90dmsquash-live/dmsquash-live-root.sh#L166) 文件夹作为 workdir。`<rd.live.overlay>`[默认](https://github.com/dracutdevs/dracut/blob/86bf2533d77762e823ad7a3e06a574522c1a90e3/modules.d/90dmsquash-live/dmsquash-live-root.sh#L125)为`/<rd.live.dir>/overlay-<LABEL>-<UUID>",其中LABEL和UUID为ROOT分区的LABEL和UUID。持久化不仅可以用文件夹，也可以使用文件，在此不介绍。

## 内核参数

- `root=live:分区`: 存放镜像的分区。
- `rd.live.overlay=分区:目录`: OverlayFS 的 upperdir，目录必须由/开头。
- `rootfstype=ext4`: root 分区的文件类型。