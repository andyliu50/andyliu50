---
title: 个人博客 — GitHub Pages篇
date: 2018-11-08 11:10:29
tags:
- hexo
categories:
- Blog
---
# 什么是GitHub Pages？ #

[GitHub Pages](https://pages.github.com/)是一项免费的静态网站托管服务（不支持PHP, Ruby, or Python.），可用于托管个人、组织或者项目的网页。

**创建和发布网站的两种方式**：  


- 利用[Jekyll Theme Chooser](https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site-with-the-jekyll-theme-chooser)在线创建网站
- 在本地计算机创建网站，然后利用[GitHub Desktop](http://desktop.github.com/) 或者 [the command line](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line)将网站发布至GitHub Pages.  

**GitHub Pages网站的两种类型**：

- 项目网站 —— 该网站与项目关联， 并且网站文件存储于项目仓库的分支中
- 个人和组织网站 —— 该网站与项目无关，并且网站文件存储于专用的仓库中

<!-- more -->


# 如何将博客发布到GitHub Pages #


1. 访问[GitHub](https://github.com/)网站，注册一个新账号  

2. 登陆GitHub网站，并创建新Repository。如果用户名是john，创建的Repository name必须为john.github.io

![](09.png)  

3. 修改Hexo的配置文件  

```bash
deploy: 
	type: git              
	repo: https://github.com/john/john.github.io.git              
	branch: master
```

4. 打开GitHub Bash, 设置GitHub的账户信息

```bash
git config --global user.name "GitHub用户名"
git config --global user.email "GitHub注册邮箱"
```

5. 安装Git部署插件

```bash
npm install hexo-deployer-git --save
```


6. 运行以下三条命令，将网页内容发布到GitHub Pages  

```bash
hexo clean    // 清除缓存，并删除Public目录  
hexo g        // 创建Public目录，并在Public目录下生成静态的网页文件  
hexo d        // 将网页文件同步到GitHub，同步前需要输入GitHub用户名和密码通过身份认证
```

7. 登陆GitHub，在Repository的Settings页面中，可以看到博客的访问URL: https://john.github.io。

![](10.png)  