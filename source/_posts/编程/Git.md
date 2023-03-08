---
title: 如何使用Git
date: 2021-12-30 14:00:00
tags: 
 - git
categories: 
 - 编程
---
 ![Git](git.jpg) 本文主要介绍如何使用Git，包括Git客户端，Github以及Gitee。

## Git客户端

下载地址：[Git (git-scm.com)](https://git-scm.com/)

文档：[Git - Reference (git-scm.com)](https://git-scm.com/docs)

## 初始设置

安装完成后，需要先设置一下用户名称和邮箱：

```
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

<!-- more -->
## 创建Git版本库

使用命令`git init`，创建一个新的Git版本库，或者重新初始化现有的版本库。创建成功后，会在当前目录创建一个隐藏目录.git。

```
$ git init
Initialized empty Git repository in C:/project/.git/
```

## 修改初始化仓库时的默认分支名称(master -> main)

初始化仓库时，Github的默认分支名称已经从master更改为main，但是Git客户端和Gitee的默认分支名称仍然为master。通过以下方式，可以修改Git客户端和Gitee的版本库名称。

### Git客户端

使用以下命令，将Git客户端创建的默认分支名称设置为main

```
git config --global init.defaultBranch main
```

### Gitee

在Gitee的后台管理界面，选择**设置**-**仓库首选项**，将master修改为main。

![Gitee](2021-12-30_141954.jpg)

## 修改本地的分支名称(master -> main)

如果初始化完成后，本地分支的名称为master，可以通过命令`git branch -m master main`修改为main。

```
$ git branch -m master main
$ git branch
* main
```

然后，通过命令`git push -u gitee main`将本地分支推送到远程分支。并且，将远程分支设置为与main关联。

```
$ git push -u gitee main
$ git remote set-head gitee main
```

## Git客户端配置文件

Git客户端的配置文件是在用户目录(~)下的`.gitconfig`文件。

文件内容如下：

```
:> Get-Content ~\.gitconfig
[user]
        email = andyliu50@outlook.com
        name = Andy Liu
[core]
        autocrlf = true
[init]
        defaultBranch = main
```

## Git本地和远程同步

### 方法一

本地已经完成仓库的初始化，在远程Github或者Gitee上创建相同名称的仓库。然后，在本地运行以下命令：

```
git remote add github git@github.com:andyliu50/mytool.git
git push -u github main
```

其中，github是远程仓库在本地的名称，默认一般用origin，由于本地要同时和Github和Gitee同步，为了便于区分，因此将远程仓库设置为github。andyliu50是github的账号名称，mytool是远程仓库的实际名称。

命令`git push`将本地分支main推送到远程仓库。

### 方法二

本地不创建仓库，在远程Github或者Gitee创建仓库，然后，在本地运行以下命令，将远程仓库克隆到本地：

```
git clone git@github.com:andyliu50/mytool.git
```

## 创建标签并推送到远程

用`git tag`可以创建标签

```
git tag v1.0
```

可以将一个标签推送到远程库

```
git push github v1.0
```

也可以将多个标签推送到远程库

```
git push github --tags
```

