##### 1.（Android）对多线程的理解

Android中的线程分为主线程和子线程。主线程叫叫UI线程，用来处理界面相关操作。子线程用来处理耗时的操作，如请求网络、IO。如果在主线程执行耗时的操作，程序就会无法及时响应，因此必须在子线程中执行耗时的操作。  

Android中开启线程的方式有：Thread、AsyncTask(异步任务)、HandleThread、IntentService和线程池

 - AsyncTask：封装了线程池和Handler，可以在子线程中执行完任务之后轻易切换到主线程刷新界面
 - HandlerThread：是一种线程，继承了Thread，通过handler消息机发送消息到子线程执行任务
 - IntentService：是一种服务，继承了Service，使用HandlerThread去开启线程，处理完成之后会关闭服务
 - 线程池：管理和重用线程，避免过多的创建和销毁
 - 


##### 2.（Android）Handler消息机制

消息机制由四部分组成：Handler、Message、MessageQueue和Lopper

 - Message：消息的载体
 - MessageQueue：消息队列，负责存放消息
 - Looper：负责读取消息，每个线程只有一个Looper
 - Handler：负责发送和处理消息，将当前线程切换到handler所关联的线程去处理消息

它的运行过程是这样的：handler发送一个消息出去，这个消息会被添加到MessageQueue中等待被处理，而Looper则会一直尝试从消息队列中取出消息，最后分发回handler的handleMessage()方法。


##### 3.（架构）MVP模式的理解

MVP全称：Model-View-Presenter，解除了View和Model之间的耦合，可以有效的降低View层的复杂性，避免业务逻辑塞进View中。

 - View：在Android中，View层由Acitivity、Fragment或者某个View充当，实现了一个逻辑接口，并持有一个presenter变量。View的操作转交给Presenter执行，执行完成后，presenter调用view层的逻辑接口方法，返回给View。View层只管显示。
 - Model：model层主要负责数据的存储功能,Persenter向Model层获取存储数据。
 - Persenter：主要负责连接View层和Model层。Presenter向Model层获取数据返回给View层，使得View和Model之间没有耦合，也将View的业务逻辑抽离出来。


##### 4.（Android）SharePerference如何保存数据

以键值对的方式在xml中保存数据
```
<map>
    <String name="name">Tom</String>
    <boolean name="select" value = "false"/>
    <int name="age", value=24/>
</map>
```
以键值对的方式在xml中保存  
签名对应数据类型  
name属性对应键  
value属性对应值  

##### 5.（Android）如何发编译APK
从apk包中解压出dex文件，通过dex2java工具将dex文件反编译冲.jar包，然后用jd-gui工具将jar包转床成java代码  
使用apkTool反编译资源


##### 6.（Android）Activity异常退出时怎么保存数据

在onSaveInstanceState()方法中通过Bundle保存数据，可以保存基本的数据类型和他们的数组，序列化对象及其数组，包裹类对象及其数组.  
在onRestoreInstanceState()方法中恢复数据。或者在onCreate()方法中恢复数据。注意onCreate()方法中的Bundle可能为空.


##### 7.（Android）自定义View常用的构造函数及其参数的作用 (3-17)

常用的有三个，一个只有Context，一个只有Context和attributeSet，另外一个主题id
```

```
只有Context参数的构造函数在代码中创建View的时候调用。  
含有两个参数context和attrSet的构造函数一般是在xml文件中使用View，加载View时调用。attributeSet保存了自定义的属性。  
含有三个参数的构造函数一般是前面两个调用


##### 8.（Android）View的绘制流程(原理)

View的绘制流程分为三个阶段：measure -> layout -> draw

 - measure: 第一步measure，即测量View的大小。从顶层View到子View，递归调用measure方法，measure又会调用onMeasure()方法完成测量过程
 - layout：第二步layout，即确定View的位置，进行页面布局。从顶层View到子View递归调用View的layout方法，根据子View的布局参数将子View放在合适的位置上。
 - draw：第三步绘制。绘制过程又分为4个步骤，
    - 绘制背景
    - 绘制View本身内容
    - 绘制子View的内容
    - 绘制滚动条
    

##### 9.（设计模式）单例模式的理解
确保某个类只有一个实例，并且向整个系统提供这个实例。  
单例模式适用于创建对象需要消耗过多的情况，例如访问IO，数据库，请求网络等。  
实现单例模式的方式有：懒汉式，饿汉式，DCL，静态内部类和枚举单例。其原理都是将构造函数私有，确保多线程单例有且只有一个，反序列化不会重新创建对象。  
常见的单例写法：

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
```java
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
```java
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

##### 10.（Android）事件分发机制

当一个MotionEvent产生后，需要把这个事件传递给一个具体的View，这个传递的过程就是事件分发的过程。这个过程由三个重要的方法完成：dispatchTouchEvent、onInterceptTouchEvent和onTouchEvent。

 - dispatchToucnEvent：负责分发事件，它的返回值受onInterceptTouchEvent和onTouchEvent的影响，表示是否消耗事件
 - onInterceptTouchEvent：用来判断是偶拦截事件
 - onTouchEvent：负责处理事件，返回值表示是否消耗了事件
 
当一个事件都够传递给某个View，那么该View的dispatchTouchEvent方法一定会被执行。如果是ViewGroup，会调用onInterceptTouchEvent方法判断是否拦截该事件。如果返回true表示拦截，那么会调用该View的onTouchEvent方法；如果返回false表示不拦截，则会调用子View的dispatchTouchEven方法，将当前事件传递给子View，如此往复知道事件最终被处理
```java
public boolean dispatchTouchEvent(){
    boolean consume;
    if(onInterceptTouchEvent()){
        consume = onTouchEvent();
    }else{
        consume = childView.dispatchTouchEvent();
    }
    return consume;
}
```
事件从Activity出发，传递给Window，再传递给顶层View，然后根据事件分发机制，传递到具体的View，事件处理的过程则相反。


##### 11.（Android）做过哪些性能优化

 - 布局优化：使用merge和include减少层级，使用ViewStup延时加载
 - Bitmap优化：根据控件大小缩放Bitmep
 - 响应速度优化：将一些耗时的操作放在子线程去处理
 - 绘制优化：onDraw不要创建局部对象；不要执行大量的循环操作；不要做耗时的操作
 - 内存优化：避免内存泄漏


##### 12.（Android）常见的内存泄漏有哪些？怎么解决？ 

 - 静态变量导致的内存泄漏，如静态的Activity、Fragment、View
 - 单例模式导致的内存泄漏，如注册量监听器回调将它保存到集合，如果不取消注册就会导致内存泄漏
 - 属性动画导致的内存泄漏，开启了循环动画没有取消
 - 非静态内部类导致的内存泄漏，由于持有外部类引用，如果执行耗时的操作，退出界面时还没执行完成，就导致内存泄漏，可以使用静态内部类 + 弱引用的方式将Activity或Fragment传进来。
 

##### 13.（Android）Activity的生命周期

onCreate -> onStart -> onResume -> onPause -> onStop -> onDestory


##### 14.（Android）Service的生命周期

以start方式启动  
onCreate -> onStartCommand -> onDestroy  
每次启动时，onCrete只执行一次，onStartCommand每次启动都执行
以bind方式启动  
onCreate -> onBind -> unBind -> onDestroy


##### 15.（Android）Fragment的生命周期

onAttach -> onCreate -> onCreateView -> onActivityCrate -> onStart -> onResume -> onPause -> onStop -> onDestroyView -> onDestroy -> onDettach


##### 16.（Android）Broadcast广播机制

广播分为：标准广播、有序广播和本地广播。

 - 标准广播：是一种异步广播，所有广播接收器几乎同时受到广播
 - 有序广播：是一种同步广播，优先级较高的广播接收器西安搜狐到广播，可拦截广播和传递数据
 - 本地广播：只有当前应用可以接收

注册广播的方式有两种：动态注册和静态注册

 - 静态注册：是指在AndroidMenifest配置中注册，这种方式注册的广播接收器白应用没运行也能收到广播
 - 动态注册：是指在代码中注册，这种方式注册的广播接收器在运行应用并且注册后才能收到广播，注意需要取消注册
动态注册的优先级比静态注册的高


##### 17.（Android）对Context的理解

Context，即上下文环境，是一种抽象类，提供程序运行的基础信息。

Context有两个子类：ContextWrapper和ContextImpl。

 - ContextWrapper是上下文功能的封装类
 - ContextImpl是上下文功能的实现类  

ContextWrapper又有三个子类：Application、Service和ContextThemeWrapper。ContextThemeWrapper是一个带主题的封装类，Activity继承ContextThemeWrapper。  

因此，Context共有三种类型：Application、Service和Activity。大多情况下，三种类型的context是通用的，但有些特殊场景，比如启动Activity和Dialog，只能用Activity类的Context。因为Dialog必须建立在Activity上面弹出(除了AlertDialog)；启动Activity必须建立在另一Activity之上，以形成返回栈。


##### 18.（Android）Activity启动模式

Activity有四种启动模式：Standard、Single'Top、SingleTask和SingleInstance

 - Standard：这是默认的启动模式，每次启动都会创建实例
 - SingleTop：栈顶复用模式，如果需要启动的Activity在栈顶，就重用该实例，否侧创建实例，如阅读类App的内容界面
 - SingleTask：栈内复用模式，如果栈内已有该Activity实例，就重用该实例，并将该Activity以上的Activity全部移除，如App的主界面。
 - SingleInstance：创建新的返回栈存放该实例。


##### 19.（）自定义View的方式

自定义View的方式有三种：组合控件、继承原有控件和完全自定义View。

 - 组合控件：将一些基础组件封装起来形成新的控件，如通用的标题栏
 - 继承原有控件：在原有控件的基础上添加方法来扩展功能，或者修改原有的显示逻辑
 - 完全自定义View：需要重写onMeasure()实现测量逻辑，需要重写onDraw方法实现绘制逻辑，需要注意支持wrap_parent和Padding


##### 20.（Android）动画种类

动画分为三种：帧动画、视图动画和属性动画

 - 帧动画：通过切换图片的方式实现动画效果
 - 视图动画：通过矩阵运算修改View的显示
 - 属性动画：通过修改对象的属性值实现动画，是真正意义上的动画。
视图动画和属性动画的区别：例如将一个View使用视图动画移动到别的位置，但它点击监听事件还在原来的区域；而使用属性动画它的点击区域也随着控件的移动而移动。


##### 21.（Android）跨进程通信方式

Android跨进程通信的方式有：Intent、文件共享、ContentProvider、广播、Socket和AIDL

 - Intent：
 - 文件共享：两个进程读写一个文件来传递数据
 - ContentProvider：Andorid专用于不同应用间共享数据
 - 广播：
 - Socket：网络上不同应用的数据交互
 - AIDL：


##### 22.（）内存溢出与内存泄漏

 - 内存溢出：指程序申请内存时，没有足够的空间供其使用，即内从不够用
 - 内存泄漏：指程序申请内存后，无法释放已申请的内存空间。过多的内存泄漏可能导致内存溢出

内存泄漏检测工具：MAT(Memory AnalyzeTool)、LeakCanary


##### 23.（Android）Fragment与Activity、Fragment的通信方式

接口回调、广播、Fragment调用Activity中的实例方法，EventBus


##### 24.（Android）UI适配

尺寸用dp、文字用sp、多实用math_parent、wrap_content


##### 25.（）


##### 26.（Java）==和equals()、equals()和hashCode()的区别

==、equals()和hachCode()的作用：

 - ==：用来比较基本类型的值是否相等；也用来比较对象地址是否相等；
 - equals()：用来比较对象的内容是否相等；Object类的equals()用==判断
 - hashCode()：对象散列码，由对象导出的整形值。Object类的hashCode()返回对象的地址

equals()和hashCode()的规律
 - 如果两个对象equals，那么它们的hashCode一定相等
 - 如果两个对象不equals，它们的hashCode有可能相等
 - 如果两个对象hashCode相等，它们不一定equals
 - 如果两个对象hashCode不相等，那么它们一定不equals
 - 任何时候复写equals，都必须复写hashCode


##### 27.（Java）基本类型和包装类型的区别

8种基本类型都有对应的包装类型。

 - 基本类型用 = = 判断相等，包装类型用equals判断，缓存区也可以用==
 - 包装类型解决了基本类型无法做到的事，如泛型类型参数，序列化、类型转换。
 - 
 

##### 28.（Java）String、StringBuildr和StringBuilder的区别

 - String：是只读字符串，对字符串的修改其都是在创建一个新的对象，并将引用指向该对象，会在常量池中缓存
 - StringBuffered：可变字符串，可以对原对象进行修改，是线程安全的
 - StringBuilder：也是可变字符串，功能和StringBuffered一样，非线程安全
> 注意：不要在循环中使用字符串拼接


##### 29.（Java）内部类的作用和分类

 - (非静态)内部类：可以访问定义该类的作用域中的数据
 - 成员内部类
 - 局部内部类：定义在方法或表达式内
 - 匿名内部类：
 - 静态内部类：


##### 30.（Java）进程和线程的区别

 - 进程：是CPU分配资源的最小单位。例如一个应用就是一个进程(也可以多个进程)，CPU为这个应用分配资源，如内存
 - 线程：是CPU调度的最小单位。CPU执行的是线程

进程之间不能共享资源，线程可以共享所在进程的资源。  
一个进程至少包含一个线程，线程依赖进程而存在。


##### 31.（Java）final、finally和finalize的区别

 - final：用于修饰类、变量或方法。final类不可继承；final变量不可修改；final方法不可重写
 - finally：与try-catch一起使用，确保出现异常也能调用finally块的语句
 - finalize：资源回收时调用


##### 32.（Java）serialized和parcelable的区别

 - Serialized：使用简单，序列化和反序列化要执行大量的IO操作，低效
 - Parcelable：使用麻烦，在内存中操作，高效
 - Serialized适用于存储到本地或网络传输；Parcelable适用于在内存中操作。


##### 33.（Java）常用集合有哪些

常用的集合分为：List、Set、Map和Queue

 - List常用的有：ArrayList、LinkedList、CopyOrWriteArrayList
 - Set常用的有：HashSet、TreeSet、LinkedHashSet
 - Map常用的有：HashMap、TreeMap、LinkedHashMap、ConcurrentHashMap


##### 34.（Java）List、Set和Map的区别

 - List：List集合以线性的方式存储，集合中可以存放重复的元素
 - Set：Set是一种最简单的集合，集合中的对象不按特定的方式排序，没有重复的对象
 - Map：Map是一种把键对象和值对象映射的集合


##### 35.（Java）ArrayList和LinkedList的区别

 - ArrayList：由数组实现，访问元素很快，插入和删除元素很慢(因为要移动元素)
 - LinkedList：由链表实现，访问很慢，插入和删除元素很快


##### 36.（Java）HshSet、TreeSet和LinkedHashSet的区别

 - HashSet：使用HashMap实现，按照哈算法来存储集合中的对象，元素无序
 - TreeSet：使用TreeMap实现，底层为树结构，按照某种规则将元素插入指定位置
 - LinkeHashSet：继承了HashSet，元素有序


##### 37.（Java）HashMap、TreeMap、LinkedHashMap和ConcurrentHshMap的区别

 - HashMap：由散列表实现
 - TreeMap：由红黑树实现，是key有序的Map集合
 - ConcurrentHashMap：线程安全的HashMap
 - LinkedHashMap：有插入顺序的HashMap


##### 38.（Java）HashMap和HashTable的区别

都是用哈希表实现

 - HashMap：是非线程安全集合，允许null键和值，效率高，初始16， 以2的指数扩展
 - HashTable：是线程安全集合，不允许null键和值，效率低，初始11，以2*old+1方式扩展


##### 39.（Java）HashMap和HashSet的区别

 - HashMap：实现Map接口，存储键值对，使用键对象计算hashCode，速度快
 - HashSet：实现Set接口，仅存储对象，使用元素对象计算hashCode，可能存在相同的hashCode，因此还要用equals判断元素相等性，速度较慢


##### 40.（Java）数组和链表的区别

 - 数组：预先分配内存，地址连续，随机访问速度快，插入删除慢
 - 链表：动态分配内存，地址不一定连续，顺序访问速度慢，插入删除快


##### 41.（Java）并发与并行的区别

 - 并发：某时间段内多个任务交替执行
 - 并行：某时间段内多个任务同时执行


##### 42.（）


##### 43.（Java）线程的各个状态

 - 新建态：new
 - 就绪状态：Runnable，调用start后，在run执行之前
 - 运行状态：Running，run
 - 阻塞状态：Bloked。包括同步阻塞、主动阻塞和等待阻塞
    - 同步阻塞：锁被其他线程占用
    - 主动阻塞：主动让出cup执行权，如Thread.sleep、thread.join()
    - 等待阻塞：wait()
 - 终止状态：dead、run()方法运行结束或异常退出


##### 44.（Java）如何保证线程安全

 - 使用ThreadLocal：线程局部变量
 - 使用只读对象，final类避免继承，final变量避免中途修改
 - 使用线程安全类，如StringBuffered、ConcurrentHashMap、CopyOrWriteArrayList
 - 使用同步和锁机制，synchronize、Lock


##### 45.（Java）并发包中有哪些类

 - 并发集合类：ConcurrentHashMap、CopyOrWriteArrayList、BlockingQueue
 - 线程管理类：线程池、Executors、ThreadPoolExecutor
 - 锁相关类：Lock接口、ReentrantLock
 - 线程同步类(不常用)：


##### 46.（Java）常用锁的实现方式

 - 并发包中的锁类：Lock接口利用volatile的可见性；ReentrantLock、...
 - 同步代码块：synchronize同步方法；使用synchronize对类和对象进行同步


##### 47.（Java）实现线程同步的方式有哪些

 - 锁和条件对象
 - 同步方法和同步块
 - 阻塞队列
 - 单任务的线程池


##### 48.（Java）volatile的作用

 - volatile修饰变量，使该变量的操作在内存中进行，以便多个线程读取到的是最新值，解决的是多线程共享变量可见性问题
 - 对volatile变量的操作并非都是原子性，如i++，i--
 - volatile适用于一读多写的并发场景，他会是线程的执行速度会变慢


##### 49.（Java）线程池的作用

管理和复用线程，控制线程最大并发数


##### 50.（Java）ThreadPoolExecutor构造方法的参数意义

 - corePoolSize：核心线程数
 - maximunPoolSize：非核心线程最大线程数
 - keepAliveTiem：线程空闲时间
 - timeUtil：keepAliveTiem的单位
 - workQueue：缓存队列，请求的线程数大于corePoolSize核心线程数时加入队列
 - threadFactory：线程工厂，用来创建线程
 - handler：拒绝策略，当缓存队列慢后，并且活动线程数大于最大线程数，通过该策略处理请求，相当于限流。常用的策略有打印日志，跳转界面


##### 51.（Java）wait()和sleep()的区别

 - object.wait()：当一个线程执行到wait()方法时，它就进入到该对象的等待池中，释放锁，使其他线程可以访问。使用notify、notifyAll或睡眠来唤醒等待池中的线程。wait、nofity、notifyAll必须在同步块中调用。
 - Thread.sleep()：使线程进入睡眠等待，挡在同步块中调用sleep方法，不会释放锁，即其他线程无法访问这个对象。


##### 52.（Java）join()和yield()的区别

 - thread.join()：使调用join()方法所在的线程进入阻塞状态；等待目标线程执行完毕后再继续
 - Thread.yield()：主动让出线程执行权给其他已就绪状态的线程


##### 53.（Java）await、signal、signalAll和wait、notify、notifyAll的区别

作用：

 - await()/wait()；都是将线程加入等待池中并释放锁
 - signal()/notify()：随机解除等待池中的一个线程
 - signalAll()/notifyAll()：解除等待池中的线程

区别：
 - await、signal、signalAll是Condiction条件对象中方法
 - wait、notify、notifyAll是Object类的方法


##### 54.（Java）

 - Object类：wait()、notify()、notifyAll()
 - Condiction类：await()、signal()、signalAll()
 - Thread类静态方法：Thread.sleep()、Thread.yield()


##### 55.（Java）中断线程interrupted()和isInterrupted()的区别
 - interrupted()：是Thread的静态方法，使当前线程中断状态置false
 - isInterrupted()：是实例方法，不改变中断状态


##### 56.（Java）JVM内存布局

 - 堆区：存放实例对象；垃圾回收区域；子线程共享
 - 元空间：存放类元信息、字段、静态属性、方法、常量等，线程共享
 - 虚拟机栈：描述Java方法执行的内存区域，线程私有。每个方法都是用一个栈帧去表示，栈帧是方法运行的基本结构，存放局部变量表，操作栈、方法返回地址；
 - 本地方法栈：功能和虚拟机栈差不多，为Native方法服务
 - 程序计数器：


##### 57. （Java）类加载流程

类的加载流程分为三个阶段：加载、链接、初始化

 - 加载：读取文件产生二进制流，并转化为特定的数据结构，初步校验cafe babe魔法数，常量池、文件长度、是否有父类等。创建java.lang.Class实例
 - 链接：链接阶段又分为是三个过程：验证、准备和解析
    - 验证：更详细的校验，比如类型是否正确，静态变量是否合理
    - 准备：为静态变量分配内存，并设定默认值
    - 解析：解析类和方法确保类与类之间的相互引用正确性。完成内存结构布局
 - 初始化：执行类构造器

