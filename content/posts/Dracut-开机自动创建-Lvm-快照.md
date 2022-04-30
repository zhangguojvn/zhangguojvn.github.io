---
title: "Dracut 开机自动创建 Lvm 快照"
date: 2022-04-30T17:09:57+08:00
tags: ['Dracut','Lvm']
categories: ['Linux']
draft: false
---

## Dracut-056之前版本
内核参数加入以下参数即可,其中 ORIG_LV 为快照源, SNAP_LV 为快照名.每次启动都会删除 SNAP_LV 并创建新的 SNAP_LV .
```bash
rd.lvm.snapshot="ORIG_LV:SNAP_LV"
```

## Dracut-056及之后版本

在Dracut-056中删除了快照功能，需要自己创建快照脚本。

