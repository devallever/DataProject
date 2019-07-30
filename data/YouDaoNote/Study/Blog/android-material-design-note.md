---
title: Material Design 笔记
date: 2017-04-21 22:08:13
tags:
 - Android
 - Material Design
categories: [Android]
---
# 1.Toolbar的使用

## 1.1设置应用的主题为NoActionBar
```
android:theme="@style/AppTheme"
```
这个主题在sytle文件中定义
```
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
<!-- Customize your theme here. -->
	<item name="colorPrimary">@color/colorGreen_300</item>
	<item name="colorPrimaryDark">@color/colorGreen700</item>
	<item name="colorAccent">@color/colorAccent</item>
</style>
```
		
## 1.2在布局文件中使用Toolbar控件
```
<android.support.v7.widget.Toolbar
android:id="@+id/id_material_design_activity_toolbar"
android:layout_width="match_parent"
android:layout_height="?actionBarSize"
android:background="@color/colorGreen700"
android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
```
## 1.3在Activity中这样用
		
```java
Toolbar toolbar = (Toolbar)findViewById(R.id.id_material_design_activity_toolbar);
setSupportActionBar(toolbar);
		```
## 1.4设置toolbar的home图标并设置监听
```java
ActionBar actionBar = getSupportActionBar();
if (actionBar != null){
actionBar.setDisplayHomeAsUpEnabled(true);
actionBar.setHomeAsUpIndicator(R.mipmap.ic_arrow_back_white_36dp);
}

```
在onOptionsItemSelected()方法中设置监听
```
case android.R.id.home:
drawerLayout.openDrawer(GravityCompat.START);
break;
```
## 1.5设置toolbar菜单
```
<?xml version="1.0" encoding="utf-8"?>
		<menu
		    xmlns:app="http://schemas.android.com/apk/res-auto"
		    xmlns:android="http://schemas.android.com/apk/res/android">
		    <item
		        android:id="@+id/id_menu_notification"
		        android:title="Notification"
		        android:icon="@mipmap/ic_notifications_active_white_24dp"
		        app:showAsAction="never"/>
		    <item
		        android:id="@+id/id_menu_sms"
		        android:title="Notification"
		        android:icon="@mipmap/ic_sms_white_24dp"
		        app:showAsAction="never"/>
		    <item
		        android:id="@+id/id_menu_person"
		        android:title="Notification"
		        android:icon="@mipmap/ic_person_outline_white_24dp"
		        app:showAsAction="never"/>
		</menu>
		```
		然后在Activity中加载这个菜单
		```
		@Override
		public boolean onCreateOptionsMenu(Menu menu) {
		        getMenuInflater().inflate(R.menu.toolbar_menu,menu);
		        return true;
		}
		
		@Override
		public boolean onOptionsItemSelected(MenuItem item) {
		        int id = item.getItemId();
		        switch (id){
		            case R.id.id_menu_notification:
		                Toast.makeText(this,"Notification",Toast.LENGTH_SHORT).show();
		                break;
		            case R.id.id_menu_sms:
		                Toast.makeText(this,"SMS",Toast.LENGTH_SHORT).show();
		                break;
		            case R.id.id_menu_person:
		                Toast.makeText(this,"Contacts",Toast.LENGTH_SHORT).show();
		                break;
		            case android.R.id.home:
		                drawerLayout.openDrawer(GravityCompat.START);
		                break;
		
		        }
		        return true;
		}
```
		
# 2.DrawerLayout与NavigationView的爱恨情仇
## 2.1在布局中使用DrawerLayout和NavigationView
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:id="@+id/id_material_design_activity_drawer_layout">
		
<android.support.design.widget.CoordinatorLayout
	android:layout_width="match_parent"
	android:layout_height="match_parent">
<android.support.v7.widget.Toolbar
	android:id="@+id/id_material_design_activity_toolbar"
	android:layout_width="match_parent"
	android:layout_height="?actionBarSize"
	android:background="@color/colorGreen700"
	android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
	app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
		
<android.support.design.widget.FloatingActionButton
	android:id="@+id/id_material_design_activity_fab"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:layout_gravity="bottom|right"
	android:layout_margin="16dp"
	android:src="@mipmap/ic_notifications_active_white_24dp"
	app:elevation="8dp"/>
		
</android.support.design.widget.CoordinatorLayout>
		
<android.support.design.widget.NavigationView
	android:id="@+id/id_material_design_navigation_view"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:layout_gravity="start"
	app:menu="@menu/nav_menu"
	app:headerLayout="@layout/nav_header_layout"/>
		        
</android.support.v4.widget.DrawerLayout>
```
其中NavigationVIew包含一个menu菜单和头布局nav_menu.xml
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto">
		
	<group android:checkableBehavior="single">
	<item
	android:id="@+id/id_nav_menu_item_like"
	android:icon="@mipmap/ic_favorite_black_24dp"
	android:title="Like"/>
	<item
	android:id="@+id/id_nav_menu_item_alarm"
	android:icon="@mipmap/ic_alarm_black_24dp"
	android:title="Alarm"/>
	<item
	android:id="@+id/id_nav_menu_item_account"
	android:icon="@mipmap/ic_account_circle_black_24dp"
	android:title="Account"/>
	<item
	android:id="@+id/id_nav_menu_item_setting"
	android:icon="@mipmap/ic_build_black_24dp"
	android:title="Setting"/>
			
	</group>
	</menu>
```
nav_header_layout
```
<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="match_parent"
	android:layout_height="180dp"
	android:padding="10dp"
	android:background="@color/colorGreen700">
			
	<de.hdodenhof.circleimageview.CircleImageView
	android:id="@+id/id_nav_header_iv_head"
	android:layout_width="70dp"
	android:layout_height="70dp"
	android:src="@mipmap/h_01"
	android:layout_centerInParent="true"/>
		
<TextView
	android:id="@+id/id_nav_header_tv_email"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:text="devallever@163.com"
	android:textColor="@color/white"
	android:layout_alignParentBottom="true"/>
			
<TextView
	android:id="@+id/id_nav_header_tv_username"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:layout_marginBottom="5dp"
	android:text="Devallever"
	android:textColor="@color/white"
	android:layout_above="@id/id_nav_header_tv_email"/>
			
</RelativeLayout>
```
## 2.2设置NavigationView的菜单监听
```
navigationView = (NavigationView)findViewById(R.id.id_material_design_navigation_view);
navigationView.setCheckedItem(R.id.id_nav_menu_item_like);
navigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
	@Override
	public boolean onNavigationItemSelected(@NonNull MenuItem item) {
	int id = item.getItemId();
	switch (id){
		case R.id.id_nav_menu_item_like:
			Toast.makeText(MaterialDesignActivity.this, "Like",Toast.LENGTH_SHORT).show();
			break;
		case R.id.id_nav_menu_item_account:
			Toast.makeText(MaterialDesignActivity.this, "Account",Toast.LENGTH_SHORT).show();
			break;
		case R.id.id_nav_menu_item_alarm:
			Toast.makeText(MaterialDesignActivity.this, "Alarm",Toast.LENGTH_SHORT).show();
			break;
		case R.id.id_nav_menu_item_setting:
			Toast.makeText(MaterialDesignActivity.this, "Setting",Toast.LENGTH_SHORT).show();
			break;
	}
	return true;
	}
});
```
## 2.3把toolbar与NavigationView关联起来，并实现home的动画效果
```
ActionBarDrawerToggle actionBarDrawerToggle = 
new ActionBarDrawerToggle(this,drawerLayout,toolbar,R.string.app_name, R.string.app_name);
actionBarDrawerToggle.syncState();
```
		
# 3FloatActionButton
## 3.1在布局中使用FloatActionButton控件
```
<android.support.design.widget.FloatingActionButton
	android:id="@+id/id_material_design_activity_fab"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"	
	android:layout_gravity="bottom|right"
	android:layout_margin="16dp"
	android:src="@mipmap/ic_notifications_active_white_24dp"
	app:elevation="8dp"/>
```
## 3.2设置监听，和一般的按钮设置监听是一样的
```
fab.setOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View v) {
	
	}
});
```
		
# 4Snackbar
当点击按钮时候弹出Snackbar
```
fab.setOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View v) {
		Snackbar.make(v, "Remind!", Snackbar.LENGTH_INDEFINITE)
			.setAction("I know.", new View.OnClickListener() {
				@Override
				public void onClick(View v) {
					Toast.makeText(MaterialDesignActivity.this,"OK", Toast.LENGTH_SHORT).show();
				}
			})
			.show();
	}
});
```
		
# 5CoordinatorLayout
加强版的FrameLayout
```
<android.support.design.widget.CoordinatorLayout
	android:layout_width="match_parent"
	android:layout_height="match_parent">
<android.support.v7.widget.Toolbar
	android:id="@+id/id_material_design_activity_toolbar"
	android:layout_width="match_parent"
	android:layout_height="?actionBarSize"
	android:background="@color/colorGreen700"
	android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
	app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
	
<android.support.design.widget.FloatingActionButton
	android:id="@+id/id_material_design_activity_fab"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:layout_gravity="bottom|right"
	android:layout_margin="16dp"
	android:src="@mipmap/ic_notifications_active_white_24dp"
	app:elevation="8dp"/>
	
</android.support.design.widget.CoordinatorLayout
```