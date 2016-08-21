---
title: 开发一个Android应用之前，应该考虑点什么？
categories: Android
tags: [Android,规范,开源库] 
---
开发一个Android应用之前，应该考虑点什么？
==
![开心一刻](http://upload-images.jianshu.io/upload_images/1915184-e22b0058b534ace3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->
**写在前面：**
昨天参加了秋百万大大组织的北京**GDG**活动，收获颇丰。又跟几个有写作习惯的朋友建立了一个“神秘组织”，每两周为一个周期，每人都会产出一篇原创的文章，互相校验和探讨，意在督促组员之间学习和分享开源技术。而简书上的平台就交给我维护了，不出意外，几天之内大家就可以看到我们的主页和第一期成员的文章啦，**敬请期待~**

前几天接手了一个老员工遗留下来的代码，别提多痛苦了，甚至有了让我推倒重写的冲动，很多地方的设计问题颇多，老大也快无法忍受了。。。上学的时候写一篇作文，要想好主题立意和分段，解数学题的时候也得在草稿纸上梳理清楚逻辑之后再下笔。其实开发一个Android也应是如此，不做准备，埋头苦写，最后会导致问题颇多。所以本文会**根据网络上优秀内容和我实际的开发经验**讨论总结一下，**开发一个Android应用之前，都应该做哪些准备**。

**编码规范**
--
**编码规范**的问题是我最先想强调的，因为我接手的项目命名就极其混乱，甚至在一个类中的命名都没有统一化（生无可恋脸）。代码可能不是你自己一个人写，**保证代码可读性**是非常必要的。而规范存在的意义就是淡化每个人的习惯而达到统一。不多说，下面就介绍**Android的编码规范**。

 - 除了注释，代码中不出现中文
 - 每个类写上必要的注释，类的说明，作者，联系方式
 - 方法加上必要的注释说明，方便以后维护

**包管理**

 - base:存放基础类的包，里面的类以Base为前缀，例如BaseActivity；
 - activity:存放activity的包，每个activity命名以Activity结尾，例如MainActivity;
 - fragment:存放fragment的包，每个fragment命名以Fragment结尾，例如ChatFragment;
 - receiver:存放receiver的包；
 - service:存放service的包；
 - adapter:存放adapter的包，每个adapter命名以Adapter结尾，例如EventItemAdapter;
 - common:存放一些公共常量，例如后端接口、SharedPreferenceKey、IntentExtra等;
 - utils:存放工具类的包，比如常见的工具类：LogUtils、DateUtils；
 - entity:存放实体类的包；
 - widget:存放自定义View的包；

以上是一些常见的包，但不局限于此，视项目的具体情况而定。

**命名**

**大驼峰命名(UpperCamelCase)：每个单词的第一个字母都大写。**

**小驼峰命名(lowerCamelCase)：除第一个单词以外，每一个单词的第一个字母大写。**

**命名的基本原则:**

 - 尽可能地使用统一的命名规范；
 - 不使用汉语拼音；
 - 除了常见的英文缩写，尽量少地使用缩写；

**包命名**

 - 小写字母，参见上文包管理；
 - 连续的单词直接连接起来，不使用下划线；

**Java类命名**

 - 大驼峰命名 UserListAdapter；
 - 除常见的缩写单词以外，不使用缩写，缩写的单词每个字母都大写 RequesURLList；
 - 公共的工具类建议以Utils、Manager为后缀，如LogUtils；
 - 接口命名遵循以上原则，以able或ible为后缀；

**变量命名**

 - 成员变量命名
  - 小驼峰命名；
  - 不推荐使用谷歌的前面加m的编码风格（如果使用团队中使用m，则统一使用）；
 - 常量命名
  - 单词每个字母均大写；
  - 单词之间下划线连接；
 - 控件变量命名
  - 小驼峰命名；
  - 建议使用 控件缩写+逻辑名称 格式，例如 tvPostTitle、etUserName；
  - 对应的控件的id的命名控件缩写_逻辑名称，单词均小写，用下划线连接，例如：tv_post_title、
  - et_user_name；
 - 常见的控件缩写如下：
控件 缩写
Linearlayout	ll
RelativeLayout	rl
TextView	tv
EditText	et
Button	btn
ImageView	iv
CheckBox	chb
ListView	lv
GridView	gv
RadioButton	rb

**方法命名**

 - 小驼峰命名；
 - Getter和Setter方法，推荐使用自动生成的，写起来也很方便。注意，bool类型的变量Getter方法写成isTrue这种；
 - 方法名应当保证见名知义的原则，尽量不使用or或者and，遵循“do one thing”原则；

**布局文件命名**

 - activity、fragment布局文件名以对应的类别名称为前缀，逻辑名称放在其后，以下划线连接，例如activity_home、fragment_chat_list，方便查找；
 - ListView、GridView的item布局文件建议以list_item、gird_item为前缀，加上对应的逻辑名称，例如
   list_item_post、grid_item_photo；
 - Dialog的布局文件以dialog为前缀，逻辑名称放在其后，下划线连接，例如dialog_warnning;
   包含项布局命名以include开头，在加上对应的逻辑名称，例如include_foot
 - 控件的id命名参见控件变量命名；

**资源命名**

 - 图标资源以ic为前缀，例如ic_chat，指聊天图标；
 - 背景图片以bg为前缀，例如bg_login，指的是登录页的背景图；
 - 按钮图片以btn为前缀，例如btn_login，指的是登录按钮的图片，不过这只有一种状态，需要加上状态的可以在后面添加，例如btn_login_pressed，表示登录按钮按下的图片；
 - 当使用shape和selector文件为背景或者按钮时，命名参照以上说明；

**项目架构**
--

**项目框架**
一个好的项目架构可以降低项目的复杂性，并且易扩展、易维护，耦合低，结构清晰，功能内聚。
**MVC**，**MVP**，**MVVM**还有一些设计模式，结合你app的业务，自行选择一个最合适的一条路走到黑，建议不要“混搭”。

**开源框架**

开源框架的选择分两大类，一是权威性毫无争议，放心选择的这类帮助我们快速开发的，比如：

 - 网络 Retrofit + OkHttp+ RxJava、Volley、android-async-http
 - 依赖注入 Dagger2、ButterKnife、RoboGuice
 - 事件总线 otto、EventBus
 - 图片加载 Fresco、Glide、Picasso
 - 数据库 GreenDao、Ormlite、LitePal
 - 日志输出 logger、LogUtils
 - 当然不能忘了Google提供的兼容support全家桶

还有一类框架虽然不是毫无争议，但是也能极大节省我们的时间，提升开发效率，对于这类项目，要在Github上长期**淘宝**了。找那种star很多，issues解决很快，长期维护的项目，这种坑会比较少。

**代码细节**
--
**抽取基类**
即使目前你没有需要，也一定要抽取一个BaseActivity和BaseFragment，因为早晚会用到，切记！
比如：

```
public abstract class BaseActivity extends Activity{

    @Override
    public void onCreate(Bundle savedInstanceState, PersistableBundle persistentState) {
        super.onCreate(savedInstanceState, persistentState);
        init();
    }

    private void init() {
        setContentView(R.layout.activity_main);
        initView();
        initData();
        initListener();
    }

    /**
     * 初始化布局
     */
    protected abstract void initView();

    /**
     * 初始化数据
     */
    protected  abstract void initData();

    /**
     * 初始化侦听
     */
    protected abstract void initListener();

}
```
BaseFragment也同理，随着业务的增加和抽象，基类会继续扩展。其实不仅仅是**BaseActivity**和**BaseFragment**，如果你一个业务模块中有很多相似和相同的逻辑，也可以抽取一个**BaseXXX**这是非常好的一个习惯。

还有一些建议，不做展开，大家请自行搜索：

 - Android Studio上好用的插件
 - selector
 - 图片的.9处理
 - Resources xml文件中，记得用注释分割每个类用到的资源，建议不共用
 - 慎用static关键字
 - 定期code review，不断代码重构

性能优化
--
推荐一个我曾经写过的总结：

[Android性能优化总结](http://www.jianshu.com/p/be05874965d4)

**写在后面：**

本文做了一个初步总结，未来会慢慢扩展和完善，如果有遗漏，大家可以补充~