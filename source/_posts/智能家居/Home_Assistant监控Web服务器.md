---
title: Home Assistant监控Web服务器
date: 2021-10-23 17:15:00
tags: 
 - home_assistant
categories: 
 - 智能家居
---

一般在企业IT运维过程中，都需要对服务器，数据库和存储设备等进行监控，以便在第一时间发现故障，从而能够在最快的时间内作出响应。如果考虑成本，通常有一些开源免费的解决方案供选择，例如Zabbix, Nagios以及Cacti等。

最近，我在折腾Home Assistant的时候，发现除了管理智能家居设备以外，也可以用来监控Web服务。话不多说，下面以监控公网上的Web服务为例，先简单介绍一下实现的原理，然后描述一下整个的配置过程。

<!-- more -->

## 准备工作

- 一台Home Assistant网关
- Web服务网址，例如：ha.example.com
- Web服务端口，例如：8123

## 实现方式

首先，Home Assistant支持**Command Line**的集成，或者说模块，简单来说，Home Assistant可以调用我们定义的命令，然后根据命令执行的结果来判断服务的运行状况。由于需要监控的是Web服务，因此，就可以用curl命令来访问远端的Web服务，根据命令执行的结果，来判断Web服务是否在正常运行。如果返回的结果与预期不符合，就发送消息给管理员。管理员收到消息后，就可以作出响应。

更进一步，如果Web服务的异常状况可以通过重启服务来解决，那么，我们甚至可以在Home Assistant上定义一个重启服务的Action（动作）。这样，Home Assistant就可以自动帮我们修复故障，实现了“自愈”功能。

## 配置Command Line集成

由于**Command Line**的功能不支持在Web界面中配置，因此，我们需要通过修改`configuration.yaml`配置文件，来完成配置。

```yaml
binary_sensor: 
	- platform: command_line                                                                                 
  		name: "HA WAN"                                                                                                           
		command: 'response=$(curl -s -L https://ha.example.com:8123 --connect-timeout 5 --cacert ./ssl/HARootCA.pem -o /dev/null -w "%{http_code}\n");test $response -eq 200 && echo "Running" || echo "Stopped"'     
		payload_on: "Running"
		payload_off: "Stopped"                         		 
		scan_interval: 60       
```

**binary_sensor:** Home Assitant支持各种类型的集成，或者说模块，binary_sensor是其中的一种类型。

**platform:** binary_sensor这种类型的集成中，command_line是其中的一种平台，其它还有小米，HomeKit，Nest等等。

**name:** 定义名称

**command:** 具体需要Home Assistant执行的命令

**payload_on:** 如果返回结果为Running，Home Assistant则判断Web服务正在运行

**payload_off:** 如果返回结果为Stopped，Home Assistant则判断Web服务停止运行

**scan_interval:** 定义检测的时间区间，单位为秒。例如，每1分钟查询一次Web服务的运行状况

下面，再简单介绍一下执行监测任务的命令：

```
'response=$(curl -s -L https://ha.example.com:8123 --connect-timeout 5 --cacert ./ssl/HARootCA.pem -o /dev/null -w "%{http_code}\n");test $response -eq 200 && echo "Running" || echo "Stopped"' 
```

其中，-L参数的作用是跟踪网址的重定向(redirection)，有些网站输入网址后，会重定向到另外一个网址，如果不设置-L参数，会导致返回以下的结果。

```
HTTP/1.1 405 Method Not Allowed
Content-Type: text/plain; charset=utf-8
Allow: GET
Content-Length: 23
Date: Mon, 18 Oct 2021 11:17:26 GMT
Server: Python/3.9 aiohttp/3.7.4.post0
```

因为我是用Docker方式安装的Home Assistant，因此Docker容器中，无法访问/etc/ssl目录下证书，导致Https访问时，无法验证服务器发送的TLS/SSL证书。因此，解决方法就是将根证书HARootCA.pem保存到Home Assistant的配置目录/PATH_TO_YOUR_CONFIG/下。我将证书保存在了/PATH_TO_YOUR_CONFIG/ssl目录下，因此，--cacert参数设置的路径值是./ssl/HARootCA.pem。

通过以下命令，可以判断curl返回的结果是否为200，如果是200，就返回Running，否则，返回Stopped。

```
test $response -eq 200 && echo "Running" || echo "Stopped"
```



## 通知模块(IFTTT)

由于我一直习惯于使用IFTTT作为消息通知App，并且Home Assistant支持IFTTT的集成，因此，决定采用IFTTT作为发送消息到手机端的方式。

简单来说，消息的传递路径就是：Home Assistant网关 -> IFTTT云服务 -> IFTTT客户端。

> 提示：IFTTT是另外一套独立的云端自动化服务，并不是Home Assistant的功能模块。IFTTT可以与Home Assistant集成使用。

首先，需要需要修改`configuration.yaml`配置文件，配置IFTTT的集成。注意：Key需要到IFTTT的网址注册申请。

```
ifttt:
    key: 'adNdkL90kkdliekdif'
```

手机端需要到应用商店下载IFTTT的客户端。

![](20211023_105828000_iOS.png)

IFTTT服务端，需要配置一个Applet，该Applet的运行逻辑是，如果Home Assistant发送消息内容给IFTTT服务端，IFTTTF服务端就将消息内容推送到用户的手机端。

![](20211023_105936000_iOS.png)

![](20211023_110011000_iOS.png)

根据IFTTT服务端的配置，Home Assistant需要在发送给IFTTT服务端的消息通知中，传递以下三个值给IFTTT服务端。

Value: IFTTT服务端定义的Applet名称

Value1: 消息的标题

Value2: 消息的正文

## 配置自动化

Home Assistant的自动化模块，可以让我们根据定义的触发器，例如，Web服务从状态On变为Off，来触发Action，例如发送消息给管理员。

我们可以通过Web界面来配置自动化，点击**配置** - 自动化 - **添加自动化**，配置**触发条件**和**动作**。

**触发条件**

- 触发条件类型：状态
- 实体：binary_sensor.ha_wan，就是之前在配置文件中定义的实体
- 从: on
- 变为: off
- 持续：3分钟

![](2021-10-23191544.png)

**动作**

动作类型：调用服务

服务：ifttt.trigger

event: general // IFTTT中定义的Applet名称

value1: 消息的标题

value2: 消息的正文

![](2021-10-23191643.png)

## 最终效果

当Web服务异常或者恢复时，手机端就能接收到IFTTT发送的消息。

![](20211023_092850000_iOS.png)
