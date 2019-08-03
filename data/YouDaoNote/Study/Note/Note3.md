##### 1.（）网络层协议

 - ISO七层协议：应用层、表示层、会话层、运输层、网络层、数据链路层、物理层
 - TCP/IP四层协议：应用层、运输层、网际层、网络接口层
 - 五层协议：应用层、运输层、网络层、数据链路层、物理层

五层协议作用：

 - 应用层：通过应用进程间通信的交互来玩成特定的网络应用。HTTP、SMTP、FTP。数据单元：报文
 - 运输层：向两个主机中进程间通信提供通信的数据传输服务。
    - 传输控制协议TCP：
    - 用户数据报协议UDP：
 - 网络层：为分组交换机上的不同主机提供通信服务。IP协议，数据单元：IP数据报
 - 数据链路层：两个相邻节点之间传输数据。点对点PPP；载波监听/碰撞检查CSMA/CD。数据单元：帧
 - 物理层：在传输媒体(介质)上传输比特流。数据单元：比特流。


##### 2.（）ArrayList和LinkedList的实现和区别


##### 3.（）如何保证线程安全

##### 4.（）List、Set和Map的区别



##### 5（）六大原则

 - 单一职责原则SRP(Single Responsibility Principle)
    - 定义：就一个类而言，应该仅有一个引起它变化的原因。[ASD]
    - 简单地说： 一个类中应该是一组相关性很高的函数，数据的封装。这满足高内聚的要求两个完全不一样的功能就不应该放在同一个类中。
   - 例如：ImageLoader中ImageLoader类只负责加载图片逻辑,ImageCache只负责图片缓存逻辑
 
 - 开放-封闭原则OCP(Open-Close Principle)
    - 定义：软件中的对象(类, 模块, 函数等)应该对于扩展是开放的, 但是对于修改是封闭的.[ASD]
    - 简单的说： 在我们写代码时候,假设变化不会发生.当发生变化时,我们就常见抽象来隔离以后发生同类变化。当软件需求发生变化,尽量通过扩展的方式来实现变化,而不是通过修改已有代码来实现.
    - 例如：最初的ImageLoader只有内存缓存MemoryCache,为了需求，添加了磁盘缓存DiskCache,后来又加入了双缓存，每一次添加一种缓存方式，都要修改ImageLoader的代码.可以将缓存类抽象成一个接口ImageCache，提供一些接口方法以上三种缓存都实现这个接口，并实现接口方法.在ImageLoader添加一个依赖注入的方法setImageCache(ImageCache imageCache)。

 - 里氏替代原则LSP(Liskov Subsitution Principle)
    - 定义：所有引用基类的地方必须能透明的使用其子类的对象。
    - 简单的说：只要父类出现的地方,子类都可以出现,而且替换为子类也不会产生任何错误或异常。
    - 例如：Shape是Renctangle的父类, Shape shape = new Shape();可以替换为Shape shape = new Renctangle();而不产生错误或异常.
 
 - 依赖倒置原则DIP(Dependence Inversion Principle)
    - 定义: 高层模块不应该依赖底层模块，两个都应该依赖抽象；抽象不应该依赖细节.，细节应该依赖抽象。
     - 简单地说:
        - 抽象就是指接口或抽象类
        - 细节就是指实现类
        - 高层模块就是调用端
        - 底层模块就是具体实现类
        - 模块间的依赖通过抽象发生,实现类之间不发生直接的依赖关系,其依赖关系是通过接口或抽象类产生的.
  
 - 接口隔离原则ISP(InterfaceSegregation Principle)
    - 定义：类间的依赖关系应该建立在最小的接口上。
    - 简单的说：就是客户端依赖的接口尽可能的小。
    - 例如：
 
```
 public static void close(Closeable closeable) {
 	if(closeable != null){
 		try{
 			closeable.close();
 		}catch(IOException e){
 			e.printlnStack();
 		}
 	}
 }
```
这个方法的基本原理就是依赖于Closeable抽象(依赖倒置)，并且建立在最小化依赖原则的基础上。它只需要知道这个对象可以关闭，也就是接口隔离。

 - 迪米特原则：
    - 定义：一个对象应该对其他对象有最少的了解。
    - 例如：我们买房子的时候，通过中介。而找房子的工作应该是中介区完成的。我们只需要把条件告知中介，中介就会去找到房子给我们。


##### 6.（）生产者与消费之问题

 - 如果一个进程能产生并释放资源，则该进程称为生产者；
 - 如果一个进程使用(消耗)资源，则该进程称为消费者；
 - 生产者-消费者问题，是进程同步、互斥关系方面的一个典型。通过缓冲区发生关系；
 - 生产者进程将生产的产品(如数据、消息)送入缓冲区；
 - 消费者进程从缓冲区中取出产品。
 - 规定：
    - 所有生产者存放产品的数量不能超过缓冲区总容量；
    - 所有消费者取出产品的数量总量不能超过生产者当前生产的产品总量。
 
 
 
##### 7.（）HashMap和HashTable的区别


##### 8.（）散列技术


##### 9.（）自定义View的三种方式


##### 10.（）VIew中比较重要的回调方法

 - onFinishInflate()
 - onSizeChanged()
 - onMeasure()
 - onDraw()
 - onLayout()
 - onTouch()
 
##### 11.（）自定义ViewGroup通常需要重写的方法

 - onMeasure()
 - onLayout()
 - onTouchEvent()


##### 12.（）事件分发拦截机制

 - 触摸事件：捕获触摸屏后产生的事件，MotionEvent
 - 事件传递顺序：Activity -> Window -> RootView -> ViewGroup -> View
 - 事件执行顺序：View -> ViewGroup -> RootView -> Window -> Activity
 - 事件传递返回值：true，表示拦截，比不继续传递；false，表示不拦截，继续传递
 - 事件处理返回值：true，表示已经处理，不用交给上级；false表示不处理，交给上级处理。


##### 13.（）自定义View的常用构造函数及其参数作用



##### 14.（）ListView的优化


##### 15.（）滑动监听

 - 常用监听ListView的两种方式：setOnTouchListener、setOnScrollListener
 - setOnTouchListener：根据触摸事件获取不同事件的触摸坐标值，对这些坐标值进行比较，实现不同的逻辑。例如判断上滑、下滑，在Down事件获取第一次触摸屏幕时的坐标，在MOVE事件中获取移动的坐标值，如果当前 - 首次 > 0，则为下滑。
 - setOnScrollListener：有两个重要回调方法：
    - onScrollStateChange()：根据它的参数scrollStae判断滚动状态
    - onScroll()：在滚动时候一直调用。它的三个参数：
        - firstVisibleItem：
        - visibleItemCount：能看到的Item数
        - totalItemCount：
     - 判断滚动方向时，只需要判断
```
OnScrollListener.SCROLL_STATE_IDLE：停止滚动
				  .FLING：由于惯性滚动
				  .TOUCH_SCROLL： 还在滚动
```
 
```
firstVisibleItem > lastVisibleItemPosition；//上滑
lastVisibleItemPosition = firstVisibleItem
```


##### 16.（）ListView加载不同布局

在Adapter中重写getItemType()，然后在getView中根据这个类型加载不同的布局。


##### 17.（）在ListView滑动时隐藏和显示Toolbar

原理：首先判断是上滑动还是下滑，然后计算滑动距离。当超过某个值的时候，认为是滑动的就执行toolbar的位移动画。


##### 18.（）RecyclerView和ListView的区别



##### 19.（）Scroll分析

滑动一个View，本质就是移动一个View。要实现滑动，就要监听用户产生的触摸事件，根据事件传入坐标，动态且不断改变View的坐标，从而实现VIew的滑动。

 - Android坐标系：以屏幕左上角的顶点为原点。
 - View坐标系：以父View左上角为原点，getXX()就是获取视图坐标系中的坐标值。
  - 触控事件：MotionEvent，封装了一些常用的事件常亮，如ACTION_DOWN、ACTION_MOVE等等。
  - 获取坐标的方法：View坐标系用getLRTB()；MotionEvent提供的getXYRawX、RawY。

##### 20.（）实现滑动的方式

基本思想：当触摸View时，记录当前触摸点的坐标；手指移动时，记录移动后的坐标，通过计算得到坐标偏移量，通过偏移量修改View的坐标。

 - layout()方式
  - params方式
  - scrollBy方式
  - scroller方式
  - 属性动画方式
 
##### 21.（）绘图机制

 - 屏幕信息
    - 屏幕大小：屏幕对角线长度，单位，寸
    - 分辨率：屏幕的像素点个数，px
    - PPI：每英寸像素，又称DPI(Dots Per Inch)

 - 2D绘图基础
    - 画布和画笔：Paint(画笔)，Canvas(画布)。通过这两个类的组合可以画出各种图案。
    - Paint的方法和功能：
        - setAntiAlias()：设置锯齿效果
        - setColor()：
        - setARGB()：
        - setAlpha()：
        - setTextSize()：
        - setStyle()：设置画笔风格，空心或实心
        - setStorkeWidth()：设置空心边框的宽度
    - Canvas的方法和功能
        - 画点：drawPoint(x, y, paint);
        - 画线：drawLine(startX, startY, endX, endY, paint)
        - 画多条直线：
        - 画矩形：drawRect(l, t, r, b, paint);
        - 画圆角矩形：drawRoundRect(l, t, r, b, radiusX, radiusY, paint);
        - 画圆：drawCircle(x, y, r, paint);
        - 画弧线：drawArc(l, t, r, b, startAngle, sweepAngle, boolean, paint);
        - 画椭圆：drawOval(l, t, r, b, paint);
        - 画文本：drawTexgt(text, x, y, paint)
        - 画路径：drawPath(path, paint)
        - 
 - XML绘图
    - 在xml中使用bitmap
    - shape：用于绘制各种形状，无论是扁平化，拟物化，还是渐变都能绘制，使用各种各样的标签实现各种效果。
        - gradient：渐变
        - solid：填充颜色
        - corners：当shape指定为rectangle时才使用，指定圆角
    - Layer：类似ps图层叠加
    - selector：帮助开发者实现事件反馈。
 
```
<?xml version="1.0" encoding = "utf-8">
<bitmap xmlns:android=""
	android:src=""/>
</>
```
这样代码中可以将图片直接转成Bitmap


#####22.（）SurfaceView

当View执行的操作逻辑太多，特别是需要频繁刷新界面，会不断阻塞主线程，从而导致画面卡顿。为了避免这一问题，Android提供了SurfaceView来解决各个问题，它会通过子线程来进行页面刷新。

SurfaceView与View的区别：

 - View适用于主动更新，SurfaceView适用于被动更新
 - VIew在主线程对画面进行刷新；SurfaceView通过子线程进行刷新
 - View绘图没有使用双缓冲；SurfaceView实现了双缓冲机制

##### 23.（）TCP三次握手

准备工作：服务器创建传输控制块TCB，准备接收客户端的连接请求。此后Server处于Listen状态，Client也创建TCB

 - C向S发送连接请求报文段(我可以向你发数据吗)
 - S收到连接请求报文段，同意，则向C发出确认，等待C发送数据(可以，你什么时候发)
 - C收到S的确认后还要向S发送确认(我现在就发)

为什么不是两次握手？

 - C还要发送一次确认，主要是为了防止已失效的连接请求报文突然又传到了S，因而发生错误。

什么是已失效的连接请求报文段？

 - C发送的第一个请求在某些网络节点上长时间滞留了，以至延误到连接释放以后的某个时间点才到达S。


##### 24.（）TCP四次握手

 - C向S发送连接释放请求报文段，并停止发送数据。
 - S收到连接释放报文段后发出确认。此时，S进入半关闭状态，即 C -> S 方向关闭。但S仍然可以向C发送数据。
 - S向C发送连接释放报文段。
 - C收到S的连接释放请求报文，发出确认。


##### 25.（）equals()和“==”的和区别


##### 26.（）equals()和hashCode()的作用和区别

##### 27.（）动画机制

 - 视图动画
 - 属性动画
 - 插值器：定义动画的变换速率，相当于物理中的加速度。例如先慢后快。
 - SVG矢量动画：优势，缩放不失真


##### 28.（）Activity启动模式

> 注意：使用singleTop或singleTask的ActivityA通过startActivityFroResult来启动ActivityB，系统直接返回RESULT_CANCEL，而不会再去等待返回。

##### 29.（）Activity启动标记

 - NEW_TASK：使用新的Task来启动Activity
 - SINGLE_TOP：与singleTop相同
 - CLEAR_TOP：与singleTask相同
 - NO_HISTORY：使用这种模式启动的ActivityA来启动Activity之后，A就销毁。

##### 30.（）Android系统信息与安全机制

 - 获取Android系统信息：android.os.Build
 - 应用管理：PackageManager
 - 安全机制：混淆、签名、数字证书、沙箱机制


##### 31.（）静态块与飞静态块

 - 相同点：在JVM加载类时且在构造方法前执行，在类中可定义多个。一般在代码块中对一些static变量进行赋值。
 - 不同点：
    - 静态块总是比非静态块先执行；
    - 静态块只在第一次new的时候执行一次，之后不执行；
    - 非静态块在每一次new的时候都执行。

##### 32.（）静态块与静态方法

 - 区别：静态块是自动执行（类加载的时候执行一次）；静态方法是被动调用；
 - 作用：静态块用来初始化一些项目中最常用的变量和对象；静态方法可以用作不创建对象亦可以执行代码。


##### 33.（）Activity退出怎么保存数据


##### 34.（）读写文件

 - 保存文本数据：
 - 保存二进制数据：
 - 读取文本文件：
 - 读取二进制文件：


##### 35.（）ANR及解决办法
 - ANR：Application Not Response，应用程序无响应。
 - 出现场景：主线程被IO操作阻塞、主线程存在耗时操作、主线程中错误的操作，如Thread.sleep
    - 应用在5s内未响应用户操作
    - BroadcastReceiver未在10s内完成相关处理
    - Service在20s内无法处理完成
    - 有些情况已经ANR，但不会弹出对话框
 - 如何避免：
    - UI线程只做跟UI相关的工作；
    - 耗时的操作单独放在线程中处理；
    - 使用AsyncTask或者Handler处理耗时的操作；
    - BroadcastReceiver中onReceive方法尽量减少耗时，建议使用IntentService处理耗时的操作。
 - 主线程有哪些：Activity生命周期回调方法、onKeyDown、onClick；AsyncTask处理doINBackground。


##### 36.（）Glide的缓存策略


##### 37.（）RxJava优缺点

 - RxJava：一个用响应式编程和观察者模式实现的异步操作库。RxJava中的响应式编程是被观察者拿到数据主动传递给观察者，将展示层和数据处理层分离，解耦了各个模块，通过不同线程操控代码运作配合变换过滤等API操作实现数据流传递。
 - 优点：异步、简洁
    - 内部支持多线程操作
    - 强大的map和flapMap保证了依赖上一次接口进行二次处理时不发生嵌套
    - 将各个模块分离
    - 支持Lamda表达式，保证RxJava代码在阅读上更加简洁
    - 随着程序逻辑复杂，依然保持简洁


##### 38.（）Android中的序列化
 - 什么是序列化：把Java对象转换成字节序列并存储到一个存储介质的过程。
 - 什么是反序列化：把字节序列恢复为Java对象。
 - Java对象的组成：变量和方法。但是序列化和反序列化仅处理变量不处理方法。
 - 原理：把Java对象的状态信息保存到存储媒介，这样可以在JVM非运行情况下，获取Java对象。也可以在其他机器的JVM上获取指定Java对象，如使用RMI(远程方法调用)、网络中传递对象。
 - 使用场景：
    - 把内存中的对象保存到一个文件或者数据库
    - 用套接字在网络上传递对象
    - 远程传输对象
 - 方式：Serializable和Parcelable

##### 39.（）Serializable和Parcelable的区别


##### 40.（）View的绘制原理(流程)


##### 41.（）冒泡排序

 - 交换排序：两两比较排序记录的关键字，一旦发现两个记录不满足要求则进行交换，知道整个序列全部满足要求为止。
 - 冒泡排序：比较相邻记录，如果逆序则进行交换，从而使得关键字小的记录如气泡一样逐渐往上漂浮(左移)，或者说关键字大的记录如石块一样逐渐往下沉(右移)。
 - 结束比较：当一轮比较之后没有发生交换，则已经排好序。因此用个标志位来记录是否发生过交换。
 - 时间复杂度：O(n^2)
 - 算法特点：
    - 稳定
    - 可用于链式存储
    - 缺点：移动次数较多，当n很大时不适用
 - Java实现：
```
    private static void popSort(int[] a){
        //最基本的冒泡排序写法
        /*int i;
        for (i=0;i<a.length;i++){
            for (int j=0;j<a.length-1;j++){
                if (a[j]>a[j+1]){
                    int temp = a[j];
                    a[j] = a[j+1];
                    a[j+1] = temp;
                }
            }
        }*/

        //改进的冒泡排序写法
        int len = a.length-1;
        boolean flag = true;//用来标识是否发生交换操作
        while (len>0 && flag){
            flag = false;
            for (int j=0;j<len;j++){
                if (a[j] > a[j+1]){
                    //交换
                    int temp = a[j];
                    a[j] = a[j+1];
                    a[j+1] = temp;
                    flag = true;
                }
            }
            show(a);
            len--;
        }
    }
```
 - C实现：
```

```

##### 42.（）直接插入排序

 - 插入排序：每一趟将一个待排记录按其关键字大小插入到已经排好序的一组记录的适当位置上，直到所有待排记录全部插入。
 - 直接插入排序：将一条记录插入到已排好序的有序表中，从而得到一个新的、记录数量增1的有序表。
 - 时间复杂度：O(n^2)
 - 特点：
    - 稳定
    - 算法简便、容易实现
    - 适用于链式存储结构
    - 适用于基本有序。当初始无序，n比较大时不适用
 - Java实现：
```java
    private static void insertSort(int[] a){
        System.out.println("插入排序算法:");
        int i, j;
        for (i=0; i<a.length; i++){
            for (j=i; j>0; j--){
                if (a[j] < a[j-1]){
                    int temp = a[j];
                    a[j] = a[j-1];
                    a[j-1] = temp;
                }
            }
            System.out.print("第" + (i+1) + "次排序后:");
            show(a);
        }
    }
```

##### 43.（）顺序表和链表比较


##### 44.（）快速排序


 - 思想：在待排记录中选择一个记录(通常第一个)作为枢纽(关键记录)pivotkey. 经过一趟排序，以枢纽记录分为两个子表，左边子表记录比枢纽记录小，右边子表比枢纽记录大。然后对每个子表重复以上过程。知道每个子表只有一个记录时，排序结束。
 - 每一趟操做：参考《大话数据结构》有详细解析
    - 选择表中一个记录作为枢纽(通常是第一个记录) 保存好记录的值pivotkey
    - 从high往左每个记录依次与pivotkey比较，比pivotkey大或相等的，high往左移(high--), 比pivotkey小的结束high移动，交换low和high的值
 	- 从low往右每个记录依次与pivotkey比较，比pivotkey小或相等的，low往右移(low++), 比pivotkey小的结束low移动，交换low和high的值
    - 返回pivotkey的值
 - 时间复杂度：O(nlogn)
 - 特点：不稳定；适用于顺序存储，不适用于链式存储。适用于n比较大，初始记录无序。
 - java实现
```
int a[10] = {4,3,2,6,1,9,5,8,7,0};
```

```
    private static void quickSort(int[] a,int low, int high){
        if (low<high){
            int povit = partition(a,low,high);
            quickSort(a,povit+1,high);  //对枢纽记录右边子表进行排序
            quickSort(a,low,povit-1);   //对枢纽记录左边的子表进行排序
        }
    }

    private static int partition(int[] a, int low, int high){
        System.out.println("low = " + low);
        int pivot = a[low]; //用子表的第一个记录作为枢纽记录
        while (low<high){   //
            while (low<high && a[high]>=pivot)  //右往左比较
                high--;                         //如果记录比枢纽记录大或者相等，high向左移动一个
            swap(a,low,high);                   //交换，比枢纽记录小的放左边
            show(a);
            while (low<high && a[low]<=pivot)   //左往右比较
                low++;                          //如果记录比枢纽记录小或者相等，high向右移动一个
            swap(a,low,high);                   //交换，比枢纽记录大的放右边
            show(a);
        }
        System.out.println("pivot = " + pivot);
        System.out.println();
        return low;
    }
```


调用
```java
quickSort(a, 0, a.length-1);
```

##### 45.（）Looper原理


##### 46.（）希尔排序

 - 思想：希尔排序的思想是使数组中任意间隔为h的元素都是有序的. 这样的数组成为h有序数组  
简单地说, 一个h有序数组就是h个互相独立的有序数组编织在一起组成的一个数组.  
在进行排序时, 如果h很大, 我们能将元素移动到很远的地方, 为实现更小的h有序数组创造方便, 用这种方式,对于任意以1结尾的h序列, 我们都能够将数组排序.  
这就是希尔排序.
 - 时间复杂度：O(n^1.3)
 - 特点：
    - 不稳定
    - 只适用于顺序存储，不适用于链式存储；适用于初始记录无序
    - n越大，效果越明显
 - Java实现
```
    private static void shellSort(int[] a){
        int N = a.length;
        int h = 1;
        while (h < (N/3)) h = h * 3 + 1;
        while (h >= 1){
            for (int i=h; i<N; i++){
                for (int j=i; j>=h;j=j-h){
                    //进行插入排序
                    if (a[j] < a[j-h]){
                        int temp = a[j];
                        a[j]= a[j-h];
                        a[j-h] = temp;
                    }
                }
            }
            System.out.print("h = " + h + ": ");
            show(a);
            h = h/3;
        }
    }
```

##### 47.（）二分查找算法

 - 思想：在有序表中，取中间记录作为比较对象。
    - 如果关键字与中间相等则查找成功；
    - 如果关键字小于中间记录，则在中间记录左半区域继续查找；
    - 如果关键字大于中间记录，则在中间记录右半区域继续查找；
    - 重复以上过程，直到查找成功，或者查找区域无记录。
 - 时间复杂度：O(logn)
 - 特点：
    - 比较次数少，效率高；
    - 只适用于顺序存储结构，且查找前要排序；
    - 不适用于经常插入删除操作的顺序表；
 - Java实现
```

```

##### 48.（）Bitmap如何避免OOM

 - 思想：使用BitmapFactory.Option加载所需尺寸的图片，即通过一定的采样率来加载缩放后的图片。
 - 采样率：inSampleSize
    - inSampleSize = 1, 不缩放。
    - inSampleSize = n，缩小图片的宽高为原来的 1/n，大小为原来的 1/n^2。建议inSampleSize为2的指数倍。
 - 流程：
    - 将Option的inJustDecodeBound设置为true，并加载图片。此时只会解析图片的宽高信息，而不会加载图片。
    - 从Option中取出图片宽高；
    - 通过一定的算法计算出适当的采样率
    - 将Option的inJustDecodeBound设为false，并重新加载图片，此时加载的就是缩小后的缩略图。
 - 获取采样率通用算法：
```

```

##### 49.（）使用线程的方式。


##### 50.（）HTTP协议

 - http的工作流程：
    - 服务器不断监听TCP的80端口，以便发现是否有客户端向它发送连接建立请求
    - 一旦监听到连接请求并建立TCP之后，客户端向服务器发出一访问某个页面的请求。
    - 最后TCP连接释放
    - 在请求和响应的交互，必须按照规定的格式和循环一定的规则。这些规则和格式就是HTTP协议。
 - HTTP的报文结构：HTTP报文分为请求报文和响应报文。都是由三部分组成：开始行、首部和实体主体。  
请求报文的开始行称为请求行；响应报文的开始行称为状态行。其他部分一样。  
    - 请求方法：GET、POST、HEAD、DELETE、PUT
    - 常见首部字段：Host、Connection、User-Agent、Accept-Language、Content-Type
    - 版本：1.1、2.0
    - 状态码：2xx表示成功；3xx表示重定向；4xx表示客户端错误；5xx表示服务器错误。
 - 请求报文举例：
```
开始行：GET、/SocialServer/LoginServer?username=xm&passwd=dixm
首部行：
实体主体：
```








 
  
 
 
 








































