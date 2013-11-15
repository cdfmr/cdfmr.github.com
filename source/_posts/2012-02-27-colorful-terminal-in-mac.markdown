---
layout: post
title: "让Mac OS X的终端多姿多彩"
date: 2012-02-27 13:57
comments: true
categories: Mac
---

与Linux相比，Mac OS X的终端总是欠缺些什么。对了，是色彩，Linux的ls命令使用不同颜色区分各种文件类型，Vim编辑器也支持语法高亮，而Mac终端却总是以黑白示人。其实，只要稍微做一些工作，Mac的终端同样可以多姿多彩，请往下看。

<!--more-->

## 彩色化ls的输出

Mac中BSD的ls命令可以使用`-G`参数彩色化输出的文件列表，需要配置LSCOLORS环境变量定义颜色，具体配置方法可以输入`man ls`查看。

不过，我推荐安装Linux使用的GNU Coreutils替换Mac的ls命令，因为：

* Coreutils提供了配置工具，定义颜色代码更加方便；  
* Coreutils包含的不仅仅是ls，同时作为Linux用户，我更习惯于使用GNU的各种shell工具。

Coreutils的安装与配置方法如下：

1. 通过[Homebrew](/blog/2012/02/25/homebrew-installation-and-usage/)安装Coreutils  
`brew install xz coreutils`  
注：Coreutils并不依赖于xz，但它的源码是用xz格式压缩的，安装xz才能解压。

2. 生成颜色定义文件  
`gdircolors --print-database > ~/.dir_colors`

3. 在`~/.bash_profile`配置文件中加入以下代码

{% codeblock lang:bash %}
if brew list | grep coreutils > /dev/null ; then
	PATH="$(brew --prefix coreutils)/libexec/gnubin:$PATH"
	alias ls='ls -F --show-control-chars --color=auto'
	eval `gdircolors -b $HOME/.dir_colors`
fi
{% endcodeblock %}

gdircolor的作用就是设置ls命令使用的环境变量LS_COLORS（BSD是LSCOLORS），我们可以修改~/.dir_colors自定义文件的颜色，此文件中的注释已经包含各种颜色取值的说明。

看看默认颜色的显示效果。  
![ls screenshot](/blog/images/2012-02-27-colorful-terminal-in-mac_ls.png)

## grep高亮显示关键字

这个很简单，加上`--color`参数就可以了，为了使用方便，可以在`~/.bash_profile`配置文件中加上alias定义。

{% codeblock lang:bash %}
alias grep='grep --color'
alias egrep='egrep --color'
alias fgrep='fgrep --color'
{% endcodeblock %}

## Vim语法高亮

在Vim中输入命令`:syntax on`激活语法高亮，若需要Vim启动时自动激活，在`~/.vimrc`中添加一行`syntax on`即可。


