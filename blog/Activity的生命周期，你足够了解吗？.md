---
title: Activity的生命周期，你足够了解吗？
categories: Android
tags: [Android,Activity,生命周期] 
---
Activity的生命周期，你足够了解吗？
==
**写在前面：**
对于Activity的生命周期，相信只要已经接触过Android的同学，一定可以说出个大概，因为Activity的生命周期真的是太重要的机制了。不过在开发中，我们在每个生命周期方法应该做些什么，还有一些比较关键的知识细节也许你还不清楚，所以本文会带着大家再来探寻一次Activity的生命周期。
![开心一刻](http://upload-images.jianshu.io/upload_images/1915184-ce9b7e9c5b9796ce?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->
最近又到了校招的季节，假设你在面试的时候遇到了一个**穿着拖鞋、衣衫褴褛**的Android面试官，拿着你的简历，眉头深锁，对着简历上面**“精通Activity的生命周期和启动模式”**的这句话，问了你以下几个问题：

 - onSaveInstanceState方法在Activity的哪两个生命周期方法之间调用？

 - 弹出一个Dialog时，onPause会调用吗？什么情况下会，什么情况下不会？

 - 横竖屏切换的时候，生命周期方法是如何调用的？如何进行配置呢？

 - Activity调用了onDestory方法，就会在Activity的任务栈消失吗？

 - 永久性质的数据，应该在哪个生命周期方法中保存？

 - 在onCreate或者onRestoreInstance方法中恢复数据时，有什么区别？

如何这些问题你都能回答出来并且懂得原理的话，好吧，你可以点击浏览器的右上角了~如果并没有**完完全全理解Activity的生命周期**，那么继续往下看。

Activity生命周期的回调意义
-----------------

**直接上图：**

![Activity的生命周期方法](http://upload-images.jianshu.io/upload_images/1915184-eb9b1e13d3636e78?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

相信找张图你百分百看过，我来简略的介绍一下Activity的生命周期：

 1. **onCreate和onDestory** 
 分别代表了一个Activity的**创建和销毁**、第一个生命周期和最后一个生命周期回调，期间包裹了一个**完整**(entire lifetime)的Activity生命周期。
 
 2. **onStart和onStop**
 分别代表了Activity已经处于**可见状态和不可见状态**，此时的Activity未处在前台，**不可以与用户交互**，可多次被调用，期间Activity处于可见(visable lifetime)状态。
 
 3. **onResume和onPause**
分别代表了Activity已经进入前台获得焦点和退出前台失去焦点，此时的Activity是可以和**用户交互的**，可多次被调用，期间的Activity处于前台(foreground lifetime)状态。

 4. **onRestart**
 表示Activity正在重新启动，正常状态下，Acitivty调用了onPause--onStop但是并没有被销毁，重新显示此Activity时，onRestory会被调用。

好了在毫无新鲜感的痛苦中，上面人尽皆知的Acitivty生命周期方法终于被我介绍完了~本文不再去写Demo来分析比如A和B相互启动，或者点击Back或者Home键时，Activity生命周期方法的调用，因为我相信你如果**真正理解了上面的Activity的生命周期方法的含义**，这些都可以分析出来。

每个生命周期方法都应该干点啥？
---------------

 1. **onCreate**
 这个方法是在Activity的此生中第一次也会是唯一一次调用，所以在这个方法中，我们应该去初始化一些总体资源比如**setContentView**或者加载一些关于这个Activity的**全局数据**。
 
 2. **onStart**
 这个生命周期方法会被重复调用中，也可以加载一些当Activity可见时，才需要加载的数据，或者注册一个广播，监听UI的变化来刷新界面。
 
 3. **onResume**
 当Activity获取焦点时，这个方法会被回调，**十分轻量级**，最好做一些轻量级的数据加载和布置，这些数据的变动应该是处在onResume---onPause这个生命圈之内的。
 
 
 4. **onPause** onPause方法是我绝对想跟大家强调的一个方法，首先onPause方法绝对不可以进行**太耗时的操作**，或者一些**重量级的释放操作**，因为这会影响下一个Activity进入前台与用户交互。也就是说，只有onPause方法调用完毕，下一个Activity的onStart才会调用。
    在一些永久数据保存上，找到了这样的一段描述：
    
    > onPause(), onStop(), onDestroy() are "killable after" lifecycle methods. This indicates whether or not the system can kill the
    process hosting the activity at any time after the method returns,
    without executing another line of the activity's code. Because
    onPause() is the first of the three, once the activity is created,
    onPause() is the last method that's guaranteed to be called before
    the process can be killed—if the system must recover memory in an
    emergency, then onStop() and onDestroy() might not be called.
    Therefore, you should use onPause() to write crucial persistent data
    (such as user edits) to storage. However, you should be selective
    about what information must be retained during onPause(), because
    any blocking procedures in this method block the transition to the
    next activity and slow the user experience.
    
    翻译过来就是：无论出现怎样的情况，比如程序突然死亡了，能保证的就是onPause方法是一定会调用的，而onStop和onDestory方法并不一定，所以这个特性使得onPause是**持久化相关数据**的最后的可靠时机。
    
    所以比如你要写入数据库的数据，就可以放到onPause保存，关于一些其他细节，推荐大家看一看我以前总结的一篇文章：
    

 5. **onStop和onDestory**
  在onStop和onDestory我们通常都会做一些**释放资源相关**的操作，当然这个资源的释放是与onStart和onCreate相匹配的，毕竟他们是成对出现的。

画一张简单的图来表达含义：

![用法解析](http://upload-images.jianshu.io/upload_images/1915184-f3a075db34bdaf6f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

PS：Activity被翻译成活动，其实不如翻译成**“界面”**更容易贴切，所以与界面展示相关资源的初始化，可以放到Activity中进行。而有些**重量级**的资源，与**界面无关的**，并且与**进程同生共死**，可以考虑放在**Application中的onCreate**方法中进行初始化。
开始填坑
----

觉得大家应该比较容易理解以上的内容，那么就开始把文章开头问题的坑给填上吧~

**onSaveInstanceState方法在Activity的哪两个生命周期方法之间调用？**

其实onSaveInstanceState方法与onPause方法的调用顺序没有先后之分，你需要记住的是，onSaveInstanceState一定在**onStop方法之前**调用。

**弹出一个Dialog时，onPause会调用吗？什么情况下会，什么情况下不会？**

首先，如果你弹出的是本Activity的Dialog，并不会有任何生命周期方法调用。你肯定不服并且说：应该是onPause方法调用，明明Activity不可点击了嘛！
我想说的是，Dialog是一个**View**，它本身就**依附在Acitivty**上，可以理解为是**属于**本Activity的，**所以它的焦点也自然是本Activity的焦点**，自然不会有什么生命周期方法调用了。
如果**其他Activity**的Dialog弹出了，onPause才会调用。

**横竖屏切换的时候，生命周期方法是如何调用的？如何进行配置呢？**

横竖屏切换时，如果不做任何配置，生命周期方法的回调顺序为：

> onPause--onSaveInstanceState--onStop--onDestory--onCreate--onStart--onResume

也就是说Activity被销毁并重建了。如果不想这样可以在清单文件中的Activity添加一行配置：

```
android:configChanges="keyboardHidden|orientation|screenSize"
```
此时再切换横竖屏，就不会销毁并再次创建了。


**在onCreate或者onRestoreInstance方法中恢复数据时，有什么区别？**

当onRestoreInstance调用时，其中的bundle参数必然不为空，而使用onCreate恢复数据时，bundle也许是空的，所以要进行一下非空判断。

**Activity调用了onDestory方法，就会在Activity的任务栈消失吗？**

什么时候onDestory会被调用呢？

可以分为两大类嘛：1.点击了back键 2.Activity被意外销毁

点击back键相当于调用了finish()方法，通常来说finish
方法会至少有两个目的，一是将Activity从返回栈中退出，二是调用onDestory方法（未必是及时调用）
而直接调用onDestory，是不会将Activity在任务栈中清除的。

**写在后面：**
不知不觉本文就要结束了，Android是一个**复杂而有机的整体**，关于Activity 的生命周期方法，可继续研究的知识非常多，在未来会给大家从源码上分析Activity 的创建流程。