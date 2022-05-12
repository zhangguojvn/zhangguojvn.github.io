---
title: "Yubikey 配置 PIV 签名"
date: 2022-05-06T19:28:44+08:00
draft: true
tags: ['Yubikey','PIK']
categories: ['SecurityToken']
---

## 设置模式

计划只使用 PIV 模式，可以在 yubikey manager 中关闭其他模式。

```bash
ykman config mode CCID
```

## 启动 pcscd

必须启动 pcscd 否则 yubico-piv-tool 无法发现设备

```bash
#systemd
systemctl start pcscd
#openRC
/etc/init.d/pcscd
#other
/usr/sbin/pcscd &
```

## 生成ECDSA密钥

```bash
openssl ecparam -out ca.key -name secp384r1 -genkey
openssl ec -in ec-secp256k1-priv-key.pem -pubout > secp256k1-pub.pem
```

## 生成 RSA 密钥

```bash
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```

## 导入 yubikey 

在以下例子中，将私钥导入 9a 插槽中。关于插槽介绍可以看 [官方](https://developers.yubico.com/yubico-piv-tool/YKCS11/Functions_and_values.html#_key_alias_per_slot_and_object_type) 的这篇文章。

```bash
yubico-piv-tool -s 9a -a import-key -i private.pem
```

依照 PIV 的標準，Yubikey 只能存入 X.509 憑證，而不能單純儲存公鑰。因此這邊得用我們剛才產生的私鑰，製作一份自我簽核 (self-signed) 的憑證：

```bash
$ yubico-piv-tool -a verify-pin -a selfsign-certificate -s 9a -S "/CN=SSH KEY/" -i public.pem -o cert.pem
```

## 设置 SSH key

```bash
pkcs15-tool --list-public-key #列出所有的公钥
pkcs15-tool --read-ssh-key 01 #输出为ssh格式 
ssh -I /usr/lib/x86_64-linux-gnu/opensc-pkcs11.so syyang@foo.bar #使用PIV进行SSH认证
```

## 参考资料
- [Harden your Linux UEFI Secure Boot using GRUB signature checking and a Yubikey](https://casualhacking.io/blog/2020/5/24/harden-your-linux-uefi-secure-boot-using-grub-signature-checking-and-a-yubikey)
- [使用 Yubikey 進行公鑰認證](https://electronic.blue/blog/2016/09/18-public-key-authentication-with-yubikey/)
- [Supported PKCS#11 Functions](https://developers.yubico.com/yubico-piv-tool/YKCS11/Functions_and_values.html#_key_alias_per_slot_and_object_type)