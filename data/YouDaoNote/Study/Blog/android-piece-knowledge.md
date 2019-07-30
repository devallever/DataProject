---
title: Android 零碎知识点和技巧
date: 2017-09-02 12:24:45
tags:
 - Android
categories: [Android]
---

# 使用DownloadManager下载文件
## 下载文件
```
        DownloadManager.Request req = new DownloadManager.Request(Uri.parse("urlString"));
        // 通过setAllowedNetworkTypes方法可以设置允许在何种网络下下载，
        // 也可以使用setAllowedOverRoaming方法，它更加灵活
        //req.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI);
        //req.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_MOBILE);
        //req.setAllowedNetworkTypes(DownloadManager.Request.NET)

        // 此方法表示在下载过程中通知栏会一直显示该下载，在下载完成后仍然会显示，
        // 直到用户点击该通知或者消除该通知。还有其他参数可供选择
        req.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);

        // 设置下载文件存放的路径，同样你可以选择以下方法存放在你想要的位置。
        // setDestinationUri
        // setDestinationInExternalPublicDir
        req.setDestinationInExternalFilesDir(context, "dirPath", "fileName");

        // 设置一些基本显示信息
        req.setTitle(mVideoData.getName());
        //req.setDescription("Title");
        req.setMimeType("video/mpeg4");
        mDownloadId = mDownloadManager.enqueue(req);
```

## 监听下载结果
```
public class DownloadReceiver extends BroadcastReceiver {
	private static final String TAG = "DownloadReceiver";
	@Override
	public void onReceive(Context context, Intent intent) {
		DownloadManager dm = (DownloadManager)
			context.getSystemService(context.DOWNLOAD_SERVICE);
		if (intent.getAction().equals(DownloadManager.ACTION_DOWNLOAD_COMPLETE)) {
		long downId = intent.getLongExtra(
			DownloadManager.EXTRA_DOWNLOAD_ID, -1);
	Cursor c = dm.query(new DownloadManager.Query().setFilterById(downId));
			if (c.moveToFirst()) {
				int status = c.getInt(c.getColumnIndex(DownloadManager.COLUMN_STATUS));
				if (status == DownloadManager.STATUS_SUCCESSFUL) {
				//downComplete(Uri.decode(c.getString(c.getColumnIndex(DownloadManager.COLUMN_LOCAL_URI)))) ;
					//downComplete( c.getString(c.getColumnIndex(DownloadManager.COLUMN_LOCAL_FILENAME))) ;
					Log.d(TAG, "onReceive: fileName = " + c.getString(c.getColumnIndex(DownloadManager.COLUMN_LOCAL_FILENAME)));
				} else {
					int reason = c.getInt(c.getColumnIndex(DownloadManager.COLUMN_REASON));
				}
			}
		}
	}
}
```

# 文件下载断点续传
1.获取已下载的文件长度.
```
long downloadedLength = 0;  //已下载文件长度
if (file.exists()){
downloadedLength = file.length();
}
```
2.获取服务器文件长度
```
OkHttpClient client = new OkHttpClient();
Request request = new Request.Builder()
	.url(url)
	.build();
Response response = client.newCall(request).execute();
long contentLength = response.body().contentLength();
```
3.设置请求，跳过已下载长度
```
OkHttpClient client = new OkHttpClient();
Request request = new Request.Builder()
	//断点下载
	.addHeader("RANGE", "bytes=" + downloadedLength + "-")
	.url(url)
	.build();
Response response = client.newCall(request).execute();
if (response != null){
	inputStream = response.body().byteStream();
	saveFile = new RandomAccessFile(file, "rw");
	saveFile.seek(downloadedLength); //跳过已下载的字节
	byte[] b = new byte[1024];
	int total = 0;
	int len;
	while ((len = inputStream.read(b)) != -1){
		total += len;
		saveFile.write(b,0,len);
		int progress = (int)((total + downloadedLength) * 100 / contentLength);
		publishProgress(progress);

	}
	response.body().close();
	return TYPE_SUCCESS;
}
```

# 判断网络类型
```
    private int getNetWorkType(){
        ConnectivityManager connectivityManager = (ConnectivityManager)  getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();

        if (networkInfo != null && networkInfo.isConnected()) {
            int type = networkInfo.getType();
            if (type == ConnectivityManager.TYPE_WIFI) {
                return NET_WORK_WIFI;
            } else if (type == ConnectivityManager.TYPE_MOBILE) {
                return NET_WORK_MOBILE;
            }
        }
        return -1;
    }
```

# HttpUrlConnection传递数据
关键代码
```
        URL url = new URL("http://119.23.30.121:80/JFitness/userController/getUserInfo");
        HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
        //设置每次传输的流大小
        httpURLConnection.setChunkedStreamingMode(128 * 1024); //128K
        //允许输入输出流
        httpURLConnection.setDoInput(true);
        httpURLConnection.setDoOutput(true);
        httpURLConnection.setUseCaches(false);
        httpURLConnection.setRequestProperty("Content-Type", "application/json");
        httpURLConnection.connect();

        // POST请求
        DataOutputStream out = new DataOutputStream(httpURLConnection.getOutputStream());
        //JSONObject obj = new JSONObject();
        //String json = java.net.URLEncoder.encode(obj.toString(), "utf-8");
        String json = "{\n" +
                "\t\"xid\":\"9875642\"\n" +
                "}";
        out.writeBytes(json);
        out.flush();
        out.close();
        // 读取响应
        BufferedReader reader = new BufferedReader(new InputStreamReader(httpURLConnection.getInputStream()));
        String lines;
        StringBuffer sb = new StringBuffer("");
        while ((lines = reader.readLine()) != null) {
            lines = URLDecoder.decode(lines, "utf-8");
            sb.append(lines);
        }
        System.out.println(sb);
        Log.d(TAG, "get user info = " + sb);
        reader.close();
        // 断开连接
        httpURLConnection.disconnect();
```
注意设置请求头数据格式为json
```
httpURLConnection.setRequestProperty("Content-Type", "application/json");
```

# 使用JsonObject创建json字符串
```
    private void createJson(){
        JSONObject jsonObj = new JSONObject();
        try {
            jsonObj.put("key_1",1);
            jsonObj.put("key_2","value");
            jsonObj.put("key_3",false);
            jsonObj.put("key_4",0.3f);
            jsonObj.put("key_5",3L);
            jsonObj.put("key_6",new Object());
            jsonObj.put("key_7",new ArrayList<>());
            Log.d(TAG, "createJson:  = \n" + jsonObj.toString());
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```
打印结果
```
{"key_1":1,"key_2":"value","key_3":false,"key_4":0.30000001192092896,"key_5":3,"key_6":"java.lang.Object@f94f4ed","key_7":"[]"}
```

# 获取外部存储路径
```
String dirType = Environment.DIRECTORY_DOWNLOADS;//或其他
String dir = Environment.getExternalStoragePublicDirectory(dirType).getPath() + "/";
```

# 根据Url获取文件名
```
String url = "https://github.com/devallever/MyCoolWeather/blob/master/app/simpleWeather.apk";
String fileName = url.substring(url.lastIndexOf("/")).split("/")[1];//不带"/"
```
打印结果
```
onCreate: fileName = simpleWeather.apk
```

# 判断第一次启动
```
//第一次启动
SharedPreferences sharedPreferences = getSharedPreferences("setting",MODE_PRIVATE);
boolean first_lanch = sharedPreferences.getBoolean("first_lanch",true);
if(first_lanch){
	Toast.makeText(this,"第一次启动",Toast.LENGTH_LONG).show();
	sharedPreferences.edit().putBoolean("first_lanch", false).commit
}else {
	Toast.makeText(this,"不是第一次启动",Toast.LENGTH_LONG).show();
}
```


# ViewPager设置手势监听和页面触摸监听
创建个手势监听器
```
    private int verticalMinistance = 100;            //水平最小识别距离
    private int minVelocity = 10;            //最小识别速度
    private GestureDetector.OnGestureListener onGestureListener = new GestureDetector.SimpleOnGestureListener() {
        @Override
        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
            if (e1.getX() - e2.getX() > verticalMinistance && Math.abs(velocityX) > minVelocity) {
                showToast("left");
            } else if (e2.getX() - e1.getX() > verticalMinistance && Math.abs(velocityX) > minVelocity) {
                showToast("right");
            } else if (e1.getY() - e2.getY() > verticalMinistance && Math.abs(velocityY) > minVelocity) {
                showToast("up");
            } else if (e2.getY() - e1.getY() > verticalMinistance && Math.abs(velocityY) > minVelocity) {
                showToast("down");
            }
            return false;
        }

        @Override
        public boolean onDown(MotionEvent e) {
            return  true;
        }
    };
```
其中,onDown()必须返回true, 否则onFling()不起效
然后创建GestureDetector对象
```
mGestureDetector = new GestureDetector(this,onGestureListener);
```
最后设置触摸监听,把事件交给手势监听器处理
```
mViewPager.setOnTouchListener(new View.OnTouchListener() {
	@Override
	public boolean onTouch(View view, MotionEvent motionEvent) {
		//把触摸事件交给手势
		mGestureDetector.onTouchEvent(motionEvent);
		return false;
	}
});
```

# LinearLayout从右到左排列
```
android:gravity="right"
```



# 使用较高API的方法
例如canvas.drawRoundRect()是minSDK21以上的方法要使用的话在方法上添加标注，然后在代码中判断当前sdk版本，进行不同的处理
```
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
@Override
protected void onDraw(Canvas canvas) {
	super.onDraw(canvas);
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
		canvas.drawRoundRect();
	}else{
		canvas.drawRect();
	}
}
```

# ListView(RecyclerView)的单选效果
在Adapter中公开一个方法,传入选中的position,然后刷新
```
public void setSelected(int selectedPosition){
	mSelectedPosition = selectedPosition;
	notifyDataSetChanged();
}
```

listView设置选项监听,在监听回调方法中,记录选中的position,调用上面方法setSelected()

在getView方法中判断position是否为mSelectedPosition,然后进行相应操作
```
if (mSelectedPosition == position){
	deviceListViewHolder.iv_select.setVisibility(View.VISIBLE);
	deviceListViewHolder.tv_device_name.setTextColor(context.getResources().getColor(R.color.code_blue));
}else {
	deviceListViewHolder.iv_select.setVisibility(View.INVISIBLE);
	deviceListViewHolder.tv_device_name.setTextColor(context.getResources().getColor(R.color.gray));
}
```


# EditText相关
## 光标颜色
在使用EditText的XML 文件中加入一个属性：
```
android:textCursorDrawable="@null"
```

android:textCursorDrawable   这个属性是用来控制光标颜色的，"@null"   是作用是让光标颜色和text color一样
比如 
```
android:textCursorDrawable="@color/black_color"
```
就可以设置成黑色。pad上面很多光标颜色和背景色一样。	

# TextView相关
##　更改字体
```
textView.setTypeface(Typeface.createFromAsset(context.getAssets(), "font/Shket-Regular_0.016.otf"));
```
字体文件放在/asset/font目录下
## 设置行距
```
android:lineSpacingExtra="4dp"
```
## 斜体加粗
```
android:textStyle="bold|italic"
```

# 修改语言

# 全屏显示


# 强制横屏
在AndroidManifest.xml中设置,在Activity节点中添加
```
android:screenOrientation="landscape" //锁定横屏
android:screenOrientation="portrait"	//锁定竖屏

//例如
<activity android:name=".BaseActivity"
	android:screenOrientation="landscape">
</activity>
```
或者在代码中添加
```
setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
```
在setContentView()方法之前

# 透明状态栏