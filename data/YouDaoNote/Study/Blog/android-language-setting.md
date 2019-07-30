---
title: Android 修改App语言
date: 2017-08-09 21:38:09
tags:
 - Android
 - Language
categories: [Android]
---

# 定义资源文件
在res目录下创建
values	默认
values-en	英文
values-zh	中文
分别存放不同的语言资源

# 保存语言设置
可以用SharePreference保存语言设置
```
public static void setLanAtr(String language){
	SharedPreferences sharedPreferences = MyApplication.getInstance().getSharedPreferences("setting_share", 0);
	SharedPreferences.Editor editor = sharedPreferences.edit();
	editor.putString("language",language);
	editor.commit();
}
```

另外相应的可以获取语言设置
```
public static String getLanAtr(){
	SharedPreferences sharedPreferences = MyApplication.getInstance().getSharedPreferences("setting_share", 0);
	String lanAtr = sharedPreferences.getString("language", Contants.LANGUAGE_CHINESE);
	return  lanAtr;
}
```

# 设置语言
在BaseActivity或者Application中开启全局语言设置
```
Utils.changeAppLanguage(getResources(), SharePreferencesUtil.getLanAtr());
```

重点:changeAppLanguage()方法如下
```
public static void changeAppLanguage(Resources resources, String lanAtr){
	Configuration configuration = resources.getConfiguration();
	DisplayMetrics displayMetrics = resources.getDisplayMetrics();
	if (lanAtr.equals(Contants.LANGUAGE_ENGLISH)){
		configuration.locale = Locale.ENGLISH;
	}else {
		configuration.locale = Locale.CHINESE;
	}
	resources.updateConfiguration(configuration,displayMetrics);
}
```