---
title: 5分钟打造Android一键退出功能
categories: Android
tags: [Android,Activity,退出] 
---
5分钟打造Android一键退出功能
==
**写在前面：**
当我们的App打开很多Activity的时候，用户挨个返回退出显然用户体验是非常不好的，所以我们有时需要提供一个**一键退出功能**。一键退出功能有很多种实现方法，本文我们选择比较常规的手段，用一个**BaseActivity**管理所有启动的**Activity**。
<!--more-->
**下面给出完整的BaseActivity代码**

```
import java.util.LinkedList;
import java.util.List;
import android.app.Activity;
import android.os.Bundle;

public abstract class BaseActivity extends Activity {
	// 管理运行的所有的activity
	public final static List<BaseActivity> mActivities = new LinkedList<BaseActivity>();
	public static BaseActivity activity;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		synchronized (mActivities) {
			mActivities.add(this);
		}
		init();
	}

	private void init() {
		initViews();
		initData();
	}

	/**
	 * 初始化Views
	 */
	public abstract void initViews();

	/**
	 * 初始化数据
	 */
	public void initData() {

	}

	@Override
	protected void onResume() {
		super.onResume();
		activity = this;
	}

	@Override
	protected void onPause() {
		super.onPause();
		activity = null;
	}

	@Override
	protected void onDestroy() {
		super.onDestroy();
		synchronized (mActivities) {
			mActivities.remove(this);
		}
	}

	/**
	 * 一键退出的方法
	 */
	public void killAll() {
		// 复制了一份mActivities 集合
		List<BaseActivity> copy;
		synchronized (mActivities) {
			copy = new LinkedList<BaseActivity>(mActivities);
		}
		for (BaseActivity activity : copy) {
			activity.finish();
		}
		// 杀死当前的进程
		android.os.Process.killProcess(android.os.Process.myPid());
	}
}

```
**代码分析：**
在项目中的所有的Activity，都**继承于BaseActivity**，在onCreate方法中，将这个Activity **add**进**LinkedList**中（这里选择用LinkedList是因为它增删快，适合于这个场景中），在onDestory方法中将这个Activity **remove**掉，这样就保证每一个启动了的Activity都存于集合LinkedList中。

然后我们写一个killAll方法，复制这个集合并且遍历退出，你可以在任何地方调用这个方法，这样我们的一键退出功能就完美实现了~