---
title: 如何在你的应用中正确使用Context
categories: Android
tags: [Android,Context,应用,技巧] 
---
## **如何在你的应用中正确使用Context** ##
**写在前面：**
**Context对象在我们的项目中实在是太常见了，启动Activity、Service、发送一个Broadcast，作为获取各种系统Resources的参数，Layout Inflation的参数，show a Dialog的参数等等。Context对象的使用不当，是可能造成内存泄漏的，当你的工程代码已经达到十几万行甚至是几十万行时，Context对象就对内存泄漏造成非常可观的影响了，所以我们应该对Context对象的使用，做到心中有数。**
<!--more-->
零：什么是Context
--

前两天刚刚对 Context 写了一篇比较长的博客，借鉴大牛们的经验，对 Context 进行了比较详细的整合与总结，花半个小时的时间耐心读一读吧！

[你足够了解 Context 吗？](http://www.jianshu.com/p/46c35c5079b4)

**一句话总结：**

Context是为一个Android程序提供各种功能、资源、服务的一个**环境**， Context 的资源在系统中**只有一套**，因为它的子类（**Application、Activity、Service**）对这同一块资源处理方式的不同，让Context 对象在功能上表现出各自之间的**差异**。

一：Context对象之间的差异
--

相信如果你是一个初学者， Context 在你手里应该是胡乱传入的，哪里有 Context 就找哪里，各种**this**乱入，O(∩_∩)O哈哈，至少当时我是这样的，但是 Context 不同的对象在使用功能上是有区别的，比如以下代码：

![获得Application的Context](http://upload-images.jianshu.io/upload_images/1915184-c31701faeb4510d3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在清单文件中做以下配置：

![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-ad3555c790c9d71e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在界面中show a Dialog

![show a Dialog](http://upload-images.jianshu.io/upload_images/1915184-6fcffee9f32d9928?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**点击按钮之后崩溃信息**

![崩溃信息](http://upload-images.jianshu.io/upload_images/1915184-94dea281679efa6e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当我使用**Application**的**Context**时，是无法弹出一个Dialog的，因为Dialog作为一个View，依附在Activity上，并且与Theme相关，当传入参数为**Actvity**的**Context**时，崩溃就解决了。

下面这张表展示出了Context对象之间使用上的**差异**：

![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-b1121435e2d90f71?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

二：Context相关的内存泄漏问题
--
在讨论内存泄漏之前，先简单的说说Android中**内存的回收**

Dalivik虚拟机扮演了常规的垃圾回收角色，为了GC能够从App中及时回收内存，我们需要时时刻刻在适当的时机来释放**引用对象**，Dalvik的GC会自动把**离开活动线程的对象**进行回收。

什么是**Android内存泄漏：**

虽然Android是一个**自动管理内存**的开发环境，但是垃圾回收器只会移除那些已经**失去引用的**、**不可达的**对象，在十几万、几十万行代码中，由于你的失误使得一个本应该被销毁的对象仍然被错误的持有，那么该对象就永远不会被释放掉，这些已经没有任何价值的对象，仍然占据聚集在你的堆内存中，GC就会被**频繁触发**，多说几句，如果手机不错，一次GC的时间**70**毫秒，不会对应用的性能产生什么影响，但是如果一个手机的性能不是那么出色，一次GC时间**120**毫秒，出现大量的GC操作，我相信用户就能感觉到了吧。这些无用的引用堆积在**堆内存**中，越积越多最终导致Crash，有关一些性能优化推荐给大家一个我总结的博客。

[Android性能优化总结](http://www.jianshu.com/p/be05874965d4)

有些跑题了，我们赶紧来看看什么情况下**Context会引发内存泄漏**

 - **错误的单例模式**


![错误的单例模式](http://upload-images.jianshu.io/upload_images/1915184-7382d3977caa4986.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们来分析一下这个**非线程安全的单例模式**，假设你在**Activity A**去getInstance获得**instance**对象，顺手传了一个**this**，好了，现在一个常驻内存的**Singleton**保存了你传入**Activity A**的对象，并且一直持有Activity A的**引用**，这样即使你Activity被销毁掉，但是因为它的引用还存在于一个Singleton中，是不可能被**GC**掉的，这样就导致了内存泄漏。

 - **View持有Activity的引用**

![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-58d7b888b311e762?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再来分析一下，有一个静态的Drawable对象，当我给ImageView设置这个Drawable时，ImageView像上面那个例子一样，保存了这个mDrawable的引用（大家可以点开源码**705行**去看，很多UI组件都是统一的操作，一直持有传入的对象），然而ImageView传入了**this**，也就是ImageView同样持有一个MainActivity的**mContext**。因为被**static**修饰的mDrawable是常驻内存的，MainActivity是它的间接引用，所以当MainActivity被销毁时，也不能被GC掉，所以也造成了**内存泄漏**。

三：使用Context的正确姿势
--

通俗一点说，Context造成的内存泄漏，几乎都是当Context销毁的时候，却还被各种不合理、无端地引用着。那么哪个Context对象是不会被销毁的呢？对了，Application的Context对象可以理解为随着进程存在的，所以当Application的Context能搞定的情况下，并且生命周期长的对象，优先使用**Application的Context**

![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-97aa372ce34d7455?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

调用一行代码：

> LaucherApplication.getContext();

回头看看上面那张表格，显然Application的Context不是万能的，涉及**UI**加载操作时，似乎我们只能使用Activity的Context，所以你当你使用Activity的Context时，你要对持有Activity的对象心中有数，**保证它能随着生命周期的销毁而被回收**，慎用static关键字，不要因为方便访问就各种static乱入。

多说一点，上表中**Layout Inflation**中只能使用**Activity**的Context，而各种View在创建时，需要传入的Context参数也是Activity的，大家懂了吧，当解析XML文件的时候，传入的参数也就统一了，相信大家一定能想明白这点。

**写在最后：**


**给大家推荐一个内存检测的自动化工具，LeakCanary，但是当你曾经写出的代码不规范不负责，已经达到十几万行，几十万行的时候，再去抽丝剥茧试图解开已经打上层层死结的引用关联，是非常难的。所以平时还是要注意下细节哈~**

**如果大家觉得喜欢有价值，就关注我，点下赞哈，你们的支持是我持续原创的动力。**