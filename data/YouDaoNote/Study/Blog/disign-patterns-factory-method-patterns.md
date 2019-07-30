---
title: 设计模式(三) 工厂方法模式
date: 2017-05-24 11:51:32
tags:
 - 设计模式
 - 工厂方法模式
categories: [设计模式]
---

# 定义
定义一个用于创建对象的接口,让子类决定实例化那个类

# UML类图
![](https://timgsa.baidu.com/timg?image&quality=80&size=b10000_10000&sec=1495597696&di=b6e459e489113ecdbc79ac8f57f17f19&src=http://pic002.cnblogs.com/images/2012/448009/2012102412230955.jpg)

解读: 

 - Product: 产品抽象类, 
 - ConcreteProduct: 产品实现类
 - Creator: 工厂抽象类, 返回一个Product实例
 - ConcreteCreator: 工厂实现类, 返回一个ConcreteProduct实例
 
# 简单实现
##　Fruit: 作为产品抽象类
```java
public abstract class Fruit {
    abstract void getName();
}
```

##　Banana: 作为产品实现类
```java
public class Banana extends Fruit {
    @Override
    void getName() {
        System.out.println("I am banana");
    }
}
```

## Apple: 作为产品实现类
```java
public class Apple extends Fruit {
    @Override
    void getName() {
        System.out.println("I am apple");
    }
}
```

## FruitFactory: 作为抽象工厂类
```java
public abstract class FruitFactory  {
    abstract Fruit createFruit();
}
```

## AppleFactory: 作为工厂实现类
```java
public class AppleFactory extends FruitFactory {
    @Override
    Fruit createFruit() {
        return new Apple();
    }
}
```

## BananaFactory: 作为工厂实现类
```java
public class BananaFactory extends FruitFactory {
    @Override
    Fruit createFruit() {
        return new Banana();
    }
}
```

## 使用
在需要哪一个产品时就生产哪个
```
FruitFactory fruitFactory = new AppleFactory();
Fruit apple = fruitFactory.createFruit();
apple.getName();
```

## 使用反射
### FruitFactory
```java
public abstract class FruitFactory  {
    abstract <T extends Fruit> T createFruit(Class<T> className);
}
```

###　ConcreteFruitFactory: 具体工厂类
```java
public class ConcreteFruitFactory extends FruitFactory {
    @Override
    <T extends Fruit> T createFruit(Class<T> className) {
        Fruit fruit = null;
        try {
            fruit = (Fruit)Class.forName(className.getName()).newInstance();
        }catch (Exception e){
            e.printStackTrace();
        }
        return (T)fruit;
    }
}
```

### 使用
需要哪个类对象就传入那个类的类型
```java
       FruitFactory fruitFactory = new ConcreteFruitFactory();
       Fruit apple = fruitFactory.createFruit(Apple.class);
       apple.getName();
```

# 简单工厂方法模式
当工厂只有一个的时候, 只需将对应的工厂方法改为静态方法, 并去掉abstract关键字, 此时就变成了简单工厂模式或者静态工厂模式
```
public class FruitFactory  {
    static Fruit createFruit(){
        return new Apple();
    }
}
```
