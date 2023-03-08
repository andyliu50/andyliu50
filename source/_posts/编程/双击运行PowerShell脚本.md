---
title: 如何双击运行PowerShell脚本
date: 2021-11-8 08:47
tags: 
 - powershell
categories: 
 - 编程
---

由于PowerShell脚本双击后，系统默认使用记事本打开脚本，导致脚本无法直接运行。本文主要介绍如何可以通过双击运行的方式，直接运行PowerShell脚本。

由于Windows系统对PowerShell做了许多安全性的防护措施，导致PowerShell不像以前的批处理文件那样，复制到电脑上就可以直接运行。目前，主要有以下一些原因会导致运行PowerShell出现问题：

- **默认情况下，PowerShell脚本的.ps1文件与记事本关联。**

  Windows系统默认将.ps1文件与记事本关联，因此，如果双击运行脚本，只会打开记事本，并显示脚本的内容，而不会通过PowerShell运行脚本。

- **PowerShell默认阻止运行来自于外部的脚本。**

​		默认情况下，PowerShell的ExecutionPolicy阻止运行任何来自于外部的脚本，例如从互		联网下载的脚本，或者来自于网络共享的脚本等。

- **有些PowerShell脚本需要管理员权限才能正常运行。**

​		有些命令需要管理员权限才能正常运行，因此需要获取管理员权限并执行脚本。

- **有些用户可能有自定义的PowerShell环境**。

​		有些高级用户，可能会修改系统的PowerShell环境，因此导致脚本运行不成功。

<!-- more -->

## 双击运行脚本

虽然双击运行PowerShell脚本，只会打开记事本。但是，我们仍然可以通过.BAT批处理文件来调用PowerShell脚本，从而实现双击运行脚本的目的。

首先，需要将批处理文件的文件名设置为与PowerShell脚本相同的文件名。例如，PowerShell脚本的文件名为MyScript.ps1，那么批处理文件的文件名为MyScript.bat。然后，将这两个文件放到相同的目录下。

在批处理文件中，写入以下代码：

```
@ECHO OFF
PowerShell.exe -Command "& '%~dpn0.ps1'"
PAUSE
```

**@ECHO OFF：**阻止命令行提示符以及脚本中的命令显示在屏幕上。

**PowerShell.exe -Command "& '%~dpn0.ps1'"：**这条命令用于调用PowerShell.exe，从而可以执行脚本文件。**'%~dpn0.ps1'**就是PS脚本文件的完整路径。

以下是关于**'%~dpn0.ps1'**的详细说明，详细内容，请参考文档：[PowerShell Tips]({filename}PowerShell Tips.md)

> 当把批处理文件和其它文件(PowerShell脚本)部署到计算机上后，如果无法知道文件所在的目录，就可以使用%~dpn0。
>
> 在批处理文件中，**%n**是一种特殊变量：**%0**: 批处理文件的完整路径（包括批处理文件的文件名和后缀）
> **%1**: 批处理文件后面的第一个参数 
> **%2**: 批处理文件后面的第二个参数 
> … 
>
> **~d**: 驱动器盘符，例如C:\, D:\
> **~p**: 路径（不包含驱动器盘符和批处理文件名） 
> **~n**: 批处理文件名
>
> 所以，如果结合起来使用就是：
>
> **%0**: 批处理文件的完整路径（**包含批处理文件的后缀**） 
> **%dp0**:  批处理文件所在的目录 
> **%dpn0**: 批处理文件的完整路径（**不包含批处理文件的后缀**）



## ExecutionPolcy

默认情况下，Windows系统阻止运行PS脚本。但是，通过批处理文件，我们可以临时-ExecutionPolicy的设置，来达到执行脚本的目的。注意：该ExecutionPolicy的设置只对当前的PowerShell会话有效。

```
PowerShell.exe -ExecutionPolicy Bypass -Command "& '%~dpn0.ps1'"
```



## 获取管理员权限

在执行某些PowerShell命令时，必须有管理员权限，才能正常运行。但是，通过批处理文件或者CMD命令，无法触发UAC来提升权限。但是，我们可以通过PowerShell的Start-Process -Verb RunAS命令，来实现提权。

以下命令开启了两个PowerShell的进程，一个用来运行Start-Process，另外一个由Start-Process生成。注意%~dpn0.ps1用了两个双引号。

```
PowerShell.exe -Command "& {Start-Process PowerShell.exe -ArgumentList '-ExecutionPolicy Bypass -File ""%~dpn0.ps1""' -Verb RunAs}"
```



## 用户自定义PowerShell配置文件

有些情况下，用户可能会修改PowerShell配置文件的默认设置，这种情况有可能导致脚本运行失败。为了避免这种情况，我们可以采用-NoProfile参数，这样，执行脚本的时候，就不会去加载用户的配置文件。

```
PowerShell.exe -NoProfile -Command "& {Start-Process PowerShell.exe -ArgumentList '-NoProfile -ExecutionPolicy Bypass -File ""%~dpn0.ps1""' -Verb RunAs}"
```

**[参考资料]** 

- [How to Use a Batch File to Make PowerShell Scripts Easier to Run (howtogeek.com)](https://www.howtogeek.com/204088/how-to-use-a-batch-file-to-make-powershell-scripts-easier-to-run/#:~:text=PowerShell.exe)

- [Check for Admin Credentials in a PowerShell Script - Scripting Blog (microsoft.com)](https://devblogs.microsoft.com/scripting/check-for-admin-credentials-in-a-powershell-script/)

