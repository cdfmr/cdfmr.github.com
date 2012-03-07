---
layout: post
title: "Xcode的点滴记录"
date: 2012-03-07 09:58
comments: true
categories: Mac&nbsp;OS&nbsp;X Xcode
---

升级到Lion以及Xcode 4之后，发现Xcode 4在配置上与Xcode 3有些不同，记录于此备忘。

<!--more-->

## 代码中的公司名称

Xcode生成代码时，自动在文件头插入注释，其中的公司名称需要设置，否则以`__MyCompanyName__`代替。

Xcode 3中设置ORGANIZATIONNAME宏定义。

{% codeblock lang:bash %}
defaults write com.apple.Xcode PBXCustomTemplateMacroDefinitions '{ORGANIZATIONNAME="YourNameHere";}'
{% endcodeblock %}

Xcode 4不再使用此宏定义，直接读取系统地址簿中的公司信息，所以需要在地址簿中给自己设定一个公司名称。

Xcode 3和Xcode 4都可以针对项目单独设置公司名称，其中Xcode 3的设置项是项目信息窗口中`General`选项卡的`Organization Name`，而Xcode 4是右边栏项目文件信息中`Project Document`分组下的`Organization`。

## 让大括号另起一行

使用Xcode自动完成功能生成的代码，左大括号是位于行末的。如果要让大括号另起一行，Xcode 3与Xcode 4的设置并不相同。

Xcode 3的设置：

{% codeblock lang:bash %}
defaults write com.apple.Xcode XCCodeSenseFormattingOptions -dict BlockSeparator "\n"
{% endcodeblock %}

Xcode 4中，自动完成功能是由代码片段库`Code Snippet Library`控制的，因此需要修改相应的代码片段模板。点击代码片段，会弹出窗口显示其内容，并提供编辑功能。但是，Apple并不允许我们修改内建的代码模板。不过，我们可以绕过Xcode进行修改。Xcode 4内建的代码片段保存在/Developer/Library/Xcode/PrivatePlugIns/IDECodeSnippetLibrary.ideplugin/Contents/Resources/SystemCodeSnippets.codesnippets文件中，这是一个plist文件，直接编辑即可。[这里](https://gist.github.com/1992813)是我修改好的文件。

## 烦人的调试权限问题

将Lion升级到10.7.3之后，每次在Xcode 4中运行程序都会弹出如下信息提示，输入用户密码才能继续。

	“Developer Tools Access”需控制另一进程，才能继续调试。键入您的密码以允许执行此操作。
	
以及

	“gdb-i386-apple-darwin”需控制另一进程，才能继续调试。键入您的密码以允许执行此操作。
	
[Stack Overflow](http://stackoverflow.com/questions/9132826/stop-developer-tools-access-needs-to-take-control-of-another-process-for-debugg)上给出了两种解决方案。

* 重新安装Xcode

* 按如下方法修改/etc/authorization文件

{% gist 1992771 %}

没有验证第一种方法，第二种方法经验证有效（修改前切记备份原文件）。