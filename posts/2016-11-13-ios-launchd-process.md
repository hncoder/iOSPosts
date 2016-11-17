---
title: iOS之初始化进程launchd
date: 2016-11-13
description: 了解初始化进程launchd。
---

了解操作系统后，我们知道，在操作系统中，内核只是一个服务提供者，而用户态中的应用程序才是系统中负责真正工作的实体。在iOS中，用户环境始于launchd，如上篇《iOS之系统启动流程》提到，进程launchd相当于Linux中的init。launchd作为第一个被内核启动的用户态进程，将负责启动系统中其它守护和代理程序。


> 守护程序（daemon）和传统的Unix概念一样，是后台服务，通常和用户没有交互，不考虑是否有用户登录进系统（OS X上）。  
> 代理程序（agent）是一类特殊的守护程序，可以和用户交互，有的程序还有GUI。


launchd由操作系统内核启动，用户没有权限去进行手动启动，但可以使用`launchctl`命令来和launchd进行交互，借此可以控制后台守护程序的启动或终止。在OS X中launchd的实例不止一个，当用户登录后，将会从系统范围中的launchd实例fork出一个属于用户范围的launchd实例。由于iOS不需要登录，所以只有一个系统范围的launchd，并且它是系统运行期间唯一不能终止的进程，当系统关闭时，它作为最后一个进程退出。 

前面提到，代理程序相当于特殊的守护程序，那么在iOS/OS X上，launchd是怎样来启动这些守护程序的呢？其原理是，launchd通过查看特定文件夹中的plist属性文件，根据这些plist文件来决定启动哪些守护程序。这几个特定的文件夹目录路径见以下表格：  

|目录 | 用途 | 
|----- | ---- | 
|/System/Library/LaunchDaemons|存放属于系统本身的守护程序plist文件|
|/Library/LaunchDaemons|存放第三方程序的守护程序plist文件|
|/System/Library/LaunchAgents|存放属于系统本身的代理程序plist文件|
|/Library/LaunchAgents|存放第三方程序的代理程序plist文件，通常为空|
|~/Library/LaunchAgents|用户自有的launch代理程序，只有对应的用户才会执行。|

如果有越狱设备，可以尝试将LaunchDaemons中某个不太重要的守护程序plist文件删除，再次启动系统，这个守护程序就不会被启动了。这点在《iOS之文件系统目录结构》中也有提到。  

前面讲到，launchd相当于传统Unix（Linux）中的init，但实际上在iOS/OS X中launchd除了承担init的职责外还负责了其它很多事情，这里不多赘述。并且，就init职责这部分，也还是有差别。具体可以参考以下对照表格：

|职责 | 传统init | launchd |
|----- | ---- | ------ |
|所有进程的祖先，PID为1|用户态第一个进程，其它进程从init fork而来。init设置的资源限制被所有后代继承。|相同。另外，launchd还会设置Mach异常端口，内核内部通过异常端口处理异常情况并生成信号。|
|支持“运行级别”|支持。运行级别：<br/>0-关闭电源<br/>1-单用户<br/>2-多用户<br/>3-多用户+NFS<br/>5-停机<br/>6-重启|<br/>不支持。只支持每个守护程序或代理程序自有的配置文件，但这和单用户模式是有区别的。|
|启动系统服务|按照词典序顺序运行/etc/rc?.d（？表示运行级别）中的每一个文件。|运行系统服务（守护程序）和用户服务（代理程序）。|
|系统服务的规范|运行服务的方式是运行shell脚本，而对于服务的内容毫不知情。|处理属性列表文件，属性列表文件中带有指定键值对。|
|退出时重启服务|init识别/etc/inittab中用于重启服务的respawn关键词。|允许守护程序或代理程序的属性列表中指定KeepAlive键。|
|默认用户|root|root，但是launchd允许在属性列表指定username键。|

iOS有相当多的守护程序，在众多守护程序当中，很有必要在这讲下的是和我们紧密相关的GUI Shell。在OS X中，它是Finder，在iOS中对应的是SpringBoard。它是用户和设备交互的第一个程序，也是用户看到的第一个图形界面程序（不考虑首次打开设备后的设置引导界面）。它的程序包放在/System/Library/CoreServices目录下。那么SpringBoard启动做了哪些事呢？  

SpringBoard在启动创建桌面UI的时候会枚举/Applications和/var/mobile/Applications目录下所有应用来设置桌面界面上的展现：一是通过应用的Info.plist来确定该应用是否将展现，以及获取其展现的应用图标；二是根据/var/mobile/Library/SpringBoard/IconState.plist文件内容来布局桌面图标。  

如果应用Info.plist文件中存在SBAppTags键并且其数组值中包含`hidden`项，表示应用已被设置为隐藏状态，那么其图标将不在桌面上出现。被隐藏起来的系统应用有DemoApp.app、iOS Diagnostics.app、Field Test.app、Setup.app以及TrustMe.app。

除了默认隐藏的系统应用，能否隐藏App Store下载应用呢？这就只能通过越狱设备，并编辑相应应用的Info.plist文件，添加SBAppTags键将其数组值里添加hidden项，才能达到目的。要取消应用隐藏，则去掉SBAppTags键即可。

另外，上述的应用图标是根据Info.plist中的CFBundleIcons属性来获取的；用户在桌面上改变了应用位置或新建了一个分组，都将修改重新保存IconState.plist属性文件。  

SpringBoard除了负责展示桌面UI，还有一个重要的职责，就是将UI事件投递到应用程序，因此，如果SpringBoard在后台被信号暂停了，前台应用也就收不到UI事件了。  

此篇提取梳理了launchd相关的一些主要特点，更多详细情况请参考《深入解析Mac OS X & iOS操作系统》。
