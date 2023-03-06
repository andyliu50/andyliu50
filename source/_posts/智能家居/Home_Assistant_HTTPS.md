---
title: Home Assistant配置HTTPS
date: 2021-10-19 15:02:00
tags: 
 - home_assistant
categories: 
 - 智能家居
---

默认情况下，Home Assistant安装完成后，是使用HTTP访问。如果在内网使用，HTTP相对还比较安全。但是，为了能让手机在外网也能访问HA网关，需要将其发布到外网（Internet）。因此，首要任务就是要配置HTTPS，确保数据的安全性。

为了节约成本，我打算使用自签的SSL证书。虽然官方也推荐了免费的TLS/SSL证书提供商：[Let’s Encrypt](https://www.home-assistant.io/docs/ecosystem/certificates/lets_encrypt/)和[Duck DNS integrating Let’s Encrypt](https://www.home-assistant.io/integrations/duckdns/)，但是考虑到自己环境的复杂性，决定暂时先用自签的证书。

<!-- more -->

## 制作自签的证书

证书的制作过程都是在树莓派上使用**openssl**完成。

### 制作根证书

首先，需要创建根证书密钥，该密钥主要用来为证书签名。安全起见，该密钥不能泄露给其他人。

```bash
openssl genrsa -des3 -out rootCA.key 4096
```

**该命令生成的Key默认使用密码保护，如果要取消密码保护，只需要移除选项-des3。**

下面，可以制作根证书。

```bash
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -out rootCA.pem
```

如果需要使用苹果设备**macOS vs 10.15 / iOS 13 (or above)**，请使用以下命令：

```bash
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 825 -out rootCA.pem
```

### 制作HA证书

请根据以下步骤，制作用于HA服务器的证书。

**创建rootCA.csr.cnf文件**

```
# rootCA.csr.cnf
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
             
[dn]
C=CN
ST=Shanghai
L=Shanghai
O=HA
OU=HAU
emailAddress=admin@ha.com
CN = ivpn.asuscomm.com   
```

**创建v3.ext文件**

```
# v3.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
extendedKeyUsage=serverAuth
                
[alt_names]
DNS.1 = ivpn.asuscomm.com
IP.1 = 192.168.1.65
```

**创建证书密钥**

```bash
openssl req -new -sha256 -nodes -out hassio.csr -newkey rsa:2048 -keyout hassio.key -config <( cat rootCA.csr.cnf )
```

如果在Windows平台上运行该命令，需要注意-config参数后面rootCA.csr.cnf文件的路径地址。请参考以下例子：

```bash
openssl req -new -sha256 -nodes -out hassio.csr -newkey rsa:2048 -keyout hassio.key -config "C:\Program Files\Git\usr\bin\rootCA.csr.cnf"
```

**创建证书**

```bash
openssl x509 -req -in hassio.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out hassio.crt -days 3650 -sha256 -extfile v3.ext
```

如果需要使用苹果设备**macOS vs 10.15 / iOS 13 (or above)**，请使用以下命令：

```bash
openssl x509 -req -in hassio.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out hassio.crt -days 825 -sha256 -extfile v3.ext
```



## 配置Home Assistant

将HA证书重新命名：

```
rename hassio.crt fullchain.pem
rename hassio.key privkey.pem
```

将证书复制到`/PATH_TO_YOUR_CONFIG/ssl`目录：

**这里需要注意的是，由于采用了Docker方式安装HA，没有权限访问/etc目录，所以，只能将证书复制到/PATH_TO_YOUR_CONFIG/ssl。**

```bash
mkdir /PATH_TO_YOUR_CONFIG/ssl
cp fullchain.pem /PATH_TO_YOUR_CONFIG/ssl/
cp privkey.pem /PATH_TO_YOUR_CONFIG/ssl/
```

修改配置文件`configuration.yaml`：

```
http:                                                                       
	ssl_certificate: ./ssl/fullchain.pem                                     
	ssl_key: ./ssl/privkey.pem  
```



## 将Home Assistant发布到公网

在HA网关上运行以下命令，该命令会将本地的8123端口映射到远端树莓派的50302端口。

```
autossh -f -M 0 -NR 0.0.0.0:50302:localhost:8123 pi@ivpn.asuscomm.com -p 50001
```

然后，创建计划任务，每次HA网关重启后，会自动执行以上命令。

```
crontab -e
@reboot autossh -f -M 0 -NR 0.0.0.0:50302:localhost:8123 pi@ivpn.asuscomm.com -p 50001 &
```

在华硕路由器上，选择**外部网络(WAN) - 端口转发 - 自定义设置** ，添加端口转发的条目。

```
服务名称：Home-Assistant
通信协议：TCP
外部端口：50302
内部端口：50302
本地IP地址：192.168.50.68
```



## iOS如何安装根证书

点击rootCA.pem文件，安装根证书。

![](20211019_001515000_iOS.png)

打开**设置**应用，找到**已下载描述文件**，点击**安装**，安装过程中需要输入手机密码。

![](20211019_001547000_iOS.png)

安装完成后，需要在**设置 - 通用 - 关于本机 - 证书信任设置**中，启用安装的根证书。

![](20211019_001714000_iOS.png)



## Android如何安装根证书

重新命名rootCA.pem文件为**rootCA.crt**。

点击安装rootCA.crt文件。

