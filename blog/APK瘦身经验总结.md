---
title: APK瘦身经验总结
categories: Android
tags: [Android,瘦身,经验] 
---
Android APK瘦身经验总结
==
**写在前面：**
无论手机的内存有多大， 我们自然都希望一个应用的安装包能越小越好，更小的APK标示着更多地用户愿意去下载和体验。本文借鉴**张明云、胡凯**等老师的博客，对常规的APK瘦身方法进行归纳和总结并附上**资料的链接**。如果能对你有所帮助，那真的再好不过了。
<!--more-->
零：瘦身的指标
--

是什么造成了APK越来越大呢？ 先来看一张解压之后的APK的目录图：

![解压APK之后的目录](http://upload-images.jianshu.io/upload_images/1915184-48bfa4535ec6721c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

应该从哪些方面入手对APK进行瘦身呢？

 - 冗余的代码，不必要的jar包；

 - 未使用的静态资源，libs；

 - 屏幕适配时，资源的重复使用；

 - 错误地预置数据

 - native code

 - **未进行图片资源的优化与压缩（重点）**

如果看到这里感觉还没形成相应的概念，没关系，接下来对这些指标进行逐个分析，相信你很快就能明了。

一：剔除冗余代码
--

 - **Proguard**
   
   Proguard是编译时对java代码进行压缩，混淆，优化，预编译等操作的集成化工具。Proguard
   会遍历你的所有代码然后找出无用处的代码。所有这些不可达（或者不需要）的代码都会在生成最终的apk文件之前被清除掉。Proguard
   也会重命名你的类属性，类和接口，然整个代码尽可能地保持轻量级水平。
   
   [Android代码混淆规则设置](http://blog.csdn.net/fengyuzhengfan/article/details/43876197)
   
   [使用Proguard混淆Android代码](http://blog.isming.me/2014/05/31/use-proguard/)
   
   有关Proguard的配置和使用，网上有很多资料，以上两个是我认为还不错的，通过Proguard工具通常能使你的APK减小200K左右。
   
 - **AndResGuard**
   
   AndResGuard是微信客户端高级工程师 shwen 的开源项目，可以做到直接处理安装包.
   不依赖源码，不依赖编译过程，仅仅输入一个安装包，得到一个混淆包。AndResGuard的功能还是非常强大，腾讯多个产品都使用到了它，具体的源码和使用方法移步到Github上学习，已开源。
   
   [AndResGuard--github](https://github.com/shwenzhang/AndResGuard)

二：剔除无用的资源
--

 - **Lint**
   
   Proguard它只会对代码进行分析，对比如图片之类的资源就束手无策了，不过这个时候Lint就可以发挥非常大的作用了。
   
   Lint 一个静态的代码分析器，你只需通过调用 **./gradlew
   lint**这个简单地命令它就能帮你检查所有无用的资源文件。它在检测完之后会提供一份详细的资源文件清单，并将无用的资源列在“**UnusedResources:Unused resources**” 区域之下。只要你不通过反射来反问这些无用资源，你就可以放心地移除这些文件了。
   
   [Android Studio中Lint的使用](http://waychel.com/shi-yong-android-studiode-lintqing-chu-wu-yong-de-zi-yuan-wen-jian/)

值得注意的是，Lint 会分析资源文件(比如 /res 文件夹下面的文件) ，但是会跳过 **assets** 文件 ( /assets 文件夹下面的文件)。事实上 assets 文件是可以通过它们的文件名直接访问的，而不需要通过Java引用或者XML引用。因此，Lint 也不能判定某个 asset 文件在项目中是否有用。这全取决于开发者对这个文件夹的维护了。所以你要去人工排查一下assets文件夹下有没有无用的资源，如果有就移除它。

三：对资源文件进行取舍
--

其实这步操作更多的针对是**屏幕适配**的知识，曾经听说一个开发者对Android所有屏幕密度下的文件夹都提供了一套图片资源（ldpi, mdpi, tvdpi, hdpi, xhdpi, xxhdpi and xxxhdpi），这是非常不理智的行为，虽然Android支持这么多的屏幕密度，但是不代表你需要为每一个都提供一套资源，下面对资源取舍的一些**建议**：

 - 尽量使用一套图片资源，对于一些图片在不同分辨率手机上变现差异过大的情况，再考虑去相应文件夹下放入这个特定的图片

 - 使用一套图、一套布局，多套dimens.xml文件，在使用最小资源的情况下搞定多分辨率适配

 - 尽量重用图片，例如对称的图片，只需要提供一张，另外一张图片可以通过代码旋转的方式实现

 - 去除无用的库，功能上重复的库，使用更轻量级的库，比如你仅仅是想做一些网络请求，使用volley即可，没必要去引入xutils，这就要根据你项目的情况去具体分析了。

 - 错误的预置一些图片资源，有一些没必要跟随程序展示的图片就在需要加载它的时候再加载它，将程序与资源尽可能的分离。

四：对图片资源进行优化
--
好了终于来到本篇博客的重头戏，其实上对一个APK大小的优化更多的就是对图片资源的一些处理技巧，我们应该在不降低图片的显示效果、保证APK显示效果的前提下压缩图片文件的大小。

 - **tinypng**
   
   [图片压缩利器tinypng](http://leonshi.com/2015/11/02/tinypng-compress/)
   
   据官网介绍，它的原理是通过合并图片中相似的颜色，通过将 24 位的 PNG 图片压缩成小得多的 8 位色值的图片，并且去掉了图片中不必要的
   metadata（元数据，从 Photoshop 等工具中导出的图片都会带有此类信息），这种方式几乎能完美支持原图片的透明度。
   
   拖拽图片即可使用，压缩率甚至达到了70%，感叹它强大的同时，有精力的同学可以去研究研究它的**算法**
   
   ![压缩率达到了70%](http://upload-images.jianshu.io/upload_images/1915184-8ff20ea64821e1e2?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   
   [开源的tinypng插件](https://github.com/mogujie/TinyPIC_Gradle_Plugin)


**webP**

[有关webP的简介](http://blog.csdn.net/rogerjava/article/details/48783743)

WEBP 是google推出的意图改变web图片JPG、PNG、GIF三分天下局势的一种图片格式。它不仅支持无损或有损压缩、alpha通道，还支持动画演示。在同画质的情况下，webp格式图片占用体积相较于jpg图片大约减少40%，相较于无损png图片大约减少30%。具不完全统计，互联网流量中60%都产生于图片，如果能用上webp格式，网站的访问速度将会大大提升。

不过android 4.0+才原生支持webp, 但是我们的app是兼容2.3+，所以4.0以下的设备将无法看到图片。但是引入兼容的SO文件，APK也会变大，这时候自己做个取舍吧！

**写在最后：**

**其实还有一些给APK瘦身的技巧和细节，并没有一一列举出，而且有些新的优化技巧涉及到一些兼容性的问题，大家多查阅资料就能明白了。**