---
title: SwipeMenuListView 基本使用
date: 2017-08-09 20:47:15
tags:
 - Android
 - SwipeMenuListView
categories: [Android]
---

# SwipeMenuListView简介
一个带侧滑删除的ListView
> github地址: [https://github.com/baoyongzhang/SwipeMenuListView](https://github.com/baoyongzhang/SwipeMenuListView)


# 基本使用
添加依赖
android studio中
```
compile 'com.baoyz.swipemenulistview:library:1.3.0'
```
eclipse中下载相应jar包


使用方式和ListView是一样的,在listView的基础上有更多设置
在布局中引用控件
```
<com.baoyz.swipemenulistview.SwipeMenuListView
	android:id="@+id/share_manage_user_ls"
	android:layout_width="match_parent"
	android:layout_height="wrap_content"/>
```

Adapter跟普通ListView的adapter是一样的,不多说了
然后就是在代码中设置菜单
```
SwipeMenuCreator swipeMenuCreator = new SwipeMenuCreator() {
	@Override
	public void create(SwipeMenu menu) {
		SwipeMenuItem swipeMenuItem = new SwipeMenuItem(getApplicationContext());
		swipeMenuItem.setBackground(R.color.app_red);
		//swipeMenuItem.setTitleColor(R.color.white);
		swipeMenuItem.setTitleColor(Color.WHITE);
		swipeMenuItem.setWidth(DisplayUtil.dip2px(ShareManageActivity.this,90));
		swipeMenuItem.setTitle(R.string.delete);
		swipeMenuItem.setTitleSize(18);
		menu.addMenuItem(swipeMenuItem);
	}
};
mSwipeMenuListView.setMenuCreator(swipeMenuCreator);

```
创建SwipeMenuCreator对象,在方法体中创建SwipeMenuItem对象,然后把这个对象添加到Menu对象中,有多少个菜单就添加多少个SwipeMenuItem.比如这个菜单中就仅有一个删除菜单项.SwipeMenu对象方法根据名字就知道干什么用的了.

设置菜单项的监听
```
//设置监听
mSwipeMenuListView.setOnMenuItemClickListener(new SwipeMenuListView.OnMenuItemClickListener() {
	@Override
	public boolean onMenuItemClick(final int position, SwipeMenu menu, int index) {
		switch (index){
			case 0:
				break;
		}
		return false;
	}
});
```
根据position判断选择的是哪一项,从左到右为: 0 1 2....
好了,这个最简单的使用就这么多了.
