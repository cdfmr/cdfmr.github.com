---
layout: post
title: "打造完美的Drupal编辑器"
date: 2012-02-23 10:06
comments: true
categories: Drupal
---
一个Octopress架设的博客，怎么上来就是一篇Drupal的文章呢？很简单，本来是打算用Drupal来架设个人博客的，结果在上线前发现了Octopress，于是叛逃了 :) 。尽管不再使用Drupal，但安装Drupal过程中的一些经验还是值得记录与分享的。

言归正传，Drupal预设的编辑器过于简陋，而我当时对“完美”的定义是：可视化、图像上传、代码高亮，并且使用 [CKEditor](http://ckeditor.com/) + [IMCE](http://drupal.org/project/imce) + [SyntaxHighlighter](http://alexgorbatchev.com/) 实现了预期目标。以下是安装过程，基于Drupal 7环境。

<!--more-->

###安装CKEditor

CKEditor是最好的在线HTML编辑器之一，也是将要实现的Drupal编辑器的核心组成部分。

下载[CKEditor 3.6 for Drupal 7](http://download.cksource.com/CKEditor%20for%20Drupal/CKEditor%203.6.2%20for%20Drupal/ckeditor_3.6.2_for_drupal_7.zip)并在Drupal模块管理界面中安装（也可直接通过URL安装）。启用CKEditor，此时Drupal已经有了一个不错的所见即所得编辑器。

###安装IMCE

IMCE是一个文件管理器，用于实现图像文件的上传。其实，CKEditor本身就捆绑了一个非常好的文件管理器CKFinder，但CKFinder是收费软件（CKEditor是免费/收费双重授权的，我们使用其免费授权），故使用免费的IMCE进行替换，本文也不讨论CKFinder的配置。

下载[IMCE](http://ftp.drupal.org/files/projects/imce-7.x-1.5.tar.gz)，安装并启用。

进入CKEditor的配置页面，编辑`Advanced`配置档案，展开`FILE BROWSER SETTINGS`，可以看到`File browser type`设置为`CKFinder`，将其修改为`IMCE`。按同样方法修改`Full`配置档案。

###代码高亮

到目前为止，Drupal编辑器已经基本成型了，可以应付绝大部分的编辑需要。不过作为~~ 酷毙 ~~苦逼的码农，怎么能缺少代码高亮功能呢。

代码高亮可以通过SyntaxHighlighter实现，并且只支持`Full HTML`文本格式。相对于CKEditor和IMCE，这个东东的安装会复杂一些，慢慢道来。

1\. 下载[SyntaxHighlighter](http://alexgorbatchev.com/SyntaxHighlighter/download/download.php?sh_current)，解压到`sites/all/libraries`目录下（新安装的Drupal是没有这个目录的，需要手工创建）。为什么不在模块管理界面中安装呢，因为这只是一个Javascript库，不是Drupal模块。

2\. 下载[SyntaxHighlighter的Drupal模块](http://ftp.drupal.org/files/projects/syntaxhighlighter-7.x-2.x-dev.tar.gz)，安装并启用。

3\. 进入`SyntaxHighlighter`的配置页面，勾选需要支持的语言，保存设置。**注意：不要修改代码使用的标签名称，因为之后安装的某个插件只识别默认的\<pre\>标签**。

4\. 进入`文本格式`的配置页面，编辑`Full HTML`格式，在`启用过滤器`分组中勾选`Syntax highlighter`，并在`过滤器处理顺序`中将其置为第一个；取消选择`将换行符号转换为HTML`，否则代码换行处会插入\<br\>标签。保存设置。

OK，现在新建一篇文章，选择格式为`Full HTML`，点击CKEditor工具栏左上的`Source`按钮切换到源文件模式，输入以下内容，预览一下，是不是已经可以显示代码高亮了？

{% codeblock lang:html %}
<code class="brush:cpp">
int main(int argc, char **argv)
{
    printf("Hello, World!\n");
    return 0;
}
</code>
{% endcodeblock %}

###让CKEditor支持代码高亮

这样显然不够，虽然实现了代码高亮显示，但怎样才能在CKEditor中直接输入代码呢？我们需要安装一个叫做[ckeditor-syntaxhighlight](http://code.google.com/p/ckeditor-syntaxhighlight/)的插件。

1\. 下载[ckeditor-syntaxhighlight](http://ckeditor-syntaxhighlight.googlecode.com/files/ckeditor-syntaxhighlight-1.0.tar.bz2)，将其中的`ckeditor-syntaxhighlight/plugins/syntaxhighlight`解压到`sites/all/modules/ckeditor/ckeditor/plugins`目录下。

2\. 编辑`sites/all/modules/ckeditor/includes/ckeditor.lib.inc`文件，在`ckeditor_load_plugins`函数的`CKEditor build-in plugins`段中加入以下代码。

{% codeblock lang:python %}
if (file_exists($_editor_path . 'plugins/syntaxhighlight/plugin.js'))
{
    $arr['syntaxhighlight'] = array
    (
        'name' => 'syntaxhighlight',
        'desc' => t('Syntaxhighlight plugin'),
        'path' => $base_path . $editor_path . 'plugins/syntaxhighlight/',
        'default' => 'f'
    );
}
{% endcodeblock %}

3\. 编辑`sites/all/modules/ckeditor/ckeditor.config.js`文件，在`Drupal.settings.cke_toolbar_DrupalFull`数组的最后加入`'Code'`，即数组内容的最后一行修改为`['DrupalBreak', 'DrupalPageBreak', 'Code']`，保存。

4\. 进入CKEditor的配置页面，编辑`Full`配置档案，展开`EDITOR APPEARANCE`，在`Plugins`分组中勾选`Syntaxhighlight plugin`，保存设置。

好了，新建一篇`Full HTML`格式的文章，看看CKEditor工具栏上最后一排是不是新增了一个`code`按钮，点击一下看看。如果看不到新增的按钮，在CKEditor的`Full`配置中，展开`EDITOR APPEARANCE`，在`工具栏`分组中，重新`Load sample toolbar: Full`，保存设置。

大功告成！

###题外话

如果你是一名程序员，不排斥控制台操作方式，并且只想简简单单地写一写博客，强烈推荐[Octopress](http://octopress.org)，个中好处请自行[Google](http://www.google.com/search?q=Octopress)之。