---
title: 你足够了解Context吗？
categories: Android
tags: [Android,Context,源码] 
---
## **你足够了解Context吗？** ##
这里有关于Context的一切
-
**写在前面：**

> 当我还是一个24K纯Android新手的时候（现在是也是个小Android萌新），拿着工具书对着电脑敲敲打打，那个时候我就有一个非常大的疑问：Context到底为何这么牛？show一个Dialog，启动一个Activity，Service，发送一个Broadcast，还有各种方法需要传入的参数。几乎在Android中，Context的身影处处可见，所以弄懂它，似乎是一件迫在眉睫的事，所以深呼吸，整理思路，来看看Context到底是什么。

<!--more-->
零：官方定义
--
好吧如果你无法翻墙，推荐你两个可以看官网文档的网站：

[Android官方文档国内镜像站点](http://wear.techbrood.com/)

[Android中文API](http://www.android-doc.com/)

**我们来看看官方文档中，Context的解释**

> Interface to global information about an application environment. This is an abstract class whose implementation is provided by the Android system. It allows access to application-specific resources and classes, as well as up-calls for application-level operations such as launching activities, broadcasting and receiving intents, etc.

 - 一个应用环境的全局信息，字面意思是上下文的意思;
 - Context是一个抽象类;
 - 允许我们通过Context获取各种资源，服务，或者去启动一个Activity，发送一个广播，等等;

怎么去理解Context呢？其实Context就是给Android应用程序提供了一个可以实现各种操作的土壤环境，Context为Android提供了各种资源、功能、服务。如果说编写一个Android程序像搭建一座房子，那Context就为Android提供了土地，木材，和染料(启动一个Activity，弹出一个Dialog)，并且能提供呼叫各种将房屋建得更完善的其他帮助(发送一个广播，启动一个服务等)。

一：继承关系
--
![Context继承关系](http://upload-images.jianshu.io/upload_images/1915184-21051266e7de1602?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过继承关系可以看到，Context直接子类为ContextIml（实现类）和ContextWrapper（包装类）

再看看ContextWrapper的子类有什么，看到熟悉的Service和Application了吧，不过看到这里你一定有个疑问，为什么Activity和他们哥俩不在一个继承层级呢？而是Activity又继承了ContextThemeWrapper，那么ContextWrapper和ContextThemeWrapper的区别在哪里呢？

看到这两个类的名字，相信你心里已经有了答案，对，区别在**Theme**。

**该类内部包含了主题(Theme)相关的接口，即android:theme属性指定的。**
只有Activity需要主题，Service不需要主题，
   所以Service直接继承于ContextWrapper类。而Activity因为含有Theme属性的缘故，所以继承自ContextThemeWrapper。

**所以说，Context所调用的资源是不同的了？**保留这个疑问，继续向下看。
   
二：源码阅读
--
看到了继承结构，我们分别来看看Context及其子类的一些源码

 - **Context**
 

```
public abstract class Context {

    // 获取应用程序包的AssetManager实例
    public abstract AssetManager getAssets();
 
    // 获取应用程序包的Resources实例
    public abstract Resources getResources();

    // 获取PackageManager实例，以查看全局package信息	
    public abstract PackageManager getPackageManager();

    // 获取应用程序包的ContentResolver实例
    public abstract ContentResolver getContentResolver();
    
    // 它返回当前进程的主线程的Looper，此线程分发调用给应用组件(activities, services等)
    public abstract Looper getMainLooper();

    // 返回当前进程的单实例全局Application对象的Context     
    public abstract Context getApplicationContext();

    // 从string表中获取本地化的、格式化的字符序列
    public final CharSequence getText(int resId) {
        return getResources().getText(resId);
    }

    // 从string表中获取本地化的字符串
    public final String getString(int resId) {
        return getResources().getString(resId);
    }

    public final String getString(int resId, Object... formatArgs) {
        return getResources().getString(resId, formatArgs);
    }

    // 返回一个可用于获取包中类信息的class loader
    public abstract ClassLoader getClassLoader();

    // 返回应用程序包名
    public abstract String getPackageName();

    // 返回应用程序信息
    public abstract ApplicationInfo getApplicationInfo();

    // 根据文件名获取SharedPreferences
    public abstract SharedPreferences getSharedPreferences(String name,
            int mode);

    // 其根目录为: Environment.getExternalStorageDirectory()
    
    public abstract File getExternalFilesDir(String type);

    // 返回应用程序obb文件路径
    public abstract File getObbDir();

    // 启动一个新的activity 
    public abstract void startActivity(Intent intent);

    // 启动一个新的activity 
    public void startActivityAsUser(Intent intent, UserHandle user) {
        throw new RuntimeException("Not implemented. Must override in a subclass.");
    }

    // 启动一个新的activity 
    // intent: 将被启动的activity的描述信息
    // options: 描述activity将如何被启动
    public abstract void startActivity(Intent intent, Bundle options);

    // 启动多个新的activity
    public abstract void startActivities(Intent[] intents);

    // 启动多个新的activity
    public abstract void startActivities(Intent[] intents, Bundle options);

    // 广播一个intent给所有感兴趣的接收者，异步机制 
    public abstract void sendBroadcast(Intent intent);

    // 广播一个intent给所有感兴趣的接收者，异步机制 
    public abstract void sendBroadcast(Intent intent,String receiverPermission);

    public abstract void sendOrderedBroadcast(Intent intent,String receiverPermission);
 
    public abstract void sendOrderedBroadcast(Intent intent,
            String receiverPermission, BroadcastReceiver resultReceiver,
            Handler scheduler, int initialCode, String initialData,
            Bundle initialExtras);

    public abstract void sendBroadcastAsUser(Intent intent, UserHandle user);

    public abstract void sendBroadcastAsUser(Intent intent, UserHandle user,
            String receiverPermission);
  
    // 注册一个BroadcastReceiver，且它将在主activity线程中运行
    public abstract Intent registerReceiver(BroadcastReceiver receiver,
                                            IntentFilter filter);

    public abstract Intent registerReceiver(BroadcastReceiver receiver,
            IntentFilter filter, String broadcastPermission, Handler scheduler);

    public abstract void unregisterReceiver(BroadcastReceiver receiver);
 
    // 请求启动一个application service
    public abstract ComponentName startService(Intent service);

    // 请求停止一个application service
    public abstract boolean stopService(Intent service);
 
    // 连接一个应用服务，它定义了application和service间的依赖关系
    public abstract boolean bindService(Intent service, ServiceConnection conn,
            int flags);

    // 断开一个应用服务，当服务重新开始时，将不再接收到调用， 
    // 且服务允许随时停止
    public abstract void unbindService(ServiceConnection conn);
 
    public abstract Object getSystemService(String name);
 
    public abstract int checkPermission(String permission, int pid, int uid);
 
    // 返回一个新的与application name对应的Context对象
    public abstract Context createPackageContext(String packageName,
            int flags) throws PackageManager.NameNotFoundException;
    
    // 返回基于当前Context对象的新对象，其资源与display相匹配
    public abstract Context createDisplayContext(Display display);
 }  
```
Context的源码算上注释有3000行之多，这里贴出一些重要代码，可以看到，Context几乎包含了所有你能想到的，一个Android程序需要的资源和操作，Context自己就像一个App一样，启动Activity、Service，发送Broadcast，拿到assets下的资源，获取SharedPreferences等等。

但Context只是一个顶层接口啊，又是谁帮他实现了操作呢？是**ContextWrapper**吗？

 - **ContextWrapper**
  

```
public class ContextWrapper extends Context {
    Context mBase; //该属性指向一个ContextIml实例

    public ContextWrapper(Context base) {
        mBase = base;
    }

    /**
     * @param base The new base context for this wrapper.
     * 创建Application、Service、Activity，会调用该方法给mBase属性赋值
     */
    protected void attachBaseContext(Context base) {
        if (mBase != null) {
            throw new IllegalStateException("Base context already set");
        }
        mBase = base;
    }

    @Override
    public Looper getMainLooper() {
        return mBase.getMainLooper();
    }

    @Override
    public Object getSystemService(String name) {
        return mBase.getSystemService(name);
    }

    @Override
    public void startActivity(Intent intent) {
        mBase.startActivity(intent);
    }
}
```
好吧，ContextWrapper好像很懒的样子，它把所有操作都丢给了mBase，mBase又是谁呢？在构造方法和attachBaseContext方法中，指向了一个Context实例，**ContextIml**，我们赶紧来看看ContextIml的源码！

 - **ContextImpl**

```
/**
 * Common implementation of Context API, which provides the base
 * context object for Activity and other application components.
 */
class ContextImpl extends Context {
    private final static String TAG = "ContextImpl";
    private final static boolean DEBUG = false;

    private static final HashMap<String, SharedPreferencesImpl> sSharedPrefs =
            new HashMap<String, SharedPreferencesImpl>();

    /*package*/ LoadedApk mPackageInfo; // 关键数据成员
    private String mBasePackageName;
    private Resources mResources;
    /*package*/ ActivityThread mMainThread; // 主线程

    @Override
    public AssetManager getAssets() {
        return getResources().getAssets();
    }

    @Override
    public Looper getMainLooper() {
        return mMainThread.getLooper();
    }

    @Override
    public Object getSystemService(String name) {
        ServiceFetcher fetcher = SYSTEM_SERVICE_MAP.get(name);
        return fetcher == null ? null : fetcher.getService(this);
    }

    @Override
    public void startActivity(Intent intent, Bundle options) {
        warnIfCallingFromSystemProcess();
        if ((intent.getFlags()&Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
            throw new AndroidRuntimeException(
                    "Calling startActivity() from outside of an Activity "
                    + " context requires the FLAG_ACTIVITY_NEW_TASK flag."
                    + " Is this really what you want?");
        }
        mMainThread.getInstrumentation().execStartActivity(
            getOuterContext(), mMainThread.getApplicationThread(), null,
            (Activity)null, intent, -1, options);
    }
}
```
其实ContextImpl才是在Context的所有继承结构中唯一一个真正实现了Context中方法的类。其它Context的子类，Application，Activity，Service，都是与ContextImpl相关联，去获取资源和服务，并没有真正自己去实现，这里就不贴上ContextThemeWrapper的源码了，它是为Activity添加了一些Theme的属性，不再赘述。

思路越来越清晰，我们现在就是要去寻找，**Activity，Service，Application**是何时与**ContextImpl**完成绑定关联的。

三：关联时机
--
我们都知道**ActivityThread**的**main**方法，是整个**Android**程序的入口，所以去探究**ActivityThread**类，也是一件非常重要的事。

推荐一篇文章，去了解下**ActivityThread**吧

[ActivityThread简介](http://blog.csdn.net/myarrow/article/details/14223493)


贴出ActivityThread的main方法部分重要的代码

```
public final class ActivityThread {  

	static ContextImpl mSystemContext = null;  
	
    static IPackageManager sPackageManager;
    
	// 创建ApplicationThread实例，以接收AMS指令并执行  
    final ApplicationThread mAppThread = new ApplicationThread();  
  
    final Looper mLooper = Looper.myLooper();  
    
	final HashMap<IBinder, ActivityClientRecord> mActivities  
            = new HashMap<IBinder, ActivityClientRecord>();  
            
	final HashMap<IBinder, Service> mServices  
            = new HashMap<IBinder, Service>();  
            
    final H mH = new H();  

	Application mInitialApplication;  
  
    final ArrayList<Application> mAllApplications  
            = new ArrayList<Application>();  
  
    static final ThreadLocal<ActivityThread> sThreadLocal = new ThreadLocal<ActivityThread>();  
    
    Instrumentation mInstrumentation;  
  
    static Handler sMainThreadHandler;  // set once in main()  

    private class H extends Handler {  
  
        public void handleMessage(Message msg) {  
            if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));  
            switch (msg.what) {  
                case LAUNCH_ACTIVITY: {  
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");  
                    ActivityClientRecord r = (ActivityClientRecord)msg.obj;  
  
                    r.packageInfo = getPackageInfoNoCheck(  
                            r.activityInfo.applicationInfo, r.compatInfo);  
                    handleLaunchActivity(r, null);  
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);  
                } break;  
                ...  
            }  
            if (DEBUG_MESSAGES) Slog.v(TAG, "<<< done: " + codeToString(msg.what));  
        }  
         
        ...  
    }  

	private class ApplicationThread extends ApplicationThreadNative {  
  
        private void updatePendingConfiguration(Configuration config) {  
            synchronized (mPackages) {  
                if (mPendingConfiguration == null ||  
                        mPendingConfiguration.isOtherSeqNewer(config)) {  
                    mPendingConfiguration = config;  
                }  
            }  
        }  
  
        public final void schedulePauseActivity(IBinder token, boolean finished,  
                boolean userLeaving, int configChanges) {  
            queueOrSendMessage(  
                    finished ? H.PAUSE_ACTIVITY_FINISHING : H.PAUSE_ACTIVITY,  
                    token,  
                    (userLeaving ? 1 : 0),  
                    configChanges);  
        }  
  
        // we use token to identify this activity without having to send the  
        // activity itself back to the activity manager. (matters more with ipc)  
        public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,  
                ActivityInfo info, Configuration curConfig, CompatibilityInfo compatInfo,  
                Bundle state, List<ResultInfo> pendingResults,  
                List<Intent> pendingNewIntents, boolean notResumed, boolean isForward,  
                String profileName, ParcelFileDescriptor profileFd, boolean autoStopProfiler) {  
            ActivityClientRecord r = new ActivityClientRecord();  
  
            r.token = token;  
            r.ident = ident;  
            r.intent = intent;  
            r.activityInfo = info;  
            r.compatInfo = compatInfo;  
            r.state = state;  
  
            r.pendingResults = pendingResults;  
            r.pendingIntents = pendingNewIntents;  
  
            r.startsNotResumed = notResumed;  
            r.isForward = isForward;  
  
            r.profileFile = profileName;  
            r.profileFd = profileFd;  
            r.autoStopProfiler = autoStopProfiler;  
  
            updatePendingConfiguration(curConfig);  
  
            queueOrSendMessage(H.LAUNCH_ACTIVITY, r);  
        }  
  
        ...  
    }
	public static void main(String[] args) {  
        SamplingProfilerIntegration.start();  
  
        // CloseGuard defaults to true and can be quite spammy.  We  
        // disable it here, but selectively enable it later (via  
        // StrictMode) on debug builds, but using DropBox, not logs.  
        CloseGuard.setEnabled(false);  
  
        Environment.initForCurrentUser();  
  
        // Set the reporter for event logging in libcore  
        EventLogger.setReporter(new EventLoggingReporter());  
  
        Process.setArgV0("<pre-initialized>");  
  
        Looper.prepareMainLooper();  
  
        // 创建ActivityThread实例  
        ActivityThread thread = new ActivityThread();  
        thread.attach(false);  
  
        if (sMainThreadHandler == null) {  
            sMainThreadHandler = thread.getHandler();  
        }  
  
        AsyncTask.init();  
  
        if (false) {  
            Looper.myLooper().setMessageLogging(new  
                    LogPrinter(Log.DEBUG, "ActivityThread"));  
        }  
  
        Looper.loop();  
  
        throw new RuntimeException("Main thread loop unexpectedly exited");  
    } 
	public void handleMessage(Message msg) {  
	    if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));  
	    switch (msg.what) {  
	        case LAUNCH_ACTIVITY: { // 创建Activity对象  
	            Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");  
	            ActivityClientRecord r = (ActivityClientRecord)msg.obj;  
  
	            r.packageInfo = getPackageInfoNoCheck(  
	                    r.activityInfo.applicationInfo, r.compatInfo);  
	            handleLaunchActivity(r, null);  
	            Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);  
	        } break;  
  
	        case BIND_APPLICATION: // 创建Application对象  
	            Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");  
	            AppBindData data = (AppBindData)msg.obj;  
	            handleBindApplication(data);  
	            Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);  
	            break;  
  
	        case CREATE_SERVICE: // 创建Service对象  
	            Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceCreate");  
	            handleCreateService((CreateServiceData)msg.obj);  
	            Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);  
	            break;  
              
	        case BIND_SERVICE:  // Bind Service对象  
	            Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceBind");  
	            handleBindService((BindServiceData)msg.obj);  
	            Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);  
	            break;  
	    }  
	}    
}
```
也许你不能完全看懂、理解这些代码，不过没关系，直接告诉你结论吧，ActivityThread的一个内部类H，里面定义了activity、service等启动、销毁等事件的响应，也就是说activity、service的启动、销毁都是在ActivityThread中进行的。

当然了，一个**Activity**或者**Service**的从创建到启动是相当复杂的，其中还涉及的**Binder**机制等等原理，推荐给大家两篇博文，去慢慢研读消化吧。

[Activity启动原理详解](http://blog.csdn.net/stonecao/article/details/6591847)

[Service启动原理分析](http://blog.csdn.net/luoshengyang/article/details/6677029)

准备工作不知不觉就做了这么多，差点忘了正事，我们还是要继续寻找**Application**、**Activity**、**Service**是何时与**ContextImpl**进行关联的。

 - **Application**

```
   // ActivityThread.java
   private void handleBindApplication(AppBindData data) { 
      try {
            // If the app is being launched for full backup or restore, bring it up in
            // a restricted environment with the base application class.
            Application app = data.info.makeApplication(data.restrictedBackupMode, null);
            mInitialApplication = app;
            ...
        } finally {
            StrictMode.setThreadPolicy(savedPolicy);
        }
   }

   // LoadedApk.java
   public Application makeApplication(boolean forceDefaultAppClass,
            Instrumentation instrumentation) {
        if (mApplication != null) {
            return mApplication;
        }

        Application app = null;

        String appClass = mApplicationInfo.className;
        if (forceDefaultAppClass || (appClass == null)) {
            appClass = "android.app.Application";
        }

        try {
            java.lang.ClassLoader cl = getClassLoader();
            ContextImpl appContext = new ContextImpl(); // 创建ContextImpl实例
            appContext.init(this, null, mActivityThread);
            app = mActivityThread.mInstrumentation.newApplication(
                    cl, appClass, appContext);
            appContext.setOuterContext(app); // 将Application实例传递给Context实例
        } catch (Exception e) {
            ...
        }
        mActivityThread.mAllApplications.add(app);
        mApplication = app;

        return app;
    }
```
每个应用程序在第一次启动时，都会首先创建一个Application对象。从startActivity流程可知，创建Application的时机在handleBindApplication()方法中，该函数位于 ActivityThread.java类

 - **Activity**

```
      private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        ...
        Activity a = performLaunchActivity(r, customIntent); // 到下一步

        if (a != null) {
            r.createdConfig = new Configuration(mConfiguration);
            Bundle oldState = r.state;
            handleResumeActivity(r.token, false, r.isForward,
                    !r.activity.mFinished && !r.startsNotResumed);
            ...
        }
        ...
     }

    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        ...    
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            ...
        }

        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);

            if (activity != null) {
                Context appContext = createBaseContextForActivity(r, activity); // 创建Context
                CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
                Configuration config = new Configuration(mCompatConfiguration);
                if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
                        + r.activityInfo.name + " with config " + config);
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config);

                if (customIntent != null) {
                    activity.mIntent = customIntent;
                }
                r.lastNonConfigurationInstances = null;
                activity.mStartedActivity = false;
                int theme = r.activityInfo.getThemeResource();
                if (theme != 0) {
                    activity.setTheme(theme);
                }


            mActivities.put(r.token, r);

        } catch (SuperNotCalledException e) {
            ...

        } catch (Exception e) {
            ...
        }

        return activity;
    }

```

```
    private Context createBaseContextForActivity(ActivityClientRecord r,
            final Activity activity) {
        ContextImpl appContext = new ContextImpl();  // 创建ContextImpl实例
        appContext.init(r.packageInfo, r.token, this);
        appContext.setOuterContext(activity);

        // For debugging purposes, if the activity's package name contains the value of
        // the "debug.use-second-display" system property as a substring, then show
        // its content on a secondary display if there is one.
        Context baseContext = appContext;
        String pkgName = SystemProperties.get("debug.second-display.pkg");
        if (pkgName != null && !pkgName.isEmpty()
                && r.packageInfo.mPackageName.contains(pkgName)) {
            DisplayManagerGlobal dm = DisplayManagerGlobal.getInstance();
            for (int displayId : dm.getDisplayIds()) {
                if (displayId != Display.DEFAULT_DISPLAY) {
                    Display display = dm.getRealDisplay(displayId);
                    baseContext = appContext.createDisplayContext(display);
                    break;
                }
            }
        }
        return baseContext;
    }
```

通过startActivity()或startActivityForResult()请求启动一个Activity时，如果系统检测需要新建一个Activity对象时，就会回调handleLaunchActivity()方法，该方法继而调用performLaunchActivity()方法，去创建一个Activity实例，并且回调onCreate()，onStart()方法等，函数位于 ActivityThread.java类。

 - **Service**

```
    private void handleCreateService(CreateServiceData data) {
        // If we are getting ready to gc after going to the background, well
        // we are back active so skip it.
        unscheduleGcIdler();

        LoadedApk packageInfo = getPackageInfoNoCheck(
                data.info.applicationInfo, data.compatInfo);
        Service service = null;
        try {
            java.lang.ClassLoader cl = packageInfo.getClassLoader();
            service = (Service) cl.loadClass(data.info.name).newInstance();
        } catch (Exception e) {
            if (!mInstrumentation.onException(service, e)) {
                throw new RuntimeException(
                    "Unable to instantiate service " + data.info.name
                    + ": " + e.toString(), e);
            }
        }

        try {
            if (localLOGV) Slog.v(TAG, "Creating service " + data.info.name);

            ContextImpl context = new ContextImpl(); // 创建ContextImpl实例
            context.init(packageInfo, null, this);

            Application app = packageInfo.makeApplication(false, mInstrumentation);
            context.setOuterContext(service);
            service.attach(context, this, data.info.name, data.token, app,
                    ActivityManagerNative.getDefault());
            service.onCreate();
            mServices.put(data.token, service);
            try {
                ActivityManagerNative.getDefault().serviceDoneExecuting(
                        data.token, 0, 0, 0);
            } catch (RemoteException e) {
                // nothing to do.
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(service, e)) {
                throw new RuntimeException(
                    "Unable to create service " + data.info.name
                    + ": " + e.toString(), e);
            }
        }
    }

```

通过startService或者bindService时，如果系统检测到需要新创建一个Service实例，就会回调handleCreateService()方法，完成相关数据操作。handleCreateService()函数位于 ActivityThread.java类

> 看到这里，相信你对Context的理解更进一步了，现在我们知道了Context是什么，它为Android提供了怎样的资源、功能、和服务，又在什么时候将Application、Activity、Service与ContextImpl相关联，但是所请求的资源是不是同一套资源呢？


在这里你一定说：“当然不是，不同的Context对象明显是有区别的，用法也不同”

但是其实他们访问的，确确实实，是**同一套**资源。

三：Context资源详解
--

来吧，看看不同Context对象的区别和用法的不同，参见以下表格。

![Context用法区别](http://upload-images.jianshu.io/upload_images/1915184-c666a310a5737c5f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这张表格是不是又支持了你的观点(也就是一直认为的，Context资源对象是不同的)，但是还是要再次强调一次，它们所请求的，确确实实是同一块资源，看看上面进行关联的源码，都走进了Context实现类的**init**方法，拨云见日，我们去看看**init**方法吧。

查看ContextImpl类源码可以看到，**getResources**方法直接返回内部的**mResources**变量

```
final void init(LoadedApk packageInfo,  
            IBinder activityToken, ActivityThread mainThread,  
            Resources container) {  
    mPackageInfo = packageInfo;  
    mResources = mPackageInfo.getResources(mainThread);  
  
    if (mResources != null && container != null  
            && container.getCompatibilityInfo().applicationScale !=  
                    mResources.getCompatibilityInfo().applicationScale) {  
        if (DEBUG) {  
            Log.d(TAG, "loaded context has different scaling. Using container's" +  
                    " compatiblity info:" + container.getDisplayMetrics());  
        }  
        mResources = mainThread.getTopLevelResources(  
                mPackageInfo.getResDir(), container.getCompatibilityInfo().copy());  
    }  
    mMainThread = mainThread;  
    mContentResolver = new ApplicationContentResolver(this, mainThread);  
  
    setActivityToken(activityToken);  
} 
```

**mResources**又是调用**LoadedApk**的**getResources**方法进行赋值。代码如下。

```
public Resources getResources(ActivityThread mainThread) {  
    if (mResources == null) {  
        mResources = mainThread.getTopLevelResources(mResDir, this);  
    }  
    return mResources;  
} 
```

从代码中可以看到，最终**mResources**的赋值是由**AcitivtyThread**的**getTopLevelResources**方法返回。代码如下。

```
Resources getTopLevelResources(String resDir, CompatibilityInfo compInfo) {  
    ResourcesKey key = new ResourcesKey(resDir, compInfo.applicationScale);  
    Resources r;  
    synchronized (mPackages) {  
        // Resources is app scale dependent.  
        if (false) {  
            Slog.w(TAG, "getTopLevelResources: " + resDir + " / "  
                    + compInfo.applicationScale);  
        }  
        WeakReference<Resources> wr = mActiveResources.get(key);  
        r = wr != null ? wr.get() : null;  
          
        if (r != null && r.getAssets().isUpToDate()) {  
            if (false) {  
                Slog.w(TAG, "Returning cached resources " + r + " " + resDir  
                        + ": appScale=" + r.getCompatibilityInfo().applicationScale);  
            }  
            return r;  
        }  
    }  
  
    AssetManager assets = new AssetManager();  
    if (assets.addAssetPath(resDir) == 0) {  
        return null;  
    }  
  
    DisplayMetrics metrics = getDisplayMetricsLocked(false);  
    r = new Resources(assets, metrics, getConfiguration(), compInfo);  
    if (false) {  
        Slog.i(TAG, "Created app resources " + resDir + " " + r + ": "  
                + r.getConfiguration() + " appScale="  
                + r.getCompatibilityInfo().applicationScale);  
    }  
      
    synchronized (mPackages) {  
        WeakReference<Resources> wr = mActiveResources.get(key);  
        Resources existing = wr != null ? wr.get() : null;  
        if (existing != null && existing.getAssets().isUpToDate()) {  
            // Someone else already created the resources while we were  
            // unlocked; go ahead and use theirs.  
            r.getAssets().close();  
            return existing;  
        }  
        // XXX need to remove entries when weak references go away  
        mActiveResources.put(key, new WeakReference<Resources>(r));  
        return r;  
    }  
}  
```
以上代码中，mActiveResources对象内部保存了该应用程序所使用到的所有Resources对象，其类型为**WeakReference**，所以当内存紧张时，可以释放**Resources**占用的资源，自然这不是我们探究的重点，ResourcesKey的构造需要resDir和compInfo.applicationScale。resdDir变量的含义是资源文件所在路径，实际指的是APK程序所在路径，比如可以是：/data/app/com.haii.android.xxx-1.apk，该apk会对应/data/dalvik-cache目录下的：data@app@com.haii.android.xxx-1.apk@classes.dex文件。

**所以结论来了：**

如果一个应用程序没有访问该程序以外的资源，那么mActiveResources变量中就仅有一个**Resources**对象。

**总结：**

当ActivityThread类中创建Application、Service、Activity的同时，完成了与ContextImpl的关联绑定，通过ContextImpl类中init方法，获得了一个唯一的**Resources**对象，根据上述代码中资源的请求机制，再加上ResourcesManager采用单例模式，这样就保证了不同的ContextImpl访问的是同一套资源。

如果这篇博客现在就结束了，你一定会杀了我 - -，现在我们就来分析下，是什么造成了唯一的这个**Resources**，却展现出了“**不同**”。

举个通俗易懂的例子，我和我老妈都拿到同一块**土豆**，但是因为我们处理这个土豆的方法有区别，导致这个土豆最后表现出来的也不一样，我想把它做成**薯片**，我妈妈把它炒成了**土豆丝**，:-D。

再具体一点，比如除了Activity可以创建一个Dialog，其它Context都不可以创建Dialog。比如在Application中创建Dialog会报错，还有Application和Service可以启动一个Activity，但是需要创建一个新的task。比如你在Application中调用startActivity（intent）时系统也会崩溃报错。

报错的原因并不是因为他们拿到的Context资源不同，拿到的都是一个Resoucres对象，但是在创建Dialog的时候会使用到Context对象去获取当前主题信息，但是我们知道Application和Service是继承自ContextWrapper，**没有实现关于主题的功能**，然而Activity是继承自ContextThemeWrapper，该类是实现了关于主题功能的，因此创建Dialog的时候必须依附于Activity的Context引用。

**结论：**

**Application、Service、Activity，它们本身对Resources资源处理方法的不同，造成了这个Resoucres最后表现出来的不一样**，这么说大家就都懂了吧！


四：Context内存泄漏
--

关于Context的内存泄漏，找到一篇比较不错的文章分享给大家。

[Android开发，中可能会导致内存泄露的问题](http://spencer-dev.com/blog/2015/androidkai-fa-zhong-ke-neng-hui-dao-zhi-nei-cun-xie-lu-de-wen-ti.html/)

**写在最后：**

> Context可能还有更多深层次的知识需要我们去了解，比如Context这些封装类，是具体如何通过Binder跟ContextImpl进行关联的；资源对象都被存储在ArrayMap，为什么ArrayMap中会有可能存在多个资源对象，如何访问其他应用程序的Context资源等等，剩下的这些就靠大家慢慢发掘了~

转载请注明出处。

**如果大家觉得喜欢有价值，就关注我，点下赞哈，你们的支持是我持续原创的动力。**