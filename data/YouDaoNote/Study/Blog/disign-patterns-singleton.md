---
title: 设计模式(一) 单例模式
date: 2017-05-12 09:40:29
tags:
 - 设计模式
 - 单例模式
categories: [设计模式]
---


# 定义
确保每个类只存在一个实例，而且自行实例化并向整个系统提供这个实例  

# 使用场景
访问IO，数据库，网络等需要消耗多资源的对象

# 实现方式
## 饿汉模式
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

## 懒汉模式
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

## 双重检查锁定
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
## 静态内部类
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

# 注意事项

以上实现方式，在反序列化时候会重新创建对象，所以必须加入以下方法

```
/**防止反序列化重新构建对象*/
private Object readResolve() throws ObjectStreamException {
        return StaticInnerSingleHolder.staticInnerSingleton;
}
```
 
# 总结
待更新...
