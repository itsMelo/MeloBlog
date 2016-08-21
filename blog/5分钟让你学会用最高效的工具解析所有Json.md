---
title: 5分钟让你学会用最高效的工具解析所有Json
categories: Android
tags: [Android,Json,技巧] 
---
5分钟让你学会用最高效的工具解析所有Json
==
> 原创博客，转载请经过本人允许，你们的点赞和关注是我长期写作的动力~

<!--more-->
如果你是一个Android开发工程师，学会解析Json字符串是你的必修课，本篇文章主要以实例的方式手把手教你怎么做，花五分钟时间阅读本篇文章你就可以学会解析所有的Json字符串啦。

**准备：**

 - json字符串
 - fastjson
 - HiJson格式化json工具

**开始教程：**

 - **fastjson：**
   
   常用工作中解析json的工具类有谷歌的GSON，jackson，fastjson，这里就不做一一比较了，博主告诉大家，fastjson就是最高效最好用的，选它就没错了。FastJson出自阿里工程师之手，是一个Json处理工具包，包括“序列化”和“反序列化”两部分，它具备如下特征：
   
    - 速度最快，测试表明，fastjson具有极快的性能，超越任其他的Java Json parser。包括自称最快的JackJson，是GSON解析速度的6倍；
   
    - 功能强大，完全支持Java Bean、集合、Map、日期、Enum，支持范型，支持自省；无依赖，能够直接运行在Java SE 5.0以上版本；支持Android；开源 (Apache 2.0)

   **下载地址：**
   
   	[fastjson jar包下载地址](http://download.csdn.net/detail/pdsyzbaozi/8199419)

 - **HiJson：**
   
   HiJson是一个将 json **字符串格式化**的工具，非常好用，让你的json字符串结构一目了然，并且可以直接复制键值，强烈推荐！
   
   ![HiJson格式化json](http://upload-images.jianshu.io/upload_images/1915184-37902f91956283a6?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   
   **下载地址：**
   
   [HiJson下载地址](http://download.csdn.net/download/u012017406/8195263)


Fastjson API入口类是com.alibaba.fastjson.JSON，常用的序列化操作都可以在JSON类上的静态方法直接完成。

```
public static final Object parse(String text); // 把JSON文本parse为JSONObject或者JSONArray 
public static final JSONObject parseObject(String text)； // 把JSON文本parse成JSONObject    
public static final  T parseObject(String text, Class clazz); // 把JSON文本parse为JavaBean 
public static final JSONArray parseArray(String text); // 把JSON文本parse成JSONArray 
public static final  List parseArray(String text, Class clazz); //把JSON文本parse成JavaBean集合 
public static final String toJSONString(Object object); // 将JavaBean序列化为JSON文本 
public static final String toJSONString(Object object, boolean prettyFormat); // 将JavaBean序列化为带格式的JSON文本 
public static final Object toJSON(Object javaObject); 将JavaBean转换为JSONObject或者JSONArray。
```
如果你从没解析过json，看不太明白没关系，现在我上面那个json字符串，手把手的教你怎么解析，学会解析这个较复杂的json串，相信其他的你也肯定也会解析了。

**json串提供给大家拿去练手**

```
{
	"status": "2000",
	"msg": "Successful!",
	"data": [{
		"details": [{
			"distance": 2847,
			"nextLat": 39.994076,
			"nextLong": 116.47764,
			"nexti": "MeloDev",
			"status": 4
		}],
		"distance": 2847,
		"imageUrl": "",
		"overview": "长期原创Android博客",
		"source": "http://www.jianshu.com/users/f5909165c1e8/latest_articles",
		"status": "SUCCESSFUL"
	}, {
		"details": [{
			"distance": 2769,
			"nextLat": 39.97691,
			"nextLong": 116.46019,
			"nexti": "MeloDev",
			"status": 4
		}],
		"distance": 2769,
		"imageUrl": "",
		"overview": "喜欢请加关注",
		"source": "http://www.jianshu.com/users/f5909165c1e8/latest_articles",
		"status": "SUCCESSFUL"
	}]
}
```

好的万事俱备，马上就开始！

把下载的fastjson的两个jar包导入libs目录下：

![导入fastjson的jar包](http://upload-images.jianshu.io/upload_images/1915184-c0e69c1f32a9b485?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在开始比较关键的一步，新建一个bean对象，去作为json解析之后的载体，代码如下：

```
public class QueryResultInfo {
	public String status;
	public String msg;
	public List<DataList> data;

	public class DataList {
		public int distance;
		public String imageUrl;
		public String overview;
		public String source;
		public String status;
		public List<DetailsList> details;

		@Override
		public String toString() {
			return "DataList [distance=" + distance + ", imageUrl=" + imageUrl + ", overview=" + overview + ", source=" + source + ", status=" + status + ", details=" + details.toString() + "]";
		}

		public class DetailsList {
			public int distance;
			public double nextLat;
			public double nextLong;
			public String nexti;
			public int status;

			@Override
			public String toString() {
				return "DetailsList [distance=" + distance + ", nextLat=" + nextLat + ", nextLong=" + nextLong + ", nexti=" + nexti + ", status=" + status + "]";
			}
		}
	}

	@Override
	public String toString() {
		return "QueryResultInfo [status=" + status + ", msg=" + msg + ", data=" + data.toString() + "]";
	}

}
```

我来告诉大家，写一个解析json之后bean对象的技巧。首先观察json格式化的结果（HiJson工具右侧视图），java代码中：

 - **数据的类型、键的名称都必须与json字符串保证一一对应** 
也就是例子中，每个变量都是以json的**键名称**命名的，不能写错，而且数据类型也必须对应，String就是String，int就是int，float就是float

 - 如果出现嵌套的数组，就写一个**内部类**，用同样的方式命名各个json字段，用List接收它，注意List的命名也得是用json中的**键名**。多层嵌套以此类推。

 - 所有变量的访问域都是**public**的。

**好了bean对象就完成了。**

我把json字符串存在了String.xml下，点击按钮之后，解析json字符串，layout文件很简单，这里就不放出了。

![MainActivity](http://upload-images.jianshu.io/upload_images/1915184-6142704e39e36afe?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，我们调用JSON.parseObject(myJson,AppInfo.class)这一行代码，我们就把json字符串的所有信息都解析到了appInfo对象中，想用什么就直接取出来就可以了。

这个json字符串相对还是复杂的，多层嵌套，所以这个你都会了，简单的你也肯定没问题了，当然fastjson的强大不止于此，如果有特殊需要，再慢慢发掘吧~！

> 喜欢请关注哦，未来要写一篇有关线程消息机制Handler的字典型博客，正在深入研究中！