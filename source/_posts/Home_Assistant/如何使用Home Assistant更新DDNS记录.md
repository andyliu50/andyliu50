---
title: 如何使用Home Assistant更新DDNS记录
date: 2024-09-26 10:05
tags:
  - home_assistant
categories:
  - Home_Assistant
updated:
---

## 缘起

由于家中的宽带运营商不提供公网IPv4地址，所以一直都没有将Home Assistant发布到公网。最近无意中从网上了解到可以使用IPv6的公网地址，经过测试，确实有效。

考虑到IPv6地址是动态分配的，间隔一段时间就会自动更新为新的IPv6地址，于是就在免费的DDNS服务商[Dynv6](https://dynv6.com/)申请了动态域名。根据Dynv6官方API文档介绍，IPv6不支持用`Remote Access API`，也就意味着不支持用[ddclient](https://ddclient.net/)作为客户端，来自动更新DDNS记录。

```
### DynDNS API

We implemented the dyn.com [Remote Access API](https://help.dyn.com/remote-access-api/) that is also known as the _Members NIC Update API_ or _DNS Update API_. It can only be used to update the IPv4 address of a zone.

To use the endpoint https://dynv6.com/nic/update please set the server in your client to 'dynv6.com'.
```

根据[REST API for dynv6](https://dynv6.github.io/api-spec/)文档，自己用Python写了一个脚本，用于自动更新DDNS记录。经过一段时间的使用，基本能满足需求，但是仍然有一些小瑕疵，比如运营商自动分配IPv6地址的过程中，有时候会出现短时间内（5-10分钟）多次分配IP地址的情况，并且网络也会变得很不稳定，这时会导致DDNS更新出现问题。

最近在使用Home Assistant的过程中，发现了`RESTful Command`的集成，于是想到为何不干脆使用Home Assistant来完成DDNS记录的更新，这样还省去了单独维护Python脚本的时间成本。

## 需求

其实需求很简单，就是当Home Assistant的IPv6地址自动更新为新地址的时候，Home Assistant自动将新地址更新到`Dynv6`的动态域名记录中。

### 具体细节

为了实现上述需求，需要考虑以下细节：

- 触发器

	Home Assistant会实时监控公网IPv6地址的变化，当IP地址发生变化，就会触发Home Assistant更新DDNS记录的动作。
	
- 触发的先决条件

	Home Assistant需要实时监控Dynv6 Rest API的网络连接，以确保可以成功更新DDNS记录。如果网络连接异常，即使公网IPv6地址发生改变，也不会触发Home Assistant的更新DDNS记录的动作。

- 动作

	Home Assistant执行更新DDNS记录的动作。

## System Monitor

**System Monitor**可以用来监控CPU、内存、硬盘等硬件信息，在这里，主要用来实时获取网口的IPv6地址信息。当IPv6地址发生改变时，就可以触发Home Assistant的自动化操作。

**System Monitor**是Home Assistant的一个集成，或者说是扩展功能，使用之前需要手工添加和配置。

以下是网口IPv6地址的实体ID，该实体的值是IPv6地址：

```
entity_id: sensor.system_monitor_ipv6_address_eth0
```

## Dynv6 Rest API

[REST API for dynv6](https://dynv6.github.io/api-spec/)

**Rest API URL**
https://dynv6.com/api/v2

Dynv6 Rest API主要分为两大类：**Zones**和**Records**。

### Zones

**GET**          Get a list of records
**POST**       Add a new record
**GET**          Get details for a record
**PATCH**     Update an existing record
**DEL**           Delete a record

### Records

**GET**          Get a list of zones
**POST**       Register a new zone
**GET**          Get details for a zone
**PATCH**     Update an existing zone
**DEL**           Delete the zone
**GET**           Get details of a zone by its name

下载：[openapi.json](openapi.json)

## Command Line

**Command Line**是Home Assistant中的一个集成功能，可以通过命令的方式，定义一种传感器实体。因此，通过该集成，可以定义一个二进制传感器，检测Dynv6 Rest API的网络连接性。

### nc工具

为了检测Dynv6 Rest API的网络连接性，可以使用命令行工具`nc`。

以下命令可以用来检测Home Assistant和Dynv6 Rest API服务之间的网络连接性。如果Home Assistant与Dynv6 Rest API服务的TCP 443端口连接成功，返回“success”，否则返回“fail”。

```
nc -6z -w 2 dynv6.com 443 > /dev/null 2>&1 && echo success || echo fail'"	
```

参数说明：

-6        指定nc使用本机的IPv6地址。使用该选项，主要是为了确保IPv6已经更新完成，并能够与Dynv6 API服务之间正常通讯，这也是为了避免前面所提到的，自己写的脚本中的“小瑕疵”。
-z         nc只做端口扫描，不传输数据。
-w        如果连接不上，2秒后超时。

### SSH远程命令

另外，由于Home Assistant是通过容器的方式安装，并且容器中`nc`工具的版本不支持IPv6，所以只能考虑用主机的`nc`。为了能调用主机的`nc`，决定使用SSH的方式远程调用命令。

```
ssh -o StrictHostKeychecking=no pi@pi -i /config/.ssh/id_rsa 'nc -6z -w 2 \
			 dynv6.com 443 > /dev/null 2>&1 && echo success || echo fail'
```

参数说明：

-o StrictHostKeychecking=no      登录时不需要验证目标主机的Key
-i /config/.ssh/id_rsa                        私钥文件的位置

### configuration.yaml

以下是Home Assistant的配置文件`configuration.yaml`中的详细配置信息：

```
- binary_sensor:	
	name: 'Dynv6 API'	
	command: "ssh -o StrictHostKeychecking=no pi@pi -i /config/.ssh/id_rsa 'nc -6z -w 2 \
			 dynv6.com 443 > /dev/null 2>&1 && echo success || echo fail'"	
	payload_on: success	
	payload_off: fail	
	scan_interval: 60	
	icon: mdi:api	
	unique_id: cmd_dynv6_api	
	device_class: connectivity
```

以上配置定义了一个二进制传感器，该传感器的名字为“Dynv6 API”，每60秒执行一次网络连接性测试，如果成功则返回“success”，否则返回“fail”。

## RESTful Command

**RESTful Command**是Home Assistant中的一个集成功能，通过该集成，可以定义一个对Rest API的操作请求，执行DDNS记录的更新。

```
rest_command:
	dynv6_update_zone:
	url: https://dynv6.com/api/v2/zones/<zone_id>
	method: patch
	content_type: application/json
	headers:	
		authorization: 'Bearer <Token>'
	payload: '{"ipv6prefix":"{{ states("sensor.system_monitor_ipv6_address_eth0") }}"}'
```

url               zone_id需要替换成与Zone对应的id。
method     更新Zone的方法需要用PATCH。
authorization        需要用Token。
payload:    更新后的IPv6地址，此处是读取传感器`sensor.system_monitor_ipv6_address_eth0`的值。

## 自动化

在Home Assistant中，创建一个名为“DDNS｜自动更新动态域名”的自动化，该自动化可以实现自动更新DDNS记录的需求。

### trigger

如果传感器`sensor.system_monitor_ipv6_address_eth0`值发生变化，触发器就会被自动触发。

to: null 传感器的值发生任何变化都会导致触发器被触发。

```
trigger:
  - platform: state
    entity_id:
      - sensor.system_monitor_ipv6_address_eth0
    to: null
    enabled: true
```

### condition

触发器被触发的先决条件是，二进制传感器`binary_sensor.dynv6_api`的状态必须是`on`。

```
condition:
  - condition: state
    entity_id: binary_sensor.dynv6_api
    state: "on"
```

### action

当自动化被触发器成功触发后，就会执行操作`rest_command.dynv6_update_zone`，该操作将执行DDNS记录的更新。

```
action:
  - action: rest_command.dynv6_update_zone
    data: {}
```

### configuration.yaml

以下是自动化的详细配置信息：

```
alias: DDNS｜自动更新动态域名
description: ""
trigger:
  - platform: state
    entity_id:
      - sensor.system_monitor_ipv6_address_eth0
    to: null
    enabled: true
condition:
  - condition: state
    entity_id: binary_sensor.dynv6_api
    state: "on"
action:
  - action: rest_command.dynv6_update_zone
    data: {}
mode: single
```