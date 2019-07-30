---
title: HandlerThread简单使用
date: 2017-07-01 16:59:50
tags:
 - Android
 - Thread
 - HandlerThread
categories: [Android]
---

# HandlerThread简介
HandlerThread继承了Thread，它是一种可以使用Handler的Thread，它的实现就是在run()方法中通过Looper.prepare()创建消息队列，并通过Looper.loop()开启消息循环。这样在实际使用中就允许在HandlerThread中创建Handler了。
由于HandlerThread的run()方法是一个无限循环，因此当明确不需要使用HandlerThread的时候可以通过Looper的quit()或quitSafely()来终止线程执行。

# 使用Handler
通常我们会在主线程中创建Handler，在子线程中调用handler.post(runnable)传递消息到主线程的消息队列中处理runnable的run方法.这样完成了子线程到主线的切换。
在onCreate()方法中
```
mainHandler = new Handler();
```

然后在子线程中post
```
btn_post_to_main_thread.setOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View v) {
		new Thread(new Runnable() {
			@Override
			public void run() {
				Log.d(TAG, "Thread id = " + Thread.currentThread().getId());
				mainHandler.post(runnable);
			}
		}).start();
                
	}
});
```

runnable
```
    private Runnable runnable = new Runnable() {
        @Override
        public void run() {
            Log.d(TAG, "Thread id = " + Thread.currentThread().getId());
        }
    };
```

运行结果
```
HandlerThreadActivity: Thread id = 10383
HandlerThreadActivity: Thread id = 1
```

# 使用HandlerThread
先创建HandlerThread实例，在onCreate()方法中
```
handlerThread = new HandlerThread("handlerThread");
handlerThread.start();
```
这样就开启了一个带Looper的子线程，因为HandlerThread是继承自Thread，它的run方法是这样定义的
```
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
```

关于Looper原理，可以参考《Android开发艺术探索》中的消息机制，我的理解是:

 - Looper.prepare();创建Looper实例
 - Looper.loop();进入一个无限循环中，不断监听消息队列中是否有消息，有则把他取出来分发给handler的handlerMessage()中处理。

因为线程中需要有一个Looper，线程绑定的handler才可以发送消息到消息队列中，那么相应的线程才会得到处理。
 
然后就是利用handlerThread获取到Looper用来创建Handler实例
```
handler = new Handler(handlerThread.getLooper());
```
此时这个handler即使实在主线程中创建，但是它与子线程的Looper关联了，所以处理消息时候也会在子线程中处理的
```
btn_post_to_sub_thread.setOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View v) {
		handler.post(runnable);
	}
});
```
运行结果：
```
HandlerThreadActivity: Thread id = 10382
HandlerThreadActivity: Thread id = 10382
HandlerThreadActivity: Thread id = 10382
```
可以知道是在子线程中处理的。

# HandlerThread和Thread的区别

 - 普通Thread主要用于在run()方法中执行一个耗时的任务
 - HandlerThread内部创建消息队列，需要handler消息方式来通知HandlerThread去执行一个具体的任务。
