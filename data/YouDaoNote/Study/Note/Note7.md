** Note-7 **

**Note7**

##### 1. (Android) Handler, Message, MessageQueue和Lopper的关系是什么?

- Handler: 发送和处理消息
- Message: 消息的载体, 即保存消息的内容
- MessageQueue: Message的队列, 存放Message
- Lopper: 消息循环系统, 用来从MessageQueue中取消息

流程: 当调用Handler的sendMessage()或postRunnable()时, 最终都是将Message放入MessageQueue中. 而Looper则一直监听着MessageQueue中的消息, 如果有新的消息, 则通过MessageQueue的next()方法取出Message, 然后调用Handler的dispatchMessage将消息分发到Handler, 最终执行Handler的handleMessage()方法或Runnable的run();方法.

##### 2. (Android) MessageQueue的数据结构是什么? 为什么要用这种结构?

- 队列, 是一种先进先出的线性数据结构. 保证了消息同步有序的执行
- 链表实现


##### 3. (Android) 如何在子线程创建Handler?

要在子线程中创建Handler, 首先要调用Looper.prepare()方法, 会创建Looper对象, 用ThreadLocal保存当前线程的Looper.
原因: 创建Handler时, 会检查当前线程是否存在Looper, 如果没有则程序崩溃.


##### 4. (Android) Handler的post()方法原理

Handler postRunnable(), 会把这个runnable对象赋值给Message的callback字段, 最终通过Handler的sendMessage()方法发送这个消息. 当后面调用Handler的dispatchMessage()方法时, 会首先判断message.callback是否为空, 不为空就执行Runnable的run()方法, 否则就调用handleMessage();

##### 5. (Android) Handler是如何切换线程

- Handler持有Looper, 创建Handler时将当前线程的Looper赋值给Handler, 或者创建Handler时传入指定线程的Looper.
- Looper持有MessageQueue, 当调用handler的sendMessage()方法, 会把当前的Handler赋值给Message的target字段, 然后将Message入队. 此时, Message就进入了指定线程.
而Looper会监听MessageQueue的消息, 当有消息时取出Message, 调用message.target的dispatchMessage方法, message.target就是Handler对象
- 由于是在指定线程的消息循环系统中调用dispatchMessage(), 所以就切换到指定线程


##### 6. (Android) Handler消息机制的一些整理

- 在ActivityThread的main()方法中调用Looper.prepareMainLooper()方法创建Looper对象, 同时添加到ThreadLocal中. 对Looper的构造方法中创建了MessageQueue
- 创建Handler时, 会从ThreadLocal中获取当前线程的Looper, 如果没有则崩溃. 因此在主线程创建Handler没问题, 而在子线程中先创建Looper, 再创建Handler. Handler会持有Looper和Looper的MessageQueue, 当调用Handler的sendMessage()方法时, 会将当前的handler设为Message的target, 然后放入MessageQueue消息队列中.
- Looper从MessageQueue的next()方法中取出Message. 调用message.target的dispatchMessage()方法. 首先判断message.callback是否为空, 这个callback就是post()方法传入地方Runnable对象. 不为空则执行run()方法. 否则判断handler的mCallback对象是否为空, 这个mCallback是构建Handler时传入的. 不为空则执行mCallback的handleMessage()方法. 它的返回值表示是否已经处理了消息, 如果返回false, 则会调用handler的handleMessage()方法, 这个方法是创建Handler时重写的方法.



##### 7. (Android) Activity的启动流程, 简要描述

- **startActivity**()有n个重载，最终调用到**startActivityForResult**()
- 在startActivityForResult()方法中调用**Instrumentation**的**execStartActivity**()方法启动Activity
- 在execStartActivity()方法通过**ActivityManagerNative.getDefault**()获取**AMS**单例，调用AMS的**startActivity**()方法，处理Activity的启动, 并且传入了**ApplicationThread**对象，进行后续的IPC，**此时启动Activity的流程转到系统进程**
  - **ApplicationThread**继承ApplicationThreadNative，ApplicationThreadNative是抽象类，继承Binder实现**IApplicationThread**接口，IApplicationThread定义了系统进程调用APP进程的接口，scheduleLaunchActivity
  - **AMS**继承ActivityManagerNative，ActivityManagerNative是抽象类，继承Binder，实现**IActivityManage**r接口。IActivityManager定义了APP进程调用系统进程的接口，如启动Activity和Service
- 在AMS的startActivity()方法，经过**ActivityStackSupervisor**和**ActivityStack**的相互调用，最终调用到**ActivityStackSupervisor**的**realStartActivityLocked**()方法启动Activity
- 在realStartActivityLocked()方法中，调用**ApplicationThread**的**scheduleLaunchActivity**()方法，**把启动Activity的流程转到App进程**
- 在scheduleLaunchActivity()方法中，通过**Handler**发送消息去启动Activity，调用到**handleLaunchActivity**()去处理启动
- 在handleLaunchActivity()方法中，调用**ActivityThread**的**performLaunchActivity**()方法，创建并返回Activity实例
  - 调用**Instrumentation**的**newActivity**方法通过**反射(ClassLoader)**的方式创建Activity实例，
  - 通过**LoadedAPK**的**makeApplication**()方法重建**Application**，如果已存在，则直接返回Application
  - 通过调用**createBaseContextForActivity**()方法创建**ContextImpl实例
    - ContextImpl是Context的具体实现，通过attach()方法和Activity建**立联系
  - 调用attach()方法完成一些初始化操作
    - 建立ContextImpl和Activity的关联
    - 创建Window，并和Activity建立关联
  - 调用**Instrumentation**的**callActivityOnCreate**()方法，最终调用Activity的**onCreate**(), 完成启动。
  - 调用**activity.performStart**()方法，最终调用Activity的**onStart()**方法
- 当创建并返回Activity，回到handleLaunchActivity()中，调用**handleResumeActivity**()方法，最终调用Activity的**onResume**()完成显示

##### 8. (Android) OkHttp请求的基本流程


##### 23. (Android) Service的启动过程, 简要描述

- startService()方法经过一系列调用最终调用到**ContextImpl**的**startServiceCommon**()方法
- 在startServiceCommon()方法中通过**ActivityManagerNative.getDefault**()获取到**AMS**，调用AMS的**startServic**e()方法将启动流程**转移到系统进程**。
- 在AMS的startService()方法中经过一系列的调用，最终调用到**ActiveServices**的**realStartServiceLocked**()方法
  - ActiveServices用来管理Service，相当于ActivityStack用来管理Activity。
- 在realStartServiceLocked()方法中调用**ApplicationThread**的**scheduleCreateService**()方法，将启动Service的流程**转移到APP进程**。
- 在scheduleCreateService()方法中，通**过Handler**发送消息去执行。最后调用到ActivityThread的**handleCreateService**()方法
- 在**handleCreateService**()方法中
  - 通过**反射(类加载)**的方式**创建Service**
  - 创建**ContextImpl**
  - [创建并]返回Application
  - 调用setvice的**attache**()方法建立ContextImpl与Service的关联
  - 调用Service的**onCreate**()方法, 完成启动
  - 缓存Service
- 回到**ActiveServices**的**realStartServiceLocked**()方法，继续调用**sendServiceArgsLocked**()方法，将启动service的处理转移到APP进程，然后通过handle方式，最终调用service的**onStartCommand**()方法。

##### 24. (Android) Service的绑定过程, 简要描述

- bindService()方法经过一系列调用，最终调用到**ContextImpl**的**bindServiceCommon**()方法
- 在bindServiceCommon()方法中
  - 通过**LoadedAPK**的**getServiceDispatcher**()方法获取一个**IServiceConnection**的接口对象，它具体是实现类是**ServiceDispatcher.InnerConnection**
    - 将客户端的**ServiceConnection**转化为**ServiceDispatcher.InnerConnection**
    - InnerConnection extends IServiceConnection.Stub，应该是**binder**通信
    - ServiceDispatcher连接ServiceConnection和InnerConnection
  - 调用**AMS**的**bindService**()方法，将绑定流程**转移到系统进程**
- 在AMS的bindService()方法中经过一系列调用，最终调用到**ActiveServices**的**realStartServiceLocked**()方法，通过调用**ApplicationThread**的**scheduleCreateService**()方法将绑定流程**转移到app进程**。
- 在**ApplicationThread**的**scheduleCreateService**()方法中通过**Handler**发送消息去处理，最终调用到**ActivityThread**的**handleBindService**()方法。
  - 调用**service**的**onBind**()方法，绑定完成
  - 调用**AMS**的**publishService**()方法，在系统进程中通知客户端已将成功连接到Service。
- 在AMS的publishService()方法中，经过一系列调用，最终调用到**ActiveServices**的**publishServiceLocked**()方法。
- 在publishServiceLocked()方法中，调用**IServiceConnection
**的**connected**()方法，具体实现是**ServiceDispatcher.InnerConnection**的**connected**()方法
- 在ServiceDispatcher.InnerConnection的connected()方法中，调用**ServiceDispatcher**的**connected**方法，用Handler post一个**RunConnection**类去执行，在RunConnection的**run**()方法中，最终回调客户端Connection的**onServiceConnected**(name, service)方法完成连接。



##### 35. (Android) BroadcastReceiver的注册流程, 简要描述

- **registerReceiver**()经过一系列调用，最终调用到**ContextImpl**的**registerReceiverInternal**()方法
- 在registerReceiverInternal()方法中
  - 创建了**IIntentReceiver**接口对象，它具体是实现类是**ReceiverDispatcher.InnerReceiver**
    - InnerReceiver实现**Binder**，用于跨进程通信，
    - IIntentReceiver则定义了进程接口回调方法
    -** ReceiverDispatcher**保存了**BroadcastReceiver**和**InnerReceiver**，这样当收到广播时，ReceiverDispatcher可以方便的调用BroadcastReceiver的onReceive()方法
  - 调用**AMS**的**registerReceiver**()方法，将注册流程**转换到系统进程**。
- 在**AMS**的**registerReceiver**()方法中，把**InnerReceiver**和**IntentFilter**保存起来，完成注册流程。


##### 36. (Android) BroadcastReceiver的发送和接收广播流程, 简要描述

- **sendBroadcast(**)经过一系列调用，最终调用到**ContextImpl**的**sendBroadcast**()方法。
- 在sendBroadcast()方法中，调用**AMS**的**broadcastIntent**()方法，将发送广播流程**转换到系统进程**
- 在AMS的broadcastIntent()方法中，调用了**AMS**的**broadcastIntentLocked**()方法。
- 在broadcastIntentLocked()方法中，
  - intent添**加FLAG_EXCLUDE_STOPPED_PACKAGES**，使停用的应用不会收到广播。
  - 根据**intentfilte**查找出满足条件的BroadcastReceiver，并添加到**BroadcastQueue**中。BoradcastQueue调**用scheduleBroadcastsLocked**()方法会将广播发送该广播接受者。
- 在BoradcastQueue的scheduleBroadcastsLocked()方法中，使用**Handler**发送消息去执行发送广播，最后调用到**processNextBroadcast**()方法去处理消息
- 在processNextBroadcast()方法中，首先发送的是无序广播，**遍历Receiver**的列表，掉用**deliverToRegisteredReceiverLocked**()方法将广播发送出去，最终调用**performReceiveLocked**()方法完成具体的发送过程。
- 此时广播**已经发送出去**
- 回到BroadcastQueue的performReceiveLocked()方法，会调用**ApplicationThread**的**scheduleRegisteredReceiver**()方法将广播的接收流程**转移到APP进程**。
- 在ApplicationThread的scheduleRegisteredReceiver()方法中，调用**InnerReceiver**的**performReceive**()方法，最终调用到R**eceiverDispatcher**的**performReceive**()方法
- 在ReceiverDispatcher的performReceive()方法中，通过**Handle**r的post()方法将**Args**的Runnable对象发送出去，在Args的**run(**)方法中会调用**BroadcastReceiver**的**onReceive**()方法，此时广播接收者会收到广播。



#####