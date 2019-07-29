**Note5**

##### 1.（）文件、SharePreference和SQLite使用场景

 - 文件：适用于存储一些简单的文本数据类型或二进制数据
 - SharePreference：适用于存储一些少量纪录的简单数据，如单个用户信息、配置信息。
 - SQLite：适用于大量记录的复杂关系型数据，如所有用户的信息
 
##### 2.（）Okhttp、Volley和Retrofit比较

 - Okhttp：对HttpUrlConnection进行封装的一个高性能的网络请求框架，支持同步、异步，支持http2.0、websocket。使用的时候需要自己再封装多一层。
 - Volley：轻量级的网络请求库，支持HttpUrlConnection、okhttp、ImageLoader，封装好。缺点就是不支持post大数据，不适合上传文件。
 - Retrofit：封装了Okhttp，RESTful风格的网络请求框架，支持Rx，解耦更彻底，没什么缺点吧。
 
##### 3.（）MVC和MVP


##### 4. （）FragmentPagerAdapter和FragmentStatePageAdapter区别

 - FragmentPageAdapter：每一个生成的Fragment都会保存到内存中，切换后不会被销毁。适用于数量少、静态的Fragment
 - FragmentStatePageAdapter：只保留当前Fragment，切换后被销毁，释放资源。适用于数量多、数据动态性大的Fragment
 
 
##### 5.（）LIstView优化方式

 - 缓存机制：getView()方法中有个view参数，就是用来缓存每一项item。如果非空的话就直接引用
 - viewHolder模式：充分利用ListView的视图缓存机制，避免每次getView的时候都要通过findViewById去实例化控件。可以大大提高ListView的性能。具体的做法是在Adapter中定义ViewHolder内部类，把Item中的控件，作为viewHolder的成员变量，把viewHolder作为tag设置给view
 
 
##### 6.（）视图动画和属性动画的区别


##### 7.（）Activity的生命周期

 - 四种状态：运行状态、暂停状态、停止状态和销毁状态
 - 回调方法：onCreate -> onStart -> onResume -> onPause -> onStop -> onDestroy
 - 当启动Activity的时候，会依次调用：onCreate > onStart -> onResume，此时Activity进入运行状态
 - 当启动另一个Activity，原来的Activity会调用：onPause -> onStoop，当调用onPause的时候进入暂停状态，特点是Activity不在前台，但是可见，如打开一个对话框。当调用onStop的时候进入停止状态，此时不在前台且不可见。
 - 当点击back返回上一个Activity，上一个Activity会调用onRestart -> onStart -> onResume。特别的，如果上一个Activity由于内存不足被杀，会调用onCreate -> onStart -> onResume，即重新创建。
 - 当点击Back退出Activity时候，会依次调用：onPause -> onStop -> onDestroy。此时为销毁状态。
 
 
##### 8.（）Activity启动模式



##### 9.（）Handler-异步消息处理机制

##### 10.（）AsyncTask异步任务

 - AsyncTask封装了Handler + 线程池，即使对异步消息处理机制完全不了解也可以十分简单从子线程切换到主线程。创建AsyncTask对象必须在主线程中，以为AsyncTask内部的Handler是静态的，加载AsyncTask类的时候会初始化Handler
 - 创建AsyncTask的子类时重写几个方法，其中doInBackground是运行在子线程，其他运行在主线程。制定了三个泛型参数，第一个是执行时传入的参数类型，也是doInBackground的传入参数，第二个是publishProgerss()的传入参数，第三个是onPostExecute()的传入参数。
 
 
##### 11.（）Servic

Service是Android中实现程序后台运行的解决方案，它适合去执行那些不需要用户交互而且还要长时间运行的任务。服务依赖于创建服务时所在的进程，当该进程被杀，所依赖的服务也被杀


##### 12.（）Service生命周期


##### 13.（）Android系统架构

 - Linux内存层：底层核心，包含Android系统核心服务。包括硬件驱动、进程管理等
 - 标准库和运行时：
    - Dalvik和ART：Android运行环境虚拟机，每个app都会分配独立的虚拟机保证相互之间不会被干扰，运行时编译。ART是安装时编译。
    - 标准库：开发者在开源环境中可以使用的开发库
 - Framework：构建应用程序用到的各种API
 
 
##### 14.（）Android控件架构


##### 15.（）View的测量

 - onMeasure()：告诉系统画一个多大的View
 - MeasureSpec：测量规格，用来获取测量模式和测量值。MeasureSpec是一个32位的int值，高2位为测量模式，低30位为测量大小，通过MeasureSpec的静态方法getMode、getSize获得。
 - 三种测量模式：
 
 
##### 16.（）ViewGroup的测量和绘制



##### 17.（）Activity退出怎么保存数据(异常情况退出)

 - 在onSaveInstanceState()方法中通过Bundle保存数据，可以保存基本的数据类型和他们的数组，序列化对象及其数组，包裹类对象及其数组.
 - 在onRestoreInstanceState()方法中恢复数据。或者在onCreate()方法中恢复数据。注意onCreate()方法中的Bundle可能为空.
 - 异常情况：启动另一Activity、点击Home、旋转屏幕、锁屏、Activity被kill

##### 18.（）自定义View常用的构造函数及参数作用
常用的有三个，一个只有Context，一个只有Context和attributeSet，另外一个主题id
```

```
只有Context参数的构造函数在代码中创建View的时候调用。
含有两个参数context和attrSet的构造函数一般是在xml文件中使用View，加载View时调用。attributeSet保存了自定义的属性。
含有三个参数的构造函数一般是前面两个调用


##### 19.（）RecyclerView和ListView的区别

 - 最大的区别：RecyclerView强制使用ViewHolder模式优化列表，
 - RecyclerView通过设置LayoutManager快速实现列表，网格布局
 - RecyclerView在添加、删除Item的时候有默认的动画
 - RecyclerView的事件监听需要自己实现
 

##### 20.（）自定义View的几种方式

自定义View的方式有三种：组合控件、继承原有控件和完全自定义View。

 - 组合控件：将一些基础组件封装起来形成新的控件，如通用的标题栏
 - 继承原有控件：在原有控件的基础上添加方法来扩展功能，或者修改原有的显示逻辑
 - 完全自定义View：需要重写onMeasure()实现测量逻辑，需要重写onDraw方法实现绘制逻辑，需要注意支持wrap_parent和Padding
 
##### 21.（）Broadcast广播机制

广播分为：标准广播、有序广播和本地广播.

 - 标准广播：是一种异步广播，所有广播接收器几乎同时受到广播
 - 有序广播：是一种同步广播，优先级较高的广播接收器西安搜狐到广播，可拦截广播和传递数据
 - 本地广播：只有当前应用可以接收

注册广播的方式有两种：动态注册和静态注册

 - 静态注册：是指在AndroidMenifest配置中注册，这种方式注册的广播接收器白应用没运行也能收到广播
 - 动态注册：是指在代码中注册，这种方式注册的广播接收器在运行应用并且注册后才能收到广播，注意需要取消注册
动态注册的优先级比静态注册的高


##### 22.（）动画的几种方式和区别
 动画分为三种：帧动画、视图动画和属性动画
 
 - 帧动画：通过切换图片的方式实现动画效果
 - 视图动画：通过矩阵运算修改View的显示
 - 属性动画：通过修改对象的属性值实现动画，是真正意义上的动画。
 
视图动画和属性动画的区别：例如将一个View使用视图动画移动到别的位置，但它点击监听事件还在原来的区域；而使用属性动画它的点击区域也随着控件的移动而移动。


##### 23.（）运行时权限 

 - 6.0以前，权限在安装的时候申请，如果不同意的权限，用户可以选择不安装
 - 6.0以后，不在安装时申请，而是把权限分为普通权限和危险权限。普通权限默认已授权，危险权限分为9组20多个，包括读取联系人、电话、位置和 存储等。
 
 
##### 24.（）Handler消息机制


消息机制由四部分组成：Handler、Message、MessageQueue和Lopper

 - Message：消息的载体
 - MessageQueue：消息队列，负责存放消息
 - Looper：负责读取消息，每个线程只有一个Looper
 - Handler：负责发送和处理消息，将当前线程切换到handler所关联的线程去处理消息

它的运行过程是这样的：handler发送一个消息出去，这个消息会被添加到MessageQueue中等待被处理，而Looper则会一直尝试从消息队列中取出消息，最后分发回handler的handleMessage()方法。


##### 25.（）

 
##### 1.（）字符和字符串所占字节数

 - char：2字节
 - String：
     - 英文字符串的字节数与英文字母相同，如“hello”占5个字节
     - 中文字符串根据编码而定，如GBK下，一个汉字占2个字节；UTF-8下，一个汉字占3个字节
     
##### 2.（）String、StringBuffered和StringBuilder的区别。（）

##### 3.（）equals与“==”的区别

 - equals：比较内容是否相等
 - ==：比较地址是否相等
	JVM有个区域叫字符串缓冲池，使用String a = "abc"时，会检查缓冲池中是否有相同内容的对象，有就直接引用，没有才新建一个值为“abc”的对象，并放入缓冲池，再把引用返回给a。而new String("abc")则创建一个新的String对象，在堆内存开辟新的空间来存放这个对象。