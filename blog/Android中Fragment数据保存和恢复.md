---
title: Android中Fragment数据保存和恢复
categories: Android
tags: [Android,Fragment,数据] 
---
Android中Fragment数据保存和恢复
==
**写在前面：**
上周我们总结了Activity中数据的保存和恢复，我们花两分钟来回顾一下：
[Android中Activity数据的保存和恢复](http://www.jianshu.com/p/6622434511f7)
<!--more-->
**一句话总结：**

 - 临时数据
对于**临时数据**，我们使用**onSaveInstanceState**方法进行保存，并且在**onCreate**方法中恢复。

 - 永久数据
对于**持久性数据**，我们要在onPause方法中进行存储，但是要注意，onPause方法中不能进行大量操作，会影响其他Activity进入任务栈栈顶。

ps：在Activity中弹出一个当前Activity的**Dialog**并不会有任何生命周期方法调用（以前我曾以为会调用onPause方法）。因为Dialog作为一个View本身就是属于当前Activity的，Activity并没有**失去焦点**。

ok，完成了回顾，下面来开始本篇博客：

**Fragment**在我们的项目中真的太实用和常见了，它的使用频率和数量甚至超过了Activity，所以本文目的是探究Fragment的数据保存和恢复。

在开始讲解之前，你应该对Fragment的**生命周期方法**有一定了解，推荐给大家一篇博客，我认为不错：

[Fragment生命周期方法详解](http://blog.csdn.net/wanghao200906/article/details/45561385)

准备工作做了这么多，下面我们正式开始吧！

![测试App截图](http://upload-images.jianshu.io/upload_images/1915184-84f1eaec23de028d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本文直接选用了《第一行代码》中Fragment模块的讲解例子，点击下面的按钮**分别跳转**这四个Fragment。为了方便观察，我重写了Fragment所有**生命周期方法**和**onSaveInstanceState**方法，并打印了Log。

我们目的是探究Fragment数据的保存和恢复，在这里我把它分为两大类的情况：

 1. 单个Fragment遭遇一些突发情况

 2. Fragment之间相互的切换或覆盖

在此之前，先引入一个**返回栈**的概念。
我想你应该知道返回栈是什么，并且你以前接触的应该是保存Activity的返回栈，类比Activity，Fragment返回栈其实是保存Fragment的栈结构。区别在于：**Fragment的返回栈由Activity管理；而Activity的返回栈由系统管理。**

在未修改之前，本文添加并切换Fragment的方式都是在返回栈中**仅有一个** fragment：

![添加切换fragment](http://upload-images.jianshu.io/upload_images/1915184-183cacefa4387d8e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不要心急，过一会再说怎么去在返回栈中压入多个fragment，我们先来处理**只有一个**的情况

 1. 单个Fragment遭遇突发情况
    
    仍然是用以下**突发情况**进行测试：
    
     - 点击back键
    
     - 点击锁屏键
    
     - 点击home键
    
     - 其他APP进入前台
    
     - 启动了另一个Activity
    
     - 屏幕方向旋转
    
     - APP被Kill
    
    不过与上篇博客不同的是，我们在清单文件中，给Activity做了如下配置：
    ![配置configChange](http://upload-images.jianshu.io/upload_images/1915184-99af7b0a6ca0f2e2?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    这么做的目的是当屏幕方向发生改变的时候，fragment所依附的Activity并不会重新销毁再创建，让**情况相对简单一点**。
    
    **测试结果：**
    
    当一个fragment孤零零地呆在返回栈时，它所处的情况与Activity如出一辙。类比Activity对数据的保存和恢复，我们可以对此得出结论：
    
     - 临时数据 对于**临时数据**，我们使用**onSaveInstanceState**方法进行保存，并且在**onCreateView**方法中恢复（请注意是onCreateView）。
    
     - 永久数据 对于**持久性数据**，我们要在onPause方法中进行存储。
 2. Fragment之间的相互切换或覆盖
当返回栈中保证**只有一个Fragment**，相互切换时，生命周期方法的调用是怎样的呢？例如本例中，从fragment03切换到fragment04：

![fragment03切换fragment04](http://upload-images.jianshu.io/upload_images/1915184-6d89724ceff62e0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
　　可以看到，上述的这种情况，两个fragment从创建到销毁，经历了所有的生命周期方法。
　　如果返回栈中fragment的数量为多个呢？首先在切换时，加上以下代码，保证将fragment放入返回栈中：

![addToBackStack](http://upload-images.jianshu.io/upload_images/1915184-d8961dd8db6cf972.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
　　使用addToBackStack方法，就能将fragment放入相应的返回栈中去了，从表象上来看区别在于进入其他fragment时，点击back键时，可以返回上一个fragment。这时候切换时，生命周期方法就是如何调用的呢？

![返回栈有多个fragment切换](http://upload-images.jianshu.io/upload_images/1915184-d0fdbf4ea4c1e944.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　对比这两张生命周期方法的图，能得出两个结论。
　　1.无论任务栈中fragment数量为多少，onSaveInstanceState方法都没有调用
　　2.当fragment任务栈中有多个fragment时，进入下一个fragment时，并不会销毁**fragment实例**，而是仅仅**销毁视图**，最终调用的方法为onDestoryView。
　　所以此时我们要去保存临时数据，并不能仅保存在onSaveInstanceState中（因为它可能不会调用），还应该在onDestoryView方法中进行保存临时数据的操作，源码如下：
![代码截图](http://upload-images.jianshu.io/upload_images/1915184-dbd03c5a2f1419ea?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为没有了系统提供的bundle参数，我们选择把数据保存在Arguments中，代码就不带着大家一步一步的看了，因为逻辑并不复杂，挺好理解的。通过这种方式，我们就挺容易的将**临时数据和fragment的一些状态**保存进bundle中并在需要时恢复了。

不知不觉本篇文章就要结束了，感兴趣的可以尝试当调用**ft.add()**方式去添加fragment时，生命周期方法又是怎样调用的呢？

**结束之前我们来一句话总结下本文：**
Fragment对**临时数据**的保存，仅仅依靠**onSaveInstanceState**方法是不行的，还需要在**onDestoryView**中进行相应操作，具体参考上面的代码。

Fragment中对于一些持久性的数据，仍应在**onPause**中保存。