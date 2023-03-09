---
title: BT种子放到目录下后自动下载
date: 2020-02-16 14:21:55
tags:
- raspberrypi
categories:
- 智能家居
---
Transmission支持将种子文件放到指定目录下后，就会自动创建下载任务。

配置参数

```
"watch-dir": "/home/pi/Downloads/torrents",
"watch-dir-enabled": true
```
配置参数*watch-dir*，指定存放种子文件的目录。为了避免权限不足，建议为目录设置775权限。
```
chmod 775 torrents
```
为了使配置生效，需要重新启动Transmission-daemon服务。
```
sudo service transmission-daemon restart
```
