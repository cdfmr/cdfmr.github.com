---
layout: post
title: "几个非官方的OpenWrt软件包"
date: 2013-10-15 15:08
comments: true
categories: OpenWrt
---

[OpenWrt](https://openwrt.org)是一个优秀的开源路由器固件，也是一个高度可定制的Linux发行版。OpenWrt包含opkg软件包管理工具，可以从软件源中轻松安装数以千计的软件包。即便如此，有时我们还是需要从官方源之外寻找其它软件包，以满足某些特殊的需求。本文就收集了我目前使用的一些非官方软件包，其中有自己创建的，但大部分来自于第三方。

<!--more-->

### busybox

官方源busybox中的ls命令将中文字符显示为问号，于是Google找到[ring0](http://www.pppei.net/blog/post/507)编译的版本，在此基础上打包成ipk文件。该软件包将busybox安装于/usr/bin目录中，且只对ls命令创建了软链接，故只替换了ls命令，不会破坏预装的/bin/busybox，卸载后即可恢复为预装版本。

[下载](/blog/files/busybox-unicode_1.19.4_ar71xx.ipk)

### dnspod-client

[DNSPod](http://www.dnspod.cn)动态域名客户端，基于[vinoca](http://www.vinoca.org/2012/06/10/openwrt%E4%BD%BF%E7%94%A8dnspod%E7%9A%84%E5%8A%A8%E6%80%81%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90ddns%E5%8A%9F%E8%83%BD)的脚本改造而来。安装完成后，修改/etc/config/dnspod文件填入正确的用户名、密码及域名信息，并且需要在DNSPod上预先创建相应的子域名。

[下载](/blog/files/dnspod-client_1.0_all.ipk)

### hfs-fsck

HFS+文件系统检查工具，解决HFS+文件系统只读的问题，参见[原文](http://www.geektu.com/post/2013-08-11-yong-openwrt-diy-time-capsule)。

[下载](/blog/files/hfs-fsck_332.25-1_ar71xx.ipk)

### luci-app-aria2

Aria2下载器的Web前端，已经忘记从哪里下载而来了。我做了部分修改，修正了脚本文件中的一些BUG，并且将[yaaw](https://github.com/binux/yaaw)替换成了[webui-aria2](https://github.com/ziahamza/webui-aria2)（个人喜好）。

[下载](/blog/files/luci-app-aria2_1.0_all.ipk)

### luci-app-file-explorer

基于Web的文件浏览器（只读）。安装此软件包会重写/etc/httpd.conf文件，用于设置访问口令。OpenWrt预装时该文件是不存在的，而且在大多数情况下也不会使用。如果在安装前已经对该文件进行过配置，请注意备份。

[下载](/blog/files/luci-app-file-explorer_1.0_all.ipk)

最后，对软件包的原作者表示感谢。
