---
title: IntentService 工作原理
date: 2017-07-02 09:22:13
tags:
 - Android
 - IntentService
 - Service
 - Thread
categories: [Android]
---

# IntentService简介
是一种特殊的Service，继承了Service并且是个抽象类，可用于执行后台耗时任务，当任务结束后自动停止，同时由于是服务的原因，它的优先级比普通线程高很多，因此不容易被系统杀死。在实现上，IntentService封装了HandlerThread和Handler。

# 基本流程

startService(intentService):
onCreate()->
onStartCommand()->
onStart()->
sendMessage()->
handlerMessage()->
onHandlerIntent()->
stopSelf(startId)->
onDestroy->



# 详细解剖
## onCreate()
当服务第一次创建的时候调用onCreate()
```
    @Override
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
```
该方法封装了HandlerThread 和 Handler

## onStartCommand()
每一次启动服务的时候调用onStartCommand()
```
    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }
```
该方法仅仅调用了onStart()方法
## onStart()
```
    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }
```
该方法把startService传来的Intent和startId封装成Message消息，然后通过handler发送消息，接着执行mServiceHandler.handleMessage()
其中ServiceHandler是IntentService的一个内部类，它的定义如下
```
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }
```

## handleMessage()
```
@Override
public void handleMessage(Message msg) {
	onHandleIntent((Intent)msg.obj);
	stopSelf(msg.arg1);
}
```
该方法会执行onHandleIntent()方法，它是个抽象方法，因此我们自定义IntentService的时候必须重写onHandleIntent()方法。因为发送消息那那个handler是HandlerThread所关联的，所以处理信息的时候是在子线程进行的。不会的可以看看HandlerThread原理。

## stopSelf()
```
@Override
public void handleMessage(Message msg) {
	onHandleIntent((Intent)msg.obj);
	stopSelf(msg.arg1);
}
```
当onHandleIntent()方法执行完毕之后，会调用stopSelf(int)方法结束服务


## onDestroy()
```
    @Override
    public void onDestroy() {
        mServiceLooper.quit();
    }
```
最后销毁服务的时候，终止线程执行