##### 1.（）IO流

 - 流
    - 输入流：可以从其中读入一个字节序列的对象。InputStream
    - 输出流：可以从其中写入一个字节序列的对象。OutputStream
    - 字符流：Reader和Writer。专门用于处理Unicode字符的单独的类层次结构。这些类拥有的读入和写出操作都是基于两个字节的Unicode码元，而不是单字节的字符。
    - 常用字节流家族图：
        - InputStream和OutputStream是基础
        - DataInputStream和DataOutputStream可以以二进制格式读写所有的基本Java类型。
        - FileInputStream和FileOutputStream可以读写文件
    - 常用字符流家族图：
 - 常用字节/字符流基本用法
    - InputStream：
        - int read(byte[] b)：读入一个字节数组，返回十几度如的字节数。
        - int read(byte[] b, int offset, int len)：offset，第一个读入字节应该被放置在b中的偏移量
        - int available()：返回不阻塞情况下可读取的字节数
    - OutputStream：
        - void write(byte[] b)：
        - void write(byte[] b, int offset, int len)：
    - FileInputStream和FileOutputStream：可以提供附着在一个磁盘文件上的输入流和输出流。只能对字节级别进行读写。
    - BufferedInputStream和BufferedOutputStream：流在默认情况下是不被缓冲区缓存的，请求一个数据块并将其置于缓冲区中会更加高效。
    - DataInput和DataOutput：
        - DataOutput：定义了以二进制格式写入字符、字符串等方法，如writeChar、writeInt、writeDouble、writeUTF...等等。
        - DataInput：定义了以二进制格式读取数据的方法，如readInt、readDouble、readUTF...等等。
    - DataInputStream和DataOutputStream：

##### 2.（）IO流读写文件

包括：保存/读取文本文件和保存/读取二进制文件

 - 保存文本文件：

Java实现
```

```
Kotlin实现
```

```

 - 读取文本文件：

Java实现
```

```
Kotlin实现
```

```
 - 保存二进制文件：

Java实现
```

```
Kotlin实现
```

```
 - 读取二进制文件：

Java实现
```

```
Kotlin实现
```

```

###### 3.（）解析xml数据

xml数据

```

```

 - Pull解析：XmlPullParser
     - 获取XmlPullParser对象：XmlPullParserFactory.newInstance().newPullParser();
     - 设置解析数据：xmlPullParser.setInput()
     - 获取事件类型：xmlPullParser.getEventType()。包括：START_DOCUMENT,END_DOCUMENT,START_TAG,END_TAG,TEXT。
     - 获取标签名：xmlPullParser.getName();
     - 解析内容：xmlPullParser.nextText()
     - 解析下一个事件：xmlPullParser.next()
     - 解析过程：文档开始，从上到下，从左到右依次解析。简单地说就是顺序解析。当解析到文本时，就跳到下一个开始标签
 - SAX解析：
     - 获取XMLReader对象：SAXParserFactory.newInstance().newSAXParser().getXMLReader();
     - 创建自定义的DefaultHandler：new MyDefaultHandler();
     - 设置解析Handler：xmlReader.setContentHandler();
     - 开始解析：xmlReader.parse()
     - 基本流程：定义一个类继承DefaultHandler，重写其中五个方法：
         - startDocument():
         - endDocument()：
         - startElement()：
         - endElement()：
         - characters()：
     - 解析节点中的内容的时候，通常会根据节点名用StringBuilder保存对应的信息。
     - 注意在获取节点内容时，characters()方法会调用多次，其中一些空白、换行也被当做内容结穴出来。
 
 
##### 4.（）解析json数据
 
```
 
```
 
 - 使用JSONObject解析
     - 创建JSONObject()：new JSONObject("jsonData");
     - getJSONObject():
     - getString();
     - getJSONArray();

```
 
```
 - 使用GSon解析
     - new GSon();
     - gSon.fromJson(jsonData, JavaBean.class)
 
```
 
```
 
##### 5.（）回调机制
 
##### 6.（）Socket通信
 
 - 套接字：包含主机和服务端口号。主机地址就是客户程序或服务器程序所在主机的IP地址；端口号则是主机彼此通信时所用的通道。
 - Socket，C-S模型：
     - 服务器首先创建ServerSocket对象，通过accept()方法等待客户端发来请求。
     - 客户端创建Socket，与服务器建立连接
     - 服务器中的accept()方法监听到来自客户端的请求，会返回Socket对象，该对象与客户端请求时的Socket建立连接
     - 通过Socket的getInputStream和getOutputStream()来传送数据
     - 而服务器中的accept()会继续监听下一个客户的连接请求。
     - 服务端多线程机制：当客户端请求过来时，服务器程序创建一个线程为客户端提供通信服务。而ServerSocket的accept()方法继续监听客户端请求。（这种方式不能满足高性能服务器的要求，参考NIO）
 - ServerSocket：


##### 7.（）异步消息处理机制


##### 8.（）AsyncTask

 - 实现原理：Handler + 线程池。Android帮我们做了很好的封装，即使对异步消息处理机制完全不了解也可以蛇粉简单的从子线程切换到主线程。
 - AsyncTask抽象类指定了3个泛型参数：Params、Progress和Result
    - Params：执行AsyncTask时传入参数，也是doInBackground()的参数
    - Progress：onProgressUpdate()的传入参数，也是在doInBackground()中调用publishProgress()的传入参数
    - Result：doInBackground()的返回参数，作为onPostExecute()的传入参数
 - 重新AsyncTask几个重要的方法：
    - onPreExecute()：后台执行之前调用
    - doInBackground()：后台执行时调用
    - onProgressUpdate()：更新进度
    - onPostExecute()：后台执行完成调用


##### 9.（）Service
Service是Android中实现程序后台运行的解决方案，它适合去执行那些不需要用户交互而且还要长时间运行的任务。服务依赖于创建服务时所在的进程，当该进程被杀，所依赖的服务也被杀

 - 生命周期：
 - Service基本用法：
    - 自定义服务，继承Service，并在AndroidManifest中注册
    - 重写3个方法：
      - onCreate()：创建服务时候调用，只调用一次。
      - onStartCommand()：启动时候调用，每次启动都调用。
      - onDestroy()：
    - 启动和停止服务：startService()、stopService()
 - Service与Activity通信：Binder
    - 自定义Binder，继承Binder，并且创建一些自定义方法，在活动与服务绑定的时候可以适当调用这些方法。在服务中onBinder()返回这个Binder。
    - 在Activity中创建ServiceConnection对象，实现其中两个方法：onServiceConnected()和onServiceDisconnected()。分别在绑定和解绑服务时调用。
    - 绑定服务：bindService(serviceIntent, serviceConnection, BIND_AUTO_CREATE)，其中BIND_AUTO_CREATOR表示绑定之后自动创建服务，执行onCreate()，绑定服务后不执行onStartCommand()，而执行serviceConnection的onServiceConnected()方法
    - 解绑服务：unBindService(serviceIntent)
 - 前台服务：startForeground()
 - IntentService：


##### 10.（）搭建博客平台


##### 11.（）下载示例

 - 保存文件的另一种方式：

```
RandomAccessFile raf = new RandomAccessFile(file, "rw");
raf.write();//跟其他一样
```
 - 跳过已下载的字节

```
raf.seek(downloadedLength);
```

 - 断点下载，指定从哪个字节传输

```
int downloadedLength = file.length();
Header: ("RANGE", "byte=" + downloadedLength + "-")
```

##### 12.（）jdbc连接数据库


##### 13.（）MaterialDesign

 - Toolbar:
 - DrawerLayout
 - NavigationView
 - FAB
 - SnackBar
 - CoordinatorLayout
 - CardView
 - AppBarLayout
 - SwipeRefreshLayout
 - CollapsingToolbarLayout
 - BottomNavigationView


##### 14.（）Android系统架构

 - Android系统机构：
    - Application：
    - Framework：
    - 标准库和运行时：
    - Linux内核层：
 - App组件架构：
    - Activity：
    - Service：
    - ContentProvider：
    - BroadcastReceiver：
    - Intent：作为信息的载体，组件与组件之间通过Intent通信，传递信息，交换数据。
 - 应用上下文环境Context：Activity、Service和Application都是(间接)继承Context。在创建Context实现类的时候就是创建Context的时机。ApplicationContext对象贯穿整个应用的生命周期，为应用提供全局的环境支撑。


##### 15.（）空间架构

 - 控件分类：ViewGroup和View。ViewGroup可作为父控件，包含其他ViewGroup和View，从而形成一个控件树木。上层控件负责下层控件的绘制和测量，并传递交互事件。
 - findViewById()方法就是在控件树中以深度优先遍历来查找对应的元素。
 - 每棵控件树的顶部都有一个ViewParent对象，是整棵树控制核心，统一调度和分配交互管理事件。
 - 每个Activity都包含一个Window对象，Window对象通常由PhoneWindow来实现。
 - PhoneWindow将一个decorView设置为整个应用窗口的根View。将要显示的具体内容呈现给PhoneWindow。
 - 所有的View的监听事件通过WindowManagerService来进行接收，并通过Activity对象来回调相应的onClickListener。
 - DecorView把屏幕分成TitleVIew和ContentView。contentView就是一个id为content的FrameLayout，setContentView就是把我们的布局添加到这个FrameLayout中。
 - 视图树的第二层装载一个LinearLayout作为ViewGroup，包含ActionBar和FrameLayout。这一层会根据参数设置不同的布局。这就解释了requestWindowFeature()方法要在setContentView()之前执行才能生效。
 - 在Activity调用setContentView后，ActivityManagerService会回调onResume，此时系统才会把整个DecorView添加到PhoneWindow中，并让器显示出来，从而最终完成界面的绘制。


##### 16.（）四种权限关键字的区别


##### 17.（）静态内部类和内部类的区别


##### 18.（）View的测量

##### 19.（）View的绘制

 - 重写View中的onDraw方法来绘制。该方法中以一个canvas参数，可以使用这个参数进行绘图。canvas就像是一个画板，使用paint就可以在上面作画了。
 - 在其他地方可以使用以下方法来创建canvas对象。传进去的bitmap与canvas进行关联，成为装载画布。bitmap用来保存作者所有绘制在canvas上的像素信息。即所有canvas.drawXXX方法都发生在这个Bitmap上。

```
Canvas canvas = new Canvas(bitmap);
```
 - 虽然我们使用Canvas的绘制API，但其实并没有将图形直接绘制在onDraw方法指定的那块画布上，而是通过改变Bitmap，让View进行重绘，从而显示改变后的bitmap。

```
//在onDraw中绘制两个bitmap
canvas.drawBitmap(bitmap1, paint);
canvas.drawBitmap(bitmap2, paint);
//把bitmap2装载到另一个Canvas中，并绘制
Canvas aCanvas = new Canvas(bitmap2);
aCanvas.drawXXX();
//刷新View的时候，onDraw方法画出来的bitmap2已经发生改变，这就是因为bitmap2承载了在aCanvas上所进行的绘图操作。
```
 


##### 20.（）ViewGroup的测量和绘制


##### 21.（）volatile和synchronize的作用和区别

##### 22.（）synchronize的用法区别

 - 修饰代码块：称为同步语句块。作用范围是{}括起来的代码；作用的对象是这个代码块的对象
 - 修饰实例方法：称为同步方法。作用范围是整个方法；作用对象是调用这个方法的对象
 - 修饰静态方法：作用范围是整个方法；作用对象是这个类的所有对象。
 - 修饰类：作用范围是synchronize后面括起来的部分；作用对象是这个类的所有对象。

##### 23.（）final的作用

 - final类：
 - final方法：
 - final变量：final成员变量必须在声明的时候初始化或者在构造方法中初始化。本地final变量必须声明时初始化。
 - 匿名类中的所有变量必须是final
 - 接口中声明的所有变量本身是final
 - 将类、方法、=和变量声明为final能提高性能，JVM进行优化。
 - final集合对象不能重新引用，但能修改里面的内容。
 - final类中的方法默认是final


##### 24.（）死锁

 - 定义：若干进程竞争有限资源，加上推进顺序不当，从而构成无限期循环等待的局面。如果没有外力作用，那么死锁涉及到的各个进程将永远处于封锁状态。
 - 条件（同时满足）：
    - 互斥：每个资源不能同时被两个或以上的进程占有
    - 不可抢占：进程所获得的资源在未使用完毕之前，资源申请者不能强行夺取资源
    - 占有且申请：已经占有资源的进程又申请新的资源
    - 循环等待：
 - 应对死锁的策略：
   - 预防：破坏必要条件之一
   - 避免：使用银行家算法，思源分配算法、安全性算法
   - 检测与恢复：