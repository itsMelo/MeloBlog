---
title: Android的性能优化总结
categories: Android
tags: [Android,性能优化,经验] 
---
Android的性能优化
--
**写在前面：**
        
      公司给了我一周的时间去学习Android性能的优化，参考了张明云老师的一片文章，并且用公司的实际项目进行测试（附有截图），还进行了一些知识点，注意事项以及很多网址链接的补充，希望这篇博文能让做性能测试的朋友们少走一些弯路。
      文中没有贴出大段代码，但是几乎所有的知识点都有链接，点进去就能看你想看的。转载注明出处。
<!--more-->
**零：性能指标**
--

 1. **布局复杂度**：布局复杂会导致布局需要更长的时间，从而导致进入应用慢、页面切换慢；
 2. **耗电量**：耗电量大会导致机器发热、缩短机器的有效使用时长；
 3. **内存**：内存消耗大会导致频繁GC，GC时会暂停其它工作，导致页面卡顿；内存泄露会导致剩余可用内存越来越小；内存不足会导致应用异常；
 4. **网络**：频繁的网络访问会导致耗电和影响应用的性能；网络交互数据大小会影响网络传输的效率；
 5. **程序执行效率**：糟糕的代码会严重影响程序的运行效率，UI线程过多的任务会阻塞应用的正常运行，长时间持有某个对象会导致潜在的内存泄露，频繁的IO操作、网络操作而不用缓存会严重影响程序的运行效率。
####**一：布局复杂度的优化**
**关于布局的优化，主要分两个大方向**
##### **1. 实现相同界面效果并且层级结构相同时，选用何种Layout最好**
在Android中单独的布局性能：

```
FrameLayout>LinearLayout>RelativeLayout

```
可供参考的网址：[LinearLayout与RelativeLayout的性能比较](http://www.jianshu.com/p/8a7d059da746)
**总结:**

 - RelativeLayout会让子View调用2次onMeasure，LinearLayout 在有weight时，也会调用子View2次onMeasure;

 - RelativeLayout的子View如果高度和RelativeLayout不同，则会引发效率问题，当子View很复杂时，这个问题会更加严重。如果可以，尽量使用padding代替margin;
 
 - 在不影响层级深度的情况下,使用LinearLayout和FrameLayout而不是RelativeLayout;

 - 使用组合控件性能要好于两个独立控件，比如一个文本旁边有一个图片，这中情况最好要用DrawableLeft的这种属性设置图片;

##### **2. 减少布局的层级结构**

 - HierarchyViewer---可查看布局层次结构，View绘制时耗时。
[HierarchyViewer的使用](http://blog.csdn.net/xyz_lmn/article/details/14222975)
 - 无线UIViewer---强烈推荐App工具，可在手机端直接实现HierarchyViewer的功能，查看任意界面的UI布局。
[无线UIViewer下载](http://download.csdn.net/detail/duantihi/9448886)

**测试图片如下：**
![使用无线UIViewer测试出的结果](http://upload-images.jianshu.io/upload_images/1915184-8a608fd303dd4794?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当前界面的UI布局层级如上图所示

**总结：**

 - 一些复用性很高的布局文件，比如一个App的标题栏，建议使用布局重用**include**标签，方便引入和共同管理。

 - 观察上图第三个层级和第四个层级，无论是Layout类型，还是所覆盖的坐标点，都是重合的，因为父FrameLayout作为一个Container，子FrameLayout作为一个子View的跟布局，这种情况，可使用**merge**标签进行布局层级的优化。

 - 有些在特定情况下才会出现的界面，比如联网之后，或者未必百分之百出现的界面，可用**ViewStub**标签进行懒加载，性能明显要优于加载出这个界面然后**gone**掉。

**布局优化相关网址：**
[三种优化标签的使用情景和优势---张业兴](http://blog.csdn.net/xyz_lmn/article/details/14524567)
[布局优化标签的源码分析](http://www.it165.net/pro/html/201409/22192.html)

有关布局优化的一些基础知识准备(郭霖老师的两篇博客)：
[Android LayoutInflater原理分析，带你一步步深入了解View(一)](http://blog.csdn.net/guolin_blog/article/details/12921889)
[ Android视图绘制流程完全解析，带你一步步深入了解View(二)](http://blog.csdn.net/guolin_blog/article/details/16330267)

**二：Android开发者模式—GPU过渡绘制**
--
**GPU过度绘制定义：**

如果你粉刷过一个房间或一所房子，就会知道给墙壁涂上颜色需要做大量的工作。假如你还要重新粉刷一次的话，第二次粉刷的颜色会覆盖住第一次的颜色，第一次的颜色就永远不可见了，等于你第一次粉刷做的大量工作就完全被浪费掉。这太可怕了。

同样的道理，如果在你的应用程序中浪费精力去绘制一些东西同样会产生性能问题。过度绘制这个名词就是用来描述屏幕上一个像素在单个帧中被重绘了多少次。

GPU过度绘制就指的是在屏幕一个像素上绘制多次(超过一次)，GPU过度绘制或多或少对性能有些影响。

**GPU过度绘制分析：**

过度绘制其实是一个性能和设计的交叉点。我们在设计上追求很华丽的视觉效果，但一般来说这种视觉效果会采用非常多的层叠组件来实现，这时候就会带来过度绘制的问题。我们再来看看具体显示在Android界面层级关系：

当我们来绘制一个界面时，会有一个windows，然后是建立Activity，在Activity里可以建立多个view，或view group，view也可以嵌套view。这些组件从上到下分布，上面的组件是可以被用户看见的，而在下面的组件是不可见的，但是我们依然要花很多时间去绘制那些不可见的组件，因为在某些时候，它也可能会显示出来。

**检测过度绘制：**

设置-开发者选项-调试GPU过度绘制-显示过度绘制区域(过度渲染等，不同机器可能不同)

![打开过度绘制的流程图](http://upload-images.jianshu.io/upload_images/1915184-bf7ba57ad4a13785?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**测试的颜色标识含义：**

![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-2d6386707155cca9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**项目测试截图：**

![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-ac12fc0ef9ab361b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-7d0b5cec974ef972?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-3e54c1605d23c069?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到项目中并不存在太大问题，有关减少过度绘制的一些建议：

 1. 太多重叠的背景
    
     
    这个问题其实最容易解决，建议前期在设计时尽量保持整体背景统一，另外开发可以检查你在布局和代码中设置的背景，有些背景是被隐藏在底下的，它永远不可能显示出来，这种没必要的背景一定要移除，因为它很可能会严重影响到app的性能。
 2. 太多重叠的view
    
     
    第一个建议是：使用ViewStub来加载一些不常用的布局，它是一个轻量级且默认不可见的视图，可以动态的加载一个布局，只有你用到这个重叠着的view的时候才加载，推迟加载的时间。第二个建议是：如果使用了类似viewpager+Fragment这样的组合或者有多个Fragment在一个界面上，需要控制Fragment的显示和隐藏，尽量使用动态地Inflation view，它的性能要比SetVisiblity好。
    
 3. 复杂的Layout层级
    
      这里的建议比较多一些，首先推荐用Android提供的布局工具Hierarchy
**三：Android中耗电量的测试**
--
[深入浅出Android App耗电量统计](http://www.cnblogs.com/hyddd/p/4402621.html)

测试截图：

![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-b9ad7e49225531eb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本人认为这一点没有过多补充的，大多数App都不会消耗过多的电量。

**四：内存、CPU、GPU**
--

应用运行时内存使用情况查看：Android Studio—Memory/CPU/GPU

通常这种测试应该使用一个自动化工具（monkey）去不停的点击App，或者切换界面，来观察内存、cpu的情况。

 1. **内存**
    
    **测试截图：**
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-61398ca5ea9cecbe?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    在地图界面不断地刷新，正常的内存成锯齿状分布。
    
    需要注意的情况：
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-e9d03f898bcfcfdb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    出现了针状分布，说明内存发生了突变，如果内存峰值不能降下来，就说明出现了内存溢出，就值得引起我们的关注了。
 2. **CPU**
    
    测试图片：
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-4b31f6ced46442ab?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 

 1. **GPU**
    
    Android Studio 1.4增加一项新功能：分析GPU渲染功能。作者详细讲解这一新功能的分析方法。
    
    在GPU选项卡下，可以在屏幕上看到图形化显示的渲染每帧所花费的时间。图形中每条都表示被渲染的一帧。颜色表示进程的不同周期：
    
    **绘画（蓝色）**
    表示View#onDraw()方法。那部分建立/更改DisplayList对象，然后转换成GPU能够理解的OpenGL命令。高的条形可能是视图复杂，而要求更多的时间绘制它们的显示列表，而许多视图在短时间内就失效了。
    
    **准备（紫色）**
    在Lollipop中，加入另一个线程，以帮助UI线程渲染更快。这个线程叫：RenderThread。它的责任是转换显示列表为OpenGL命令，再发送给GPU。这样在渲染过程中，UI线程可以开始处理下一个帧。这时UI线程将所有资源传送给RenderThread。如果有许多资源要传递（如许多/繁重显示列表），这一步可能需要较长时间。
    
    **处理（红色）**
    执行显示列表产生OpenGL命令。由于需要视图重绘，如果有许多/复杂显示列表要执行转换，这一步可能需要较长时间。当视图无效或是移动时，都要要重绘视图。
    
    **执行（黄色）**
    发送OpenGL命令到GPU。由于CPU发送这些缓存的命令到GPU，并期待收回干净缓存，这就阻塞调用了。缓存数量有限，并且GPU也很忙
    ——
    CPU会发现自己必须先等待缓存释放。因此，如果在这一步我们见高的条形，就可能意味着GPU在绘制UI时非常忙，这个绘制在短时间内太复杂了。
 
 测试截图：
![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-3c27ec3ed25d38bf?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**结论：**

可以通过切换界面，看图形的峰值和颜色去判断绘制View每个阶段所花费的时间，然后根据你的需求进行优化。

**五：程序的执行效率**
--

 1. 静态代码检查工具：Android studio—Analyze—Inspect Code…/Code cleanup… ，用于检测代码中潜在的问题、存在效率问题的代码段并提供改善方案；

 2. DDMS—TraceView，用于查找程序运行时具体耗时在哪；

 3. StrictMode：用于查找程序运行时具体耗时在哪，需要集成到代码中；


####**六：知名的三方性能优化工具**

 1. **LeakCanary** 
    LeakCanary是一个检测内存泄露的开源类库。你可以在 debug
    包种轻松检测内存泄露。强烈推荐LeakCanary，大多数公司都在使用它进行内存泄漏的测试。
    
    以下是我找到的学习资料，写的非常棒： 
    
    [LeakCanary:
    让内存泄露无所遁形](http://www.liaohuqiu.net/cn/posts/leak-canary/)
    
    [LeakCanary
    中文使用说明](http://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)
    
    具体使用请参考以上两个链接，下面给出一个测试截图，供大家直观感受其便捷和强大的功能。
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-9eac174d478347f7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-e01c35d7c95a01eb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    **结论：** LeakCanary非常直观的展现了MainActivity出现了内存泄漏，并且指出引用路径中的哪个引用是不该有的，然后修复问题。总而言之非常好用，处理内存泄漏首选的工具。
 2. **GT** GT是腾讯开发的一款APP的随身调测平台，利用GT，可以对CPU、内存、流量、点亮、帧率/流畅度进行测试，还可以查看开发日志、crash日志、抓取网络数据包、APP内部参数调试、真机代码耗时统计等等，需要说明的是，应用需要集成GT的sdk后，GT这个apk才能在应用运行时对各个性能进行检测。
    
    [GT官方网址](http://gt.qq.com/)
    
    下面是使用GT测试项目的截图：
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-3a9f005484fdfaa5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-8292f2b33b7532c5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-fa051d42cda87d9b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-deba30db8bba7cac?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    ![这里写图片描述](http://upload-images.jianshu.io/upload_images/1915184-4d539ff1d892e076?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    具体图片是什么含义，大家去点击官网去了解学习就可以，GT还是很全面好用的，慢慢发掘吧。

> 第一次写技术博客，并没有贴出大量代码，如果大家需要了解原理，点进链接去看就好了，如果有什么建议和问题大家可以给我留言。有什么好的工具和方法可以同大家一起分享。

**如果对大家喜欢请点赞收藏哈，你们的认可是我写作的动力O(∩_∩)O**