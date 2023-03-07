---
title: 华硕路由器VPN功能
date: 2021-10-02 09:01:00
tags: 
 - raspberrypi
 - vpn
categories: 
 - 智能家居
---

平时，家中的路由器很少会受到关注，除非无法上网或者网速变慢，才会想到去检查路由器。因此，可以说，家用路由器的存在感是比较弱的。但是，最近帮朋友解决了一个家中的网络问题后，改变了我对路由器的看法。

朋友是一个比较注重个人隐私和数据安全的人，因此，总是把一些重要文件、照片等存放在家中的电脑上面。但是，如果他不在家中，就无法访问家里的电脑了。因此，朋友向我咨询是否有什么办法，可以让他即使在外面，也可以访问家中电脑上的数据。于是，我建议他使用网盘同步文件，但是朋友比较担心文件的安全性，不想使用网盘。

我想了想，那只能使用**虚拟私人专用网络(VPN)** 了，但是，这项功能一般只会在企业级的设备上使用，家用路由器通常不支持这项功能。跟朋友解释了以后，朋友说他买的路由器功能比较多，也许支持这项功能。

<!-- more -->

## VPN服务器

于是，抱着试试的心态，我在朋友家中打开了路由器的管理界面，在左侧菜单果然看到了**VPN**的选项。并且， 支持三种类型的VPN: PPTP, OpenVPN和IPSec VPN，我决定采用IPSec VPN。开启IPSec VPN的功能以后，以下配置信息需要记录下来，因为配置VPN客户端的时候会用到：

- **服务器IP地址：vpn.asuscomm.com**
- **预共享密钥：123456**
- **用户名称与密码：test/123456**

![](2021-10-02094926.jpg)

## iOS客户端

路由器上的配置完成以后，接下来需要在手机端配置VPN功能，下面以苹果手机为例。

在手机上打开**设置 - 通用 - VPN与设备管理 - VPN**，然后点击**添加VPN配置...**。

![](IMG_3185.png)

以下是具体配置信息：

- **类型：IPSec**
- **描述：MyVPN**
- **服务器：vpn.asuscomm.com**
- **账户：test**
- **密码：123456**
- **密钥：123456**

![](IMG_3187.png)

配置完成后，点击连接VPN，如果显示为**已连接**，说明配置成功。

![](IMG_3186.png)

最后，除了iOS客户端以外，还支持**Windows，Android**和**MacOS**。如果想要了解更多关于VPN配置的信息，可以点击关注我。

## 树莓派客户端

1  安装**Network Manager**网络管理工具。

```bash
sudo apt install network-manager network-manager-gnome
```

安装完成后，可以在桌面右上角看到该工具的图标。点击该图标，可以看到配置VPN连接的选项。

![](IMG_3188.png)

2  安装**Network Manager**的插件**network-manager-strongswan**。

```bash
sudo apt-get install network-manager-strongswan
```

安装成功后，添加VPN类型中，会出现**IPSec/IKEv2(strongswan)** 的选项。

![](IMG_3189.png)

3  安装**Strongswan**的插件**libstrongswan-eap-mschapv2.so**, 由于之前没有安装该插件，导致一直无法连接成功。

```bash
sudo apt-get install libcharon-extra-plugins
```

安装成功后，可以在`/usr/lib/ipsec/plugins`目录下，查看**libstrongswan-eap-mschapv2.so**文件。

![](IMG_3190.png)

4 在VPN的选项下，配置详细的信息：

   **Address: ivpn.asuscomm.com**

   **Certificate: 华硕路由器导出的根证书，可以在Windows系统上先转换成Base64编码的.cer文件**

   **Authentication: EAP**

   **Username: rpi**

   **Password: 123456**

   **打开选项Request an inner IP address**

![](IMG_3191.png)

5 VPN连接成功后，图标上会出现一个小锁，用命令`ip addr`查看IP地址信息，发现并没有出现虚拟网卡，但是物理网卡上绑定了一个新的IP地址。

![](IMG_3192.png)

![](IMG_3193.png)

另外，**Network Manager**除了IPSec VPN以外，还支持OpenVPN, Cisco IPSec VPN, L2TP/IPSec VPN，只需要安装相应的插件即可。其中，安装L2TP/IPSec VPN插件时，需要使用树莓派默认的软件仓库，国内软件仓库安装过程中会出现缺少dnsmasq包的404报错。

**OpenVPN**

```bash
sudo apt-get install network-manager-openvpn network-manager-openvpn-gnome
```



**Cisco IPsec VPN**

```bash
sudo apt-get install network-manager-vpnc vpnc network-manager-vpnc-gnome
```



**L2TP/IPSec VPN**

```bash
sudo apt-get install network-manager-l2tp network-manager-l2tp-gnome
```

