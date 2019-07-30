---
title: Android WebView使用及与JavaScript互调
date: 2017-08-16 20:33:40
tags:
 - Android
 - WebView
 - JavaScript
categories: [Android]
---

# WebView基本使用
## 在布局中使用控件
```
<WebView
	android:id="@+id/id_web_test_2_wb"
	android:layout_width="match_parent"
	android:layout_height="match_parent"/>
```

## 在代码中
```
private void initWebView() {
	webView = (WebView)findViewById(R.id.id_web_test_2_wb);
	//不跳转到其他浏览器
	webView.setWebViewClient(new WebViewClient() {
		@Override
		public boolean shouldOverrideUrlLoading(WebView view, String url) {
			view.loadUrl(url);
			return true;
		}
	});
	WebSettings settings = webView.getSettings();
	//支持JS
	settings.setJavaScriptEnabled(true);
	//加载本地html文件
	webView.loadUrl("file:///android_asset/JavaAndJavaScriptCall.html");
	//加载网页
	//webView.loadUrl("https://www.baidu.com/");
}
```

监听返回键
```
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
	// TODO Auto-generated method stub
	if ((keyCode == KeyEvent.KEYCODE_BACK) &&   webView.canGoBack()) {
		webView.goBack();
		return true;
	}
	return super.onKeyDown(keyCode, event);
}
```

# 原生(Java)与JavaScript互调
例如加载这个本地html
在输入框中随便输入东西，然后点击确定，调用js方法，下面显示欢迎xxxx
点击网页中的按钮，调用原生方法。toast一个。
![](https://github.com/devallever/DataProject/blob/master/data/allsample/web-js.png?raw=true)

## html页面
在main目录下新建assets文件夹，在assets目录下新建JavaAndJavaScriptCall.html
```
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    <script type="text/javascript">

    function javaCallJs(arg){
         document.getElementById("content").innerHTML =
             ("欢迎："+arg );
    }

    </script>
</head>
<body>
<div id="content"> 请在上方输入您的用户名</div>
<input type="button" value="点击Android被调用" onclick="window.Android.showToast('JS中传来的参数')"/>
</body>
</html>
```
## 原生布局
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/ll_root"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="20dp"
        android:background="#000088">
        <EditText
            android:id="@+id/et_user"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:hint="输入WebView中要显示的用户名"
            android:background="#008800"
            android:textSize="16sp"
            android:layout_weight="1"/>
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="40dp"
            android:layout_marginRight="20dp"
            android:textSize="16sp"
            android:text="确定"
            android:onClick="click"/>
    </LinearLayout>

    <WebView
        android:id="@+id/id_web_test_2_wb"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```
一看就懂，不解释。

## 加载本地html
```
webView.loadUrl("file:///android_asset/JavaAndJavaScriptCall.html");
```

## 调用JavaScript方法
在点击确定之后触发调用
```
//java调用JS方法
webView.loadUrl("javascript:javaCallJs(" + "'" + et_user.getText().toString()+"'"+")");
```
这里比较难看，log出来的url是这样的
```
javascript:javaCallJs('inputValue')
```
javaCallJs()就是JavaScript代码中的方法名，里面是参数
```
<script type="text/javascript">
function javaCallJs(arg){
	document.getElementById("content").innerHTML =
	("欢迎："+arg );
}
</script>
```

## JavaScript调用原生方法
通常会建一个类用来封装被JavaScript调用的方法
```
private class JSInterface {
//JS需要调用的方法
	@JavascriptInterface
	public void showToast(String arg){
		Toast.makeText(WebTest2Activity.this,arg,Toast.LENGTH_SHORT).show();
	}
}
```
使用 @JavascriptInterface标注的方法就是被JavaScript调用的方法

然后设置
```
webView.addJavascriptInterface(new JSInterface(),"Android");
```
Android是随便起的，是JavaScript中的对象名，
源码如下：
```
	/**
     * @param object the Java object to inject into this WebView's JavaScript
     *               context. Null values are ignored.
     * @param name the name used to expose the object in JavaScript
     */
	public void addJavascriptInterface(Object object, String name) {
		checkThread();
		mProvider.addJavascriptInterface(object, name);
	}
```
再看一下html中的代码
```
<input type="button" value="点击Android被调用" onclick="window.Android.showToast('JS中传来的参数')"/>
```
其中Android就是webview添加JavaScript接口时指定的JavaScript对象名
当点击html中按钮时，就会调用showToast()方法弹出Toast