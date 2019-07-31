**Note5**

##### 1.（）文件、SharePreference和SQLite使用场景

 - 文件：适用于存储一些简单的文本数据类型或二进制数据
 - SharePreference：适用于存储一些少量纪录的简单数据，如单个用户信息、配置信息。
 - SQLite：适用于大量记录的复杂关系型数据，如所有用户的信息
 
##### 2.（）Okhttp、Volley和Retrofit比较

 - OkHttp：对HttpUrlConnection进行封装的一个高性能的网络请求框架，支持同步、异步，支持http2.0、websocket。使用的时候需要自己再封装多一层。
 - Volley：轻量级的网络请求库，支持HttpUrlConnection、okhttp、ImageLoader，封装好。缺点就是不支持post大数据，不适合上传文件。
 - Retrofit：封装了Okhttp，RESTful风格的网络请求框架，支持Rx，解耦更彻底，没什么缺点吧。
 
##### 3.（）MVC和MVP

MVC
 - Model-View-Controller：模型-视图-控制器
 - 目的：将数据与视图分离，以控制器连接两者实现解耦。
 - Android中的MVC，View层由XML负责，M层数据存取，Controller连接View和Model
 - 优点：易理解，开发和维护成本低
 - 缺点：没有明确定义，Activity既可以做Controller也可以做View，使得掺杂了各种业务逻辑代码变得臃肿

MVP：
 - 作用：解决了MVC中M层与V层的耦合，带来更良好的可扩展性，保证系统整洁与灵活
 - V：Activity、Fragment或某个View负责显示与刷新
 - M：数据存取
 - P：逻辑分发，连接M层与V层

区别：
 - MVC中，V层可以直接访问M层
 - MVP中，V层不能直接访问M层，必须通过Presenter层获取，再由P层调用V层的接口实现刷新。



##### 4. （）FragmentPagerAdapter和FragmentStatePageAdapter区别

 - FragmentPageAdapter：每一个生成的Fragment都会保存到内存中，切换后不会被销毁。适用于数量少、静态的Fragment
 - FragmentStatePageAdapter：只保留当前Fragment，切换后被销毁，释放资源。适用于数量多、数据动态性大的Fragment
 
 
##### 5.（）LIstView优化方式

 - 缓存机制：getView()方法中有个view参数，就是用来缓存每一项item。如果非空的话就直接引用
 - viewHolder模式：充分利用ListView的视图缓存机制，避免每次getView的时候都要通过findViewById去实例化控件。可以大大提高ListView的性能。具体的做法是在Adapter中定义ViewHolder内部类，把Item中的控件，作为viewHolder的成员变量，把viewHolder作为tag设置给view
 
 
##### 6.（）视图动画和属性动画的区别

 - 视图动画：通过矩阵运算修改View的显示
 - 属性动画：通过修改对象的属性值实现动画，是真正意义上的动画。

视图动画和属性动画的区别：例如将一个View使用视图动画移动到别的位置，但它点击监听事件还在原来的区域；而使用属性动画它的点击区域也随着控件的移动而移动。



##### 7.（）Activity的生命周期

 - 四种状态：运行状态、暂停状态、停止状态和销毁状态
 - 回调方法：onCreate -> onStart -> onResume -> onPause -> onStop -> onDestroy
 - 当启动Activity的时候，会依次调用：onCreate > onStart -> onResume，此时Activity进入运行状态
 - 当启动另一个Activity，原来的Activity会调用：onPause -> onStoop，当调用onPause的时候进入暂停状态，特点是Activity不在前台，但是可见，如打开一个对话框。当调用onStop的时候进入停止状态，此时不在前台且不可见。
 - 当点击back返回上一个Activity，上一个Activity会调用onRestart -> onStart -> onResume。特别的，如果上一个Activity由于内存不足被杀，会调用onCreate -> onStart -> onResume，即重新创建。
 - 当点击Back退出Activity时候，会依次调用：onPause -> onStop -> onDestroy。此时为销毁状态。
 
 
##### 8.（）Activity启动模式

Activity有四种启动模式：Standard、Single'Top、SingleTask和SingleInstance
 - Standard：这是默认的启动模式，每次启动都会创建实例
 - SingleTop：栈顶复用模式，如果需要启动的Activity在栈顶，就重用该实例，否侧创建实例，如阅读类App的内容界面
 - SingleTask：栈内复用模式，如果栈内已有该Activity实例，就重用该实例，并将该Activity以上的Activity全部移除，如App的主界面。
 - SingleInstance：创建新的返回栈存放该实例。


##### 9.（）Handler-异步消息处理机制

消息机制由四部分组成：Handler、Message、MessageQueue和Lopper

 - Message：消息的载体
 - MessageQueue：消息队列，负责存放消息
 - Looper：负责读取消息，每个线程只有一个Looper
 - Handler：负责发送和处理消息，将当前线程切换到handler所关联的线程去处理消息

它的运行过程是这样的：handler发送一个消息出去，这个消息会被添加到MessageQueue中等待被处理，而Looper则会一直尝试从消息队列中取出消息，最后分发回handler的handleMessage()方法。


##### 10.（）AsyncTask异步任务

 - AsyncTask封装了Handler + 线程池，即使对异步消息处理机制完全不了解也可以十分简单从子线程切换到主线程。创建AsyncTask对象必须在主线程中，以为AsyncTask内部的Handler是静态的，加载AsyncTask类的时候会初始化Handler
 - 创建AsyncTask的子类时重写几个方法，其中doInBackground是运行在子线程，其他运行在主线程。制定了三个泛型参数，第一个是执行时传入的参数类型，也是doInBackground的传入参数，第二个是publishProgerss()的传入参数，第三个是onPostExecute()的传入参数。
 
 
##### 11.（）Service

Service是Android中实现程序后台运行的解决方案，它适合去执行那些不需要用户交互而且还要长时间运行的任务。服务依赖于创建服务时所在的进程，当该进程被杀，所依赖的服务也被杀


##### 12.（）Service生命周期

 - 以start方式启动：onCreate -> onStartCommand -> onDestroy

每次启动时，onCrete只执行一次，onStartCommand每次启动都执行
 - bind方式启动：onCreate -> onBind -> unBind -> onDestroy

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
    - EXACTLY：精确值模式。指定控件的宽高为具体数值，或者match_parent
    - AT_MOST：最大值模式，控件宽高为wrap_center，控件大小随着内容变化而变化，此时控件的尺寸只要不超过父控件允许的最大值即可
    - UNSPECIFIED：不指定其大小的测量模式，View想要多大就多大，绘制自定义View的时候才使用
    - View的onMeasure()默认使用EXACTLY模式，如果让自定义View支持wrap_centent，必须重写onMeasure方法来指定wrap_centent时的大小

 
 
##### 16.（）ViewGroup的测量和绘制



##### 17.（）Activity退出怎么保存数据(异常情况退出)

 - 在onSaveInstanceState()方法中通过Bundle保存数据，可以保存基本的数据类型和他们的数组，序列化对象及其数组，包裹类对象及其数组.
 - 在onRestoreInstanceState()方法中恢复数据。或者在onCreate()方法中恢复数据。注意onCreate()方法中的Bundle可能为空.
 - 异常情况：启动另一Activity、点击Home、旋转屏幕、锁屏、Activity被kill

##### 18.（）自定义View常用的构造函数及参数作用
常用的有三个，一个只有Context，一个只有Context和attributeSet，另外一个主题id
```java
public class CustomView extends View {
    public CustomView(Context context) {
        super(context);
    }
    public CustomView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }
    public CustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public CustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }
}
```

```kotlin
class CustomView : View {
    constructor(context: Context) : super(context) {}
    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs) {}
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr) {}
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int, defStyleRes: Int) 
            : super(context, attrs, defStyleAttr, defStyleRes) {
    }
}
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


##### 25.（）MessageQueue工作原理

MessageQueue有两个主要的方法：
 - enqueueMessage()：将消息入队
 - next()：将消息出队。内部是单链表实现的队列，因为要频繁的入队出队操作。next()方法是个阻塞方法，没有消息会一直阻塞在那里。


##### 26.（）Looper工作原理

 - 工作流程：Looper会不停滴从MessageQueue中查看是否有新消息，有则立刻处理，否则一直阻塞。
 - 构造方法：在构造方法中创建了MessageQueue并且保存了当前线程对象。
 - 创建Looper：Looper.prepare()或主线程中使用prepareMainLopper()。
 - 开启消息循环：Looper.loop()，该方法有个死循环，循环调用MessageQueue的next()方法来获取新消息。如果返回新消息，就会通过handler的dispatchMessage方法分发给响应的Handler。Handler的dispatchMessage方法是创建Handler时所使用的Looper中执行。这样就切换到指定的线程中去执行。
 - 退出消息循环：当Lopper调用了MessageQueue中的quit()或quitSafely()方法时，next()方法会返回null，这时即可退出Looper死循环。quiet()是立即退出；quitSafely()是处理完消息之后才退出。


##### 27.（）Handler工作原理

Handler的主要工作是发送消息和处理消息。post()方法最终会调用sendMessage()方法去发送消息，而sendMessage()最终会调用到MessageQueue的enqueMessage()方法，即把消息移入到消息队列中去，
这就完成了。Looper则取出消息，调用handler的dispatchMessage方法。dispatchMessage()方法有两种处理方式：
 - 当handler使用post方法发送消息，message.callback不为空，此时会选择调用handleCallback()方法，该方法会调用callback的run方法。CallBack实际上是Runnable对象，这里就是执行post()时的Runnable。如果msg.callback为空，则会调用handlerMessage，这里会调用创建Handler子类时重写的handleMessage()方法。



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