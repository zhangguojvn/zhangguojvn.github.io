---
title: "在 Windows 下安装 go-oci8"
date: 2022-06-24T17:20:29+08:00
draft: true
tags: ['Oracle','Golang','go-oci8']
categories: ['Golang']
draft: false
description: 介绍如何在 Windows 下安装 Go 的 Oracle 数据库驱动。
---

## 安装 TDM-GCC-64

从 TDM-GCC 的[项目地址](https://sourceforge.net/projects/tdm-gcc/)下载最新版的 TDM-GCC 64位版本安装。

## 设置 TDM-GCC-64 的环境变量

在 PATH 中加入 `(TDM-GCC 安装目录)/bin` 例如 `C:\TDM-GCC-64\bin` .
![TDM-GCC 环境变量](/images/posts/install-go-oci-in-windows/00.png)

## 下载 instantclient-basic 及 instantclient-sdk

在 Oracle [官网](https://www.oracle.com/cn/database/technologies/instant-client/downloads.html) 下载 basic 及 SDK.并将两者解压,例如 `C:\installclient`.

## 设置 instantclient-basic 的环境变量

在 PATH 中加入 `instantclient-basic目录` 例如 `C:\installclient` .
![installclient 环境变量](/images/posts/install-go-oci-in-windows/01.png)

## 安装 pkg-config-lite

从 pkg-config-lite [官网](https://sourceforge.net/projects/pkgconfiglite/) 下载最新版将bin和share内文件解压到 TDM-GCC 对应目录下.

## 设置 pkg-config-lite

创建新文件夹用于储存 *.pc 文件,本例子中位C:\pkg_config_path. 并将环境变量中加入 PKG_CONFIG_PATH 并指向刚刚创建的文件.
![pkg-config 环境变量](/images/posts/install-go-oci-in-windows/02.png)

## 创建 oci8.pc

在刚刚创建的文件夹内创建 oci.pc 文件,并将 `C:/instantclient/sdk` 替换为 instantclient-sdk 的安装目录
```
C:\pkg_config_path\oci.pc
----------
Name: oci8
Description: Oracle Instant Client
Version: 12.2
Cflags: -IC:/instantclient/sdk/include
Libs: -LC:/instantclient/sdk/lib/msvc -loci
```

## 安装 go-oci8

```
go get github.com/mattn/go-oci8
```