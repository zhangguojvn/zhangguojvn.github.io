---
title: "在Dracut使用GPG解密硬盘"
date: 2022-06-25T17:20:57+08:00
tags: ['Dracut','GPG']
categories: ['Linux']
draft: false
---

## 配置加密

将配置写入 `/etc/crypttab` ,将$name $uuid $keyfile 替换为自己的配置, keyfile 为.gpg 结尾时,会启用 crypt-gpg.
```
/etc/crypttab
-----
$name UUID=$uuid $keyfile
```

## 设置公钥

将公钥放入 `/etc/dracut.conf.d/crypt-public-key.gpg`

## 设置 dracut.conf

需要手动加入 crypt-gpg 模块,将 $keyfile 替换为自己的密钥
```
/etc/dracut.conf
-----
hostonly="no"
install_items+=" /etc/crypttab $keyfile "
add_dracutmodules=" crypt-gpg "
```