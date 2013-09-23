---
layout: post
title: "用Shell实现简单的Web爬虫"
date: 2013-09-15 22:47
comments: true
categories: Shell
---

对于结构不太复杂的网页，使用grep和sed分析提取目标URL，之后使用wget下载。

以下是两个例子。

<!--more-->

* 抓取[煎蛋](http://jandan.net)“妹子图”栏目

{% gist 6571304 %}

* 抓取[Panoramio](http://www.panoramio.com)网站某个用户上传的所有照片

{% gist 6606109 %}
