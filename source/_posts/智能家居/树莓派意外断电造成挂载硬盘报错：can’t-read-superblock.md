---
title: 树莓派意外断电造成挂载硬盘报错：can’t read superblock
date: 2019-05-19 08:57:37
tags:
- raspberrypi
categories:
- 智能家居
---
最近，因为意外断电的缘故，当再次开机重新挂载硬盘的时候出现报错无法挂载，<u>**mount: /dev/sda: can't read superblock**</u>

## 一、tune2fs简介

tune2fs是调整和查看ext2/ext3文件系统的文件系统参数，Windows下面如果出现意外断电死机情况，下次开机一般都会出现系统自检。Linux系统下面也有文件系统自检，而且是可以通过tune2fs命令，自行定义自检周期及方式。

**找到Blocks per group后面的数字，我这是32768**

```
tune2fs -l /dev/sda1
```

> pi@raspberrypi:~ $ sudo tune2fs -l /dev/sda1
tune2fs 1.43.4 (31-Jan-2017)
Filesystem volume name:   HDD160GB
Last mounted on:          /mnt/HDD160GB
Filesystem UUID:          4de0e171-8799-4d96-b4b5-dc8cc1b5302e
Filesystem magic number:  0xEF53
Filesystem revision #:    1 (dynamic)
Filesystem features:      has_journal ext_attr resize_inode dir_index filetype needs_recovery extent 64bit flex_bg sparse_super large_file huge_file dir_nlink extra_isize metadata_csum
Filesystem flags:         unsigned_directory_hash 
Default mount options:    user_xattr acl
Filesystem state:         clean
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              9773056
Block count:              39072470
Reserved block count:     1953623
Free blocks:              10899774
Free inodes:              9772811
First block:              0
Block size:               4096
Fragment size:            4096
Group descriptor size:    64
Reserved GDT blocks:      1024
<u>**Blocks per group:         32768**</u>
Fragments per group:      32768
Inodes per group:         8192
Inode blocks per group:   512
Flex block group size:    16
Filesystem created:       Sat Jan  5 12:03:02 2019
Last mount time:          Sat May 18 12:52:32 2019
Last write time:          Sat May 18 12:52:32 2019
Mount count:              1
Maximum mount count:      -1
Last checked:             Sat May 18 11:40:19 2019
Check interval:           0 (<none>)
Lifetime writes:          1032 MB
Reserved blocks uid:      0 (user root)
Reserved blocks gid:      0 (group root)
First inode:              11
Inode size:	          256
Required extra isize:     32
Desired extra isize:      32
Journal inode:            8
Default directory hash:   half_md4
Directory Hash Seed:      80d6dc25-a820-4596-952a-3f8cb2ad24fe
Journal backup:           inode blocks
Checksum type:            crc32c
Checksum:                 0x169cbdf9

## 二、fsck命令

在出现系统故障之后，总是在文件系统上运行 fsck 命令。矫正的动作也许会导致某些数据的丢失。对于每一个一致性的矫正，缺省的操作就是等待操作员输入 yes 或 no。如果对于已经受到影响的文件系统您没有写的许可，那么无论您的实际响应是什么，fsck 命令缺省的动作都是 no。
注：
对于一个已经安装好了的文件系统，fsck 命令不会做出矫正。
fsck 命令出于某些原因可以在一个已经安装好了的文件系统中运行，但不是进行修复。但是当文件系统安装完毕之后，也许会返回不准确的错误消息。
fsck 命令检查并以交互方式修复不连贯的文件系统。在安装文件系统之前，应该运行此命令。您必须能够读设备文件，在此设备上驻留着文件系统（例如 /dev/hd0 设备）。通常，文件系统是连贯的，fsck 命令仅仅是报告文件系统中文件的数量、被使用的块和空闲的块。如果文件系统是不连贯的，fsck 命令显示关于那些找到的不连贯性的信息并且提示您修复它们的许可。

fsck 命令在修复中是有保留的并且会尽力避免那些可能导致有效数据丢失的动作。在特定的情况下，fsck 命令会建议破坏已经损坏的文件。如果您不允许 fsck 命令进行必要的修复，那么或许会产生一个不连贯的文件系统。安装一个不连贯的文件系统也许会导致系统的崩溃。

如果 JFS2 文件系统有快照，fsck 命令将试图保留这些快照。如果此操作失败，那么无法保证快照包含来自捕捉到的文件系统的全部先前就存在的映像。fsck 命令将删除快照和快照逻辑卷。如果 fsck 命令对文件系统进行了修改，那么内部快照将被删除。

```
fsck.ext4 -b 32768 /dev/sda1
```