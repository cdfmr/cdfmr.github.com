---
layout: post
title: "诡异的NSURLErrorDomain －3001错误"
date: 2012-12-27 21:36
comments: true
categories: Mac Cocoa
---

最近在使用NSURLDownload下载文件时，遇到NSURLErrorDomain －3001错误，具体的错误日志如下。

*The operation couldn’t be completed. (NSURLErrorDomain error -3001.)*

Google无果，最后居然发现是因为目标目录只读所致。由于很难从错误信息得知真正原因，故记录于此备忘。
