---
title: Android动画实例
date: 2017-05-24 14:54:12
tags:
 - Android
 - 动画
categories: [Android]
---

# 灵动菜单

## 实现思路
把菜单项目放在菜单按钮同一位置, 当点击带单按钮时候, 每个菜单项开始动画, 从原来位置,移动到指定位置  
## 代码
布局文件:
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageView
        android:id="@+id/id_dynamic_activity_fab_left"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"
        android:layout_centerInParent="true"
        android:visibility="visible"/>

    <ImageView
        android:id="@+id/id_dynamic_activity_fab_top"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"
        android:layout_centerInParent="true"
        android:visibility="visible"/>

    <ImageView
        android:id="@+id/id_dynamic_activity_fab_right"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"
        android:layout_centerInParent="true"
        android:visibility="visible"/>

    <ImageView
        android:id="@+id/id_dynamic_activity_fab_bottom"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"
        android:layout_centerInParent="true"
        android:visibility="visible"/>

    <ImageView
        android:id="@+id/id_dynamic_activity_fab_main"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@mipmap/ic_launcher"
        android:layout_centerInParent="true"/>

</RelativeLayout>
```

展开菜单:
```java
private void startMenu(){
        ObjectAnimator mainAnimator = ObjectAnimator.ofFloat(ivMain,"alpha",1f,0.5f);

        float leftCurrentX = ivLeft.getTranslationX();
        ObjectAnimator leftAnimator = ObjectAnimator.ofFloat(ivLeft,"translationX",leftCurrentX,-200f);

        float rightCurrentX = ivRight.getTranslationX();
        ObjectAnimator rightAnimator = ObjectAnimator.ofFloat(ivRight,"translationX",rightCurrentX,200f);

        float topCurrentY = ivTop.getTranslationY();
        ObjectAnimator topAnimator = ObjectAnimator.ofFloat(ivTop,"translationY",topCurrentY,-200f);

        float bottomCurrentY = ivBottom.getTranslationY();
        ObjectAnimator bottomAnimator = ObjectAnimator.ofFloat(ivBottom,"translationY",bottomCurrentY,200f);


        AnimatorSet animatorSet = new AnimatorSet();
        animatorSet.playTogether(mainAnimator,leftAnimator,rightAnimator,topAnimator,bottomAnimator);
        animatorSet.setDuration(500);
        animatorSet.setInterpolator(new BounceInterpolator());
        animatorSet.start();

        flag = false;
}
```
关闭菜单:
```java
private void closeMenu(){
        ObjectAnimator mainAnimator = ObjectAnimator.ofFloat(ivMain,"alpha",0.5f,1f);

        float leftCurrentX = ivLeft.getTranslationX();
        ObjectAnimator leftAnimator = ObjectAnimator.ofFloat(ivLeft,"translationX",leftCurrentX, 0f);

        float rightCurrentX = ivRight.getTranslationX();
        ObjectAnimator rightAnimator = ObjectAnimator.ofFloat(ivRight,"translationX",rightCurrentX,0f);

        float topCurrentY = ivTop.getTranslationY();
        ObjectAnimator topAnimator = ObjectAnimator.ofFloat(ivTop,"translationY",topCurrentY,0f);

        float bottomCurrentY = ivBottom.getTranslationY();
        ObjectAnimator bottomAnimator = ObjectAnimator.ofFloat(ivBottom,"translationY",bottomCurrentY,0f);


        AnimatorSet animatorSet = new AnimatorSet();
        animatorSet.playTogether(mainAnimator,leftAnimator,rightAnimator,topAnimator,bottomAnimator);
        animatorSet.setDuration(500);
        animatorSet.setInterpolator(new BounceInterpolator());
        animatorSet.start();

        flag = true;
}
```
对主菜单按钮监听
```java
ivMain.setOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View v) {
		if (flag) startMenu();
		else closeMenu();
	}
});
```

# 下拉展开动画
## 实现思路
两个LinearLayout, 一个visibility为显示,另一个为gone. 当点击showVIew时,把隐藏的LinearLayout设置为visibility,对这个过程使用动画.


## 代码
布局文件:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:id="@+id/id_show_hide_ll_show_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_vertical"
        android:orientation="horizontal">
        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@mipmap/ic_launcher"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="click me"/>


    </LinearLayout>

    <LinearLayout
        android:id="@+id/id_show_hide_ll_hide_view"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:gravity="center_vertical"
        android:visibility="gone"
        android:background="@color/colorPrimary">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@mipmap/ic_launcher"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="I am hide"/>

    </LinearLayout>

</LinearLayout>
```

获取ValueAnimator:
```
private ValueAnimator createValueAnimator(final View view, int start, int end){
        final ValueAnimator animator = ValueAnimator.ofInt(start,end);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                int value = (int)valueAnimator.getAnimatedValue();
                ViewGroup.LayoutParams params = view.getLayoutParams();
                params.height = value;
                view.setLayoutParams(params);
            }
        });
        return animator;
}
```
展开动画:
```
private void openAnimation(View view){
        view.setVisibility(View.VISIBLE);
        ValueAnimator valueAnimator = createValueAnimator(view,0,measureHiddenViewHeight);
        valueAnimator.start();
}
```

关闭动画:
```
private void closeAnimation(final View view) {
	int origHeight = view.getHeight();
	ValueAnimator valueAnimator = createValueAnimator(view,origHeight,0);
	valueAnimator.addListener(new AnimatorListenerAdapter() {
		@Override
		public void onAnimationEnd(Animator animation) {
			view.setVisibility(View.GONE);
		}
	});
	valueAnimator.start();
}
```