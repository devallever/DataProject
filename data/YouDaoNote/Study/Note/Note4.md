##### 1.（Java）进程与线程的区别


 - 进程：是CPU分配资源的最小单位。例如一个应用就是一个进程(也可以多个进程)，CPU为这个应用分配资源，如内存
 - 线程：是CPU调度的最小单位。CPU执行的是线程

进程之间不能共享资源，线程可以共享所在进程的资源。一个进程至少包含一个线程，线程依赖进程而存在。


##### 2.（Android）实现多进程的方式

 - 给四大组件指定process属性
 - 通过jni在native层去fork一个新的进程(非常规)
 - 多进程带来的问题
    - 静态成员和单例完全失效
    - 线程同步机制失效
    - sharepreference可靠性下降
    - Application多次创建
    

##### 3.（Android）实现IPC的方式

Android跨进程通信的方式有：Intent、文件共享、ContentProvider、广播、Socket和AIDL
 - Intent：
 - 文件共享：两个进程读写一个文件来传递数据
 - ContentProvider：Andorid专用于不同应用间共享数据
 - 广播：
 - Socket：网络上不同应用的数据交互
 - AIDL：

##### 4.（设计模式）观察者模式

 - 定义：定义对象间一种一对多的关系，当一个对象(Subject)改变状态，则所有依赖它的对象多会得到通知并自动更新
 - 使用场景：关联行为场景；事件多级触发场景；跨系统的消息交换场景，如消息队列、事件总线


##### 5.（设计模式）单例模式

 - 饿汉模式

这家伙太饥饿难耐啦，什么也不干，也不想想在多线程时候怎么保证单例对象的唯一性。

```java
public class HungrySingleton {
    private static final HungrySingleton hungrySingleton = new HungrySingleton();
    private HungrySingleton(){}

    public static HungrySingleton getInstance(){
        return hungrySingleton;
    }
    /**防止反序列化重新构建对象*/
    private Object readResolve() throws ObjectStreamException{
        return hungrySingleton;
    }
}
```

 - 懒汉模式

这家伙懂事点点，考虑到多线程时候怎么保证单例对象的唯一性。就加个synchronized关键字嘛，每次调用该方法都进行同步，但是反应还是有点迟钝。
```
public class LazySingleton {
    private static LazySingleton lazySingleton = null;
    private LazySingleton(){}

    /**偷懒！ 只判断一次，造成每次调用该方法都进行同步*/
    public static synchronized LazySingleton getInstance(){
        if (lazySingleton == null){
            lazySingleton = new LazySingleton();
        }
        return lazySingleton;
    }
    /**防止反序列化重新构建对象*/
    private Object readResolve() throws ObjectStreamException {
        return lazySingleton;
    }
}

```

 - 双重检查锁定

还是这家伙还是挺老实的，干活不怕苦不怕累，使用了两次判空操作，第一次判空防止不必要的同步，第二次判空保证在null情况下才创建实例。

```java
public class DCLSingleton {
    private static volatile DCLSingleton dclSingleton = null;
    private DCLSingleton(){}
    public static DCLSingleton getInstance(){
        if (dclSingleton == null){  //避免不必要的同步
            synchronized (DCLSingleton.class){
                if (dclSingleton == null){
                    dclSingleton = new DCLSingleton();
                }
            }
        }
        return dclSingleton;
    }
    /**防止反序列化重新构建对象*/
    private Object readResolve() throws ObjectStreamException {
        return dclSingleton;
    }
}
```
 - 静态内部类

这家伙比较靠谱了，既保证线程安全，也保证单例唯一性，又延迟了单例的实例化，mua
```
public class StaticInnerSingleton {
    private StaticInnerSingleton(){}
    public static StaticInnerSingleton getInstance(){
        return StaticInnerSingleHolder.staticInnerSingleton;
    }

    private static class StaticInnerSingleHolder{
        private static final StaticInnerSingleton staticInnerSingleton = new StaticInnerSingleton();
    }
    /**防止反序列化重新构建对象*/
    private Object readResolve() throws ObjectStreamException {
        return StaticInnerSingleHolder.staticInnerSingleton;
    }
}
```

 - 注意事项

以上实现方式，在反序列化时候会重新创建对象，所以必须加入以下方法

```
/**防止反序列化重新构建对象*/
private Object readResolve() throws ObjectStreamException {
        return StaticInnerSingleHolder.staticInnerSingleton;
}
```


##### 6.（Java）内存泄漏与内存溢出

 - 内存溢出：指程序申请内存时，没有足够的空间供其使用，即内从不够用
 - 内存泄漏：指程序申请内存后，无法释放已申请的内存空间。过多的内存泄漏可能导致内存溢出

内存泄漏检测工具：MAT(Memory AnalyzeTool)、LeakCanary

##### 7.（Android）如何避免内存泄漏

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


##### 9.（Android）性能优化
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

性能优化工具：
 - hierarchyviewer：布局检测工具，sdk/tools/hierachyview
 - Lint：代码优化提示，工具栏/Analyze/Inspect Code
 - Memory Monitor
 - TraceView：可视化性能调查工具，用来分析TraceView日志
 - MAT：分析App内存状态
 - Android Device Monitor


##### 10.（Android）动画分类、原理和区别

动画分为三种：帧动画、视图动画和属性动画
 - 帧动画：通过切换图片的方式实现动画效果
 - 视图动画：通过矩阵运算修改View的显示
 - 属性动画：通过修改对象的属性值实现动画，是真正意义上的动画。
视图动画和属性动画的区别：例如将一个View使用视图动画移动到别的位置，但它点击监听事件还在原来的区域；而使用属性动画它的点击区域也随着控件的移动而移动。



##### 11.（Android）IPC各种方式优缺点及应用场景

方式 | 优点 | 缺点 | 使用场景
---|---|---|---
Bundle | 简单易用 | 只能传输Bundle所支持的数据类型 | 四大组件的通信
文件共享 | 简单易用 | 不适合高并发，无法x到进程即时通信 | 无高并发访问，交换简单实时性不高的场景



##### 12.（Android）Binder的简单介绍

 - IPC角度：是Android中一种跨进程通信方式
 - Android Framework角度：是SeviceManager连接各种Manager和相应MessageService的桥梁
 - Android应用层角度：是客户端与服务端进行通信交互的媒介，bindService返回Binder


##### 13.（Android）Pull与SAX解析XML比较

SAX解析XML
 - 事件驱动型，不会将整个文档度入内存，而是在读取文档的时候就解析它。会遍历所有节点，那么如果只要用文档中的一小部分数据，这时使用SAX解析就浪费处理器的资源
 - 给予“推”的解析方式，速度快，占用内存小，实现起来比较麻烦，只支持XML读取，不支持写入
 - SAX的工作方式是自动将事件推入注册事件的处理器进行处理，因此不能控制事件的处理主动结束
 - 只要开始解析，就必须解析完成

Pull解析XML
 - 基于“拉”的解析方式，即应用程序根据自己的需要控制解析器读取。这种方式速度快，占用内存小，接口简单，容易编程。
 - Pull的工作方式是主动从解析器中获取事件，因此可以在满足的条件后不再获取事件，结束解析。
 - Pull使用一个循环结构，主动从解析器中获取事件和值，可以随时跳出

> 推荐使用Pull


##### 14.（Android）Serializable和Parcelable的原理和区别

 - Serialized：使用简单，序列化和反序列化要执行大量的IO操作，低效
 - Parcelable：使用麻烦，在内存中操作，高效
 - Serialized适用于存储到本地或网络传输；Parcelable适用于在内存中操作。


##### 15.（数据结构和算法）常用排序算法及其时间复杂度

 - 简单选择排序（n^2）
 - 直接插入排序（n^2）
 - 希尔排序（n^1.3）
 - 折半插入排序（）
 - 冒泡排序（n^2）
 - 快速排序（n*logn）


##### 16.（数据结构和算法）顺序表和链表比较


顺序表 | 链表
---|---
预先分配内存，空间浪费或溢出 | 动态分配
随机存取，按位置访问元素O(1) | 顺序存取O(n)
插入删除O(n) | 插入删除O(1)
适用于长度变化不大，很少进行插入删除操作，经常按位置访问元素 | 频繁进行插入或删除操作


##### 17.（数据结构和算法）二叉树常用的性质

 - 第i层最多有2^(i-1)个结点
 - 深度为h的二叉树最多有(2^h)-1个结点
 - 叶子数 = 度为2的结点数 + 1


##### 18. （数据结构和算法）反转字符串的方法

 - stringBuilder.reverse()
 - 递归交换
 - 字符串拼接
 - 数组倒叙遍历


##### 19.（数据结构和算法）二叉树的先序序列为ABDFCEGH，中序遍历为BFDAGEHC，求后序遍历。
FDBGHECA


##### 20.（Android）Service和IntentService的区别

 - Service：普通服务，运行在主线程，需要开启线程去执行耗时任务，且任务执行完成之后不会自动停止服务。
 - IntentService：继承了Service，可用于执行后天耗时的任务，当任务结束后自动停止。因为IntentService是服务组件，因此优先级比普通线程高很多，比较适合执行一些优先级较高的后台任务，不易被杀。IntentService封装了HandlerThread和Handler


##### 21.（Android）HnadlerThread和Thread的区别

 - Thread：普通线程，主要用于在run方法中执行一些耗时的任务
 - HandlerThread：继承了Thread，在内部创建了消息队列，在外界需要Handler的消息方式去通知HandlerThread执行一个具体的任务。


##### 22.（数据结构和算法）已知结点的权集W={10,12,4,7,5,18,2}，构造一棵哈夫曼树，并求带权路径长度


##### 23.（Java）位运算基本操作

 - \>>：右移
 - <<：左移


规则：
 - 正数右移 高位补0 
 - 负数右移 高位补1
 - 左移都补0

例如
```
System.out.println(5<<2);
//结果20
```

解析：
 - (int)5(2) = 0000 0000 0000 0000 0000 0000 0000 0101
 - (int)2(2) = 0000 0000 0000 0000 0000 0000
0001 0100

右移同理

与或非异或基本操作

| | | 与 & | 或 \| | 异或 ^ 
---|---|---|---|---
0|0|0|0|0
0|1|0|1|1
1|0|0|1|1
1|1|1|1|0

例如
```
System.out.println(5|3);
//结果7
```
... 0101  
... 0011  
\| ... 0111


##### 24.（Android）RelativeLayout、FrameLayout和LinearLayout性能比较

 - 三个布局的onLayout、onDraw耗时相差不大
 - 在onMeasure中，RelativeLayout耗时比其他两个大很多，原因是RelativeLayout会对其子View进行onMeasure两次，横向和纵向。因为RelativeLayout中子View的排列方式是基于彼此的依赖关系。而LinearLayout只需要一次measure，当weight属性存在时，调用两次measure
 - 首选LinearLayout > FrameLayout > RelativeLayout
 

##### 25.（Android）Picasso和Glide比较

 - 占用内存： Glide < Picasso， Glide的Bitmap使用RGB_565, Picasse 使用RGB_8888
 - 缓存：Picasso缓存原始图片，Glide缓存适当尺寸图片
 - 加载速度：Glide > Picasso
 - GIF：Glide支持， Picasso不支持
 - 包大小：Glide > Picasso


##### 26.（Android）测量的三种模式

 - EXACTLY：精确值模式。指定控件的宽高为具体数值，或者match_parent
 - AT_MOST：最大值模式，控件宽高为wrap_center，控件大小随着内容变化而变化，此时控件的尺寸只要不超过父控件允许的最大值即可
 - UNSPECIFIED：不指定其大小的测量模式，View想要多大就多大，绘制自定义View的时候才使用
 - View的onMeasure()默认使用EXACTLY模式，如果让自定义View支持wrap_centent，必须重写onMeasure方法来指定wrap_centent时的大小


##### 27.（Java）线程的各种状态

 - 新建态：new
 - 就绪状态：Runnable，调用start后，在run执行之前
 - 运行状态：Running，run
 - 阻塞状态：Bloked。包括同步阻塞、主动阻塞和等待阻塞
    - 同步阻塞：锁被其他线程占用
    - 主动阻塞：主动让出cup执行权，如Thread.sleep、thread.join()
    - 等待阻塞：wait()
 - 终止状态：dead、run()方法运行结束或异常退出


##### 28.（Android）View中两种重绘的区别

 - invalidate()：在UI线程中调用，Handler + invalidate
 - postInvalidate()：在非UI线程中调用（封装了Handler可以直接在非UI线程中调用）


##### 29.（网络）XML和Json的区别

Json体积小，传输快


##### 30.（网络）TCP和UDP区别

TCP：

 - 传输控制协议
 - 面向连接
 - 点对点(一对一)
 - 可靠交付
 - 全双工通信

TCP三次握手

 - C向S发送连接请求报文段(我可以向你发数据吗)
 - S收到连接请求报文段，同意，则向C发出确认，等待C发送数据(可以，你什么时候发)
 - C收到S的确认后还要向S发送确认(我现在就发)

UDP：
 - 无连接
 - 尽最大努力交付
 - 面向报文
 - 没有拥塞控制
 - 支持一对一，一对多、多对一、多对多

区别：
 - TCP：面向连接，可靠，慢，适用于大量数据
 - UDP：非面向连接，不可靠，快，适用于小量数据
 - TCP：点对点
 - UPD：点对点、一对多、多对一、多对多
 - TCP：通过Socket建立连接
 - UDP：通过DatagramSocket


##### 31.（Java）String所占字节数


##### 32.（数据结构和算法）1加到100

```

```

##### 33.（Android）NDK和JNI
 - JNI：Java Native Interface，即Java本地接口，为了方便Java与C、C++等本地代码相互调用所封装的一层接口。通过JNI，用户可以与C、C++ 所编写的本地代码进行交互。
 - NDK：Android中的本地开发工具包。可以使用C、C++编写的库，编译成so文件，让Java调用。
 - 关系：JNI是目的，NDK是手段。在Android开发环境中，通过NDK，从而实现JNI


##### 34.（Android）MVC和MVP原理和区别

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


##### 35.（Java）子类与超类构造器

 - 如果子类构造器没有显式调用超类的构造器，自动调用超类默认(无参)的构造器
 - 如果子类构造器没有显式调用超类的构造器，但如果超类没有不带参数的构造器，编译不过
 - super调用超类的方法，调用超类构造器需放在第一句。


##### 36.（Android）Retrofit原理

 - 通过动态代理创建请求，默认是Okhttp请求。OkhttpCall
 - 通过注解配置请求，通过反射获取注解内容配置请求。ServiceMethod
 - 使用Converter进行数据转换
 - CallAdapter，返回类型，默认是DefaultCallAdapter
 - 动态代理：InvocationHandler
 
总结：Retrofit使用注解去配置Http请求，将一个OkHttp请求抽象成一个接口，然后用动态代理和反射，将接口的注解翻译成一个http请求。


##### 37.（Android）RxJava


##### 38-52推送相关


##### 53.（网络）Http协议的工作流程

首先服务器会不断监听TCP的80端口(http默认端口)，以便发现客户端向它发出建立连接的请求。当C向S发出建立连接的请求时，S与C先建立TCP连接(三次握手)，然后C向S发出访问某个页面或文件的请求，然后S返回所请求的页面或文件作为响应，最后TCP连接释放。


##### 54.（网络）HTTP报文结构

HTTP报文分为请求报文和响应报文。都是由三部分组成：开始行、首部和实体主体。  
请求报文的开始行称为请求行；响应报文的开始行称为状态行。其他部分一样。  
 - 请求方法：GET、POST、HEAD、DELETE、PUT
 - 常见首部字段：Host、Connection、User-Agent、Accept-Language、Content-Type
 - 版本：1.1、2.0
 - 状态码：2xx表示成功；3xx表示重定向；4xx表示客户端错误；5xx表示服务器错误。


##### 55.（网络）GET和POST的区别

 - GET：请求参数放在URL后面，不能大于20KB，一般用于查询
 - POST：请求参数放在实体主体，大小无限制，一般用于上传更新信息


##### 56.（Android）对多线程的理解

Android中的线程分为主线程和子线程。主线程叫叫UI线程，用来处理界面相关操作。子线程用来处理耗时的操作，如请求网络、IO。如果在主线程执行耗时的操作，程序就会无法及时响应，因此必须在子线程中执行耗时的操作。  

Android中开启线程的方式有：Thread、AsyncTask(异步任务)、HandleThread、IntentService和线程池  
 - AsyncTask：封装了线程池和Handler，可以在子线程中执行完任务之后轻易切换到主线程刷新界面
 - HandlerThread：是一种线程，继承了Thread，通过handler消息机发送消息到子线程执行任务
 - IntentService：是一种服务，继承了Service，使用HandlerThread去开启线程，处理完成之后会关闭服务
 - 线程池：管理和重用线程，避免过多的创建和销毁


##### 57.（Android）getWidth()和getMeasureWidth()的区别

 - getMeasureWidth()：获取的是view原始大小。也就是这个View在XML中配置或者是代码中设置的大小。
 - getWidt()：获取的是这个View最终显示的大小。这个大小有可能等于View的原始大小，也有可能不等。


##### 58.（Android）缩放Bitmap不失真的方法

通过inSampleSize的方式加载合适尺寸的图片不会失真。



##### 59.（Java）堆区、栈区和方法区的区别。笔记18


##### 60.（Java）内部类和静态内部类

内部类对象总有一个隐式引用，它指向创建它的外部类对象。这个引用在内部类默认的构造器中设置。这个构造器中添加了外部类引用的参数。


##### 61.（Android）子线程更新UI的方式

Handler、AsyncTask、runOnUI、view.postRunnable


##### 62.（Android）Intent可以传递的数据类型

 - 基本数据类型及其数组
 - 字符串及其数组
 - 序列化对象及其数组
 - 包裹类对象及其数组


##### 63.（Android）图片列表卡顿现象及解决方案

 - 现象：在一个列表加载很多(成千上万)张图片，有可能会造成卡顿
 - 原因：如果是同步加载，卡顿是必然的，因为加载图片是耗时的操作；如果是异步加载，在快速滑动的时候会产生大量异步任务，即一瞬间存在大量更新UI的操作，也会造成卡顿。
 - 解决办法：
    - 使用异步加载图片
    - 当列表停止滑动的时候才加载图片，可以用ScrollerListener实现。判断滑动的状态，IDEL表示停止滑动。


##### 64.（Android）View的生命周期

 - 构造方法：代码创建View或从XML中加载
 - onFinishInflate()：View和子View加载完成后调用，通常是在Activity的onCreate()后调用
 - onVisibilityChanged()：View或父View的可见性改变时调用。如果View不可见或GONE，该方法第一个被调用。
 - onAttachToWindow()：View附着一个Window时调用，Activity第一次执行完onResume后调用
 - onMeasure()：测量View的大小时调用
 - onSizeChanged()：measure方法之后且测量大小与之前不一致的时候调用。
 - onLayout()：确定View的位置时调用
 - onDraw()：绘制View的时候调用
 - onWindowFocusChanged()：包含当前View的Windows获得或失去焦点时调用，View进入销毁阶段

其他相关回调方法:
 - onFocusChanged()
 - onKeyDown()和onKeyUp
 - onTouchEvent()
 - onSavedInstanceState()


##### 65.（Android）View的onMeasure、onLayout和onDraw回调次数

在加载一次的情况下：
 - onMeasure：执行两次，performTranversal执行了两次
 - onLayout：执行两次
 - onDraw：执行一次


##### 66.（数据结构和算法）求阶乘

```
private int factorial(int n){
    if(n==0) return 1;
    return n * factorial(n-1);
}
```

##### 67.（Android）图片三级缓存

LruCache -> SoftReference/WeakReference -> SD -> NetWork  
加载一张图片时，先从内存缓存中找，有则取出，没有则去引用缓存中找。有则取出并加到内存缓存，没有则去磁盘缓存中找。有则取出并加到内存缓存，没有则从网络下载，下载完成后保存到磁盘，加到内存缓存。


##### 68.（Android）Intent/Bundle传递数据大小
1MB以下，TransactionToolLargeException中提到


##### 69.（Android）为什么不能再子线程更新UI

 - Android中的UI控件不是线程安全的。如果在多线程中并发访问可能会导致UI控件处于不可预期状态。
 - 为什么不对UI控件的访问加锁？
    - 加锁会使访问UI变得更复杂
    - 会阻塞线程
 - 解决办法：单线程处理UI


##### 70.（Android）Android为什么不推荐用枚举类型Enum

文档中提到，枚举类型比静态常量多了两倍内存。原因：占用内存多，增加apk体积。具体是增加了dex的体积


##### 71.（设计模式）代理模式和动态代理

代理模式：
 - 为其他对象提供一种代理以控制对这个对象的访问
 - 通过代理对象间接访问委托对象的方法
 - 代理对象和委托对象实现相同的接口，代理对象持有委托对象的引用

按代码可分为静态代理和动态代理：
 - 静态代理：具体的代理类，固定代码
 - 动态代理：通过反射动态生成代理对象，实现InvocationHandler


##### 72.（Android）Activity、Window和View之间的关系

 - Activity：负责界面展示，用户交互与业务逻辑处理
 - Window：负责界面展示以及交互，相当于Activity的下属。
 - View：就是放在Window容器的元素，Window是View的载体，View是Window的具体展示

三者关系
 - Activity通过Window来实现视图元素的展示，Window可以理解为一个容器，盛放着一个个View，用来执行具体的展示工作。
 - Window和View的关系：window是一个界面的窗口，是存放View的容器，window是view的管理者，view附着在window上
 - Activity和Window的关系：Activity内部持有一个Window对象，用来管理View





 
