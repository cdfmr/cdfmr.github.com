---
layout: post
title: "Cocoa菜单栏右侧图标的实现"
date: 2012-02-28 21:35
comments: true
categories: Cocoa Mac
---

与Windows中的Tray Icon不同，Mac OS X中类似的UI元素位于屏幕右上角菜单栏的右侧，官方命名为Status Item，我们可以使用NSStatusBar和NSStatusItem类在菜单栏上为自己的应用添加图标。

{% codeblock lang:objc %}
NSStatusBar *statusBar = [NSStatusBar systemStatusBar];
NSStatusItem *statusItem = [statusBar statusItemWithLength:NSVariableStatusItemLength];
[statusItem setImage:image];
[statusItem setMenu:menu];
[statusItem setHighlightMode:YES];
[statusItem retain];
{% endcodeblock %}

使用这种方法创建的图标，位于菜单栏图标区域的最左侧，如果菜单栏上图标太多，很容易就被程序菜单遮挡了。那么，有没有办法在菜单栏的右侧添加图标呢？

<!--more-->

根据Apple的开发文档，这是不可能实现的。不过，Apple留了后门，通过调用NSStatusBar类的一些非公开API，是可以将图标移动到菜单栏右侧的。我们来看看怎么做。

首先要声明将要调用的非公开API。

{% codeblock lang:objc %}
@interface NSStatusBar (NSStatusBar_Private)
- (id)_statusItemWithLength:(float)l withPriority:(int)p;
- (id)_insertStatusItem:(NSStatusItem *)i withPriority:(int)p;
@end
{% endcodeblock %}

与NSStatusBar的常规接口相比，最关键的是新增的withPriority参数，这个参数指明了图标的优先级，实际上就是图标在菜单栏上的位置。

那么，怎么使用这两个API呢？根据我从茫茫网络收集的资料，有不完全相同的两种方法，各有优劣。

**方法一**

{% codeblock lang:objc %}
if ([statusBar respondsToSelector:@selector(_statusItemWithLength:withPriority:)] &&
    [statusBar respondsToSelector:@selector(_insertStatusItem:withPriority:)])
{
    int priority = INT_MAX - 1;
    if (!statusItem)
    {
        statusItem = [statusBar _statusItemWithLength:0 withPriority:priority];
        [statusItem retain];
    }
    [statusBar removeStatusItem:statusItem];
    [statusBar _insertStatusItem:statusItem withPriority:priority];
    [statusItem setLength:NSVariableStatusItemLength];
}
{% endcodeblock %}

**方法二**

{% codeblock lang:objc %}
- (void)restartSystemUIServer 
{
    NSTask *killSystemUITask = [[[NSTask alloc] init] autorelease];
    NSMutableArray *args = [NSMutableArray array];
    [args addObject:@"SystemUIServer"];
    [args addObject:@"-HUP"];
    [killSystemUITask setLaunchPath:@"/usr/bin/killall"];
    [killSystemUITask setArguments:args];
    [killSystemUITask launch];
}

- (void)createStatusBar
{
    if ([statusBar respondsToSelector:@selector(_statusItemWithLength:withPriority:)])
    {
        if (!statusItem)
        {
            statusItem = [statusBar _statusItemWithLength:0 withPriority:INT_MAX];
            [statusItem retain];
        }
        [statusItem setLength:NSVariableStatusItemLength];
        [self restartSystemUIServer];
    }
}
{% endcodeblock %}

方法一不需要重新启动SystemUIServer服务，但是在Lion的全屏模式下，右侧一部分会被Spotlight的图标所遮挡；方法二没有此问题，但重启SystemUIServer服务的过程会删除菜单栏上所有图标并重建，视觉效果不好。

另外，方法一在Snow Leopard下运行时，如果用户输入`killall SystemUIServer`重启SystemUIServer服务，会导致图标错位到Spotlight图标的右侧。解决方法是捕获系统的“com.apple.menuextra.added”通知（这个通知也是非公开的），调用方法一重新添加图标。

{% codeblock lang:objc %}
[[NSDistributedNotificationCenter defaultCenter] addObserver:self
                                                    selector:@selector(menuExtrasWereAddedHandler:)
                                                        name:@"com.apple.menuextra.added"
                                                      object:nil];
{% endcodeblock %}

两种方法都在Snow Leopard和Lion操作系统中通过测试。

另，Mac OS X系统内建的各种图标并非由上述方法创建，而使用了另一个非公开的类NSMenuExtra，可以按住⌘键拖动调整位置。

方法一来自[Tunnelblick](http://code.google.com/p/tunnelblick/)项目，方法二来自[Day-O](http://shauninman.com/archive/2011/10/20/day_o_mac_menu_bar_clock)作者Shaun Inman的指导，特此感谢！
