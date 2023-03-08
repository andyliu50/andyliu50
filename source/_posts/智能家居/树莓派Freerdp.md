---
title: 树莓派｜FreeRDP
date: 2021-11-21 16:57
tags: 
 - raspberrypi
categories: 
 - 智能家居
---
![Raspberry Pi4](20211031_044118441_iOS.jpg)由于台式机的风扇噪音比较大，因此一直想购买一款低功耗的电脑，放在卧室使用。之前在网上关注到有人将树莓派作为瘦终端来使用，于是购买了一台内存为8G的树莓派4来试用一下。

一直以来，由于台式机的风扇噪音比较大，所以我比较青睐低功耗的产品。但是，由于性能的问题，一直无法满足实际的使用需求。随着科技的不断发展，树莓派等低功耗产品的性能得到了显著提升。之前在网上关注到有人将树莓派作为瘦终端来使用，于是购买了一台内存为8G的树莓派4来试用一下。<!-- more -->

![Raspberry Pi4](20211031_042816577_iOS.jpg)

根据网上的资料，适合于树莓派的瘦终端系统有以下几种，并且都是付费产品：

- **WTWare**
- **Stratodesk’s NoTouch OS**
- **TLXOS**

最后，决定采用开源免费的[FreeRDP](https://www.freerdp.com/)软件，该软件可以通过RDP协议远程访问Windows系统的桌面。

使用了一段时间以后，得到以下结论：

- 响应速度比较快，基本感觉不到延时。
- 播放音频的声音效果还可以，但是使用蓝牙音箱的时候，声音会有卡顿现象。
- 目前只能支持1080p的分辨率，不支持2k, 4k等分辨率。
- 鼠标在操作滚动条的时候，感到有延时。
- 播放高清视频会出现卡顿，还没有详细研究相关配置参数，不清楚是否可以解决。
- 每次都要输入命令以后，才能完成远程连接的操作，比较麻烦。
- 总体来说，简单做一些办公应该没有什么问题。

在树莓派4上，必须要选择Legacy的显示驱动，否则远程桌面的画面会有延时。这个问题和V3D显示驱动有关系，问题的详细描述，可以参考[这里](https://github.com/raspberrypi/linux/issues/4353)。

```
sudo raspi-config

- select "7 Advanced Options"
- select "A7 GL Driver"
- select "G1 Legacy"
- then reboot.
```

在树莓派上，如果要使用双屏显示远程桌面，需要做以下配置：

```
To enable dual screen, I have created this file /usr/share/X11/xorg.conf.d/60-dualscreen.conf :

#
# Configuration double ecran pour Raspberry Pi 4 et les drivers "Legacy"
# The order of precedence is Display, Screen, Monitor, Device.
# 
Section "Device"
        Identifier      "Rpi4 HDMI1"
        Driver          "fbturbo"
        Option          "fbdev" "/dev/fb0"
        Option          "ShadowFB" "off"
        Option          "SwapbuffersWait" "true"
EndSection

Section "Device"
        Identifier      "Rpi4 HDMI2"
        Driver          "fbturbo"
        Option          "fbdev" "/dev/fb1"
        Option          "ShadowFB" "off"
        Option          "SwapbuffersWait" "true"
EndSection

Section "Monitor"
        Identifier      "HDMI1"
        Option          "Primary" "true"
EndSection

Section "Monitor"
        Identifier      "HDMI2"
        Option          "LeftOf" "HDMI1"
EndSection

Section "Screen"
        Identifier      "screen0"
        Device          "Rpi4 HDMI1"
        Monitor         "HDMI1"
EndSection

Section "Screen"
        Identifier      "screen1"
        Device          "Rpi4 HDMI2"
        Monitor         "HDMI2"
EndSection

Section "ServerLayout"
        Identifier      "default"
        Option          "Xinerama" "on"
        Option          "Clone" "off"
        Screen 0        "screen0"
        Screen 1        "screen1" RightOf "screen0"
EndSection
```

最后，在树莓派上安装并使用freerdp。

```
sudo apt-get install freerdp2-x11
xfreerdp /version  // 查看版本
xfreerdp /help 	   //查看帮助
xfreerdp /f /u:"" /p:"" /sound:sys:alsa,format:1,quality:high /rfx /gfx /gfx-h264 /multitransport /network:auto -bitmap-cache -glyph-cache /gdi:hw -fonts /v:SERVERNAME

/u: 用户名
/p: 密码
/v: 远程主机名或IP地址
```