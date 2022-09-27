---
title: "解决 UOS(Deepin) 无法进入桌面的问题"
date: 2022-09-27T07:39:32Z
tags: ['UOS','Deepin','信创']
draft: true
---

## 1. 问题描述

密码后无法进入桌面，回退到登陆页面。

## 2. 日志记录

查看 lightdm 位于 `/var/log/lightdm/lightdm.log` 的日志，发现以下异常。

```
[+4.55s] DEBUG: Session: Failed during authentication
[+4.55s] DEBUG: Seat seat0: Session stopped
```

## 3. 发现问题

发现用户目录下和验证有关的文件 `.Xauthority` 为 root 所有，删除即可。