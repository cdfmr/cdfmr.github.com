---
layout: post
title: "Xcode: 使用Git自动设置项目版本"
date: 2012-03-27 15:13
comments: true
categories: Mac&nbsp;OS&nbsp;X Xcode Git SVN
---

本来一直使用SVN进行版本管理，并习惯于将代码版本（revision）作为Xcode项目的编译号（CFBundleVersion）。最近开始了解Git，被其种种美好所诱惑，于是一股脑将个人的所有项目都转移到Git，然后发现CFBundleVersion不好设置了。

<!--more-->

与SVN不同，Git使用散列值作为每次提交的标记，而不是像SVN那样使用递增序列，因此不便直接用作CFBundleVersion。正愁以后每次都要麻烦地手工设置了，苦思冥想一番之后豁然开朗，虽然Git没有数值化的版本号，但我可以自己数一数提交次数啊，于是有了这个脚本。

{% codeblock lang:bash %}
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion `git log | grep -P 'commit [\da-f]{40}' | wc -l`" ${TARGET_BUILD_DIR}/${INFOPLIST_PATH}
{% endcodeblock %}

很简单，数一下Git版本库的提交次数，使用PlistBuddy将其设置为CFBundleVersion。

顺便附上以前使用的SVN脚本，稍稍不同的是将版本号补齐到3位。

{% codeblock lang:bash %}
rev=`svnversion -n`
while [ ${#rev} -lt 3 ] ; do
    rev="0${rev}"
done
echo -n ${TARGET_BUILD_DIR}/${INFOPLIST_PATH} | xargs -0 /usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${rev}"
{% endcodeblock %}

使用方法：新建一个`Run Script`类型的`Build Phase`。