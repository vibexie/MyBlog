---
title: Mac制作U盘启动盘
date: 2016-08-11 23:20:43
tags: 工具
---
很多用Mac的朋友可能需要在mac上制作U盘启动盘安装系统，其实，Mac比Windows更容易制作，打开你的终端：
<!-- more -->
* diskutil list命令列出所有磁盘
``` bash
VibeXie-MBP:Blog vibexie$ diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *121.3 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:          Apple_CoreStorage Macintosh HD            120.5 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
/dev/disk1 (internal, virtual):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS Macintosh HD           +120.1 GB   disk1
                                 Logical Volume on disk0s2
                                 205B2D4F-05D4-42E2-9D3D-5655149159C0
                                 Unlocked Encrypted
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     Apple_partition_scheme                        *16.0 GB    disk2
   1:        Apple_partition_map                         8.2 KB     disk2s1
   2:                  Apple_HFS                         5.4 MB     disk2s2
   3:                  Apple_HFS ANACONDA                12.0 MB    disk2s3
```
我们发现了disk2这个16G的U盘。

* diskutil unmountDisk /dev/disk2命令解挂载U盘
``` bash
VibeXie-MBP:~ vibexie$ diskutil unmountDisk /dev/disk2
Unmount of all volumes on disk2 was successful
```

* 写入系统镜像到U盘
如我的镜像是Fedora24,路径是/Users/vibexie/Downloads/Fedora-Workstation-Live-x86_64-24-1.2.iso，如下，执行dd命令刻入U盘。
``` bash
VibeXie-MBP:~ dd if=/Users/vibexie/Downloads/Fedora-Workstation-Live-x86_64-24-1.2.iso of=/dev/disk2 bs=1m
1470+0 records in
1470+0 records out
1541406720 bytes transferred in 606.418695 secs (2541819 bytes/sec)
```
