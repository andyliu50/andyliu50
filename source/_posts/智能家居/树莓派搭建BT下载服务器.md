---
title: 树莓派搭建BT下载服务器
date: 2019-01-20 10:12:23
tags:
- raspberrypi
categories:
- 智能家居
---
# 安装

```
sudo apt-get install transmission-daemon
```

# 配置

修改配置文件：/etc/transmission-daemon/settings.json 

```
"download-dir": "/mnt/HDD160GB/Video",  #配置下载目录，下载后的文件将存放在该目录下
"rpc-username": "transmission",        #登录BT Web界面的用户名
"rpc-password": "transmission",        #设置密码，保存配置文件后，密码自动变成密文

```

# 重启服务

```
sudo service transmission-daemon reload
sudo service transmission-daemon restart
```

# 访问方式

通过浏览器访问地址：http://192.168.1.3:9091/ ，用户名和密码：transmission/transmission

