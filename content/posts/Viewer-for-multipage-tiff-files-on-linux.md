---
title: "Linux 上的 tiff 文档查看"
date: 2022-10-13T02:41:06Z
draft: false
---

## 方案1: 转换成 PDF
使用 libtiff 的 tiff2pdf 将 tiff 文档转换为 pdf 进行查看.
### 安装
- Debian 系( Ubuntu/Debian/Deepin/UOS )安装 `libtiff-tools`
```
sudo apt install -y libtiff-tools
```
- Arch 系安装 `libtiff`
```
sudo pacman -S libtiff
```
### 运行
将 source.tif 替换为源文件名 output.pdf 替换为输出文件名
```
tiff2pdf  source.tif -o output.pdf
```

## 方案2: 直接读取
使用 evince 直接读取 tiff 文档.

### 安装
- Debian 系( Ubuntu/Debian/Deepin/UOS )安装 `evince`
```
sudo apt install -y evince
```
- Arch 系安装 `evince`
```
sudo pacman -S evince
```
### 运行
将 source.tif 替换为源文件名
```
evince  source.tif 
```