---
layout: post
title: "在沙盒环境中使用Finder打开文件夹"
date: 2012-06-23 15:39
comments: true
categories: Cocoa Mac&nbsp;OS&nbsp;X App&nbsp;Sandbox
---

从6月1日开始，Apple要求所有提交到App Store的应用程序都必须运行在沙盒环境中，这可苦了我们这些悲催的码农。一则受到沙盒环境的种种限制，一些很常用的操作都没法实现；二则App Sandbox本身就是BUG重重，这个更是有理没处说。

举个例子，常规环境中，一般使用Shared File List API来设置程序随系统自动运行。到了沙盒环境中，Apple不许我们使用此API了，而建议使用Service Management来实现类似功能。Service Management不如传统方法直接，必须将一个Helper程序设置为启动项，由Helper程序唤起主程序，而且创建的启动项不能显示在系统设置中。这就算了，问题是Helper程序怎么也不能唤起主程序，这个BUG直到10.7.4才解决。

言归正传，本文要说的是另一个BUG。通常，如果需要展示某个文件夹的内容，我们会使用NSWorkspace的OpenURL方法。即使在沙盒环境中，只要已经取得了该文件夹的相关权限，此方法显然也应该没问题。但Apple不这么想，直到目前的Lion 10.7.4中，即便是打开沙盒容器中的目录都无法成功，可以在控制台看见这样的信息。

<!--more-->

`CoreServicesUIAgent: Quarantine resolution refused to pid 5872 because it is not allowed to read /Users/xxxx/Library/Containers/com.xxxxx.xxxxx/Data/Library/Application Support`

这显然是一个BUG，而且实际上程序是可以对沙盒目录进行读写的，不可能真的没有权限。

于是上网寻求解决方案，最终在[The Unarchiver](http://code.google.com/p/theunarchiver/)的代码中找到了。

{% codeblock lang:objc %}
static void OpenFolderWithAppleScriptBecauseTheSandboxIsTerrible(NSString *path)
{
        FSRef ref;
        bzero(&ref,sizeof(ref));
        if(FSPathMakeRef((UInt8 *)[path fileSystemRepresentation],&ref,NULL)!=noErr) return;

        static const OSType signature='MACS';
        AppleEvent event={typeNull,nil};
        AEBuildError builderror;

        AEDesc filedesc;
        AEInitializeDesc(&filedesc);
        if(AECoercePtr(typeFSRef,&ref,sizeof(ref),typeAlias,&filedesc)!=noErr) return;

        if(AEBuildAppleEvent(
        kCoreEventClass,kAEOpenDocuments,
        typeApplSignature,&signature,sizeof(OSType),
        kAutoGenerateReturnID,kAnyTransactionID,
        &event,&builderror,
        "'----':(@)",&filedesc)!=noErr) return;

        AEDisposeDesc(&filedesc);

        AppleEvent reply={typeNull,nil};

        AESendMessage(&event,&reply,kAENoReply,kAEDefaultTimeout);

        AEDisposeDesc(&reply);
        AEDisposeDesc(&event);

        NSArray *apps=[NSRunningApplication runningApplicationsWithBundleIdentifier:@"com.apple.finder"];
        [[apps objectAtIndex:0] activateWithOptions:0];
}
{% endcodeblock %}

看看函数名称，看来抱怨App Sandbox的人不仅仅只是我。:)

使用这个方法，必须在App Sandbox的授权文件中为Finder增加特例。

{% codeblock lang:c %}
Key:   com.apple.security.temporary-exception.apple-events
Value: com.apple.finder
{% endcodeblock %}

至于这样能否通过App Store的审核，要看审核人员的心情了，至少The Unarchiver是通过了，我的App还在等待审核，前途未卜。说到这个，小小发下牢骚，为了实现将文件解压到压缩包所在的目录，The Unarchiver在授权文件中加了一堆目录的读写权限；而我为了实现类似的功能（从PDF文件中提取图片）使用了相同的方法，死活不能通过审核！

补充：又一次被拒了，就是因为这个Finder的例外。苹果的双重标准太恶心了，而且还在文档中明确指出不能以别的App通过审核作为申诉的理由，去TNND。