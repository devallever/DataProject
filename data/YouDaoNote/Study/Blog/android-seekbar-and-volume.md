---
title: Android Seekbar调节系统音量
date: 2017-08-09 17:28:01
tags:
 - Android
 - SeekBar
categories: [Android]
---

# SeekBar简介
就是拖动的进度条, 可以用来调节音量

# 基本使用
需要两张图片,一张是背景,一张是选中的]时候的样式,
还有一张图片,用来显示当前进度的指示图标
这些可用用图片,亦可以永drawable绘图

## 背景图片
volume_control_bg.xml
```
<?xml version="1.0" encoding="utf-8"?>

<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@android:id/background"
        android:drawable="@drawable/set_voice_bar_nor" />
    <item
        android:id="@android:id/progress"
        android:drawable="@drawable/set_voice_bar_sel" />
</layer-list>
```

控制点volume_control_point.xml
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">

    <solid
        android:color="@color/white"/>
    <size
        android:width="15dp"
        android:height="15dp"/>
    <stroke
        android:color="@color/code_blue"
        android:width="2dp"/>

</shape>
```

## 使用SeekBar
在布局文件中
```
<SeekBar
	android:id="@+id/setting_volume_control_sb"
	android:layout_width="200dp"
	android:layout_height="wrap_content"
	android:layout_centerVertical="true"
	android:layout_toLeftOf="@id/setting_loud_iv"
	android:thumb="@drawable/volume_control_point"
	android:thumbOffset="0dp"
	android:maxHeight="2dp"
	android:minHeight="2dp"
	android:progressDrawable="@drawable/volume_control_bg"
/>
```
> thumb:指定控制点
> progressDrawable:指定背景
> maxHeight:指定图片最大高度

在代码中,不用任何操作,就可以看到该空间,并拖动了

## 控制音量
首先获取AudioManager,用来管理音量
```
mAudioManager = (AudioManager)getSystemService(AUDIO_SERVICE);
```

获取最大音量和当前音量设置给seekbar
```
int maxVolume = mAudioManager.getStreamMaxVolume(AudioManager.STREAM_MUSIC);//获取媒体声音最大值
seekBar.setMax(maxVolume);
int currentVolume = mAudioManager.getStreamVolume(AudioManager.STREAM_MUSIC);
seekBar.setProgress(currentVolume);//设置当前进度
```

监听seekbar的拖动
```
seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
	@Override
	public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
		if(fromUser){
		//设置媒体音量
			mAudioManager.setStreamVolume(AudioManager.STREAM_MUSIC, progress, 0);
			int currentVolume = mAudioManager.getStreamVolume(AudioManager.STREAM_MUSIC);
			seekBar.setProgress(currentVolume);
		}
	}

	@Override
	public void onStartTrackingTouch(SeekBar seekBar) {

	}

	@Override
	public void onStopTrackingTouch(SeekBar seekBar) {

	}
});
```

监听系统音量变化
当系统音量改变时,会发送广播,因此需要定义广播接收器监听广播
```
private class VolumeReceiver extends BroadcastReceiver{

	@Override
	public void onReceive(Context context, Intent intent) {
		if(intent.getAction().equals("android.media.VOLUME_CHANGED_ACTION")){
		int currentVolume = mAudioManager.getStreamVolume(AudioManager.STREAM_MUSIC);
		seekBar.setProgress(currentVolume);
		}
	}
}
```