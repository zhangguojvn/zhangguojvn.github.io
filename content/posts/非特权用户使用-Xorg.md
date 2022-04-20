---
title: "非特权用户使用 Xorg"
date: 2022-04-20T19:29:29+08:00
draft: true
tags: ['Non-Root','Xorg']
categories: ['Linux']
draft: false
description: 非特权用户使用 Xorg
---

## 问题
在 Gentoo 下 startx 启动 i3wm 时，报告
```bash
parse_vt_settings cannot open /dev/tty0 permission
```
## 解决方法
```bash
rc-update add elogind boot
/etc/init.d/elogind start
```

##参考

[Non root Xorg](https://wiki.gentoo.org/wiki/Non_root_Xorg)