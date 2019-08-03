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




 
  
 
 
 








































