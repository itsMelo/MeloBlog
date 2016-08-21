---
title: Android消息机制字典型探究（二）
categories: Android
tags: [Android,handler,源码] 
---
Android消息机制字典型探究（二）
==
![开心一刻](http://upload-images.jianshu.io/upload_images/1915184-8d34278385fceccd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**写在前面：**
[Android消息机制字典型探究（一）](http://www.jianshu.com/p/8c06b1d7ca68)
为了完成整个**Android消息机制的探究**，我准备将知识点细分成一个个模块。在连载的第一篇文章中，在子线程更新UI导致崩溃，我们去分析探究了Android中**不允许子线程更新UI**的原因，是由于**线程安全**的问题。
当然我们**目前**分析的东西和写出文字都与Android消息机制无关。不过我其实是想给大家展示学习编程，或者说学习Android的一些好的习惯和解决问题的思路，总结起来就是：**实践去发现问题，全面的理解问题，寻找最优解**。Android本身就是一个复杂而有机的整体，**由一个知识点可以牵出一条知识线。从而构成相关的知识体系**。
<!--more-->
**这种学习方式会让你知道的越来越多，也能站在一定的高度上体会Android在设计之时的巧妙，全局的理解Android，做到融会贯通。也能在你的代码中，收获很多有益的启发。**

说完以上这些，就可以正式开始本文的话题了。既然在我们只可以在主线程更新UI，那解决这个问题，一共有几种方式呢？我就来直接告诉大家，解决子线程更新UI问题的方式，**一共有三种。**

 1. **runOnUiThread**
    
    ![runOnUiThread的使用](http://upload-images.jianshu.io/upload_images/1915184-ee3a4f5a9a294b01?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    简单讲解一下代码，点击crash按钮，开启一个子线程，显然我们在子线程中直接给 **iv_handler**
    设置一张图片，是肯定崩溃的。当我们调用**runOnUiThread**方法，并且传入一个**Runnable**对象，并且在其中设置**更新UI**的逻辑，问题就解决了。相信你也和我一样对此非常好奇，那就赶紧点进去看看，源码中是如何实现的吧！
    
    ![runOnUiThread实现原理](http://upload-images.jianshu.io/upload_images/1915184-215bb46aaa0df885?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    先来翻译一下这段注释：
    
    > 在UI线程中执行该Runnable.  * 如果当前线程是UI线程,那么该Runnable会立即执行.  * 如果当前的线程不是UI线程则调用UI线程handler的post()方法将其放入UI线程的消息队列中.  *
    注意:勿在runOnUiThread(Runnable runnable)中做耗时操作
    
    首先我想说明的是，runOnUiThread方法是属于**Activity**的，也就是说我们能拿到Activity才能使用该方法。我们本文的这个例子，明显是执行了**mHandler.post(action)**方法。我们目前不去研究**handler.post**方法，因为一会你就知道为什么了。再来看看第二种解决问题的办法。
 2. **view.post** ![view.post方法使用](http://upload-images.jianshu.io/upload_images/1915184-80f462b718e89b2a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    代码跟上一个解决办法如出一辙，我们还是来看看源码，分析一下这个方法的实现方式。
    
    ![view.post源码](http://upload-images.jianshu.io/upload_images/1915184-40b751ac6dcb8c73?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    先来翻译一下注释的意思：
    
    > 将Runnable对象添加到message queue中，并且这个runnable对象会跑在UI线程中。
    
    翻译完注释再来看代码，当View和Activity完成attach操作时，会产生一个**attachInfo**参数，在attachInfo参数中取出来了属于activity的mHandler，仍然去调用了mHandler.post(action)方法。也就是说无论我们是选择第一种方法还是第二种方法去解决这个崩溃问题，都是**殊途同归**的，最后经过层层封装，都走到了**handler.post**方法中。

**handler.post**

![发送message到主线程](http://upload-images.jianshu.io/upload_images/1915184-f35b59a4217f4ad7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![handler处理信息](http://upload-images.jianshu.io/upload_images/1915184-e4f720124308ab1b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

啊哈哈，我们连载通篇的主角**Handler**今天终于正式登场了，没错上面的代码就是将子线程的消息发送到主线程并处理的标准写法。等等，post方法在哪里？别着急，在本篇文章中，我并不打算给大家展开整个Handler知识体系的研究。我们先来看看post方法调用层级

![post](http://upload-images.jianshu.io/upload_images/1915184-2d101a24320abf9c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![sendMessageDelayed](http://upload-images.jianshu.io/upload_images/1915184-2c64da2ca248d98a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![sendMessageAtTime](http://upload-images.jianshu.io/upload_images/1915184-5dc923d75bd384d4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，方法的调用顺序为post--sendMessageDelayed--sendMessageAtTime.

在本文中我们不再继续深究下去，未来我会对这些方法的调用层级、顺序、以及使用场景为大家进行一下完整地整理总结，本文不再赘述。

**想强调的是：**
其实在这篇博客的目的：
1.从解决问题并寻找方法的角度引出Handler，真正开启Handler的知识体系。

2.思考问题，解决问题的过程。回首第一篇文章中，我因为在子线程更新UI而造成了崩溃，然后：

 - 在寻找解决办法之前，带着强烈的好奇心，去寻找了为何不能在子线程更新UI原因。在这个过程中，我理解了什么**线程安全**，深入理解了**Context**。
 
 - 去寻找解决问题的所有办法（三种），并且去探究了这几种方法的原理，试图选择在本例中的最优解（事实上这几种方法本质上没区别）。

 - 将问题简单化，所有问题的解决办法都指向了Handler，所以我们只需要探究Handler即可。

**写在最后：**

很多初学者认为Handler就是为了解决子线程更新UI的问题而存在的，事实上这种理解是**错误**的。Handler作为Android的线程间通信的机制，意义远不止此。下一篇中，Melo将带大家真正的理解Android的消息机制。