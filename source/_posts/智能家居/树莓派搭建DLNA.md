---
title: 树莓派搭建DLNA
date: 2019-01-20 09:42:34
tags:
- raspberrypi
categories:
- 智能家居
---
# 安装mimidlna

在命令行输入以下命令：
```
sudo apt-get update
sudo apt-get install minidlna
```

# 配置

设置/etc/minidlna.conf文件，在文件尾部添加如下内容：

```
#A表示这个目录是存放音乐的，当minidlna读到配置文件时，它会自动加载这个目录下的音乐文件
media_dir=A,/samba/DLNA/Music
media_dir=P,/samba/DLNA/Picture
media_dir=V,/samba/DLNA/Video
#配置minidlna的数库数据的存放目录
db_dir=/samba/DLNA/db
#配置日志目录
log_dir=/samba/DLNA/log
```
以上配置中，/samba/DLNA/* 这个目录可以自定义，一定要确保目录存在且设置权限为可读写。例子中的是已经配置好的 Samba 所在的目录，这样可以把 DLNA 目录共享在局域网之中，更方便的管理媒体文件。

# 随机启动

```
sudo update-rc.d minidlna defaults
```

# 常用命令

启动 minidlna 服务

```
sudo service minidlna start
```

当你修改配置文件及媒体资源更新时，需要强制刷新，以便minidlna将最新的媒体文件进行索引

```
sudo service minidlna force-reload
```

取消 minidlna 的开机自动启动

```
sudo update-rc.d -f minidlna remove
```

停止 minidlna 服务

```
sudo service minidlna stop
```

停止 minidlna 所有进程

```
sudo killall minidlna
```

卸载 minidlna

```
sudo apt-get remove --purge minidlna
```