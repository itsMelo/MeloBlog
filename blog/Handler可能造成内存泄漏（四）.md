---
title: Handler可能造成内存泄漏（四）
categories: Android
tags: [Android,handler,内存泄漏] 
---
Handler可能造成内存泄漏（四）
==
**写在前面：**
不知不觉中我们已经进行了三篇有关Android消息机制的研究，温故知新，我们先来回顾一下：
<!--more-->
[子线程为何不能更新UI（一）](http://www.jianshu.com/p/8c06b1d7ca68)

第一篇中我们探究了，在Android设计之时，为何子线程允许更新UI。官方给出的解释是由于**线程安全**（Thread Safe）问题。（当然也有一些其他方面的猜想）

[解决在子线程更新UI崩溃问题（二）](http://www.jianshu.com/p/8501d3b0c359)

第二篇中，我们总结了三种崩溃解决的办法。

 - Activity.runOnUIThread();

 - View.post();

 - Handler;

我们本着寻找最优解的思路比较了三种解决办法，发现代码的实现方式都为Handler，从而真正引出了Android消息机制，Handler。

[带着这篇去通关所有Handler的提问（三）](http://www.jianshu.com/p/fad4e2ae32f5)

第三篇文章是我们这个系列的重头戏，我用一个还算生动的故事，为大家解释了Handler和相关核心类的关系，推荐阅读。

OK，回顾之后，我们正式来开始本篇文章。

内存泄漏是怎么一回事？
--

可以看到，标题中有一个显眼的名词，就是**内存泄漏**，我想有必要给初学的朋友们讲讲何为内存泄漏。

在讨论内存泄漏之前，先简单的说说Android中**内存的回收**

**Dalivik**虚拟机扮演了常规的垃圾回收角色，为了**GC**能够从App中及时回收内存，我们需要时时刻刻在适当的时机来释放**引用对象**，Dalvik的GC会自动把离开**活动线程**的对象进行回收。

什么是**Android内存泄漏**：

虽然Android是一个**自动管理内存**的开发环境，但是垃圾回收器只会移除那些已经失去引用的、不可达的对象，在十几万、几十万行代码中，由于你的失误使得一个本应该被**销毁的对象仍然被错误的持有**，那么该对象就永远不会被释放掉，这些已经没有任何价值的对象，仍然占据**聚集在你的堆内存中**，GC就会被频繁触发，多说几句，如果手机不错，一次GC的时间70毫秒，不会对应用的性能产生什么影响，但是如果一个手机的性能不是那么出色，一次GC时间120毫秒，出现大量的GC操作，我相信用户就能感觉到了吧。这些无用的引用堆积在堆内存中，越积越多最终导致Crash。

有关一些性能优化推荐给大家一个我总结的博客。

[Android性能优化总结](http://www.jianshu.com/p/be05874965d4)

扯得好像远了一点，既然明白了内存泄漏是怎么回事，那与**Handler**又有什么关系呢？

Handler造成的内存泄漏
--

参考一篇外文：

[inner-class-handler-memory-leak](http://www.androiddesignpatterns.com/2013/01/inner-class-handler-memory-leak.html)

![使用Handler时，给出的warnning](http://upload-images.jianshu.io/upload_images/1915184-cdcc17e0dcd66c8c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当我们正常使用Handler时，会给出一个**warning**提示。翻译过来就是，这个Handler类应该是静态的，否则可能造成内存泄漏。

当我们简单的使用Handler的时候，并不会踩到内存泄漏这个坑，不过当Handler作为一个**内部类或者匿名类**时，这个问题就可能发生。

在上篇中，我们知道，当Android启动之时，**ActivityThread**类中，会创建**UI**线程的**Looper**和**MessageQueue**

MessageQueue中的消息会被一个接一个处理。应用的所有事件(比如**Activity生命周期**回调方法，**按钮点击**等等)都会被当做一个消息对象放入到Looper的消息队列中，然后再被逐一执行。UI线程的Looper存在于整个应用的**生命周期**内。

当在UI线程中创建Handler对象时，它就会和UI线程中Looper的消息队列进行关联。发送到这个消息队列中的消息会持有这个Handler的**引用**，这样当Looper最终处理这个消息的时候framework就会调用Handler#handleMessage(Message)方法来处理具体的逻辑。

**在Java中，非静态的内部类或者匿名类会隐式的持有其外部类的引用，而静态的内部类则不会。**

那么，内存到底是在哪里泄露的呢？其实泄漏发生的还是比较隐晦的，但是再看看下面这段代码：

```
public class SampleActivity extends Activity {

  private final Handler mLeakyHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
      // ...
    }
  }

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // Post a message and delay its execution for 10 minutes.
    mLeakyHandler.postDelayed(new Runnable() {
      @Override
      public void run() { /* ... */ }
    }, 1000 * 60 * 10);

    // Go back to the previous Activity.
    finish();
  }
}
```
当activity被finish的时候，延迟发送的消息仍然会存活在UI线程的消息队列中，直到10分钟后它被处理掉。这个消息持有activity的Handler的引用，Handler又隐式的持有它的外部类(这里就是SampleActivity)的引用。这个引用会一直存在直到这个消息被处理，所以垃圾回收机制就没法回收这个activity，内存泄露就发生了。需要注意的是：15行的匿名Runnable子类也会导致内存泄露。非静态的匿名类会隐式的持有外部类的引用，所以context会被泄露掉。

解决这个问题也很简单：在新的类文件中实现Handler的子类或者使用static修饰内部类。静态的内部类不会持有外部类的引用，所以activity不会被泄露。如果你要在Handler内调用外部activity类的方法的话，可以让Handler持有外部activity类的弱引用，这样也不会有泄露activity的风险。关于匿名类造成的泄露问题，我们可以用static修饰这个匿名类对象解决这个问题，因为静态的匿名类也不会持有它外部类的引用。

```
public class SampleActivity extends Activity {

  /**
   * Instances of static inner classes do not hold an implicit
   * reference to their outer class.
   */
  private static class MyHandler extends Handler {
    private final WeakReference<SampleActivity> mActivity;

    public MyHandler(SampleActivity activity) {
      mActivity = new WeakReference<SampleActivity>(activity);
    }

    @Override
    public void handleMessage(Message msg) {
      SampleActivity activity = mActivity.get();
      if (activity != null) {
        // ...
      }
    }
  }

  private final MyHandler mHandler = new MyHandler(this);

  /**
   * Instances of anonymous classes do not hold an implicit
   * reference to their outer class when they are "static".
   */
  private static final Runnable sRunnable = new Runnable() {
      @Override
      public void run() { /* ... */ }
  };

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // Post a message and delay its execution for 10 minutes.
    mHandler.postDelayed(sRunnable, 1000 * 60 * 10);

    // Go back to the previous Activity.
    finish();
  }
}
```

静态和非静态内部类的区别是非常微妙的，但这个区别是每个Android开发者应该清楚的。那么底线是什么？如果要实例化一个超出activity生命周期的内部类对象，避免使用非静态的内部类。建议使用静态内部类并且在内部类中持有外部类的弱引用。