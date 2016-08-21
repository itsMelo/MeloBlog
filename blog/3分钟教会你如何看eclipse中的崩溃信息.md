---
title: 3分钟教会你如何看eclipse中的崩溃信息
categories: Android
tags: [Android,eclipse,技巧] 
---
3分钟教会你如何看eclipse中的崩溃信息
==
> 本文原创，转载请注明出处，坚持长期原创博客，喜欢请加关注哦，你们支持就是我动力的源泉~

<!--more-->
**写在前面：**

前一阵花了足足一周的时间去研究了Context的源码，发布出来一篇文章，我觉得写得已经OK了却反响平平。前天写了一个解析json的教程却得到很多朋友门点赞认可。说明还有相当一部分刚刚入行的朋友们希望得到一些相对初级的知识和技巧，话不多说，**我来一点点的教大家看崩溃Log。**

> 以后准备每周出一篇入门、一篇进阶博客~

我相信很多初学者用的开发工具是Eclipse，并且很多初级书籍也不会教大家怎么去看崩溃日志，虽然不难，但是靠自己琢磨还是挺浪费时间的，我们就写一个Demo来看看吧！

![注释掉一行代码，让程序崩溃](http://upload-images.jianshu.io/upload_images/1915184-1b61f232e90373a1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**注释掉创建ViewHolder对象的代码，让程序崩溃。**

![MainActivity中展示一个ListView](http://upload-images.jianshu.io/upload_images/1915184-3bcff3ce97214b64?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这个Demo很简单，就是在MainActivity中展示一个ListView，《第一行代码》中的例子。
现在我们注释掉了创建ViewHolder对象的代码，连上手机，运行程序，看看崩溃信息吧！

![崩溃信息](http://upload-images.jianshu.io/upload_images/1915184-ad7db11573d197f7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看图片中黑色的箭头，左边的箭头指向自己的**包名过滤器**，表示只显示我这个应用的logcat，右边箭头把信息的等级过滤为**error**级别。

这时候我们进一步去**缩小**范围：

![黑框中的信息](http://upload-images.jianshu.io/upload_images/1915184-1d014a2b4b6f139c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实黑框中的信息就是我们最主要关心的，但是为什么要这两个地方的信息呢？

 - java.lang.xxxException，这个标明你的错误类型，如果没见过，用搜索工具搜一下，就能明白，在我们这个例子里是空指针异常。


 - 第二个黑框是我们**自己应用**的包名（第31行出了问题），说明这个错误就是我们自己的代码导致的，双击可以进入java代码中，后面那些android.widget开头的崩溃信息是一些牵连信息，也是可以提供参考的。

![双击进入31行](http://upload-images.jianshu.io/upload_images/1915184-be4177455837afaf?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这行报了空指针异常，分析一下，说明我们的**ViewHolder**没有创建对象。

**写在后面：**

其实本文例子中的错误并不复杂，看错误日志也是一个经验活。遇到崩溃要理清头绪，寻找错误位置，分析可能造成的原因，看得多了，也就慢慢会看了。

> 喜欢请加关注，最近应该会出产很多文章~