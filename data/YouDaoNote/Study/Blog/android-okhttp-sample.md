---
title: OkHttp的简单使用
date: 2017-06-30 20:07:13
tags:
 - Android
 - OkHttp
 - Network
categories: [Android]
---


# OkHttp简介
关于Okhttp的简介，相信大家都不陌生了，这里就不讲了。

# 初级用法

## Get请求
比如这里的使用get方法进行登录操作
```java
    private void doLoginGet(){
        final String url = "xxxxx?username=allever?password=123456"
        OkHttpUtil.loginGet(url, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {

            }

            @Override
            public void onResponse(Call call, final Response response) throws IOException {
                handleLogin(response.body().string(););
            }
        });
    }
```

loginGet方法如下：
```java
    public static void loginGet(String url, Callback callback){
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url(url)
                .build();
        client.newCall(request).enqueue(callback);
    }
```

是不是很简单，创建OkhttpClient实例，创建请求，然后执行请求，然后进行回调

处理结果
```
    private void handleLogin(final String result){
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Gson gson = new Gson();
                LoginRoot loginRoot = gson.fromJson(result,LoginRoot.class);
                User user = loginRoot.getUser();
                tv_result.setText(user.getNickname() + "\n" + user.getSignature());
            }
        });
    }
```
因为回调方法在子线程中，所以要切回到主线程中，方法有很多，可以使用Handler，这里直接runOnUiThread()就好了。简单起见，另外Gson处理json数据不在讨论范围之内，

## Post请求
例子同上，登录操作
```java
    private void doLoginPost(){
        OkHttpUtil.loginPost(username, password, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {

            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                handleLogin(response.body().string());
            }
        });
    }
```

```
    public static void loginPost(String username, String password, Callback callback){
        String url = BASE_URL + "LoginServlet";
        OkHttpClient client = new OkHttpClient();
        FormBody.Builder builder = new FormBody.Builder();
        builder.add("username",username);
        RequestBody requestBody = builder.build();
        Request request = new Request.Builder()
                .url(url)
                .post(requestBody)
                .build();
        client.newCall(request).enqueue(callback);

    }
```
唯一不同的就是这里啦，首先还是创建OkhttpClient实例，这里使用了FormBody表单, 用这个表单创建请求体，然后创建请求，最后执行请求，回调
最后的处理方法同上。

## 异步请求
上面两个例子都是异步请求，因为不用咋们手动开启子线程，它会自动的开启线程去处理的

## 同步请求
例如，退出登录的例子
```java
private void doLogoutTongBu(){
        OkHttpUtil.logoutTongBu(handler);
    }
```

```java
    public static void logoutTongBu(final Handler handler){
        new Thread(new Runnable() {
            @Override
            public void run() {
                String url = "xxxxx/logout";
                OkHttpClient client = new OkHttpClient();
                FormBody.Builder builder = new FormBody.Builder();
                if (!SharePreferenceUtil.getSessionId().equals("")){
                    RequestBody requestBody = builder.build();
                    Request request = new Request.Builder()
                            .url(url)
                            .post(requestBody)
                            .addHeader("Cookie", "JSESSIONID=" + SharePreferenceUtil.getSessionId())
                            .build();
                    Response response = null;
                    try {
                        response = client.newCall(request).execute();
                        String result = response.body().string();
                        Message message = new Message();
                        message.what = MESSAGE_LOGOUT;
                        message.obj = result;
                        handler.sendMessage(message);
                    }catch (IOException ioe){
                        ioe.printStackTrace();
                    }

                }else {
                    Message message = new Message();
                    message.what = MESSAGE_LOGOUT;
                    message.obj = "未登录";
                    handler.sendMessage(message);
                }

            }
        }).start();
    }
```
这里手动开启了一个线程进行网络请求，还是使用Post方法，跟上面的没啥区别，注意这里
```
addHeader("Cookie", "JSESSIONID=" + SharePreferenceUtil.getSessionId())
```
添加Cookie的头部信息，Cookie就是用来持久化用户登录
还有就是执行请求的方法有些区别，同步执行是：
```
response = client.newCall(request).execute();
```
异步执行是：
```
client.newCall(request).enqueue(callback);
```

同步执行的结果为Response对象，就是服务器返回的数据

# 高级用法
## 上传文件和表单数据
例如发一条说说，包括图片和文字
```java
    private void doAddNews(String content){
        OkHttpUtil.addNews(content, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {

            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Log.d(TAG, "onResponse: result = " + response.body().string());
            }
        });
    }

```

```
    public static void addNews(String content,Callback callback){
        OkHttpClient client = new OkHttpClient();
        MultipartBody multipartBody = new MultipartBody.Builder()
                .setType(MultipartBody.FORM)
                .addFormDataPart("content",content)
                .addFormDataPart("city","番禺")
                .addFormDataPart("longitude","111.22")
                .addFormDataPart("latitude","22.22")
                .addPart(Headers.of("Content-Disposition",
                        "form-data; name=\"part1"+ "\""),
                        RequestBody.create(
                                MediaType.parse("application/octet-stream"),
                                new File(Environment.getExternalStorageDirectory().getPath()+"/","jiaxin.png")))
                .build();

        if (SharePreferenceUtil.getSessionId().equals("")){
        }
        Request request = new Request.Builder()
                .url(BASE_URL + "AddNewsServlet")
                .post(multipartBody)
                .addHeader("Cookie", "JSESSIONID=" + SharePreferenceUtil.getSessionId())
                .build();

        client.newCall(request).enqueue(callback);
    }
```
设置表单类型
```
setType(MultipartBody.FORM)
```
使用MultipartBody对象把表单数据和文件数据形式封装，表单使用
```
addFormDataPart("content",content)
```
用键值对保存

文件用
```
.addPart(Headers.of("Content-Disposition",
                        "form-data; name=\"part1"+ "\""),
                        RequestBody.create(
                                MediaType.parse("application/octet-stream"),
                                fileObject))
```
其中：part1是服务器根据键找到part的那个键
```
request.getPart("part1");
```
如果多文件的话就多次调用addPart()
> 必须设置表单类型

```
setType(MultipartBody.FORM)
```