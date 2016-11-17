---
title: iOS之系统文件目录
date: 2016-10-31
description: 介绍一下系统文件目录结构。
---

因为iOS的封闭性，很大可能上，较多的工程师做了很长时间开发也不知道设备上的系统文件目录结构到底是怎样的。数据存储时，我们也基本是通过代码访问沙盒中默认的几个文件夹，但对沙盒在设备上的全路径未必关注过。  

尽管不了解系统目录存放并不影响正常业务功能开发，但对系统目录在脑袋里有个基本印象总是好的，说不定某个细节点就可避免犯常识性的低级错误。 

查看设备上的系统目录，在Mac上安装iTools连接一个越狱设备即可。下图是截自越狱设备iPhone 5s、系统版本为8.3的文件目录。接下来，将依次介绍下系统相关的目录。

![file system](https://raw.githubusercontent.com/hncoder/hncoder.github.io/master/assets/images/ios_file_system_1.png)

## 系统根目录下一级目录

iOS源自OS X，而OS X也是使用Unix系统内核，因此其目录结构基本符合Unix系统目录结构。实际上，在根目录下包含两类目录，一类是保留的Unix传统目录，一类是iOS/OS X特有的目录。

### 保留的Unix传统目录

`./bin`：“binary”的简称，存放传统Unix命令（用户级基础功能二进制文件），如ls、ps、rm、mv等。  
`./boot`：存放能使系统成功启动的所有文件，iOS中此目录为空。  
`./dev`：“device”的简称，存放BSD设备文件。每个文件代表系统的一个块设备或字符设备，一般来说，“快设备”以快为单位传输数据，如硬盘；而“字符设备”以字符为单位传输数据，如调制解调器。  
`./etc`：“Et Cetera”的简称，存放系统脚本及配置文件，如passwd、hosts等。iOS/OS X中/etc实际指向`./private/etc`。
`./lib`：存放系统库文件、内核模块及设备驱动等，iOS中此目录为空。  
`./mnt`：“mount”的简称，存放临时文件系统挂载点，iOS中此目录为空。  
`./sbin`：“system binaries”的简称，存放Unix管理类命令（系统级基础功能的二进制文件），如netstat、reboot、fdisk、ifconfig等。  
`./tmp`：存放临时文件目录，其权限为所有人任意读写，在OS X中实际指向`./private/tmp`。  
`./usr`：存放大量工具和程序，第三方程序安装目录，其中`./usr/lib`中存放了动态链接库。  
`./var`：“variable”的简写，存放一些经常更改的文件，如日志、用户数据、临时文件等，iOS/OS X中`./var`实际指向`./private/var`。某些文件在`./Library`和`./var`都存在，比如Keychains数据、系统日志等。  

### iOS/OS X特有的目录

`./Applications`：存放系统默认预装的应用，不包括从App Store下载安装的应用。  
`./Developer`：存放与开发调试相关的文件和工具二进制文件，当设备连接Xcode时选择了“Use for Development”才会被创建。  
`./Library`：存放系统应用数据、帮助文件、文档等。  
`./System`：只包含一个名为Library的目录，这个子目录中存放了系统的绝大部分组件，如各种framework，内核模块，字体文件等。  
`./User`：用户目录，存放用户的个人资料和配置，iOS中实际指向`./var/mobile`。  
`./cores`：内核转储文件存放目录，当一个进程崩溃时，如果系统允许则会产生转储文件。  
`./private`：存放/etc、/var两个链接目录的目标目录，分别是`./private/etc和/private/var`。  


## 部分系统目录

`./var/root/Library/Lockdown`：设备激活证书存放目录。  
`./Library/Keychains`：设备系统级密码等存放目录。  
`./Library/Logs`：系统日志存放目录，`./var/logs`也指向此目录。  
`./Library/Logs/CrashReporter`：系统进程崩溃日志目录。  
`./System/Library/LaunchDaemon`：系统启动进程plist文件存放目录，若想不启动某进程，删除该目录下对应的plist的文件（操作需慎重，删除之前先备份）。  
`./System/Library/Frameworks`：公有框架（开发者可使用）存放目录。  
`./System/Library/PrivateFrameworks`：私有框架（开发者不可使用）存放目录。  
`./System/Library/CoreServices/SpringBoard.app`：桌面管理器应用，用户与系统交互的中介。  
`./var/mobile/Containers`：存放App Store应用相关文件，其中，子目录`/Bundle`存放应用可执行文件，子目录`/Data`存放应用数据。  


## 部分系统应用资料存放目录

`./var/wireless/Library/CallHistory`：存放通话记录，网络流量，使用时间等记录。  
`./var/mobile/Library/AddressBook`：存放联系人数据。  
`./var/mobile/Library/Calendar`：存放日历及提醒事项记录文件。  
`./var/mobile/Library/Maps`：存放地图搜索书签记录。  
`./var/mobile/Library/SMS`：存放短信。  
`./var/mobile/Library/Notes`：存放备忘录。  
`./var/mobile/Library/Safari`：存放Safari保存的书签等。  
`./var/mobile/Library/Mail`：存放电子邮件数据。  
`./var/mobile/Library/Preferences/com.apple.accountsettings.plist`：存放邮箱设置。  
`./var/mobile/Library/Preferences/com.apple.mobilephone.speeddial.plist` ：存放个人收藏（快速拨号）。  
`./var/mobile/Media/Recordings`：存放语音备忘录。  
`./var/mobile/Media/iTunes_Control`：存放iTunes 同步的电影，歌曲等媒体文件。  
`./var/mobile/Media/DCIM`：存放照片里面的胶卷。  
`./var/mobile/Media/PhotoData`：存放照片里面的图片(含相机胶卷的识别库缩略图等)。  
`./var/mobile/Media/Books`：存放iBooks同步的书籍。  
`./var/mobile/Media/PhotoStreamsData`：存放照片流。  


## 应用沙盒目录

下图中红线框就是某个应用的沙盒目录，应用只能访问自己沙盒目录里面的文件（某些目录文件在用户授权的情况下可访问，如系统通讯录、照片等媒体文件）。在应用开发中，如果要保存沙盒中某个文件路径，注意不要保存全路径，只能保存在沙盒中的相对路径，要不会导致路径访问错误。这是因为每次重新编译安装应用时，沙盒目录路径会改变。以下是对每个文件夹的作用进行说明：

![sandbox](https://raw.githubusercontent.com/hncoder/hncoder.github.io/master/assets/images/ios_file_system_2.png)

`Documents`：用来存放仅限于不可再生的数据文件，会被iTunes同步。  
`Documents/Inbox`：用来存放由外部应用请求当前应用程序打开的文件，会被iTunes同步。  
`Library`：用来存放默认设置或其它状态信息，除Caches子目录之外的文件会被iTunes同步。  
`Library/Preferences`：使用NSUserDefaults写的设置数据都会保存到该目录下的一个plist文件中，会被iTunes同步。  
`tmp`：用来存放应用再次启动时不需要的临时文件，该目录下的东西随时可能被系统清理掉，不会被iTunes同步。

[查看Github Page](https://github.com/hncoder/hncoder.github.io/blob/master/_posts/2016-10-31-ios-file-system-directories.md)
