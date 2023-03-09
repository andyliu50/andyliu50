---
title: 用树莓派打造家用NAS
date: 2018-11-21 13:31:02
tags:
- raspberrypi
categories:
- 智能家居
---
# 安装Raspbian系统


1. 下载系统文件，下载地址：http://www.raspberrypi.org/downloads

2. 解压系统zip文件，例如：unzip ~/Downloads/2014-09-09-wheezy-raspbian.zip

3. 电脑插入SD卡后，在终端命令后，输入：df -h，查看SD卡的分区，例如，/dev/disk3s1

4. 使用以下命令卸载分区
```bash
sudo diskutil unmount /dev/disk3s1
```
5. Using the device name of the partition work out the raw device name for the entire disk, by omitting the final "s1" and replacing "disk" with "rdisk" (this is very important: youwill lose all data on the hard drive on your computer if you get the wrong device name). Make sure the device name is the name of the whole SD card as described above, not just a partition of it (for example, rdisk3, not rdisk3sSimilarly you might have another SD drive name/number like rdisk2 or rdisk4, etc. -- recheck by using the df -hcommand both before & after you insert your SD card reader into your Mac if you have any doubts!): For example, /dev/disk3s1 => /dev/rdisk3

6. 在终端命令行中，输入命令：  
```bash
sudo dd bs=1m if=~/Downloads/2014-09-09-wheezy-raspbian/2014-09-09-wheezy-raspbian.img of=/dev/rdisk3
```
7. 卸载SD卡

# SSH免密码登录

主机A(SSH Client), 主机B(SSH Server)

1. 在主机A上生成公钥（id_rsa.pub）和私钥(id_rsa)对。 公钥和私钥文件的位置：~/.ssh/
```bash
bogon:.ssh andy$ ssh-keygen -t rsa -P ''
		
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/andyliu/.ssh/id_rsa):
Your identification has been saved in /Users/andyliu/.ssh/id_rsa.
Your public key has been saved in /Users/andyliu/.ssh/id_rsa.pub.
The key fingerprint is:
3f:46:7c:d2:45:65:d6:69:7f:34:b2:31:38:1d:c1:94 andy@bogon
The key's randomart image is:
+--[ RSA 2048]----+
|            =B==B|
|           o E*o=|
|            ... o|
|         . . .  .|
|        S + o    |
|         o o     |
|          +      |
|         . .     |
|                 |
+————————+
```
2. 将公钥文件id_rsa.pub拷贝到主机B。
```bash
bogon:.ssh andy$ scp id_rsa.pub pi@192.168.1.68:/home/pi/
```
3. 将id_rsa.pub的内容拷贝到.ssh/authorized_keys，如果没有该文件，可以手动创建。
```bash
cat id_rsa.pub >> .ssh/authorized_keys 
```
# 用树莓派和aria2打造下载机

## Web服务器

1. 安装服务器Nginx
```bash
sudo apt-get update
sudo apt-get install nginx
```
2. 启动nginx服务
```bash
sudo /etc/init.d/nginx start
```
## Aria2

1. 安装Aria2
```bash
sudo apt-get update
sudo apt-get install aria2 git
```
2. 配置文件
```bash
mkdir .aria2
touch .aria2/aria2.conf
```
aria2.conf配置文件内容如下：
```
		continue=true
		daemon=true
		dir=/home/pi/Downloads/		//下载文件的存储目录
		enable-rpc=true
		file-allocation=falloc
		force-sequential=true
		input-file=/home/pi/.aria2/aria2.session
		log=/home/pi/.aria2/aria2.log
		log-level=notice
		max-concurrent-downloads=2
		max-connection-per-server=5
		parameterized-uri=true
		rpc-allow-origin-all=true
		rpc-listen-all=true
		rpc-save-upload-metadata=false
		save-session=/home/pi/.aria2/aria2.session
		save-session-interval=60
		split=5
		disable-ipv6=true
		max-tries=0     //解决百度网盘下载经常中断问题
```
3. 创建文件
```bash
touch /home/pi/.aria2/aria2.session
touch /home/pi/.aria2/aria2.log
```
4. 启动aria2 服务
```bash
aria2c --conf-path=/home/pi/.aria2/aria2.conf
```

5. 配置aria2随机启动  
安装killall工具:
```bash
		sudo apt-get install psmisc
		sudo vi /etc/init.d/aria2
		sudo chmod 755 /etc/init.d/aria2
```
让aria2服务随机自动启动
```bash
		sudo update-rc.d aria2 defaults
```
脚本内容如下：
```
		#!/bin/sh
		### BEGIN INIT INFO
		# Provides: aria2
		# Required-Start: $remote_fs $network
		# Required-Stop: $remote_fs $network
		# Default-Start: 2 3 4 5
		# Default-Stop: 0 1 6
		# Short-Description: Aria2 Downloader
		### END INIT INFO
		USER=pi
		CONF=/home/pi/.aria2/aria2.conf
		case "$1" in
		start)
		echo "Start aria2c"
		umask 0002
		su - $USER -c "aria2c --conf-path=$CONF -D"
		;;
		stop)
		echo "Stopping aria2c, please wait..."
		killall -w aria2c
		;;
		restart)
		echo "Stopping aria2c, please wait..."
		killall -w aria2c
		echo "Start aria2c"
		umask 0002
		su - $USER -c "aria2c --conf-path=$CONF -D"
		;;
		*)
		echo "$0 {start|stop|restart|status}"
		;;
		esac
		exit
```
6. 安装Web界面
```bash
cd /var/www/html/
sudo git clone https://github.com/ziahamza/webui-aria2.git
```
注意：由于系统时间错误，可能导致下载时出现证书验证错误  
访问地址: http://192.168.1.68/webui-aria2/

![](17.png)
![](18.png)

## 浏览器插件
下载并安装插件 

https://chrome.google.com/webstore/detail/baiduexporter/mjaenbjdjmgolhoafkohbhhbaiedbkno

# 配置NTP
```bash
sudo vi /etc/ntp.conf
server time.asia.apple.com
```

# 扩展root分区容量

使用命令：sudo raspi-config，选择“Expand root partition to fill SD card”选项。

![](19.png)

# 配置Samba文件共享

1. 安装Samba服务
```bash
sudo apt-get update
sudo apt-get install samba samba-common-bin
```

2. 配置Samba服务随机启动
```bash
sudo update-rc.d aria2 defaults
```

3. 配置共享目录（配置文件：/etc/samba/smb.conf）  
在配置文件的末端，添加以下内容：
```
		[Downloads]
   	    comment = Downloads
   	    valid users = pi,root
   	    path = /home/pi/Downloads
   	    browseable = yes
   	    writable = yes
   	    create mask = 0664
   	    directory mask = 0775
```
4. 创建Samba用户，必须是Linux已经存在的用户
```bash
sudo smbpasswd -a pi
```

5. 重启Samba服务
```bash
sudo /etc/init.d/samba restart
```