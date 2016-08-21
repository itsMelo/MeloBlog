---
title: Android中突发情况Activity数据的保存和恢复
categories: Android
tags: [Android,Activity,数据] 
---
Android中突发情况Activity数据的保存和恢复
==
**写在前面：**
在我们的APP使用的过程中，总有可能出现各种手滑、被压在后台、甚至突然被杀死的情况。所以对APP中一些临时数据或关键持久型数据，就需要我们使用正确的方式进行保存或恢复。
<!--more-->
突发情况都有哪些？
--
因为本文讨论的是当一些突发情况的出现时，对数据的保存和恢复。所以现在总结一下**突发情况**应该都有哪些？

 - 点击back键

 - 点击锁屏键

 - 点击home键

 - 其他APP进入前台

 - 启动了另一个Activity

 - 屏幕方向旋转

 - APP被Kill

当这些**突发情况**发生的时候，有哪些关键的方法会被调用呢？

写了一个简单的demo，我用上述的突发情况进行测试，代码中我重写了**所有Activity的生命周期方法**和**onSaveInstanceState方法**，并打印对应的log在控制台，下面是demo图：
![demo图](http://upload-images.jianshu.io/upload_images/1915184-35ac3a175abba667?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![部分代码示例](http://upload-images.jianshu.io/upload_images/1915184-f544616ba3090b78?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里就不贴出测试的过程了，直接来告诉大家**测试的结果**吧：

当我的APP处在前台，能与用户交互的情况下，出现上述的**突发事件**时，只有点击**back键**，**onSaveInstanceState**方法不会调用。其余的情况下， 该方法一律都会调用，这又是为什么呢？并且**onPause**方法是必然会调用的，这又给我们保存数据提供了怎样的思路呢？

onSaveInstanceState
--
好吧，相信当你看到本文标题的时候，你就应该想到了这个方法。因为当我们学习Android基础知识时，用onSaveInstanceState方法进行数据恢复是你必然学到过的。所以前面我营造出的一些悬念看似是失败了，不过对于onSaveInstanceState你理应知道更多知识：

 1. **何时调用：**
     
    
    > Android calls onSaveInstanceState() before the activity becomes vulnerable to being destroyed by the system, but does not bother
    calling it when the instance is actually being destroyed by a user
    action (such as pressing the BACK key)
    
    
找到了以上一段话，翻译过来就是当某个activity变得“**容易**”被系统销毁时，该activity的onSaveInstanceState就会被执行，除非该activity是被用户**主动销毁**的，例如当用户按BACK键的时候。
    
    结合我们以上的例子，其实都在说明一个词，就是**被动**。当Activity并不是由我主动点击back键而丧失焦点时，onSaveInstanceState方法就一定会调用。就例如我上述列举的那些除了点击back键的“**突发情况**”。
 2. **何地调用：** 
![何地调用](http://upload-images.jianshu.io/upload_images/1915184-81e0043a524fd228?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
   在我写的这个demo中，**onSaveInstanceState**的调用是处于**onPause**和**onStop**之间的，（下面关于Activity的生命周期方法，会讲解一些值得大家注意的），我查阅了一下资料，能保证的是onSaveInstanceState方法会在onStop之前调用，但是是否在onPause之前就不一定了。

**结论：** google工程师们对onSaveInstanceState如此设计就是让其完成对一些**临时的、非永久数据**存储并进行恢复。什么样的数据属于临时数据呢？举个例子，比如EditText中输入的内容，CheckBox是否勾选，ScrollView的滑动位置，目前视频的播放位置等等。

当我还没有自学Android时，玩着一些APP就会产生一个疑问，比如我在一个输入框中输入了大量文字没有提交或者保存。此时来了一个电话，如果退回的时候，输入框里面的文字消失了，那我可能会砸了电话，所以这个保存数据的操作，是Android开发者做的吗？

然而是不需要的，因为Android的View本身自己就实现了onSaveInstanceState方法，这些控件自己就具有保存临时数据和恢复临时数据的能力。

例如**TextView**中的部分源码：

![TextView中的实现](http://upload-images.jianshu.io/upload_images/1915184-8f86d2b28227cc2e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其他View控件都有相似的实现原理。值得一提的是，只有当你给这个wiget在xml中指定**id**时，它才具有保存数据并且恢复的能力，并且不同的wiget还不能共用这个id，否则会出现**数据覆盖**的情况。具体的源码有兴趣大家可以自己去看，这里因为篇幅的原因不再贴出，关于onSaveInstanceState我们先说这些，赶紧看看使用姿势。


onSaveInstanceState的使用姿势
--
比如我们要保存当前视频的**播放进度**，这个显然控件没有帮我们实现onSaveInstanceState，所以就只能靠自己了，代码如下所示。

![保存临时数据](http://upload-images.jianshu.io/upload_images/1915184-65b77385f772bee6?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![恢复临时数据](http://upload-images.jianshu.io/upload_images/1915184-3858067a294ecc4f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当在onCreate取出临时数据时，记得加一个**非空判断**。

看到这里，也许你认为本文该就此结束了，不过在回过头看看，我们刚才一直强调的是**临时数据**，毕竟onSaveInstanceState本身就是为临时数据服务的，但是一些**永久性质**的数据，比如**插入数据库的操作**，我们应该在什么方法中对其进行保存呢？

onPause
--
在介绍onPause方法之前，还是想聊聊Activity的生命周期方法，相信大家对它应该有了初步的了解，不过在相应的生命周期方法中，我们应该做什么操作呢？推荐给大家一篇文章，我觉得不错。

[Activity生命周期详解](http://blog.csdn.net/lonelyroamer/article/details/8927940)

关于onPause，我找到了一下关于它的特性：

> onPause(), onStop(), onDestroy() are "killable after" lifecycle methods. This indicates whether or not the system can kill the process hosting the activity at any time after the method returns, without executing another line of the activity's code. Because onPause() is the first of the three, once the activity is created, onPause() is the last method that's guaranteed to be called before the process can be killed—if the system must recover memory in an emergency, then onStop() and onDestroy() might not be called. Therefore, you should use onPause() to write crucial persistent data (such as user edits) to storage. However, you should be selective about what information must be retained during onPause(), because any blocking procedures in this method block the transition to the next activity and slow the user experience.

翻译过来就是：无论出现怎样的情况，比如程序突然死亡了，能保证的就是onPause方法是一定会调用的，而onStop和onDestory方法并不一定，所以这个特性使得**onPause是持久化相关数据的最后的可靠时机**。当然onPause方法不能做大量的操作，这会影响下一个Activity入栈。

刚才我们的测试结果还说明了一个道理，onSaveInstanceState并不是**百分百**调用的（比如点击了back键），显然一些永久性的数据，我们并不能在此中保存。

**关于本文的结论就显而易见了，我们来一句话总结一下：**

**临时数据使用onSaveInstanceState保存恢复，永久性数据使用onPause方法保存。**

下一篇准备给大家总结一下**Fragment**的数据保存和恢复，敬请期待哈~