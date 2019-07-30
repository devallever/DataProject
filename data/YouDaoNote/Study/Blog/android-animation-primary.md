---
title: Android 动画 简单使用
date: 2017-05-22 16:54:53
tags:
 - Android
 - Animation
categories: [Android]
---

> 顺便学习一下Kotlin语言

# 介绍
动画能带来良好的用户体验,作为Android程序员,必须掌握最最最基本的几种动画实现方式,包括透明度,位移,旋转,缩放.

# 属性动画
属性动画是Android 3.0推出的动画框架,能实现视图动画所有动画效果,并且能完成视图动画不具备的功能. 例如一个在左上角的按钮使用视图动画将他移动到右下角,点击时候发现原来的点击监听已经不起作用了. 简单说,视图动画仅仅改变了控件的绘制,并没有改变其属性值.如果使用属性动画将它移动,那么原来的监听还是有的,这是真正的移动啊.属性动画就是这么强大  

 属性动画最重要的两个类

 - ValueAnimator
 - ObjectAnimator
 
> 原理: 
属性动画作用于任何对象, 变化的属性名必须有该对象要有对应的getXXX和setXXX方法.
 
下面逐一讲讲

## ValueAnimator
```
    fun startValueAnimation(){
        val valueAnimator = ValueAnimator.ofFloat(0f,1f)
        valueAnimator.duration = 3000
        valueAnimator.repeatCount = 2
        valueAnimator.repeatMode = ValueAnimator.RESTART    //重复
        //valueAnimator.repeatMode = ValueAnimator.REVERSE    //倒回
        valueAnimator.start()
    }
```
 - ofFloat()传入的值类型为float, 表示该值从1f 变化到 3f, 1f是初始值, 3f是最终值, ofFloat()传入的是可变参数, 即可以传入多个值,表示变化过程, 如: ofFloat(1f, 3f, 2f, 5f), 这里逐步并不是一下子从1f直接变化到3f,而是有个过程,每个过程值都不一样,
 - 也许这里看不到变化, 但是动画所设置的值是变化了的,

## ObjectAnimator
ObjectAnimator继承了ValueAnimator, 所以可以使用父类方法对动画进行一些通用设置, 不同的是创建ObjectAnimator对象

### 透明动画
```
fun startAlpha(view: View){
        val alphaAnimator = ObjectAnimator.ofFloat(view,"alpha", 1f,0f,1f)
        alphaAnimator.duration = 1000
        alphaAnimator.start()
}
```
 - view: 动画作用的对象
 - "alpha": 值变化属性名, 及view对象中必须要有getAlpha()和serAlpha()方法,
 - 1f,0f,1f: 表示变化过程, 不透明->全透明->不透明
 

### 旋转动画
如旋转30度:

```
fun startRotate(view: View){
        val rotateAnimator = ObjectAnimator.ofFloat(view,"rotation", 0f,360f)
        rotateAnimator.duration = 1000
        rotateAnimator.start()
}
```

### 位移动画
#### X轴位移
```
fun startTranslateX(view: View){
        val currentX: Float= view.translationX
        val translateXAnimator = ObjectAnimator.ofFloat(view,"translationX",currentX,-500f,currentX,500f,currentX)
        translateXAnimator.duration = 1000
        translateXAnimator.start()
}
```
 - currentX: view原来的X坐标值
 - 动画过程,从原来位置开始,左移500像素,回到原来位置,右移500像素,回到原来位置
#### Y轴位移
```
fun startTranslateY(view: View){
        val currentY: Float= view.translationY
        val translateYAnimator = ObjectAnimator.ofFloat(view,"translationY",currentY,-500f,currentY,500f,currentY)
        translateYAnimator.duration = 1000
        translateYAnimator.start()
}
```

### 缩放动画
同样，支持Ｘ方向　和Ｙ方向
#### X轴缩放
```
fun startScaleX(view: View){
        val scaleXAnimator = ObjectAnimator.ofFloat(view,"scaleX",1F,3F,1F)
        scaleXAnimator.duration = 1000
        scaleXAnimator.start()
}
```

### 动画集合
```
fun startAnimatorSet(view: View){
        val animatorSet = AnimatorSet()

        val alphaAnimator = ObjectAnimator.ofFloat(view,"alpha", 1f,0f,1f)

        val rotateAnimator = ObjectAnimator.ofFloat(view,"rotation", 0f,360f)

        val scaleXAnimator = ObjectAnimator.ofFloat(view,"scaleX",1F,3F,1F)

        animatorSet.play(alphaAnimator).after(rotateAnimator).after(scaleXAnimator)
        animatorSet.duration = 2000
        animatorSet.start()
}
```

## 使用XML动画
在res建立animator文件夹  
在animator下创建动画的xml文件,如:
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="sequentially">
    <objectAnimator
        android:propertyName="alpha"
        android:duration="500"
        android:valueFrom="1"
        android:valueTo="0"
        android:valueType="floatType">
    </objectAnimator>

    <objectAnimator
        android:propertyName="alpha"
        android:duration="500"
        android:valueFrom="0"
        android:valueTo="1"
        android:valueType="floatType">
    </objectAnimator>
    <set
        android:ordering="together">
        <objectAnimator
            android:propertyName="rotation"
            android:duration="1000"
            android:valueFrom="0"
            android:valueTo="360"
            android:valueType="floatType">
        </objectAnimator>
        <set
            android:ordering="sequentially">
            <objectAnimator
                android:propertyName="scaleY"
                android:duration="500"
                android:valueFrom="1"
                android:valueTo="3"
                android:valueType="floatType">
            </objectAnimator>
            <objectAnimator
                android:propertyName="scaleY"
                android:duration="500"
                android:valueFrom="3"
                android:valueTo="1"
                android:valueType="floatType">
            </objectAnimator>
        </set>
    </set>
</set>
```
在代码中这样调用
```
fun startXMLAnimation(view: View){
        val animator = AnimatorInflater.loadAnimator(this,R.animator.animator_test)
        animator.setTarget(view)
        animator.start()
}
```

# 视图动画

## 透明度动画
```
    /**
     * 1:不透明
     * 0:全透明**/
    fun startAlpha(){
        val alphaAnimation = AlphaAnimation(0f,1f)
        alphaAnimation.duration = 1000
        tv_target?.startAnimation(alphaAnimation)
    }
```

## 旋转动画
```
    /**
     * 参数1: 开始角
     * 参数2: 旋转角
     * 参数3: **/
    fun startRotate(){
        //val rotateAnimation = RotateAnimation(0f,360f,0f,0f)

        //以自身中心旋转
        val rotateAnimation = RotateAnimation(
                0f,360f,
                RotateAnimation.RELATIVE_TO_SELF,0.5F,
                RotateAnimation.RELATIVE_TO_SELF,0.5F
        )
        rotateAnimation.duration = 1000
        tv_target?.startAnimation(rotateAnimation)
    }
```

## 位移动画
```
    /**
     * fromX
     * toX
     * fromY
     * toY**/
    fun startTranslate(){
        val translateAnimation = TranslateAnimation(0f,200f,0f,300f)
        translateAnimation.duration = 1000
        tv_target?.startAnimation(translateAnimation)
    }
```

## 缩放动画
```
    /**
     * fromX
     * toX
     * fromY
     * toY**/
    fun startScale(){
        //val scaleAnimation = ScaleAnimation(0f,2f,0f,2f)
        //设置中心点缩放
        val scaleAnimation = ScaleAnimation(0f,2f,0f,2f,
                ScaleAnimation.RELATIVE_TO_SELF,0.5F,
                ScaleAnimation.RELATIVE_TO_SELF,0.5F)
        scaleAnimation.duration = 1000
        tv_target?.startAnimation(scaleAnimation)
    }
```

# SVG动画
使用步骤

## 创建静态SVG图形
在drawable的资源文件夹下创建xml文件,line_svg_vector.xml
```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:height="200dp"
    android:width="200dp"
    android:viewportHeight="100"
    android:viewportWidth="100">
    <group>
        <path
            android:name="path1"
            android:strokeColor="#3F51B5"
            android:strokeWidth="5"
            android:strokeLineCap="round"
            android:pathData="M 20,80
                              L 50,80,80,80"/>

        <path
            android:name="path2"
            android:strokeColor="#3F51B5"
            android:strokeWidth="5"
            android:strokeLineCap="round"
            android:pathData="M 20,20
                              L 50,20,80,20"/>
    </group>



</vector>
```

## 创建动画资源xml文件
在animator下创建, 每个图形都要有一个动画过程  
line_path_1.xml
```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/bounce_interpolator"
    android:duration="500"
    android:propertyName="pathData"
    android:valueFrom="M 20,80 L 50,80,80,80"
    android:valueTo="M 20,80 L 50,50,80,80"
    android:valueType="pathType"
    >

</objectAnimator>
```

line_path_2.xml
```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/bounce_interpolator"
    android:duration="500"
    android:propertyName="pathData"
    android:valueFrom="M 20,20 L 50,20,80,20"
    android:valueTo="M 20,20 L 50,50,80,20"
    android:valueType="pathType"
    >

</objectAnimator>
```

## 把静态SVG和动画资源联合起来
在drawable-v21目录下创建xml文件:line_animated_vector.xml
```
<?xml version="1.0" encoding="utf-8"?>
<animated-vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/line_svg_vector">
    <target
        android:animation="@animator/line_path_1"
        android:name="path1"/>
    <target
        android:animation="@animator/line_path_2"
        android:name="path2"/>

</animated-vector>
```

## 使用SVG资源
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/id_svg_animation_activity_iv_line_svg"
        android:layout_width="150dp"
        android:layout_height="150dp"
        android:layout_gravity="center_horizontal"
        android:src="@drawable/line_animated_vector"/>

    <ImageView
        android:id="@+id/id_svg_animation_activity_iv_sun_system_svg"
        android:layout_width="150dp"
        android:layout_height="150dp"
        android:layout_gravity="center_horizontal"
        android:src="@drawable/sun_system_animated_vector"/>
</LinearLayout>
```

## 开启动画
点击时候开启动画
```
fun startLineSVGAnimation(imageView: ImageView){
        val drawable = imageView.drawable
        (drawable as Animatable).start()
}
```
> kotlin语言

