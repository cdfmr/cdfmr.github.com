---
layout: post
title: "实现Mac应用的自我重启"
date: 2012-03-26 19:31
comments: true
categories: Mac&nbsp;OS&nbsp;X Cocoa
---

对于应用程序的某些配置项，需要重新启动应用才能生效。比较友好的方式是提醒用户，并在用户确认后完成自我重启。

实现自我重启的方法很多，易于理解且实现简单的一种如下。

1. 启动一个子进程；
2. 主进程退出；
3. 子进程延时一定时间后拉起主进程，或者检测主进程已经关闭后重新拉起。

<!--more-->

子进程可以是一个简单的命令行程序，不过最简易的方法莫过于使用Shell，以下是代码片段。

{% gist 2204627 %}

以上代码只实现了延时重启，并未检测主进程状态。

使用方法：

{% codeblock lang:objc %}
[NSApp relaunchAfterDelay:2.0];
{% endcodeblock %}
