---
title: Home_Assistant的定位功能
date: 2021-10-25 14:42:00
tags: 
 - home_assistant
categories: 
 - 智能家居
---

众所周知，苹果手机的**查找**功能，可以让我们方便的查找家人以及朋友的位置信息，但前提是大家使用的都是iPhone。如果使用Android手机，是否可以实现相同的功能呢？并且，如果家人既有iPhone，又有Android，如何实现在同一个界面中查找位置呢？其实，通过Home Assistant就可以实现这些功能。

> 关于如何安装Home Assistant的介绍，请参考文章：Home Assistant: 安装篇

以下图片中展示的是通过Home Assistant实现的效果，既可以在地图上**查找家人的位置**，也可以根据状态显示家人是**在家**还是**离开**。

![](20211023_234305000_iOS.png)

下面，介绍一下如何在Home Assistant中实现位置的定位和追踪。

<!-- more -->

## iOS

对于iOS手机，只需要安装Home Assistant客户端软件，并且Home Assistant的定位服务权限设置为始终允许。iPhone手机就会自动上报位置给HA网关。

![](20211025_071159000_iOS.png)

iPhone手机注册到HA网关后，就会出现实体ID: device_tracker.iPhone，可以通过该实体ID来配置位置信息。

![](2021-10-25154947.png)

## Android

对于Android手机，**由于Home Assistant客户端的定位服务依赖于Google Play Service，因此，在国内无法使用Home Assistant的定位功能**。但是，我们可以使用另外一个开源软件**OwnTracks**，该软件可以与Home Assistant集成。

首先，OwnTracks for Android有两个版本：**gms**和**oss**，gms依赖于Google Play Service，并且使用的是谷歌地图，而oss不需要依赖于Google Play Service，并且使用的地图是 OpenStreetMap。所以，**我们应该下载并安装oss版本的OwnTracks**。

![](2021-10-25151814.png)

安装成功后，需要将连接模式更改为HTTP。

![](20211025173103.jpg)

Host设置为HA网关的Web API地址，具体内容请参考以下**HA网关配置**章节的**配置OwnTracks集成**。

![](20211025173127.jpg)

另外，建议可以将上传数据的间隔设置为300s，默认为900s。

![](20211025173136.jpg)

> 提示：OwnTracks分为几种运行模式：
>
> Move mode: 只能当OwnTracks在前台运行的时候，才能启用Move模式。这种模式下，当设备出现位置变化，就会发布位置消息。好处是数据比较精确，但是比较耗电。
>
> Significant location change mode: 该模式可以在后台运行，并且只有当距离变化大于500米时(iOS，Android略微不同)，才会按照一定的时间间隔，例如5分钟，发布位置消息。这种模式比较省电。
>
> 对于Android，建议将locatorDisplacement设置为200，该参数的作用是当距离大于200米，OwnTracks才会发布位置消息。
>
> Manual mode: 在OwnTracks应用中手动发布位置消息。
>
> Quiet mode: 与Manual mode相同，但是不发布区域事件。
>
> 一般建议使用Significant location change mode。

## HA网关配置

### 配置OwnTracks集成

首先，在HA网关中，需要先配置与OwnTracks的集成，这样Android手机才会自动将位置信息上传到HA网关。

在**配置 - 集成 - 添加集成**中，查找并添加OwnTracks。添加成功后，会出现提示框，显示HA网关的Web API地址，该地址必须记录下来，配置OwnTracks客户端时，需要使用。

Web API地址的格式如下：

```
https://ha.example.com:8123/api/webhook/1323dcce3365adfaf3dafsaf511afdaf8e3259dfafafafasf121324454
```

如果忘了记录地址，可以到Home Assistant的配置文件所在目录下，打开文件.storage/core.config_entries，查找关键字OwnTracks，找到webhook_id对应的值，然后替换URL中webhook/后面的id就可以了。

![](2021-10-25154637.png)



### 创建用户

通过创建用户，可以将人和设备关联起来，这样就可以定位人员的位置了。对于同一个用户，可以定义多个追踪设备，除了GPS以外，还可以定义蓝牙，Wi-Fi等。

![](2021-10-25155436.png)



### 创建视图

我们可以创建两种视图，一种是地图卡片，会将人员显示在地图上。配置卡片时，需要将实体设置为需要追踪的人员。

![](2021-10-25160029.png)

另外一种视图是概览视图，可以显示人员的位置状态，例如在家，离开。配置卡片时，需要将实体设置为需要追踪的人员。

![](2021-10-25160611.png)

