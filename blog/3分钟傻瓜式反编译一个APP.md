---
title: 3分钟傻瓜式反编译一个APP
categories: Android
tags: [Android,反编译,技巧] 
---
3分钟傻瓜式反编译一个APP
==
**写在前面：**
最近工作有些忙，一段时间没更新博客了，趁着刚吃完晚饭，来更新一下~
前几天，需求上有一个功能没思路，反编译了一下同类型的APP，找到了一个关键类，问题得以解决。网络上有很多比较成熟的文章，不过我个人对于反编译这块，有些需求过剩，不够简单粗暴，所以特来介绍一个方便的工具来进行反编译操作。
<!--more-->
反编译是为了啥？
--

我们什么时候需要反编译呢？

 - 想获得目标APP的资源（图片等）

 - 有功能不会写了，参考（copy）一下同类APP

 - 某些“羞羞”的事情

前两条需求还是蛮常见的，最后一条是开个玩笑，别做**坏事**就~

准备工具
--

 - onekey decompile apk （一键反编译APK工具）
 
 - 目标APK

	[onekey decompile apk下载链接](http://download.csdn.net/download/g_bird0622/7145155)

正确姿势
--

下载工具压缩包

![下载压缩包](http://upload-images.jianshu.io/upload_images/1915184-7095016c09c79e8b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

多说一句，这个工具集成了三个反编译的工具的功能，一步到位。如果你对这三个工具各自的功能使用感兴趣，**自行搜索学习**一下。

解压到C盘根目录

![解压到C盘根目录](http://upload-images.jianshu.io/upload_images/1915184-203b3fb4209cf8dd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里强调一下，最好是放在C盘根目录下，放到别的盘反编译可能会失败。我就失败过一次，具体原因是什么不得而知~

**得到以下文件：**

![得到以下文件](http://upload-images.jianshu.io/upload_images/1915184-e3b98a03c4269c2d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将要反编译的APK放到这个目录下：

![放APK到目录下](http://upload-images.jianshu.io/upload_images/1915184-0b0cdfd317a1af22?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将apk文件**拖拽**到`_onekey-decompile-apk.bat`上

![拖拽](http://upload-images.jianshu.io/upload_images/1915184-d84f2f1dae7043a3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**然后耐心等待十几秒......**

**源代码弹出，反编译完成！**

![反编译完成](http://upload-images.jianshu.io/upload_images/1915184-378d927bd2257158?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 会在`onekey-decompile-apk`目录下生成和apk同名的目录(放置了apktools反编译出来的东西)
*   会在`onekey-decompile-apk`目录下生成和apk同名的jar文件(dex2jar反编译出来的class)

图片资源会很完整，有些代码被混淆了，不过还是能看懂个大概的~

**写在后面：**

这个工具的作用不止于此，有需要再慢慢研究吧~