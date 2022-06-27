---
title: 将密钥封存至 TPM
date: 2022-06-25T14:50:51+08:00
tags: ['SecureBoot','Grub','TPM']
categories: ['Linux']
draft: true
---

## 创建主键


```bash
tpm2_createprimary -Q -C o -c ~/tpm/primary.context
```

## 创建当前 TPM 的签名 （可选）

pcr_bank 可选 `sha1 sha256` pcr_id 功能如下表,一般来说 "0,1,3,7" 即可
|PCR|USE|NOTES|
|--|--|--|
|PCR0|核心系统固件可执行代码（又名固件）|如果您升级 UEFI，可能会发生变化|
|PCR1|核心系统固件数据（又名 UEFI 设置）|
|PCR2|扩展或可插入的可执行代码|
|PCR3|扩展或可插拔固件数据|在 Boot Device Select UEFI 引导阶段设置|
|PCR4|引导管理器代码和引导尝试|测量引导管理器和固件尝试从中引导的设备|
|PCR5|引导管理器配置和数据|可以测量引导加载程序的配置；包括 GPT 分区表|
|PCR6|从 S4 和 S5 电源状态事件恢复|
|PCR7|安全启动状态|包含 PK/KEK/db 的全部内容，以及用于验证每个启动应用程序的特定证书|
|PCR8|内核命令行的哈希|由grub和systemd-boot支持|
|PCR9|initrd 的哈希|预定 linux v5.17|
|PCR10|保留供将来使用|
|PCR11|BitLocker 访问控制|
|PCR12|数据事件和高波动事件|
|PCR13|引导模块详细信息|
|PCR14|引导权限|
|PCR15-23|保留供将来使用|
```bash
tpm2_pcrread -Q "sha256":"0,1,2,3,7" -o "$TMP"/pcr.digest
```

## 使用当前 TPM 签名创建 TPM 政策

哈希算法可选 `sha1 sha256`,如果没有生成 TPM 签名可以忽略 -f 参数.

```bash
tpm2_createpolicy -Q -g "$hash" --policy-pcr -l "sha256":"0,1,3,7" -f ~/tpm/pcr.digest -L ~/tpm/pcr.policy
```

## 将密钥存入 TPM

```bash
tpm2_create -C primary.ctx -L ~/tpm/pcr.policy -c ~/tpm/seal.ctx -i ~/tpm/key
```

## 永久化 TPM

```bash
tpm2_evictcontrol -C o -c seal.ctx 0x81010002
```