---
title: BT下载完成后发送通知
date: 2020-02-16 14:21:55
tags:
- raspberrypi
categories:
- 智能家居
---

# 需求

在树莓派上，每次通过Transmission下载文件时，都要通过浏览器查看文件是否已下载完成。最好能在文件下载完成后，通过手机自动发送下载完成通知。

# 解决方案

Transmission支持当文件下载完成后，可以运行指定路径的脚本。结合IFTTT，可以将下载完成的文件名发送到手机端。


# 脚本

创建脚本文件*btfinish.sh*：
```
touch /home/pi/Documents/btfinish.sh
chmod 775 /home/pi/Documents/btfinish.sh
```
脚本内容如下：
```
#!/bin/bash

/home/pi/Documents/bt_ifttt.py "$TR_TORRENT_NAME"
```

创建Python脚本文件*bt_ifttt.py*，该脚本将会向IFTTT服务发送下载完成的消息。
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

'This is a module used to communicate with IFTTT platform.'

__author__ = 'Andy'

import requests
import subprocess
import sys

class Notification(object):
    
	mykey = ''
	def __init__(self, event, **values):
		self.__url = "https://maker.ifttt.com/trigger/%s/with/key/%s" % (event, self.mykey)
		self.__values = values
	
	def sent(self):
		requests.post(self.__url, data=self.__values)

	@property
	def url(self):
		return self.__url

	@property
	def values(self):
		return self.__values

if __name__ == '__main__':
        bt_file_name = sys.argv[1]  # 从脚本外部获取参数
        n = Notification('bt_download', value1 = bt_file_name) # IFTTT的事件名称和参数值1
        n.sent()
```


# IFTTT

![](415.png)
![](416.png)
![](417.png)
![](418.png)
![](419.png)

# 配置文件

修改Tranmission-daemon配置文件settings.json。

```
sudo vi /etc/transmission-daemon/settings.json
```

修改以下参数：
```
"script-torrent-done-enabled": true, # 下载完成后，调用并运行脚本
"script-torrent-done-filename": "/home/pi/Documents/btfinish.sh", # 指定脚本的位置
```


# 参考资料

https://github.com/transmission/transmission/wiki/Scripts

