---
title: ESP32写入MicroPython固件
date: 2021-10-23 12:30:00
tags: 
 - ESP32
categories: 
 - 智能家居
---

最近，刚入手了一块ESP32开发板。板子比两枚1元硬币略大，相比其它品牌的ESP32产品，价钱稍微贵了些，但是感觉做工还不错。通过这块开发板，我们可以玩比较热门的物联网，可穿戴设备，智能音箱等。

![](20211023_013244373_iOS.jpg)

<!-- more -->

该开发板可以支持**Arduino**和**MicroPython**两种开发环境，由于对Python比较熟悉，所以我首选**MicroPython**。

在烧录固件以前，需要先准备以下内容：

- 一台电脑，操作系统可以是Windows、Linux或者Mac
- 一条Type-C数据线
- 下载Python IDE软件[Thonny](https://thonny.org/)
- 下载[MicroPython](http://www.micropython.org/download/esp32/)固件

首先，需要通过Type-C数据线，将ESP32开发板和电脑相连，其中数据线Type-C口接开发板，USB口接电脑。由于我使用的是Windows 10系统，所以，需要先安装串口芯片的驱动程序，系统才能识别设备。我购买的设备使用的是CH340串口芯片，可以通过以下地址下载驱动。

驱动下载地址：[CH340芯片驱动](https://wiki.dfrobot.com.cn/_SKU_DFR0654_FireBeetle_Board_ESP32_E#target_30)

安装好CH340驱动以后，打开**计算机管理**，查看到**USB-SERIAL CH340K(COM3)** ，说明设备已经被正确识别。

![](2021-10-23113915.png)

打开软件**Thonny**，选择**Run - Select interpreter...**，打开**Interpreter**属性选项。

![](2021-10-23114015.png)

选择**Interpreter**(解释器)的类型为**MicroPython(ESP32)**，选择**Port or WebREPL**为**USB-SERIAL CH340K(COM3)** ，然后点击右下角的**Install or update firmware**。

![](2021-10-23114050.png)

选择Port为**USB-SERIAL CH340K(COM3)** ，点击**Browser...**，找到Fireware的下载路径。我使用的Firmware版本是esp32-idf4-20210202-v1.14.bin。

![](2021-10-23114253.png)

需要注意的是，一开始由于下载的是**SPIRAM**的固件版本，刷入固件后，出现了以下报错信息。更换固件后，就不再出现报错信息。

> E (621) spiram: SPI RAM enabled but initialization failed. Bailing out
> E (656) spiram: SPI RAM not initialized

![](2021-10-23114815.png)

> 提示：SPIRAM是一种通过SPI接口连接的外部存储，只有型号为ESP32-WROVER才带有SPIRAM。ESP32-WROOM不带SPIRAM。

固件烧录成功以后，就可以看到在交互式Shell中的Python提示符。接下来，就可以开心的玩耍啦。

![](2021-10-23121931.png)

