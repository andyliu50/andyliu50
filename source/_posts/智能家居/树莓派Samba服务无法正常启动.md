---
title: 树莓派Samba服务无法正常启动
date: 2019-06-02 13:03:24
tags:
- raspberrypi
categories:
- 智能家居
---
## 问题描述

最近，无法通过文件共享的方式，访问树莓派上的文件。在树莓派上通过以下命令查看Samba服务，发现服务没有正常启动。

```
pi@raspberrypi:~ $ sudo service samba status
● samba.service
   Loaded: masked (/dev/null; bad)
   Active: inactive (dead)
```
通过以下命令重启Samba服务，无法正常启动。

```
pi@raspberrypi:~ $ sudo service samba restart
Failed to restart samba.service: Unit samba.service is masked.
```

## 解决方法

1. 通过以下命令重启服务。

```
sudo systemctl restart smbd.service
```

2. 或者使用以下命令。

```
sudo /etc/init.d/samba restart
```

通过以上命令重启Samba服务以后，如果使用sudo service samba status查看服务状态，会显示服务状态仍然是inactive。 但是，此时Samba服务已经可以正常访问。