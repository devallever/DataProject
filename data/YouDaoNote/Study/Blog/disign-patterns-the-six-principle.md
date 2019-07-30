---
title: 设计模式之六大原则
date: 2017-05-11 08:50:16
tags:
 - 设计模式
 - 六大原则
categories: [设计模式]
---

# 单一职责原则SRP(Single Responsibility Principle)
定义: 就一个类而言，应该仅有一个引起它变化的原因。[ASD]  

简单地说：  

 - 一个类中应该是一组相关性很高的函数，数据的封装。这满足高内聚的要求
 - 两个完全不一样的功能就不应该放在同一个类中。
 - 
 
 例如:ImageLoader中ImageLoader类只负责加载图片逻辑,ImageCache只负责图片缓存逻辑
 
# 开放-封闭原则OCP(Open-Close Principle)
定义: 软件中的对象(类, 模块, 函数等)应该对于扩展是开放的, 但是对于修改是封闭的.[ASD]

简单的说:

 - 在我们写代码时候,假设变化不会发生.当发生变化时,我们就常见抽象来隔离以后发生同类变化.
 - 当软件需求发生变化,尽量通过扩展的方式来实现变化,而不是通过修改已有代码来实现.
 -  
 
例如:
最初的ImageLoader只有内存缓存MemoryCache,为了需求,添加了磁盘缓存DiskCache,后来又加入了双缓存,每一次添加一种缓存方式,都要修改ImageLoader的代码.可以将缓存类抽象成一个接口ImageCache,提供一些接口方法以上三种缓存都实现这个接口,并实现接口方法.在ImageLoader添加一个依赖注入的方法setImageCache(ImageCache imageCache);

# 里氏替代原则LSP(Liskov Subsitution Principle)
定义: 所有引用基类的地方必须能透明的使用其子类的对象

简单的说:

 - 只要父类出现的地方,子类都可以出现,而且替换为子类也不会产生任何错误或异常
 
例如:
Shape是Renctangle的父类, Shape shape = new Shape();可以替换为Shape shape = new Renctangle();而不产生错误或异常.
 
# 依赖倒置原则DIP(Dependence Inversion Principle)
定义: 

 - 高层模块不应该依赖底层模块.两个都应该依赖抽象
 - 抽象不应该依赖细节. 细节应该依赖抽象.
 
 简单地说:
 
  - 抽象就是指接口或抽象类
  - 细节就是指实现类
  - 高层模块就是调用端
  - 底层模块就是具体实现类
  - 模块间的依赖通过抽象发生,实现类之间不发生直接的依赖关系,其依赖关系是通过接口或抽象类产生的.
  
# 接口隔离原则ISP(InterfaceSegregation Principle)
定义: 类间的依赖关系应该建立在最小的接口上

简单的说:

 - 