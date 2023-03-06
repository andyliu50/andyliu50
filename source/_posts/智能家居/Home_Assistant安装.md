---
title: Home Assistant安装
date: 2021-10-17 15:02:00
tags: 
 - home_assistant
categories: 
 - 智能家居
---

## 安装Docker

1 更新树莓派的系统。

```bash
sudo apt-get update
sudo apt-get upgrade
```

2 下载脚本，然后安装Docker。

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

**需要注意的是，--mirror Aliyun可以指定通过阿里云镜像安装Docker，提高安装速度。如果用默认的安装源，很慢，甚至无法安装成功。**

3 默认情况下，只有拥有管理员权限账号才可以运行Docker，如果登录的账号是普通用户，可以通过sudo来运行docker。或者，也可以将普通用户账号添加到Docker用户组，这样也可以允许该用户运行docker的命令。

以下命令可以将Pi用户添加到Docker用户组中。

```bash
sudo usermod -aG docker Pi
```

4 查看Docker的版本信息和运行信息。

```bash
docker version
docker info
```

<!-- more -->

## 安装Home Assistant

### Raspberry pi 3

```bash
docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=MY_TIME_ZONE \
  -v /PATH_TO_YOUR_CONFIG:/config \
  --network=host \
  ghcr.io/home-assistant/raspberrypi3-homeassistant:stable
```

### Raspberry pi 4

```bash
docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=MY_TIME_ZONE \
  -v /PATH_TO_YOUR_CONFIG:/config \
  --network=host \
  ghcr.io/home-assistant/raspberrypi4-homeassistant:stable
```

以上安装命令中，一定要将`/PATH_TO_YOUR_CONFIG`修改成指定的安装路径，例如`/home/pi/homeassistant`，如果忘记修改，配置文件就会保存到`/PATH_TO_YOUR_CONFIG`目录。

安装成功后，打开浏览器访问`http://<host>:8123`，就可以登录Home Assistant的Web界面。

