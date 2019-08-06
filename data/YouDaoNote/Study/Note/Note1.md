##### 1.（）Android网络请求

- 原则：单一职责、专注
- OkHttp：Square公司开源的针对Java和Android，封装的一个高性能http网络请求库。对HttpUrlConnection封装，支持http2.0、webSocket；支持同步异步；封装了线程池、数据转换、参数使用和错误处理
- Volley：Google的一套小而巧异步请求库；支持HttpUrlConnection、OkHttp、ImageLoader；不支持post大数据；不适合上传文件
- Retrofit：Square基于OkHttp封装的一套RestFul网络请求框架；涉及一大堆设计模式、注解；JsonConverter序列化数据；支持RxJava
- Volley VS OkHttp：
  - Volley：封装更好；支持OkHttp；首选
  - OkHttp：需要再封装；基于NIO和OkIo
- OhHttp VS Retrofit：
  - Retrofit基于OkHttp；首先Retrofit
- Volley VS Retrofit：
  - Retrofit默认使用OkHttp，性能上比Volley占优势；解耦更彻底，职责细分；与RxJava配合使用
- Retrofit > Volley > OkHttp

##### 2.（）MVC在Android中的应用

##### 3.（）RecyclerView的基本用法

- RecyclerView是support-v7的组件，把ViewHolder的实现封装起来，用户只要实现自己的ViewHolder
- RecyclerView的使用方法与ListView的基本类似，需要Item布局、Adapter；重点在于Adapter的编码
- 自定义Adapter：
  - MyViewHolder是自定义的ViewHolder
  - MyAdapter的构造方法一般要传入context、dataList和layoutResId。
  - MyAdapter需要重写3个方法：
    - onCreateViewHolder：加载布局，创建ViewHolder
    - onBindViewHolder：为控件设置数据
    - getItem：
- 自定义ViewHolder：通常在MyAdapter内部创建
  - 重写构造方法：传入itemView，在这个构造方法中实例化控件(findViewById)
- 在Activity中使用RecyclerView：
  - 设置LayoutManager：
  - 设置Adapter
```JAVA
public class MyAdapter extends RecyclerView.Adapter<MyViewHolder> { }
```


##### 4.（）ViewPager的基本用法

- 使用ViewPager需要创建自己的Adapter、ViewPager的每一页都是一个Fragment(不一定)，因此，也需要创建Fragment去填充每一个页面。ViewPager的每一页的标题，需要用到TabLayout的控件，需要依赖design包
- 自定义Adapter：
  - 创建MyPagerAdapter，继承自FragmentPagerAdapter(或FragmentStatePagerAdapter)；需要创建构造方法传入fragmentManager，并调用父类的构造方法。在构造方法通常传入fragmentList，以及字符串数组作为每个页面的标题。
  - 重写getItem
  - 重写getCount
  - 重写getPageTitle
- 使用ViewPager：
  - 设置Adapter
  - 关联tabLayout：tabLayout.setupWithViewPager，关联后，tabLayout才能显示相应的标题

```
```

##### 5.（）ListView基本用法


##### 6.（）ToolBar用法


##### 7.（）属性动画的用法

##### 8.（）Retrofit2的基本用法

- 同步请求：
- 异步请求：
- Get请求：
- Post请求：
- 上传简单表单数据：
- 上传文件/图片复杂数据：
- 下载文件：小文件、大文件
- Retrofit + RxJava/RxAndroid
- Retrofit封装使用：
- 常用注解：
- Retrofit头部、cookie、token、session
- 多文件上传


##### 9.（）