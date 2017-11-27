---
title: linux无损调整分区大小
permalink: linux无损调整分区大小
date: 2017-05-17 22:00:42
categories:
- 运维
tags:
- 运维
- 磁盘分区
---

## summary
- 系统环境: Red Hat 4.8.5-11
- 情况：
  1. home：500G
  2. root：50G
  3. root分区不够用
- 思路：把home分区的空间划一部分到root分区

```bash
# 设置home分区大小为200G，释放300G空间
$ lvreduce -L 200G /dev/centos/home

# 将空闲空间扩展到root分区
$ lvextend -l +100%FREE /dev/centos/root

# 使用XFS文件系统自带的命令集增加分区空间
$ xfs_growfs /dev/mapper/centos-root
```
## 实例
### situation
挂载在根目录的分区 `/dev/mapper/centos-root` 爆满，占用100%
```bash
$ df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   50G   50G   19M 100% /
devtmpfs                  32G     0   32G   0% /dev
tmpfs                     32G     0   32G   0% /dev/shm
tmpfs                     32G  2.5G   29G   8% /run
tmpfs                     32G     0   32G   0% /sys/fs/cgroup
/dev/mapper/centos-home  476G   33M  476G   1% /home
/dev/sda1                497M  238M  259M  48% /boot
tmpfs                    6.3G     0  6.3G   0% /run/user/0
```
### analyze
挂载在根目录的分区空间太小，只有50G，而服务器 `home` 目录为非常用目录，挂在了近500G的空间。

思路：从 `centos-home` 分区划出300G空间到 `centos-root` 分区。

### operation
#### 1.查看各分区信息
```bash
$ lvdisplay
--- Logical volume ---
LV Path                /dev/centos/home
LV Name                home
VG Name                centos
LV UUID                1fAt1E-bQsa-1HXR-MCE2-5VZ1-xzBz-iI1SLv
LV Write Access        read/write
LV Creation host, time localhost, 2016-10-26 17:23:47 +0800
LV Status              available
# open                 0
LV Size                475.70 GiB
Current LE             121778
Segments               1
Allocation             inherit
Read ahead sectors     auto
- currently set to     256
Block device           253:2

--- Logical volume ---
LV Path                /dev/centos/root
LV Name                root
VG Name                centos
LV UUID                lD64zY-yc3Z-SZaB-dAjK-03YM-2gM8-pfj4oo
LV Write Access        read/write
LV Creation host, time localhost, 2016-10-26 17:23:48 +0800
LV Status              available
# open                 1
LV Size                50.00 GiB
Current LE             12800
Segments               1
Allocation             inherit
Read ahead sectors     auto
- currently set to     256
Block device           253:0
```
#### 2.减少/home分区空间
```bash
# 释放 /dev/centos/home 分区 300G 的空间
# 命令设置 /dev/centos/home 分区 200G空间
$ lvreduce -L 200G /dev/centos/home
WARNING: Reducing active logical volume to 200.00 GiB.
 THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce centos/home? [y/n]: y
 Size of logical volume centos/home changed from 475.70 GiB (121778 extents) to 200.00 GiB (51200 extents).
 Logical volume centos/home successfully resized.
```

#### 3.增加/root分区空间
```bash
$ lvextend -l +100%FREE /dev/centos/root
Size of logical volume centos/root changed from 50.06 GiB (12816 extents) to 325.76 GiB (83394 extents).
Logical volume centos/root successfully resized.
```

#### 4.扩展XFS文件空间大小
```bash
$ xfs_growfs /dev/mapper/centos-root
meta-data=/dev/mapper/centos-root isize=256    agcount=4, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 13107200 to 85395456
```
完成
