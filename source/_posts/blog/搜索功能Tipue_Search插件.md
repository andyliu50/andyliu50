---
title: 搜索功能Tipue_Search插件
date: 2021-11-6 9:54:00
tags: 
 - pelican
categories: 
 - Blog
---

本文主要介绍如何为Pelican博客添加搜索功能，详细介绍了Tipue_Search插件的安装和使用过程。由于Tipue Search插件的开发者已经从2018年起不再维护该插件，因此，相关的参考资料比较少。

随着博客文章数量的增加，搜索功能显得越来越重要。在Pelican官方插件库里面查询了一番，只有Tipue Search插件支持搜索功能。但是，该插件的开发者已经于2018年起不再维护该插件，所以相关参考资料非常少。但是，目前Pelican的志愿开发者正在接手维护工作。
![查找页面](2021-11-06095804.jpg)
<!-- more -->

## 安装

Tipue Search依赖于BeautifulSoup，因此首先需要安装BeautifulSoup。

```
pip install beautifulsoup4
```

到目前为止，Tipue Search还不支持通过Pip安装，所以，只能从Github上下载[插件](https://github.com/pelican-plugins)，然后将文件复制到Pelican所在的目录。

需要注意的是，下载下来的插件，目录结构如下：

```
tipue-search-main
	|-pelican
		|-plugins
			|-tipue_search
				|-__init__.py
				|-tipue_search.py
```

把**tipue_search目录复制到/blog/plugins**目录下，plugins目录需要手动创建。

## 配置Pelicanconf.py

修改Pelicanconf.py的配置文件，添加以下内容：

```
PLUGIN_PATHS = ["plugins", "./plugins"]                                                                                         PLUGINS = ["tipue_search"]                                                                                                       DIRECT_TEMPLATES = ['index', 'authors', 'categories', 'tags', 'archives', 'search']                                             MENUITEMS = [('首页', '/index.html'), ('标签', '/tags.html'), ('归档', '/archives.html'), ('查找', '/search.html')]   
```

由于无法通过Pip方式安装Tipue Search插件，所以，只能使用传统的方式来加载插件。首先，需要通过参数**PLUGIN_PATHS**，指定插件的所在目录。之前，插件已经复制到/blog/plugins目录下，所以，这里可以配置成"./plugins"。

```
PLUGIN_PATHS = ["plugins", "./plugins"]  
```

然后，指定需要加载的插件名称，通过**PLUGINS**参数，指定tipue_search目录名。

```
PLUGINS = ["tipue_search"] 
```

**Direct_TEMPLATES**参数用于指定需要生成.html文件的模板，因此，需要把search添加到该参数中（后面会介绍如何写search.html模板）。

```
DIRECT_TEMPLATES = ['index', 'authors', 'categories', 'tags', 'archives', 'search']       
```

为了能在首页菜单中显示搜索选项，需要配置参数**MENUITEMS**，添加('查找', '/search.html')。

```
MENUITEMS = [('首页', '/index.html'), ('标签', '/tags.html'), ('归档', '/archives.html'), ('查找', '/search.html')]  
```


## 下载JS文件

Tipue Search插件基于JQuery开发，因此需要用到以下JS文件：

- jquery-3.6.0.min.js
- tipuesearch.js
- tipuesearch.min.js
- tipuesearch_set.js

其中，文件jquery-3.6.0.min.js可以从JQuery[官方网站](https://jquery.com/download/)下载。

另外三个文件，由于tipue的官方网址已经无法访问，因此，可以从[Elegant主题5.0.0](https://github.com/Pelican-Elegant/elegant/tree/V5.0.0)的版本中获取。下载Elegant主题后，js文件的所在路径为elegant-5.0.0\static\tipuesearch。

## 编写Search.html

Search.html的详细内容如下：

```
{% extends "base.html" %}

{% block title %}
  Search {{ super() }}
{% endblock title %}

{% block extra_head %}
  <link rel="stylesheet" type="text/css" href="/theme/css/main.css">
  <script src="/theme/js/jquery-3.6.0.min.js"></script>
  <script src="/tipuesearch_content.js"></script>
  <script src="/theme/js/tipuesearch_set.js"></script>
  <script src="/theme/js/tipuesearch.min.js"></script>
  <script src="/theme/js/tipuesearch.js"></script>

  <script>
		$( document ).ready(function() {
			$('#tipue_search_input').tipuesearch({
				'show': 10,               // shows 10 found entries
			});
		});
  </script>
  
{% endblock %}

{% block content %}
<aside id="featured" class="body">
	<h1>查找</h1>
	<form action="search.html">
		<div class="tipue_search_group">
			<input type="text" name="q" id="tipue_search_input">
		</div>
	</form>
		<div>
			<div id="tipue_search_content"></div>
		</div>
</aside><!-- /#featured -->
{% endblock content %}
```


以下内容用于指定css和js文件的所在位置，其中tipuesearch_content.js是Tipue Search根据博客内容自动生成的文件。

```
  <link rel="stylesheet" type="text/css" href="/theme/css/main.css">
  <script src="/theme/js/jquery-3.6.0.min.js"></script>
  <script src="/tipuesearch_content.js"></script>
  <script src="/theme/js/tipuesearch_set.js"></script>
  <script src="/theme/js/tipuesearch.min.js"></script>
  <script src="/theme/js/tipuesearch.js"></script>
```


## 注意事项

在搜索框输入搜索关键词的时候，**关键词必须要用引号包含**，否则无法搜索到任何内容。



## 下载文件

以下文件中包含了所有的文件，包括Tipue_Search, JS以及Search.html。

点击下载：[tipue_search.zip](tipue_search.zip)

