---
title: 用Home Assistanat跟踪汽油价格
date: 2024-9-29 16:48:00
tags:
  - home_assistant
categories:
  - Home_Assistant
---

## 缘起

虽然新能源汽车越来越普及，但是就目前而言，传统的燃油车仍然是占比最高的车型。因此，每次油价的调整自然会备受车主的关注。

由于平时懒得打开App查看油价，所以每次加油时才会看一眼加油站的标价。因此，其实本人对汽油的价格并不感冒。但是，最近一直在“折腾”Home Assistant，脑海中浮现的想法都是围绕着它展开，于是就想到了用Home Assistant来跟踪油价的变化。

## 需求

首先，汽油的价格能够被实时获取，并且获得的数据能够被存储到Home Assistant中。当汽油的价格发生变化时，例如上涨或者下跌，Home Assistant可以发送通知到本人的手机。

## 数据源

在网上查找了一下关于汽油价格的网站，发现以下网站的信息比较全面，并且提供的油价可以精确到区级。

http://www.qiyoujiage.com/

下面以上海为例，访问的网址是： http://www.qiyoujiage.com/shanghai.shtml ，网页的部分源代码如下：

\<dl> \<dt> \<dd>是HTML中的一套组合标签，可以用于制作网页中的表格。

```
<div id="youjia">
	<dl>
		<dt>上海92#汽油</dt>
		<dd>7.34</dd>
	</dl>
	<dl>
		<dt>上海95#汽油</dt>
		<dd>7.81</dd>
	</dl>
	<dl>
		<dt>上海98#汽油</dt>
		<dd>9.71</dd>
	</dl>
	<dl>
		<dt>上海0#柴油</dt>
		<dd>6.99</dd>
	</dl>
</div>
```

## Beautiful Soup

`Beautiful Soup`是一款非常著名的Python第三方库，可以用来解析HTML文本，且非常简单易用。

首先，用`Requests`获取网页的内容，并将返回的结果存储到`response`变量。

```python
import requests
from bs4 import BeautifulSoup
import json
from datetime import datetime

url = 'http://www.qiyoujiage.com/shanghai.shtml'
response = requests.get(url, timeout=(5, 10))
```

用`Beautiful Soup`来解析网页内容（`response.content`），先查找所有`dl`标签，然后用`for`循环语句迭代每一个`dl`标签的内容，如果找到关键字`92`，就结束循环，并返回`name`和`price`。

为了便于Home Assistant处理脚本返回的值，此处最终结果以JSON字符串的格式输出。

```python
format_string = '%Y-%m-%d %H:%M:%S'
now = datetime.strftime(datetime.now(), format_string)

youjia_92 = {}
soup = BeautifulSoup(response.content, 'html.parser')
for dl in soup.find_all('dl'):
	name = dl.find('dt').text.strip()
	value = float(dl.find('dd').text.strip())
	if '92' in name:
		youjia_92['name'] = name
		youjia_92['price'] = value
		youjia_92['last_update'] = now
		print(json.dumps(youjia_92))
		break
```

运行以上代码，就会得到下面的结果：

```python
{"name": "上海92#汽油", "price": 7.34, "last_update": "2024-09-28 12:33:01"}
```

## Command Line

打开并编辑Home Assistant的配置文件`configuration.yaml`，添加传感器的配置内容。

```yaml
command_line:
	- sensor:
		name: "上海92#油价"
		command: python3 ./scripts/oil_price/oil_price.py
		json_attributes:
			- name
			- last_update
		value_template: "{{ value_json.price }}"
		device_class: monetary
		unit_of_measurement: 元
		unique_id: command_oil_price
		scan_interval: 7200
		command_timeout: 60
```

配置参数说明：

name                         传感器的名称
command                需要执行的命令或者脚本
json_attributes       传感器的属性，此处有两个属性：name和last_update
value_template      获取命令或者脚本返回的值
device_class           实体的类型，Home Assistant会根据设置的类型，在Dashboard中适配相应的图表
unit_of_measurement   值的单位
scan_interval          执行脚本的时间间隔，默认单位为“秒”
command_timeout  命令或者脚本默认15秒后超时，此处设置为60秒

完成以上配置，并重载命令行配置以后，在Home Assistant中就会新增一个实体，该实体的标识符为
`sensor.shang_hai_92_you_jie`。

该实体的状态属性如下：

```
name: 上海92#汽油
last_update: "2024-09-29 16:19:12"
price_changed: 油价下跌
unit_of_measurement: 元
device_class: monetary
icon: mdi:gas-station
friendly_name: 上海92#油价
```

## 油价涨跌变化

为了能跟踪油价的涨跌变化，并且可以在第一时间接收到关于油价涨跌的信息，需要根据油价在发生变化前后的比较，来判断油价的涨跌。

**弯路：** 根据文档[automation | trigger state](https://www.home-assistant.io/docs/automation/templating/#state)，自动化的触发器可以提供触发前后的数据：`trigger.from_state`和`trigger.to_state`，但是实际测试过程中，发现如果触发器是基于状态（state）的变化，而不是数值区间（numeric_state）的变化，则无法在消息通知中通过`jinja`模板来进行数值对比。

因此，我决定通过执行脚本来反映油价的涨跌情况。那么，如何在脚本中获取油价变化前的数据呢？此时，我想到了用[home assistant rest api](https://developers.home-assistant.io/docs/api/rest/)。在获取最新油价之前，先从Home Assistant获取当前的油价信息。

```python
from requests.packages.urllib3.exceptions import InsecureRequestWarning

rest_url = 'https://localhost:8123/api/states/<entity_id>'
token = <token id>

headers = {
	"Authorization": f"Bearer {token}",
	"Content-Type": "application/json"
}
```

请将`entity_id`换成实体的真实ID标识符。长期有效的Token需要在Home Assistant管理界面的个人配置文件中生成，具体可以参考文档：[Authentication](https://www.home-assistant.io/docs/authentication/#your-account-profile)

```python
previous_price = 0
try:
	requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
	response_api = requests.get(
		rest_url, headers=headers, verify=False, timeout=(5, 10)
	)
	previous_price = float(response_api.json()['state'])
except Exception as err:
	pass
```

因为用`requests`访问API的时候会出现SSL证书错误的告警信息，且访问会失败，所以需要添加参数`verify=Fales`，这样就不会对SSL证书进行检查。但是，仍然会打印告警信息。因此，还需要用以下方法关闭告警信息。

```python
from requests.packages.urllib3.exceptions import InsecureRequestWarning

requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
```

通过以上代码，就获取到了油价变化前的价格`previous_price`。

通过跟最新油价的比较，就可以判断油价的上涨和下跌。这样，油价传感器的属性又增加了一项`price_changed`。

```python
current_price = youjia_92['price']
if current_price > previous_price:
	youjia_92['price_changed'] = '油价上涨'
elif current_price < previous_price:
	youjia_92['price_changed'] = '油价下跌'
else:
	youjia_92['price_changed'] = 'N/A'
```

## 自动化

### trigger

当实体`sensor.hui_shan_92_you_jie`的属性`price_changed`值从`N/A`变化成任何其它状态，自动化都会被触发。

```
trigger:
  - platform: state
    entity_id:
      - sensor.shang_hai_92_you_jie
    attribute: price_changed
    from: N/A
```

### action

自动化触发后，执行通知动作，将通知消息发送到目标手机，消息的标题是实体`sensor.shang_hai_92_you_jie`的属性值`price_changed`，消息的正文是自动化触发前的价格`{{ trigger.to_state.state }}`和触发后的价格`{{ trigger.from_state.state }}`。

```
action:
  - action: notify.mobile_app_iphone
    metadata: {}
    data:
      message: 当前价格：{{ trigger.to_state.state }} 以前价格：{{ trigger.from_state.state }}
      title: "{{ state_attr('sensor.shang_hai_92_you_jie', 'price_changed') }}"
      data:
        push:
          sound: Doorbell.caf
```

手机上收到的通知消息见下图：

![通知消息](F8B7D9EE-6145-4C58-AF0D-72244715D40B_1_201_a.jpeg)

## 开发者工具

Home Assistant的开发者工具中有两项功能对于测试很有帮助，分别是：**设置状态**和**动作**。

### 设置状态

设置状态可以改变实体的状态值，例如，可以改变实体`sensor.shang_hai_92_you_jie`的油价，这样就可以帮助测试自动化的运行结果是否正确。例如，真实油价是7.34，我们就可以将油价改成7.50，然后当传感器再次获取到真实油价的时候，就可以触发自动化的运行。

打开**开发者工具**，在**状态**中可以**设置状态**的选项。

![开发者工具_设置状态](Pasted_image_20240929162823.png)

### 动作

由于传感器只能根据设定的时间间隔更新数据，没有手动更新的选项。但是，测试时需要立即获取到最新的数据，这时可以用`homeassistant.update_entity`来实现。

打开**开发者工具**，进入**动作**页面，进入**YAML模式**，输入以下内容，或者也可以在**用户界面模式**下操作。点击**执行动作**后，就会更新实体`sensor.shang_hai_92_you_jie`。

```
action: homeassistant.update_entity
data:
  entity_id:
    - sensor.shang_hai_92_you_jie
```

## 参考资料

[templating](https://www.home-assistant.io/docs/configuration/templating/)
[jinja | templates](https://jinja.palletsprojects.com/en/latest/templates/)
[memory of previous state](https://community.home-assistant.io/t/memory-of-previous-state/290288)
[automation | trigger state](https://www.home-assistant.io/docs/automation/templating/#state)
[manually refresh rest sensors](https://community.home-assistant.io/t/manually-refresh-rest-sensor/353208)
[perform actions](https://www.home-assistant.io/docs/scripts/perform-actions/#homeassistant-services)
[Automation: template value should be a string for dictionary value @ data[‘value_template’]. Got None](https://community.home-assistant.io/t/automation-template-value-should-be-a-string-for-dictionary-value-data-value-template-got-none/419519)
[home assistant rest api](https://developers.home-assistant.io/docs/api/rest/)
[# Python ‘requests’ Module: How to Disable Warnings](https://www.slingacademy.com/article/python-requests-module-how-to-disable-warnings/)
