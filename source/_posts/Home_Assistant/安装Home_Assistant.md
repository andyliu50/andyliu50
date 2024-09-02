---
title: 安装Home Assistant
date: 2021-10-17 15:02:00
tags:
  - home_assistant
categories:
  - Home_Assistant
updated: 2024-9-02 09:35:00
---

## 通过Docker的方式安装Home Assistant
### 安装Docker

1 更新树莓派的系统。

```bash
sudo apt-get update
sudo apt-get upgrade
```

2 下载Docker安装脚本，然后执行脚本安装Docker。

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

**需要注意的是，由于国内访问Docker官方的安装源很慢，因此可以添加参数`--mirror Aliyun`，将安装源更改为阿里云镜像，从而提高安装速度。**

<!-- more -->

3 默认情况下，只有拥有管理员权限的账号才可以运行docker，如果登录的账号是普通用户，需要通过sudo来运行docker。或者，也可以将普通用户账号添加到docker用户组，这样该用户就可以直接运行docker的命令。

以下命令可以将用户`pi`添加到docker用户组中。

```bash
sudo usermod -aG docker pi
```


4 使用命令`docker version`和`docker info`查看Docker的版本信息和运行状态。

```bash
$ docker version

Client:
 Version:           20.10.5+dfsg1
 API version:       1.41
 Go version:        go1.15.15
 Git commit:        55c4c88
 Built:             Mon May 30 18:34:49 2022
 OS/Arch:           linux/arm64
 Context:           default
 Experimental:      true

Server:
 Engine:
  Version:          20.10.5+dfsg1
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.15.15
  Git commit:       363e9a8
  Built:            Mon May 30 18:34:49 2022
  OS/Arch:          linux/arm64
  Experimental:     false
 containerd:
  Version:          1.4.13~ds1
  GitCommit:        1.4.13~ds1-1~deb11u4
 runc:
  Version:          1.0.0~rc93+ds1
  GitCommit:        1.0.0~rc93+ds1-5+deb11u2
 docker-init:
  Version:          0.19.0
  GitCommit:

docker info
```

```bash
$ docker info

Client:
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc., v0.10.4)
  compose: Docker Compose (Docker Inc., v2.17.3)

Server:
 Containers: 1
  Running: 1
  Paused: 0
  Stopped: 0
 Images: 1
 Server Version: 20.10.5+dfsg1
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 1.4.13~ds1-1~deb11u4
 runc version: 1.0.0~rc93+ds1-5+deb11u2
 init version:
 Security Options:
  seccomp
   Profile: default
  cgroupns
 Kernel Version: 6.1.21-v8+
 Operating System: Debian GNU/Linux 11 (bullseye)
 OSType: linux
 Architecture: aarch64
 CPUs: 4
 Total Memory: 7.629GiB
 Name: pi4-8g
 ID: AT4S:WDKG:SCHA:D56T:SRNU:TJJB:ZEMH:YR67:6GRF:KLVC:B6MI:SVIE
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:
  https://docker.mirrors.ustc.edu.cn/
 Live Restore Enabled: false

WARNING: No memory limit support
WARNING: No swap limit support
WARNING: Support for cgroup v2 is experimental
```

### 安装Home Assistant

### 先决条件

- Docker至少是19.03.9或者以上版本
- `libseccomp`的版本至少是 2.4.2或者以上

> **提示**
> 通过Docker方式安装Home Assistant，不支持[add-ons]([Home Assistant Add-ons - Home Assistant (home-assistant.io)](https://www.home-assistant.io/addons))（通过安装第三方的应用程序来扩展Home Assistant的功能），并且无法从管理界面中更新Home Assistant的版本。

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
  -v /run/dbus:/run/dbus:ro \
  --network=host \
  ghcr.io/home-assistant/raspberrypi4-homeassistant:stable
```

以上安装命令中，一定要将`/PATH_TO_YOUR_CONFIG`修改成指定的安装路径，例如`/home/pi/homeassistant`，如果忘记修改，配置文件就会保存到`/PATH_TO_YOUR_CONFIG`目录。

更改`MY_TIME_ZONE`为本地时区，例如`Asia/Shanghai`。

如果要使用[Bluetooth integration](https://www.home-assistant.io/integrations/bluetooth), 需要设置`D-Bus`，即`-v /run/dbus:/run/dbus:ro`

安装成功后，打开浏览器访问`http://<host>:8123`，登录Home Assistant的Web界面。

## 更新Home Assistant

下载最新版本的Home Assistant，如果命令结果返回"Image is up to date"，则说明目前已经是最新版本。

```bash
docker pull ghcr.io/home-assistant/home-assistant:stable
```

停止运行容器中的Home Assistant

```bash
docker stop homeassistant
```

删除容器中的Home Assistant，**该操作不会删除原有的配置文件**。

```bash
docker rm homeassistant
```

运行最新版本的Home Assistant，将`PATH_TO_YOUR_CONFIG`设置为原配置文件所在的路径，例如`/home/pi/homeassistant`。

```bash
docker run -d \
	--name homeassistant \
	--restart=unless-stopped \ 
	--privileged \ 
	-e TZ=MY_TIME_ZONE \ 
	-v /PATH_TO_YOUR_CONFIG:/config \ 
	-v /run/dbus:/run/dbus:ro \ 
	--network=host \ 
	ghcr.io/home-assistant/home-assistant:stable
```