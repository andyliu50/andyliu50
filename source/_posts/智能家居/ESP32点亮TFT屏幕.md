---
title: ESP32点亮TFT屏幕
date: 2021-10-26 19:32:00
tags: 
 - ESP32
categories: 
 - 智能家居
---

最近，购买的1.3寸TFT屏幕已经到手，但是数据线不匹配，所以又等了几天。今天，终于可以好好折腾一下这块屏幕啦。

![](20211024_091435141_iOS.jpg)

<!-- more -->

## 接线

首先，肯定是要将屏幕和ESP32开发板用杜邦线连接起来。由于我打算采用MicroPython，因此，特意在Github上找到了专门为MicroPython编译的ST7789驱动。但是，由于接线方式的不同，为此花费了一番时间。

以下是经过多次尝试后，得到的正确结果。

| ST7789 | ESP32       |
| ------ | ----------- |
| GND    | GND         |
| VCC    | 3.3V        |
| SCL    | GPIO18/SCK  |
| SDA    | GPIO23/MOSI |
| RES    | GPIO04      |
| DC     | GPIO02      |
| BLK    | GPIO15      |

**注意：BLK端口必须连接，否则屏幕点不亮。**

## 初始化

打开Thonny，通过串口连上ESP32。然后执行以下命令，完成屏幕的初始化：

```
import machine
import st7789
spi = machine.SPI(2, baudrate=40000000, polarity=1, sck=machine.Pin(18), mosi=machine.Pin(23))
display = st7789.ST7789(spi, 240, 240, reset=machine.Pin(4, machine.Pin.OUT), dc=machine.Pin(2, machine.Pin.OUT))
display.init()
```



## 最终效果

执行以下命令，将整屏显示为黄色。

```
display.fill(YELLOW)
```

![](20211026_072012179_iOS.jpg)



