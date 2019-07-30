---
title: 设计模式(二) 建造者模式
date: 2017-05-18 21:27:59
tags:
 - 设计模式
 - 建造者模式
categories: [设计模式]
---

# 定义

> 将一个复杂对象的构建与它的表示分离,使得同样的构建过程可以创建不同的表示

你看懂了吗? 反正我是一脸懵逼

# UML类图

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1495123628735&di=c7dad79192ca076a4d7b2dd4266c3c42&imgtype=0&src=http%3A%2F%2Fimg.my.csdn.net%2Fuploads%2F201302%2F03%2F1359892948_5468.jpg)

解读:  

 - Product产品类: 产品抽象类  
 - Builder: 抽象Builder类,规划产品组建,一般是由子类实现具体的组建过程
 - ConcreteBuilder: 具体的Builder类
 - Director: 统一组建过程
 

好吧, 还是说的有点虚呀, 来一个简单例子

# 简单实现

## Computer抽象类类：Product角色，即产品抽象类

```
/**
 * Created by allever on 17-5-18.
 * 计算机抽象类: Product角色*/

public abstract class Computer {

    private String strBoard;
    private String strDisplay;
    private String strOS;

    protected Computer(){}
    /**CPU核心数*/
    public void setBoard(String board){
        this.strBoard = board;
    }
    public void setDisplay(String display){
        this.strDisplay = display;
    }

    public void setStrOS(String os){
        this.strOS = os;
    }

    @Override
    public String toString() {
        String result = "Computer:\nboard=" + strBoard + "\ndisplay = " + strDisplay + "\nOS = " + strOS;
        return result;
    }
}

```

## MacBook类，具体Computer类

```
/**
 * Created by allever on 17-5-18.
 */

public class MacBook extends Computer {
    protected MacBook(){}

}
```

##Builder抽象类  
负责规范产品组建，一般是由子类实现具体的组建过程
```
/**
 * Created by allever on 17-5-18.
 * 抽象Builder类*/

public abstract class Builder {
    public abstract void buildBoard(String board);
    public abstract void buildDisplay(String display);
    public abstract void buildOS(String os);
    public abstract Computer build();
}
```

## MacBookBuilder：具体Builder类，ConcreteBuilder角色  

```
/**
 * Created by allever on 17-5-18.
 * Builder实现类*/

public class MacBookBuilder extends Builder {

    private Computer computerMacBook = new MacBook();

    @Override
    public void buildBoard(String board) {
        computerMacBook.setBoard(board);
    }

    @Override
    public void buildDisplay(String display) {
        computerMacBook.setDisplay(display);
    }

    @Override
    public void buildOS(String os) {
        computerMacBook.setStrOS(os);
    }

    @Override
    public Computer build() {
        return computerMacBook;
    }
}

```

## Director类：同一组装过程

```
/**
 * Created by allever on 17-5-18.
 * Director类： 负责构建Computer*/

public class Director {
    private Builder builder = null;
    public Director(Builder builder){
        this.builder = builder;
    }

    //构建对象
    public void construct(String board, String display, String os){
        builder.buildBoard(board);
        builder.buildDisplay(display);
        builder.buildOS(os);

    }
}
```

## 客户端调用

```
Builder builder = new MacBookBuilder();
Director director = new Director(builder);
director.construct("英特尔酷睿i7","15.6寸","Ubuntu 16.04");
Log.d(TAG, builder.build().toString());
Toast.makeText(BuildPatternsActivity.this,builder.build().toString(),Toast.LENGTH_LONG).show();

```


# 小提示
通常,Director角色会被省略,直接使用Builder构建对象,这个Builder通常为链式调用,即buildXXX()方法返回自身,return this;

## 修改后的Builder类

```
/**
 * Created by allever on 17-5-18.
 * 抽象Builder类*/

public abstract class Builder {
    public abstract Builder buildBoard(String board);
    public abstract Builder buildDisplay(String display);
    public abstract Builder buildOS(String os);
    public abstract Computer build();
}
```
  
  
  
## MacBookBuilder:

```
/**
 * Created by allever on 17-5-18.
 * Builder实现类*/

public class MacBookBuilder extends Builder {

    private Computer computerMacBook = new MacBook();

    @Override
    public MacBookBuilder buildBoard(String board) {
        computerMacBook.setBoard(board);
        return this;
    }

    @Override
    public MacBookBuilder buildDisplay(String display) {
        computerMacBook.setDisplay(display);
        return this;
    }

    @Override
    public MacBookBuilder buildOS(String os) {
        computerMacBook.setStrOS(os);
        return this;
    }

    @Override
    public Computer build() {
        return computerMacBook;
    }
}
```


## 在客户端中的调用

```
Builder builder = new MacBookBuilder();
//Director director = new Director(builder);
//director.construct("英特尔酷睿i7","15.6寸","Ubuntu 16.04");
Log.d(TAG, builder.buildBoard("英特尔酷睿i7")
                    .buildDisplay("15.6寸")
                    .buildOS("Max OS")
                    build().toString());
```

现在你是不是感觉到很熟悉的呢? 就像Android中构建一个Notification对象

```
                Notification notification = new NotificationCompat.Builder(this)
                        .setContentTitle("This is content title")
                        .setContentText("This is content text")
                        .setSmallIcon(R.mipmap.ic_launcher)
                        .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
                        .setWhen(System.currentTimeMillis())
                        //设置点击行为
                        .setContentIntent(pendingIntent)
                        //点击后消失
                        .setAutoCancel(true)
                        //设置声音
                        .setSound(Uri.fromFile(new File("/system/media/audio/ringtones/Luna.ogg")))
                        //设置震动 震动1s 静止1s 震动1s 要申明权限
                        .setVibrate(new long[]{0, 1000,1000,1000})
                        //设置呼吸灯闪烁
                        .setLights(Color.GREEN, 1000, 1000)
                        //设置默认
                        .setDefaults(NotificationCompat.DEFAULT_ALL)
                        //设置长文本
                        .setStyle(new NotificationCompat.BigTextStyle().bigText("An Activity is an application component that provides a screen with which users can interact in order to do something, such as dial the phone, take a photo, send an email, or view a map."))
                        //设置展开大图,,设置了长文本会看不到
                        .setStyle(new NotificationCompat.BigPictureStyle().bigPicture(BitmapFactory.decodeResource(getResources(),R.mipmap.expensive)))
                        //设置通知重要程度
                        .setPriority(NotificationCompat.PRIORITY_MAX)
                        .build();
```