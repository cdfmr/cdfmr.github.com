---
layout: post
title: "Mac OS X下使用bindfs实现mount的目录绑定功能"
date: 2012-02-28 00:02
comments: true
categories: Mac&nbsp;OS&nbsp;X
---
Linux下的mount命令有一个`--bind`参数，将目录挂载到另一个目录下。Mac OS X的mount命令不支持`--bind`，不过我们可以使用[bindfs](code.google.com/p/bindfs/)实现相同的功能。

bindfs是一个基于[FUSE](http://fuse.sourceforge.net/)的文件系统实现，并非Mac OS X的预装工具，但通过[Homebrew](../../../../2012/02/25/homebrew-installation-and-usage/)安装非常简单。

<!--more-->

`brew install bindfs`

Homebrew会自动安装bindfs以及其依赖的gettext和fuse4x。如果出现未找到pkg-config的错误，请先输入`brew install pkg-config`安装。

安装完成后，需要在系统启动时加载FUSE内核扩展。

{% codeblock lang:bash %}
sudo cp -rfX /usr/local/Cellar/fuse4x-kext/0.8.14/Library/Extensions/fuse4x.kext /System/Library/Extensions
sudo chmod +s /System/Library/Extensions/fuse4x.kext/Support/load_fuse4x
{% endcodeblock %}

重启生效。

bindfs的使用也非常简单，跟`mount --bind`基本一样。

`bindfs 源目录 挂载点`

更多信息可以在终端里输入`man bindfs`查阅。