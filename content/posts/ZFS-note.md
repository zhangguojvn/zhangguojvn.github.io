---
title: "ZFS 学习笔记"
date: 2022-12-30T17:22:29+08:00
draft: true
tags: ['ZFS']
categories: ['Linux']
draft: false
description: 非特权用户使用 Xorg
---

## 1.建立池

池相当于 raid0 池里面可以有很多的 vdev

```
# zpool create <poolname> [mirror/raidz/raidz2] <disk-id>
```

以下为建立 RAID1 阵列命令

```
# zpool create <poolname>   \
    mirror                  \
        <disk-1>            \
        <disk-2>            \
        <disk-3>            \
        ...
```
以下为建立 RAID10 阵列命令

```
# zpool create <poolname>   \
    mirror                  \
        <disk-1>            \
        <disk-2>            \
    mirror                  \
        <disk-3>            \
        <disk-4>            
```

## 2.增加 vdev 到 pool

增加 vdev 到 pool 需要和目前存在的 vdev 类型一样
即 mirror 组成的 vdev 则需要增加 mirror 到pool

```
# zpool add <poolname> [mirror/raidz/raidz2] <disk-id>
```

以下为 RAID1 升级为 RAID10 命令

```
# zpool add <poolname>  \
    mirror              \
        <disk-1>        \
        <disk-2>        \
```

以下为 RAID10 增加 vdev 的命令
```
# zpool add <poolname>  \
    mirror              \
        <disk-1>        \
        <disk-2>        \
```

## 3.增加硬盘到 vdev

对于池中的 vdev 可以增加新的硬盘到 vdev 中

```
# zpool attach <poolname> <disk-in-vdev> <new-disk>
```

## 4. 删除硬盘从 vdev

对于池中的 vdev 命令 zpool detach 可以从中删除硬盘
```
# zpool detach <poolname> <disk-in-vdev>
```

## 5. 替换硬盘

当硬盘需要替换(如损坏，定期更换)可以用以下命令对硬盘进行替换

```
# zpool replace <poolname> <disk-in-vdev> <new-disk>
```

替换硬盘是一个缓慢的过程，可以使用`zpool status`查看进度。在替换过程中可以安全的进行重启，关机等操作.
- [参考资料](https://docs.oracle.com/cd/E19253-01/819-7065/gbcus/index.html)

如果将池属性 autoreplace 设置为 on，则会自动对在先前属于该池的设备的同一物理位置处找到的任何新设备进行格式化和替换。启用此属性时，你无需使用 zpool replace 命令。此功能可能并不是在所有硬件类型上都可用。对于热备件，autoreplace 为 on 时，则在插入新设备并完成联机操作后，备件会自动分离并回到备件池。以下命令为设置方法。

```
# zpool set autoreplace=on <poolname>
```

将大小大于要替换设备的替换设备添加到池中后，它不会自动扩展到完整大小。池属性 autoexpand 的值决定当磁盘添加到池中时，是否将替换 LUN 扩展到其完整大小。缺省情况下，autoexpand 属性禁用。您可以在较大的 LUN 添加到池中之前或之后，启用此属性以扩展 LUN 大小。

```
# zpool set autoexpand=on <poolname>
```
以下为例子
```
# zpool create pool mirror c1t16d0 c1t17d0
# zpool status
  pool: pool
 state: ONLINE
 scrub: none requested
config:

        NAME         STATE     READ WRITE CKSUM
        pool         ONLINE       0     0     0
          mirror     ONLINE       0     0     0
            c1t16d0  ONLINE       0     0     0
            c1t17d0  ONLINE       0     0     0

zpool list pool
NAME   SIZE   ALLOC  FREE    CAP  HEALTH  ALTROOT
pool  16.8G  76.5K  16.7G     0%  ONLINE  -
# zpool replace pool c1t16d0 c1t1d0
# zpool replace pool c1t17d0 c1t2d0
# zpool list pool
NAME   SIZE   ALLOC  FREE    CAP  HEALTH  ALTROOT
pool  16.8G  88.5K  16.7G     0%  ONLINE  -
# zpool set autoexpand=on pool
# zpool list pool
NAME   SIZE   ALLOC  FREE    CAP  HEALTH  ALTROOT
pool  68.2G   117K  68.2G     0%  ONLINE  -
```
- [参考资料](https://docs.oracle.com/cd/E19253-01/819-7065/gayrd/index.html)

## 6. 清扫阵列

作为阵列定期进行清扫可以较早的发现问题。在此建议至少每个月清扫一次。

以下命令可以开启清扫。

```
# zpool scrub <poolname>
```

清扫是一个缓慢的过程，使用一下命令查看清扫进度。

```
# zpool scrub -v <poolname>
```

清扫会在重启后继续，使用一下命令可以安全的结束清扫。

```
# zpool scrub -s <poolname>
```

- [参考资料](https://docs.oracle.com/cd/E19253-01/819-7065/gbbxi/index.html)

## 7. 清除错误

要将错误计数器清零，请使用 zpool clear 命令，此命令只能删除错误，不能解决错误

```
# zpool clear <poolname>
```

## 8. 管理 ZFS 存储池属性

您可以使用 zpool get 命令来显示池属性信息。例如：

```
# zpool get all mpool
NAME  PROPERTY       VALUE       SOURCE
pool  size           68G         -
pool  capacity       0%          -
pool  altroot        -           default
pool  health         ONLINE      -
pool  guid           601891032394735745  default
pool  version        22          default
pool  bootfs         -           default
pool  delegation     on          default
pool  autoreplace    off         default
pool  cachefile      -           default
pool  failmode       wait        default
pool  listsnapshots  on          default
pool  autoexpand     off         default
pool  free           68.0G       -
pool  allocated      76.5K       -
```

可以使用 zpool set 命令设置存储池属性。例如：

```
# zpool set autoreplace=on mpool
# zpool get autoreplace mpool
NAME  PROPERTY     VALUE    SOURCE
mpool autoreplace  on       default
```


| 属性名 | 类型 | 缺省值 | 说明 | 
|----|----|----|----|
|allocated|字符串|N/A|只读值，表示池中物理分配的存储空间量。|
|altroot|字符串|off|标识备用根目录。如果进行了设置，则该目录会被前置到池中的任何挂载点。检查未知池时，如果不能信任挂载点，或挂载点在备用根环境中（其中典型的路径无效），则可以使用此属性。|
|autoreplace|布尔值|off|控制设备的自动替换。如果设置为 off，则必须使用 zpool replace 命令启动设备替换。如果设置为 on，则会自动对在先前属于池的设备的同一物理位置处找到的任何新设备进行格式化和替换。该属性缩写为 replace。|
|bootfs|布尔值 |N/A |标识根池的缺省可引导数据集。此属性通常由安装和升级程序进行设置。
|cachefile|字符串|N/A|控制在何处缓存池配置信息。系统引导时会自动导入高速缓存中的所有池。但是，安装和群集环境可能需要将此信息高速缓存到不同的位置，以便不会自动导入池。可设置此属性以将池配置信息高速缓存于不同位置。以后可以使用 zpool import - c 命令导入此信息。大多数 ZFS 配置不使用此属性。
|capacity|数字|N/A|用于标识已用池空间百分比的只读值。此属性的缩写为 cap。|
|delegation|布尔值 |on|控制是否可以向非特权用户授予为数据集定义的访问权限。有关更多信息，请参见第 9 章。|
|failmode|字符串 |wait|控制发生灾难性池故障时的系统行为。这种情况通常是由于失去与底层存储设备的连接或池中所有设备出现故障而导致的。这种事件的行为由下列值之一决定：<br> wait－阻止所有对池的 I/O 请求，直到设备连接恢复且使用 zpool clear 命令清除错误为止。这种状态下，对池的 I/O 操作被阻止，但读操作可能会成功。在设备问题得到解决之前，池一直处于 wait 状态。<br> continue－对任何新的写入 I/O 请求返回 EIO 错误，但允许对其余任何运行状况良好的设备执行读取操作。任何未提交到磁盘的写入请求都会被阻止。重新连接或替换设备后，必须使用 zpool clear 命令清除错误。<br> panic–向控制台打印一则消息并产生系统崩溃转储。|
|free|字符串 |N/A |只读值，表示池中未分配的块数。|
|guid|字符串 |N/A |用于标识池的唯一标识符的只读属性。|
|health|字符串 |N/A |用于标识池的当前运行状况（例如 ONLINE、DEGRADED、FAULTED、OFFLINE、REMOVED 或 UNAVAIL）的只读属性。。|
|listsnapshots|字符串 |on|控制使用 zfs list 命令是否可显示与此池有关的快照信息。如果禁用了此属性,可以通过 zfs list -t snapshot 命令显示快照信息。|
|size|数字 |N/A |用于标识存储池总大小的只读属性。|
|version|数字 |N/A |标识池的当前盘上版本。尽管在为了实现向后兼容性而需要一个特定版本时可以使用此属性，但首选的池更新方法是使用 zpool upgrade 命令。可以将此属性设置为 1 与 zpool upgrade -v 命令所报告的当前版本之间的任何数值。|