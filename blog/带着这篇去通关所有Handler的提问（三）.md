---
title: 带着这篇去通关所有Handler的提问（三）
categories: Android
tags: [Android,handler,源码] 
---
带着这篇去通关所有Handler的提问（三）
==
![开心一刻](http://upload-images.jianshu.io/upload_images/1915184-5b46d5a0584720ff?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**写在前面：**
大家久等了，Melo前阵花了一周的时间去毕业旅行，所以更新就拖延了一阵，话不多说，我们来回顾一下本系列的前两篇文章的**思路**和**知识点**：
<!--more-->
[Android消息机制字典型探究（一）](http://www.jianshu.com/p/8c06b1d7ca68)

在第一篇文章中，我们总结了Android系统不允许在子线程更新UI的原因，本质上是**线程安全问题**，从而引出了Handler。

[Android消息机制字典型探究（二）](http://www.jianshu.com/p/8501d3b0c359)

在第二篇文章中，我们又分析了三种在子线程更新UI的方法，分别是：**View.post(param); Activity.runOnUIThread(param); Handler**，当我们对这三种方法的源码进一步分析发现，其实都是对Handler做了一些封装，所以本文我们就来正式全面探究有关**Handler**的知识点。

当时我去面试的四家公司，都问到了Handler的相关知识，有深有浅，所以重要程度**不言而喻**。面试官拿起你的简历，让你谈谈Handler，你仅仅在表象上回答了Android线程通信的机理，然后面试官紧接着问了你如下的几个问题：

 - Handler是属于哪个类的？

 - Handler、Looper、MessageQueue何时建立的相互关系？

 - 主线程的Looper和MessageQueue是何时创建的？
 
 - 在同一线程中，Looper和MessageQueue是怎样的数量对应关系，与Handler又是怎样的数量对应关系？

 - MessageQueue中消息为空，线程阻塞挂起等待，为什么不会造成ANR？

 - 有关Handler的内存泄漏是怎么一回事？


![一脸萌比](http://upload-images.jianshu.io/upload_images/1915184-7a0d4ec7091a39fc?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

so...光知道表象很可能是不够的，而且还给自己挖了一个坑，所以我们对于一个知识点的探寻要全面充分一点。下面正式开始本文。

Windows和Android消息机制的区别
--

现在的操作系统普遍采用消息驱动模式。Windows操作系统就是典型的**消息驱动模型**。但是，Android的消息处理机制和Windows的消息处理机制又不太相同。我给大家画了图，看看二者的区别。

![Windows进程消息模型](http://upload-images.jianshu.io/upload_images/1915184-989ad8fda07a2608?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Android进程消息模型](http://upload-images.jianshu.io/upload_images/1915184-d41410cd42ed3d8d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过消息机制图的对比，Windows消息处理模型中，存在一个系统的消息队列，这个队列是整个进程的核心，几乎所有的动作都要转换成消息，然后放到这个队列中，由主线程统一处理。

而Android没有全局的消息队列，消息队列是和某个线程相关联在一起的。每个线程最多有一个消息队列，消息的取出和处理，也在这个线程本身中完成。

也就是说，Android中，如果你想在当前线程使用消息模型，则必须构建一个消息队列，而消息机制的相关主要类是：**Looper、Handler、MessageQueue、Message。**

我们并不着急去翻看这些类的源码，理清楚底层实现的逻辑，而且先在宏观表象上看看，Android消息机制是如何运行的？

Android消息机制的宏观原理
--

先来看一张**Android消息处理类之间的关系图**

![Android消息处理机制](http://upload-images.jianshu.io/upload_images/1915184-514571cad65f7171?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们从表象上解释一下原理，Handler负责将Message发送至当前线程的MessageQueue中，Looper时时刻刻监视着MessageQueue，将符合时间要求的Message取出，再带给发送消息的那个Handler通过HandleMessage处理。

对于消息机制的理解不能仅仅停留在这一步，下面我们从源码的角度分析一下具体的逻辑细节。

Android消息机制相关类的源码分析
--

其实写这篇文章之前，我就一直在思考，站在什么角度展开这个机制的描述，更容易让大家理解接受。思来想去，我觉得还是以一个Message游历的形式去描写，会显着有趣和清晰一点。

**Message：**

人在边境X（**子线程**）服役的士兵Message慵懒得躺在一个人数为50（池中最大数量）的军营（Message池）中。不料这时突然接到了上司的obtain()命令（据说obtain命令更加节省军费），让他去首都（**主线程**）告诉中央领导一些神秘代码。小Message慌乱地整理了下衣角和帽子，带上信封，准备出发。

上司让士兵Message收拾完毕之后等待一个神秘人的电话，并且嘱咐他：到了首都之后，0是这次任务的暗号。

![Message的创建和携带信息](http://upload-images.jianshu.io/upload_images/1915184-080d1ddd6f3223a8?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Message是消息的载体，Message设计成为**Parcelable**类的派生类，这表明Message可以通过**binder**来跨进程发送。
通常我们都会用**obtain()**方法去创建Message，如果消息池中有Message有，则取出，没有，再重新创建。这样可以防止对象的重复创建，节省资源。

![obtain方法源码](http://upload-images.jianshu.io/upload_images/1915184-8982fddc999074a1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

"铃铃铃..."小Message接到了一个陌生男子的电话。
“我叫handler，来自activity大本营，是你这次任务的接受者，一会我带你去首都的消息中心去报道。”

Handler
--

来自Activity大本营Handler部门是整个消息机制系统的核心部门，当然部门下有**很多个** Handler，这次协助小Message任务的叫mHandler。Handler部门下的员工都有一个特点，就是只关心自己的message。

Handler属于Activity，创建任何一个Handler都属于重写了Activity中的Handler。

![Activity中定义了Handler](http://upload-images.jianshu.io/upload_images/1915184-39a9617e0ae54894?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在Handler的构造中，默认完成了对当前线程Looper的绑定，至于Looper是谁，一会再谈。

![Handler的构造方法](http://upload-images.jianshu.io/upload_images/1915184-52c8a9bd457f43b5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过Looper.myLooper()获取了当前线程保存的Looper实例，又通过mLooper.mQueue获取了Looper中的MessageQueue实例。在此时，mhandler实例与looper和messageQueue实例，关联上了。

mHandler神情骄傲得对小Message说：我已经跟首都的消息中心打好了招呼，准备接收你了，现在有两种车，一种车名叫“**send**”，一种叫“**post**”，你想坐哪辆去首都都可以，不过要根据你上司的命令，选择车种类下对应的型号哦~

 - **send**
![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-304ec3faab1a3176?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 - **post**
![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-0b9ad4781e0b1861?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从代码的实现上来看，post方法也是在使用send类的方法在发送消息，只是他们的参数要求是Runnable对象。

通过对Handler源码的分析，发现除了sendMessageAtFrontOfQueue方法之外，其余任何send的相关方法，都经过层层包装走到了sendMessageAtTime方法中，我们来看看源码：

![sendMessageAtTime源码](http://upload-images.jianshu.io/upload_images/1915184-fd684532b5c792e5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时小Message和mHandler一同上了车牌号为“sendMessage”的车，行驶在一条叫“enqueueMessage”的高速公路上，mHandler向一无所知的小Message介绍说，每个像他一样的Message都是通过**enqueueMessage**路进入MessageQueue的。我们是要去首都的MessageQueue中心，其实你的消息到时候也是我处理的，不过现在还不是时候哦，因为我很忙。

![enqueueMessage源码](http://upload-images.jianshu.io/upload_images/1915184-12ec385d2ec1164f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

enqueueMessage是MessageQueue的方法，用来将Message根据时间排序，放入到MessageQueue中。其中msg.target = this，**是保证每个发送Message的Handler也能处理这个Message。**

Looper
--
路上的时间不短不长，mHandler依然为小Message热心介绍着MessageQueue和Looper
“在每个驻扎地（线程）中，只有一个MessageQueue和一个Looper，他们两个是相杀相爱，同生共死的好基友，Looper是个跑不死的邮差，一直负责取出MessageQueue中的Message”
"不过通常只有首都（主线程）的Looper和MessageQueue是创建好的，其他地方需要我们人为地创建哦~"

![prepare方法](http://upload-images.jianshu.io/upload_images/1915184-bed816d7d28d1c87?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Looper类提供了prepare方法来创建Looper。可以看到，当重复创建Looper时，会抛出异常，也就是说，每个线程只有一个Looper。

![Looper构造](http://upload-images.jianshu.io/upload_images/1915184-9fe5b7b2e249c771?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

紧接着在Looper的构造方法中，又创建了与它一一对应的MessageQueue，既然Looper在一个线程中是唯一的，所以MessageQueue也是唯一的。

在Android中，ActivityThread的main方法是程序的入口，主线程的Looper和MessageQueue就是在此时创建的。

![ActivityThread的main方法](http://upload-images.jianshu.io/upload_images/1915184-2a78bc7e32ab5990?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，在main方法中，既创建了Looper，也调用了**Looper.loop()**方法。

mHandler和小Message通过enqueueMessage路来到了MessageQueue中，进入之前，门卫仔仔细细地给小Message贴上了以下标签：
“mHandler负责带入”
“处理时间为0ms”
并且告诉小Message，一定要按照时间顺序排队。
进入队伍中，Looper大哥正在不辞辛劳的将一个又一个跟小Message一样的士兵带走。

![loop方法](http://upload-images.jianshu.io/upload_images/1915184-663c57da42f084c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

分析一下loop方法，有一个for的死循环，不断地调用queue.next方法，在消息队列中取Message。并且在Message中取出target，这个target其实就是发送消息的handler，调用它的dispatchMessage方法。

首都的MessageQueue中心虽然人很多，但是大家都井井有条的排着队伍，Looper老哥看了一眼手里的名单，叫到了小Message的名字，看了一眼小Message身上的标签，对他说：“喔，又是mHandler带来的人啊，那把你交给他处理了”

忐忑不安的小Message看到了一个熟悉的身影，mHandler就在面前，显然mHandler有些健忘，可能是接触了太多跟小Message一样的人，为了让mHandler想起自己，小Message说出了上司交给他的暗号0.


![dispatchMessage方法](http://upload-images.jianshu.io/upload_images/1915184-b6daebfefccb8e0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看见dispatchMessage方法中的逻辑比较简单，具体就是如果mCallback不为空，则调用mCallback的handleMessage()方法，否则直接调用Handler的handleMessage()方法，并将消息对象作为参数传递过去。

在handlerMessage()方法中，小Message出色的完成了自己的任务。

**写在后面：**

下一篇中，我们会探讨一下为什么loop方法中for死循环不会造成ANR，有一些有关Handler的使用技巧，以及可能造成的内存泄漏，敬请期待。