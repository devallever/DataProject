---
title: Android 拖拽基本使用
date: 2017-08-15 21:27:03
tags:
 - Android
 - DragListener
categories: [Android]
---

# onDragListener 简介
使用Android拖放框架，你可以允许用户使用图形化的拖拽手势将数据从一个视图移动到另一个视图。这个框架包括一个拖拽事件类、拖放侦听器和帮助器方法和类。
尽管该框架主要是为数据移动设计的，但您可以将其用于其他UI操作。例如，当用户拖动一个颜色图标到另一个图标上时，你可以创建一个混合颜色的应用程序。

# 最基本的使用姿势
## 设置监听
需要监听拖拽事件的区域设置监听
```
rl_destination.setOnDragListener(this);//RelativeLayout
```

## 重写onDrag
这是最重要的
```
@Override
public boolean onDrag(View v, DragEvent event) {
	return true;
}
```
必须返回true,才能退拽

event封装了各种事件
```
int action = event.getAction();
switch (action){
	case DragEvent.ACTION_DRAG_STARTED:
		//Toast.makeText(TargetItemSelectedTestActivity.this, "开始拖动", Toast.LENGTH_LONG).show();
		break;
	case DragEvent.ACTION_DRAG_ENTERED:
		//Toast.makeText(TargetItemSelectedTestActivity.this, "进入目标区域", Toast.LENGTH_LONG).show();
		//这里可以改变目标区域的背景色
		break;
	case DragEvent.ACTION_DRAG_EXITED:
		//Toast.makeText(TargetItemSelectedTestActivity.this, "离开目标区域", Toast.LENGTH_LONG).show();
		break;

	case DragEvent.ACTION_DROP:
		//Toast.makeText(TargetItemSelectedTestActivity.this, "放手", Toast.LENGTH_LONG).show();
		break;
	case DragEvent.ACTION_DRAG_LOCATION:
		//Ignore the event
		break;

}
```

## 开始拖拽
```
targetView.startDrag(null,new View.DragShadowBuilder(targetView),null,0);
```
这样就可以有拖拽效果了.
