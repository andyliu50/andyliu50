---
title: 黑苹果（Hackintosh）
date: 2022-07-31 11:38:23
tags:
- macos
categories:
- 计算机
---

# 黑苹果（Hackintosh）

## 设置Homebrew镜像源

### 中科大源

```bash
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
brew update
```

### 清华源

```bash
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
brew update
```

## bottles镜像

以中科大源为例， 从`macOS Catalina`(10.15.x) 版开始，`Mac`使用`zsh`作为默认`Shell`，对应文件是`.zprofile`，所以命令为：

```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles/bottles' >> ~/.zprofile
source ~/.zprofile
```

### 恢复默认源

```bash
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask.git
brew update
```

`homebrew-bottles`配置只能手动删除，将 `~/.zprofile` 文件中的 `HOMEBREW_BOTTLE_DOMAIN=https://mirrors.xxx.com`内容删除，并执行 `source ~/.zprofile`。

## Homebrew工具-“fatal: Could not resolve HEAD to a revision”错误解决

### 问题描述

在进行brew update时，发生错误，如下：

```bash
% brew update
fatal: Could not resolve HEAD to a revision
Already up-to-date.
```

### 解决方法

思路：

- 找到出错的homebrew的目录；

- 进入到该目录，执行fetch 和 pull命令。

1）运行“brew update --verbose”，找到出错的homebrew的目录：

```bash
% brew update --verbose
Checking if we need to fetch /usr/local/Homebrew...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core...
Fetching /usr/local/Homebrew...
Updating /usr/local/Homebrew...
Branch 'master' set up to track remote branch 'master' from 'origin'.
Switched to and reset branch 'master'
Your branch is up to date with 'origin/master'.
Switched to and reset branch 'stable'
Current branch stable is up to date.
Updating /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core...
fatal: Could not resolve HEAD to a revision
Already up-to-date.
```

2）进入到出错的目录，然后先fetch，后pull，如下：

注意：执行以下命令前，最好先更换成国内的镜像源。

```bash
% cd /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
% pwd
/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
% git fetch --prune origin
% git pull --rebase origin master
 From https://github.com/Homebrew/homebrew-core
* branch                    master     -> FETCH_HEAD
Updating files: 100% (6435/6435), done.
```



## 无法成功冷启动，提示“busy timeout[1], (60s): 'IOUSBHostDevice' ”

冷启动就是指电脑断电（拔掉电源线）一段时间，然后再次加电的启动过程。所以，每次冷启动的时候，需要先进入Windows系统，重启电脑，然后再进入Mac系统。

每次冷启动失败的时候，启动日志的信息提示：busy timeout[1], (60s): 'IOUSBHostDevice'，并且是每隔60s就会出现Intel Firmware驱动初始化的过程。

![](202207311229.jpeg)

根据'IOUSBHostDevice'的关键字信息在网上查询后得知，'IOUSBHostDevice'应该是指蓝牙设备。使用`Hackintool`工具查询得知，系统目前的蓝牙驱动版本是`IntelBluetoothFireware-v1.1.2`。于是，到开发者网站[IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware)上查询后，发现最新版是`v2.1.0`。

通过更新蓝牙驱动到最新版本v2.1.0后，可以成功实现冷启动直接进入Mac系统。

## 如何更新Hackintosh的驱动或者说内核扩展

如果要更新黑苹果的驱动文件，即.kext文件，在Mac系统中，需要下载软件[OpenCore Configurator](https://mackie100projects.altervista.org/download-opencore-configurator/)，打开该软件以后，加载并打开EFI分区。

![](202207311230.jpeg)

然后把.kext文件（`ntelBluetoothFirmware.kext`和`IntelBluetoothInjector.kext`）复制到EFI分区的`/EFI/OC/Kexts`目录下：

![](202207311231.jpeg)

## 从休眠中唤醒后，系统自动重启

系统每次从休眠中唤醒后，都会自动重启。再次进入系统后，会弹出macOS问题报告，报告内容提示**“thunderbolt power on failed”**。

![](202207311244.jpeg)

**解决方法**

进入**BIOS**设置，选择`Advanced > Devices > Onboard Devices`，禁用`Thunderbolt Controller`。

## Mac版Things试用到期后，如何重复使用

Mac版本的[Things](https://culturedcode.com/things/)应用只有15天的试用期，到期后就无法正常使用。即使删除Things文件后，重新下载并打开Things，还是无法使用。

为了能重复使用试用版的Things，可以下载趋势科技的**Cleaner One**应用。当Things快到期或者已过期的时候，右键点击Things，然后选择“移到废纸篓”，然后清倒废纸篓，在此过程中，**Cleaner One**应用会自动删除一些Things的残留文件，这些文件应该与Things限制到期无法使用的功能有关。

删除Things成功后，从以前下载的Things3.zip压缩文件中重新解压出Things，并双击打开应用，这是可以正常使用Things，并且试用期恢复为15天。

![](cleaner_one.jpeg)

## 如何通过命令行打开VMWare Fusion虚拟机

可以通过`vmrun`命令，打开虚拟机，其中：

参数`-T`可以指定主机的类型: `ws`和`fusion`,`ws`是Windows版VMWare，`fusion`是Mac版VMWare。

`start`:开启虚拟机

`suspend`：挂起虚拟机

`'/Users/andyliu/Virtual Machines.localized/win10.vmwarevm'`: 虚拟机文件的路径

```bash
vmrun -T fusion start '/Users/andyliu/Virtual Machines.localized/win10.vmwarevm'
vmrun -T fusion suspend '/Users/andyliu/Virtual Machines.localized/win10.vmwarevm'
```

如果想要通过SSH的方式，远程开启后和关闭虚拟机，需要使用以下命令：

注意：通过这种方式，需要指定`vmrun`的完整路径`'/Applications/VMware Fusion.app/Contents/Public/vmrun'`。

```bash
ssh andyliu@mac "'/Applications/VMware Fusion.app/Contents/Public/vmrun' -T fusion start '/Users/andyliu/Virtual Machines.localized/win10.vmwarevm'"

ssh andyliu@mac "'/Applications/VMware Fusion.app/Contents/Public/vmrun' -T fusion suspend '/Users/andyliu/Virtual Machines.localized/win10.vmwarevm'"
```

## 如何设置三码

(1) 打开`Hackintool`工具，选择**系统 - 序列号生成器**，找到三码（**序列号、主板序列号和SmUUID**），并将其复制到文本编辑器。如果想要更换三码，可以点击右下角的刷新按钮，重新生成三码。

![](hackintool_sn_generator.png)

(2) 打开`Opencore Configurator`，挂载EFI分区，然后在**EFI - EFI - OC**目录下，打开`config.plist`

(3) 选择**[Platforminfo-机型平台设置] - [DataHub - Generic - PlatformNVRAM]**， 在**Generic**栏，填入三码。

![](oc_sn.png)

(4) 保存并关闭`config.plist`。



## 休眠后，无法唤醒显示器

(1) 打开`Opencore Configurator`，挂载EFI分区，然后在**EFI - EFI - OC**目录下，打开`config.plist`

(2) 选择NVRAM-随机访问存储器设置 - 7C436110-AB2A-4BBB-A880-FE41995C9F82，双击boot-args，添加参数：

```
igfxonln=1
```

![](oc_bootargs.png)



## 如何制作MacOS Ventura系统DMG安装镜像

1. 通过App Store，下载最新版的MacOS操作系统版本。
2. 打开磁盘工具，选择文件 - 新建映像 - 空白映像。

![](empty_image.png)

3. 根据下载的操作系统版本文件的大小，例如12GB，创建的空白镜像文件大小为13-15GB。

![](create_empty_image.png)

4. 双击挂载新建的空白镜像文件。
5. 打开终端工具，运行以下命令。

```
$ sudo /Applications/Install\ macOS.app/Contents/Resources/createinstallmedia --volume /Volumes/macOS\ Ventura --applicationpath ~/Desktop/macOS/Install\ macOS.app --nointeraction
Password:（输入您的登录密码，回车）
Erasing Disk: 0%... 10%... 20%... 30%...100%...
Copying installer files to disk...
Copy complete.
Making disk bootable...
Copying boot files...
Copy complete.
Done.
```

