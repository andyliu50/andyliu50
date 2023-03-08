---
title: PowerShell Tips
date: 2021-04-15 12:06:16
tags: 
 - powershell
categories: 
 - 编程
---

本文主要介绍关于PowerShell中的一些小技巧，以便于需要时查询。

## 如何知道脚本文件所在路径

有时候为了能访问与脚本在相同目录的文件，需要知道脚本的路径位置。通过变量$MyInvocation可以获取脚本的路径和名称。

$MyInvocation是一种自动变量（Automatic Variables），自动变量一种用于存储PowerShell状态的特殊变量，由PowerShell创建和维护。关于自动变量的详细介绍，可以通过命令`Help about_Automatic_Variables`查看。

```
$ScriptPath = $MyInvocation.MyCommand.Path
$ScriptName = $MyInvocation.MyCommand.Name
$ScriptDir = Split-Path $ScriptPath
```

`$MyInvocation.MyCommand.Path`返回的是脚本文件的绝对路径，通过命令`Split-Path`，可以获取脚本文件所在目录的绝对路径。


## 批处理文件中%~dpn0

当把批处理文件和其它文件(PowerShell脚本)部署到计算机上后，如果无法知道文件所在的目录，就可以使用%~dpn0。

在批处理文件中，**%n**是一种特殊变量：

**%0**: 批处理文件的完整路径（包括批处理文件的文件名和后缀） 
**%1**: 批处理文件后面的第一个参数 
**%2**: 批处理文件后面的第二个参数 
… 

另外，以下参数可以和%0结合使用，用于表达文件路径：

**~d**: 驱动器盘符，例如C:\, D:\  
**~p**: 路径（不包含驱动器盘符和批处理文件名） 
**~n**: 批处理文件名

所以，如果结合起来使用就是：

**%0**: 批处理文件的完整路径（**包含批处理文件的后缀**） 
**%dp0**:  批处理文件所在的目录  
**%dpn0**: 批处理文件的完整路径（**不包含批处理文件的后缀**）

举例：

1. 如果在同一个目录下，有两个文件：test.bat, test.ps1，由于这两个文件的文件名相同，后缀不同，因此，可以在test.bat中，使用以下代码，通过运行test.bat文件来执行PS脚本。

```
@ECHO OFF
PowerShell.exe -Command "& '%~dpn0.ps1'"
```

2. 如果在同一个目录下，有两个文件：test.bat, myscript.ps1，由于这两个文件的文件名不同，因此，在test.bat中，需要使用以下代码。由于~n代表批处理文件的文件名，此处需要删除n，替换成PS脚本的文件名。

```
@ECHO OFF
PowerShell.exe -Command "& '%~dp0myscript.ps1'"
```


