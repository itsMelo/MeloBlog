---
title: Android消息机制字典型探究（一）
categories: Android
tags: [Android,handler,思路] 
---
Android消息机制字典型探究（一）
==
子线程为啥不能更新UI？
--
![开心一刻](http://upload-images.jianshu.io/upload_images/1915184-96038570640d0e76?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**写在前面：**
看到Android消息机制这几个字眼，相信大家脑海中第一时间就浮现出了**Handler**这个单词，关于这个知识点，几乎是**面试必问的问题，重要程度不言而喻**。我曾花了大致一周多的时间去研究它，本打算将其有关的所有知识点完完全全地写出，但发现篇幅会过于冗长而影响阅读。所以准备拆分成几个知识点模块，循序善诱，一步步带领大家弄清楚**Android的消息机制**。
<!--more-->
既然没有了篇幅限制，自然可以全面的去讲一讲有关Handler的一切，我先来说说当时是怎么接触到**Handler**这个类的。

在我自学Android过程中，写了一个**访问网络请求图片并显示**的Demo，在子线程中我直接给**ImageView**设置了图片，造成了崩溃。崩溃信息如下：

![崩溃信息](http://upload-images.jianshu.io/upload_images/1915184-b562b658369b0d8c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> Only the original thread that created a view hierarchy can touch its views.

这个错误信息在字面上翻译过来就是：**只有创建视图层级的原始线程，有权利处理它的视图**。说白了，就是我们经常说的“子线程不能更新UI”。这句话体现了主线程在处理视图上具有唯一权力，**这也就是为什么主线程也可以称为UI线程**。

在解决这个崩溃问题之前，我对**Android中子线程不能更新UI**产生了非常大的好奇心。

**Google**如此设计的原因是什么呢？

 1. 表象
我们先从表象上分析一下，假设可以在子线程更新UI，会产生那些后果呢？
如果不同的线程控制同一块UI，因为时间的延时性，网络的延迟性，很有可能界面图像会乱套，会花掉。而且出了问题也非常不容易排查问题出在了哪里。从硬件上考虑，每个手机只有一个显示芯片，根本上不可能同时处理多个绘制请求)，减少更新线程数，其实是提高了更新效率。
 2. 本质
如果可以并发的更新UI，事实上是 “is not thread safe”的，也就是线程不安全。我们都知道，线程安全问题其实就是，不同的线程对同一块资源的调用。在更新UI的同时，会涉及context资源的调用，所以产生了线程安全问题。

**相关阅读：**

[你足够了解Context吗？](http://www.jianshu.com/p/46c35c5079b4)

所以在Android中是不允许在子线程更新UI的、

本文开了一个小头，在下一篇中，将讨论如何解决本文中的崩溃问题。**卖个关子，一共有三种方式哦，敬请期待~**