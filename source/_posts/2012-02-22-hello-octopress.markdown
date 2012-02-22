---
layout: post
title: "Hello, Octopress!"
date: 2012-02-22 10:15
comments: true
categories: 
---
{% codeblock lang:bash %}
cd octopress
rake new_post["hello octopress"]
vi source/_post/2012-02-22-hello-octopress.markdown
rake generate
rake deploy
{% endcodeblock %}
