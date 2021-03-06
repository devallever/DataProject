**Note5**

##### 1.（Android）文件、SharePreference和SQLite使用场景

 - 文件：适用于存储一些简单的文本数据类型或二进制数据
 - SharePreference：适用于存储一些少量纪录的简单数据，如单个用户信息、配置信息。
 - SQLite：适用于大量记录的复杂关系型数据，如所有用户的信息
 
##### 2.（Android）Okhttp、Volley和Retrofit比较

 - OkHttp：对HttpUrlConnection进行封装的一个高性能的网络请求框架，支持同步、异步，支持http2.0、websocket。使用的时候需要自己再封装多一层。
 - Volley：轻量级的网络请求库，支持HttpUrlConnection、okhttp、ImageLoader，封装好。缺点就是不支持post大数据，不适合上传文件。
 - Retrofit：封装了Okhttp，RESTful风格的网络请求框架，支持Rx，解耦更彻底，没什么缺点吧。
 
##### 3.（Android）MVC和MVP

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



##### 4. （Android）FragmentPagerAdapter和FragmentStatePageAdapter区别

 - FragmentPageAdapter：每一个生成的Fragment都会保存到内存中，切换后不会被销毁。适用于数量少、静态的Fragment
 - FragmentStatePageAdapter：只保留当前Fragment，切换后被销毁，释放资源。适用于数量多、数据动态性大的Fragment
 
 
##### 5.（Android）LIstView优化方式

 - 缓存机制：getView()方法中有个view参数，就是用来缓存每一项item。如果非空的话就直接引用
 - viewHolder模式：充分利用ListView的视图缓存机制，避免每次getView的时候都要通过findViewById去实例化控件。可以大大提高ListView的性能。具体的做法是在Adapter中定义ViewHolder内部类，把Item中的控件，作为viewHolder的成员变量，把viewHolder作为tag设置给view
 
 
##### 6.（Android）视图动画和属性动画的区别

 - 视图动画：通过矩阵运算修改View的显示
 - 属性动画：通过修改对象的属性值实现动画，是真正意义上的动画。

视图动画和属性动画的区别：例如将一个View使用视图动画移动到别的位置，但它点击监听事件还在原来的区域；而使用属性动画它的点击区域也随着控件的移动而移动。



##### 7.（Android）Activity的生命周期

 - 四种状态：运行状态、暂停状态、停止状态和销毁状态
 - 回调方法：onCreate -> onStart -> onResume -> onPause -> onStop -> onDestroy
 - 当启动Activity的时候，会依次调用：onCreate > onStart -> onResume，此时Activity进入运行状态
 - 当启动另一个Activity，原来的Activity会调用：onPause -> onStoop，当调用onPause的时候进入暂停状态，特点是Activity不在前台，但是可见，如打开一个对话框。当调用onStop的时候进入停止状态，此时不在前台且不可见。
 - 当点击back返回上一个Activity，上一个Activity会调用onRestart -> onStart -> onResume。特别的，如果上一个Activity由于内存不足被杀，会调用onCreate -> onStart -> onResume，即重新创建。
 - 当点击Back退出Activity时候，会依次调用：onPause -> onStop -> onDestroy。此时为销毁状态。
 
 
##### 8.（Android）Activity启动模式

Activity有四种启动模式：Standard、Single'Top、SingleTask和SingleInstance

 - Standard：这是默认的启动模式，每次启动都会创建实例
 - SingleTop：栈顶复用模式，如果需要启动的Activity在栈顶，就重用该实例，否侧创建实例，如阅读类App的内容界面
 - SingleTask：栈内复用模式，如果栈内已有该Activity实例，就重用该实例，并将该Activity以上的Activity全部移除，如App的主界面。
 - SingleInstance：创建新的返回栈存放该实例。


##### 9.（Android）Handler-异步消息处理机制

消息机制由四部分组成：Handler、Message、MessageQueue和Lopper

 - Message：消息的载体
 - MessageQueue：消息队列，负责存放消息
 - Looper：负责读取消息，每个线程只有一个Looper
 - Handler：负责发送和处理消息，将当前线程切换到handler所关联的线程去处理消息

它的运行过程是这样的：handler发送一个消息出去，这个消息会被添加到MessageQueue中等待被处理，而Looper则会一直尝试从消息队列中取出消息，最后分发回handler的handleMessage()方法。


##### 10.（Android）AsyncTask异步任务

 - AsyncTask封装了Handler + 线程池，即使对异步消息处理机制完全不了解也可以十分简单从子线程切换到主线程。创建AsyncTask对象必须在主线程中，以为AsyncTask内部的Handler是静态的，加载AsyncTask类的时候会初始化Handler
 - 创建AsyncTask的子类时重写几个方法，其中doInBackground是运行在子线程，其他运行在主线程。制定了三个泛型参数，第一个是执行时传入的参数类型，也是doInBackground的传入参数，第二个是publishProgerss()的传入参数，第三个是onPostExecute()的传入参数。
 
 
##### 11.（Android）Service

Service是Android中实现程序后台运行的解决方案，它适合去执行那些不需要用户交互而且还要长时间运行的任务。服务依赖于创建服务时所在的进程，当该进程被杀，所依赖的服务也被杀


##### 12.（Android）Service生命周期

 - 以start方式启动：onCreate -> onStartCommand -> onDestroy

每次启动时，onCrete只执行一次，onStartCommand每次启动都执行
 - bind方式启动：onCreate -> onBind -> unBind -> onDestroy

##### 13.（Android）Android系统架构

 - Linux内存层：底层核心，包含Android系统核心服务。包括硬件驱动、进程管理等
 - 标准库和运行时：
    - Dalvik和ART：Android运行环境虚拟机，每个app都会分配独立的虚拟机保证相互之间不会被干扰，运行时编译。ART是安装时编译。
    - 标准库：开发者在开源环境中可以使用的开发库
 - Framework：构建应用程序用到的各种API
 
 
##### 14.（Android）Android控件架构


##### 15.（Android）View的测量

 - onMeasure()：告诉系统画一个多大的View
 - MeasureSpec：测量规格，用来获取测量模式和测量值。MeasureSpec是一个32位的int值，高2位为测量模式，低30位为测量大小，通过MeasureSpec的静态方法getMode、getSize获得。
 - 三种测量模式：
    - EXACTLY：精确值模式。指定控件的宽高为具体数值，或者match_parent
    - AT_MOST：最大值模式，控件宽高为wrap_center，控件大小随着内容变化而变化，此时控件的尺寸只要不超过父控件允许的最大值即可
    - UNSPECIFIED：不指定其大小的测量模式，View想要多大就多大，绘制自定义View的时候才使用
 - View的onMeasure()默认使用EXACTLY模式，如果让自定义View支持wrap_centent，必须重写onMeasure方法来指定wrap_centent时的大小
 - 重写onMeasure方法时，默认调用父类的onMeasure，在父类的onMeasure方法中会调用setMeasureDimension()把测量后的宽高值设置进去。而我们重写onMeasure的目的就是把我们重新定义的宽高值设置进去。
 - 自定义测量值的模板代码

```

```

 
 
##### 16.（Android）ViewGroup的测量和绘制



##### 17.（Android）Activity退出怎么保存数据(异常情况退出)

 - 在onSaveInstanceState()方法中通过Bundle保存数据，可以保存基本的数据类型和他们的数组，序列化对象及其数组，包裹类对象及其数组.
 - 在onRestoreInstanceState()方法中恢复数据。或者在onCreate()方法中恢复数据。注意onCreate()方法中的Bundle可能为空.
 - 异常情况：启动另一Activity、点击Home、旋转屏幕、锁屏、Activity被kill

##### 18.（Android）自定义View常用的构造函数及参数作用
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


##### 19.（Android）RecyclerView和ListView的区别

 - 最大的区别：RecyclerView强制使用ViewHolder模式优化列表，
 - RecyclerView通过设置LayoutManager快速实现列表，网格布局
 - RecyclerView在添加、删除Item的时候有默认的动画
 - RecyclerView的事件监听需要自己实现
 

##### 20.（Android）自定义View的几种方式

自定义View的方式有三种：组合控件、继承原有控件和完全自定义View。

 - 组合控件：将一些基础组件封装起来形成新的控件，如通用的标题栏
 - 继承原有控件：在原有控件的基础上添加方法来扩展功能，或者修改原有的显示逻辑
 - 完全自定义View：需要重写onMeasure()实现测量逻辑，需要重写onDraw方法实现绘制逻辑，需要注意支持wrap_parent和Padding
 
##### 21.（Android）Broadcast广播机制

广播分为：标准广播、有序广播和本地广播.

 - 标准广播：是一种异步广播，所有广播接收器几乎同时受到广播
 - 有序广播：是一种同步广播，优先级较高的广播接收器西安搜狐到广播，可拦截广播和传递数据
 - 本地广播：只有当前应用可以接收

注册广播的方式有两种：动态注册和静态注册

 - 静态注册：是指在AndroidMenifest配置中注册，这种方式注册的广播接收器白应用没运行也能收到广播
 - 动态注册：是指在代码中注册，这种方式注册的广播接收器在运行应用并且注册后才能收到广播，注意需要取消注册
动态注册的优先级比静态注册的高


##### 22.（Android）动画的几种方式和区别
 动画分为三种：帧动画、视图动画和属性动画
 
 - 帧动画：通过切换图片的方式实现动画效果
 - 视图动画：通过矩阵运算修改View的显示
 - 属性动画：通过修改对象的属性值实现动画，是真正意义上的动画。
 
视图动画和属性动画的区别：例如将一个View使用视图动画移动到别的位置，但它点击监听事件还在原来的区域；而使用属性动画它的点击区域也随着控件的移动而移动。


##### 23.（Android）运行时权限 

 - 6.0以前，权限在安装的时候申请，如果不同意的权限，用户可以选择不安装
 - 6.0以后，不在安装时申请，而是把权限分为普通权限和危险权限。普通权限默认已授权，危险权限分为9组20多个，包括读取联系人、电话、位置和 存储等。
 
 
##### 24.（Android）Handler消息机制


消息机制由四部分组成：Handler、Message、MessageQueue和Lopper

 - Message：消息的载体
 - MessageQueue：消息队列，负责存放消息
 - Looper：负责读取消息，每个线程只有一个Looper
 - Handler：负责发送和处理消息，将当前线程切换到handler所关联的线程去处理消息

它的运行过程是这样的：handler发送一个消息出去，这个消息会被添加到MessageQueue中等待被处理，而Looper则会一直尝试从消息队列中取出消息，最后分发回handler的handleMessage()方法。


##### 25.（Android）MessageQueue工作原理

MessageQueue有两个主要的方法：

 - enqueueMessage()：将消息入队
 - next()：将消息出队。内部是单链表实现的队列，因为要频繁的入队出队操作。next()方法是个阻塞方法，没有消息会一直阻塞在那里。


##### 26.（Android）Looper工作原理

 - 工作流程：Looper会不停滴从MessageQueue中查看是否有新消息，有则立刻处理，否则一直阻塞。
 - 构造方法：在构造方法中创建了MessageQueue并且保存了当前线程对象。
 - 创建Looper：Looper.prepare()或主线程中使用prepareMainLopper()。
 - 开启消息循环：Looper.loop()，该方法有个死循环，循环调用MessageQueue的next()方法来获取新消息。如果返回新消息，就会通过handler的dispatchMessage方法分发给响应的Handler。Handler的dispatchMessage方法是创建Handler时所使用的Looper中执行。这样就切换到指定的线程中去执行。
 - 退出消息循环：当Lopper调用了MessageQueue中的quit()或quitSafely()方法时，next()方法会返回null，这时即可退出Looper死循环。quiet()是立即退出；quitSafely()是处理完消息之后才退出。


##### 27.（Android）Handler工作原理

Handler的主要工作是发送消息和处理消息。post()方法最终会调用sendMessage()方法去发送消息，而sendMessage()最终会调用到MessageQueue的enqueMessage()方法，即把消息移入到消息队列中去，
这就完成了。Looper则取出消息，调用handler的dispatchMessage方法。dispatchMessage()方法有两种处理方式：

 - 当handler使用post方法发送消息，message.callback不为空，此时会选择调用handleCallback()方法，该方法会调用callback的run方法。CallBack实际上是Runnable对象，这里就是执行post()时的Runnable。如果msg.callback为空，则会调用handlerMessage，这里会调用创建Handler子类时重写的handleMessage()方法。

##### 28.（Android）使用线程的方式和原理

 - Thread：Java传统的线程
 - AsyncTask：线程池 + Handler
 - HandlerThread：带Handler的Thread
 - IntentService：HandlerThread + Handler
 - 线程池：


##### 29.（Android）AsyncTask的工作原理

 - 当执行AsyncTask的execute()时，会调用executeExecutor()方法。在这个方法里面，先调用preExecutor()，然后开始线程池的执行。
 - 首先把任务放到一个任务队列中去，如果当前没有活动的任务，就执行下一个任务。这时，另一线程池将会执行任务队列中的任务。
 - 可以看出，执行这些任务是串行的。任务的run()方法最终会调用mWorker的call()方法。因此这个call()方法就是线程池主要执行的工作。
 - call()中调用了doInBackground()方法，该方法方返回值作为postResult()方法的参数。该方法通过Handler发送执行完成这个消息，然后回调Handler的handleMessage()方法。该方法会调用AsyncTask的finish()方法，并且传入DoInBackground()的返回参数。
 - finish()最终会调用onPostExecute(),即切换到主线程去处理UI交互。
 - AsyncTask里面的Handler是静态的，在类加载到时候会初始化，因此必须在主线程中创建AsyncTask。


##### 30.（Android）HandlerThread原理

 - HandlerThread也是线程，它继承了Thread。
 - 它的run()方法创建了一个Looper对象并开启消息循环。
 - 在外部使用HandlerThread的时候需要创建Handler，并与HandlerThread的Looper进行关联，通过handler消息机制去执行一个特定的任务。


##### 31.（Android）IntentService原理

 - IntentService是一种服务组件，继承了Service，并且是一个抽象类，必须使用IntentService的子类。
 - IntentService内部使用HandlerThread + Handler。
 - 在onCrate()方法中创建了HandlerThread对象和Handler对象，并且与HandlerThread的Looper进行关联。
 - 然后在onStartCommand()方法中仅仅调用onStart()方法。
 - 在onStart()方法中，把Intent封装成Message，用handler发送出去。
 - 回调到handler的handleMessage()，该方法会调用onHandleIntent()和stopSelf()方法。
 - onHandleIntent()方法就是我们创建IntentService子类时重写的那个方法，这个方法会在子线程中执行，因为handler关联了HandlerThread的Looper，而HandlerThread属于子线程，因此，handleMessage()就在子线程中执行。
 - 执行完毕后，就会调用stopSelf()去结束服务。


##### 32.（Android）实现多进程的方式

 - 给四大组件指定process属性
 - 通过jni在native层去fork一个新的进程(非常规)
 - 多进程带来的问题
    - 静态成员和单例完全失效
    - 线程同步机制失效
    - sharepreference可靠性下降
    - Application多次创建
    

##### 33.（Android）线程与进程

 - 进程：是CPU分配资源的最小单位。例如一个应用就是一个进程(也可以多个进程)，CPU为这个应用分配资源，如内存
 - 线程：是CPU调度的最小单位。CPU执行的是线程

进程之间不能共享资源，线程可以共享所在进程的资源。  
一个进程至少包含一个线程，线程依赖进程而存在。


##### 34.（Android）IPC进程间通信的方式

Android跨进程通信的方式有：Intent、文件共享、ContentProvider、广播、Socket和AIDL

 - Intent：
 - 文件共享：两个进程读写一个文件来传递数据
 - ContentProvider：Andorid专用于不同应用间共享数据
 - 广播：
 - Socket：网络上不同应用的数据交互
 - AIDL：

##### 35.（Android）IPC各种方式的优缺点及应用场景

##### 36.（Android）Binder介绍

 - IPC角度：是Android中一种跨进程通信方式
 - Android Framework角度：是SeviceManager连接各种Manager和相应MessageService的桥梁
 - Android应用层角度：是客户端与服务端进行通信交互的媒介，bindService返回Binder


##### 37.（）内存泄漏与内存溢出
 - 内存溢出：指程序申请内存时，没有足够的空间供其使用，即内从不够用
 - 内存泄漏：指程序申请内存后，无法释放已申请的内存空间。过多的内存泄漏可能导致内存溢出


##### 38.（Android）引起内存泄漏的场景

 - 使用轻量级的数据结构，如ArrayMap、SparseAray来代替HashMap
 - 不要使用枚举Enum
 - Bitmap高效加载
 - 不要使用String进行字符串拼接
 - 非静态内部类导致的内存泄漏，非静态内部类是有外部类的隐式引用，如果内部类生命周期长于Activity，会导致Activity无法回收。建议使用静态内部类 + WeakReference方式引用外界资源
 - 匿名内部类，跟非静态内部类一样持有外部类的隐式引用，比较常见的有耗时的handler、耗时的Thread都会造成内存泄漏
 - context持有导致内存泄漏，ActivityContext传递到其他实例可能导致自身被引用而发生内存泄漏。解决办法：对于大部分非必须ActivityContext(Dialog)应该使用ApplicationContext
 - 谨慎使用static对象，static对象生命周期过长
 - 谨慎使用单例中不合理的持有，单例中的对象生命周期应用应用一致，注意不要在单例中进行不必要的外界持有引用，如非必要，需要外部变量生命周期结束的时候解除引用，赋值null
 - 关闭无用的连接，cursor、io、数据库、网络
 - 属性动画导致的内存泄漏，退出界面时没有停止动画


##### 39.（Android）内存泄漏检测

内存泄漏检测工具：MAT(Memory AnalyzeTool)、LeakCanary

##### 40.（Android）性能优化的方式
主要包括：布局、绘制、内存、响应速度、ListView、Bitmap、线程

 - 线程优化(补充)：尽量使用线程池，而不是每次都创建一个Thread

性能优化建议：

 - 避免创建过多对象
 - 不要过多使用枚举，枚举占用空间比整形大
 - 常量使用static final修饰
 - 使用Android特定的数据结构：ArrayMap、SparseArray、Pair
 - 适当使用软引用和弱引用
 - 采用内存缓存和磁盘缓存
 - 尽量采用静态内部类，这样可以避免潜在由于内部类而导致的内存泄漏


##### 41.（Android）性能优化工具

 - hierarchyviewer：布局检测工具，sdk/tools/hierachyview
 - Lint：代码优化提示，工具栏/Analyze/Inspect Code
 - Memory Monitor
 - TraceView：可视化性能调查工具，用来分析TraceView日志
 - MAT：分析App内存状态
 - Android Device Monitor

##### 42.（Android）Activity的生命周期


##### 43. （Android）从网络上加载图片的思路

 - 先从内存中看有没有该图片的缓存，没有的话访问磁盘，没有的话才从网上下载。
 - 下载的时候下载原图，缓存的磁盘的时候可以根据情况对图片进行压缩处理。
 - 如何压缩？通过Bitmap的Option对象的inSampleSize属性，可以设置采样率，默认值为1，表示不压缩。改为n的时候，图片的宽高都会变成原来的1/n，大小为1/n^2。


##### 44.（Android）图片加载框架

 - 从网络上下载图片作缓存的时候是缓存原图
 - 从磁盘加载图片是加载压缩后的图片，从磁盘缓存添加到内存缓存也是压缩后的图片。
 - 利用Bitmap的Option的inSampleSize和inJustDecodeBound
 - LruCache和DiskLruCache，分别作为内存缓存和磁盘缓存
 - 同步加载：需要手动开启线程
 - 异步加载：创建Runnable添加到线程池
 - 图片错位：检查imageUrl与ImageView的Tag是否一致


##### 45.（Android）断点下载的思路

 - 首先判断本地文件是否存在，如果存在表示下了一部分(或以下载完)，获取文件大小(长度)。在Http头部添加RANGE属性，指明从哪个字节开始传递。
 - 存储的时候使用RandomAssetFile进行读写，通过seek()方法指定重哪个位置开始写入。




##### 46.（Java）字符和字符串所占字节数

 - char：2字节
 - String：
     - 英文字符串的字节数与英文字母相同，如“hello”占5个字节
     - 中文字符串根据编码而定，如GBK下，一个汉字占2个字节；UTF-8下，一个汉字占3个字节
     
##### 47.（Java）String、StringBuffered和StringBuilder的区别。（）

##### 48.（Java）equals与“==”的区别

 - equals：比较内容是否相等
 - ==：比较地址是否相等
	JVM有个区域叫字符串缓冲池，使用String a = "abc"时，会检查缓冲池中是否有相同内容的对象，有就直接引用，没有才新建一个值为“abc”的对象，并放入缓冲池，再把引用返回给a。而new String("abc")则创建一个新的String对象，在堆内存开辟新的空间来存放这个对象。

```
String str1 = "abc";
String str2 = "abc";
str1.equals(str2); //true
```

```
String str1 = "abc";
String str2 = new String("abc");
str1.equals(str2); //false
```

equals()是Object类的方法，内部通过比较对象地址是否相等。而String类重写了equals()方法，使得String类的equals()能够比较字符串的内容是否相等。"= =" 亦可以比较基本类型的值是否相等。

##### 49.（Java）八种基本类型及其封装类

 / | 类型 | 字节数 | 封装类
--- | --- | --- | --- 
整形 |byge | 1  | Byte
整形 | short | 2 | Short
整形 | int | 4 | Integer
整形 | long | 8 | Long
浮点型 | float | 4 | Float
浮点型 | double | 8 | Double
字符型 | char | 2 | Character
布尔型 | boolean | | Boolean


##### 50.（Java）Object类常用方法

 - getClass()：获取运行时类型
 - toString()：
 - hashCode()：JVM在每new一个对象时都会把对象丢到一个哈希表中，下次做对象比较或取出对象时，根据hashCode从哈希表中取出这个对象，提高读取对象的效率。用于哈希查找，可以减少在查找中使用equals()的次数
 - equals()
 - finalize()
 - wait()：使当前线程等待该对象的锁。直到其他线程调用该对象的notify()/notifyAll()、时间点到或其他线程使用interrupt，此时该线程卡已被调度。
 - notify()/notifyAll()：

##### 51.（Java）静态内部类和非静态内部类的区别



##### 52.（Java）异常的分类

 - Throwable
    - Error
    - Exception
        - IOException
        - RuntimeException

根据派生分类：

 - 派生于RuntimeException：数组越界、空指针、类型转换、
 - 非派生于RuntimeException：在文件尾部读写数据、打开一个不存在的文件

根据检查分类：

 - 未检查的异常：派生于Error或RuntimeException
 - 已检查的异常：其他所有异常

##### 53.（Java）List、Set和Map的区别

 - List：List集合以线性的方式存储，集合中可以存放重复的元素
 - Set：Set是一种最简单的集合，集合中的对象不按特定的方式排序，没有重复的对象
 - Map：Map是一种把键对象和值对象映射的集合


##### 54.（Java）线程安全集合与非线程安全集合

 - 非线程安全集合：ArrayList、LinkedList、HashMap、HashSet
 - 线程安全集合：CopyOrWriteArrayList、ConcurrentHashMap


##### 55.（Java）ArrayList和Vector的区别

 - Vector：线程安全、低效、数组实现、可设置增长因子
 - ArrayList：非线程安全、高效、数组实现、不可设置增长因子、适用于数据量较大的集合


##### 56.（Java）四种权限关键字的区别：

 - 默认：包访问权限
 - private：当前类可访问
 - protect：包内和子类可访问
 - public：


##### 57.（Java）final的作用

 - 修饰变量：不可修改
 - 修饰方法：不可重写
 - 修饰类：不可继承


##### 58.（Java）JVM的基本构成

 - 类加载器(ClassLoader)：将class文件加载到JVM
 - 内存区/运行时数据区(Runtime Data Area)：是在JVM运行的时候操作所分配的内存区
 - 执行引擎(Execution Engine)：执行class文件中包含的字节码指令
 - 本地方法接口：调用C和C++实现的本地方法


##### 59.（Java）JVM的内存模型

 - 堆区：存放实例对象；垃圾回收区域；子线程共享
 - 元空间：存放类元信息、字段、静态属性、方法、常量等，线程共享
 - 虚拟机栈：描述Java方法执行的内存区域，线程私有。每个方法都是用一个栈帧去表示，栈帧是方法运行的基本结构，存放局部变量表，操作栈、方法返回地址；
 - 本地方法栈：功能和虚拟机栈差不多，为Native方法服务
 - 程序计数器：


##### 60.（Java）堆内存模型

 - 堆内存分两部分，新生代：老年代 = 1:2 
 - 新生代分为三部分，Eden：survive0：survive1 = 8:1:1
 - 老年代存放生命周期较长的存活对象
 - Eden区存放新生对象
 - survivor区存放没测垃圾回收后的存活对象

##### 61.（Java）GC机制

主要完成两件事：检查出垃圾、回收垃圾


##### 62.（Java）检测垃圾的方法

 - 引用计数法：给对象添加一个引用计数器，每当有地方引用该对象，计数器+1，引用失效-1，当计数器为0时，即表示垃圾。缺点，无法检测循环引用。
 - 可达性分析法(根搜索算法)：以根集对象为起点惊醒搜索，如果有对象不可达，就是垃圾。这里的根集包括Java栈中引用的对象、常量池中引用的对象和本地方法中引用的对象。


##### 63.（Java）GC垃圾回收算法

 - 标记清除法：使用可达性分析法，将可达的对象做上标记，遍历完成之后，垃圾收集器对堆内存从头到尾线性遍历，回收那些没被标记的对象。缺点，容易产生垃圾。
 - 复制算法：将内存区分为活动区和空闲区，比例1:1。对象分配到活动区，活动区满了，触发复制算法，将活动区中存活对象严格按照内存地址依次排序，全部复制到空闲区，并更新对象的内存引用地址。将空闲区与活动区交换的同时，垃圾对象已经=一次性回收。
 - 标记整理算法：先标记可达对象，然后移动所有存活对象，按照内存地址依次排列，最后将末端内存地址以后的内存全部回收。
 - 分代回收算法：
    - 分配对象到Eden区
    - 当Eden区满了，触发一次MinerGC，将Eden区存活对象复制到survivor0区，然后清空Eden区
    - 当survivor0区满了，将Eden区和survivor0区存活对象复制到survivor1区，清空Eden和survivor0区
    - 此时，survivor0区为空，交换survivor0和survivor1区，即保持survivor1区为空，如此往复
    - 当survivor1区不足以存放Eden和survivor0区的存活对象时，直接将存活对象放到老年代
    - 当老年代满了，触发FullGC，也就是将新生代好老年代都回收。
   

##### 64.（Java）常用线程池及其构造方法参数含义

 - FixPoolExecutor：只有核心线程，可设置核心线程数，没有非核心线程，执行效率高，响应快
 - CachePoolExecutor：只有非核心线程，线程数无限制，等待时长60s，适用于大量耗时少的任务
 - SchedulePoolExecutor：核心数 = 非核心线程数，适用于周期性任务
 - SinglePoolExecutor：只有一个核心线程，适用于同步任务，不必考虑线程同步问题。



##### 65.（Java）强弱软虚四种引用

 - 强引用：直接的对象引用。new，只要强引用存在，GC必不回收
 - 弱引用：系统内存不足时，该对象被GC回收
 - 软引用：发生GC时，必定会收。
 - 虚引用：


##### 66.（Java）反射机制

 - 在运行中，对任意一个类，都能够知道这个类的所有属性和方法；对任意一个对象，都能调用它任意一个方法和属性。
 - 这种动态获取信息及动态调用对象方法的功能成为Java的反射机制。

##### 67.（Java）xml和json的区别

 - 可读性： xml > json
 - 传输线： xml < json
 - 性能：   xml < json
 - 解析复杂性：xml > json





















