##### 2019.07.30，（）Glide停止请求和重新请求

```
Glide.with(context).resumeRequests()
Glide.with(context).pauseRequests()
```

##### 2019.07.24，（）Glide加载https图片不显示的问题

 - 原因：
 - 解决办法：通过修改加载图片源码的方式

做法：

 - 从 implementation "com.github.bumptech.glide:okhttp3-integration:4.7.1"这个包中复制下面几个文件并修改如下：

OkHttpGlideModule：
```java
import android.content.Context;
import android.support.annotation.NonNull;
import com.bumptech.glide.Glide;
import com.bumptech.glide.GlideBuilder;
import com.bumptech.glide.Registry;
import com.bumptech.glide.load.model.GlideUrl;
import com.bumptech.glide.module.GlideModule;
import okhttp3.OkHttpClient;

import javax.net.ssl.*;
import java.io.InputStream;
import java.security.KeyManagementException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.cert.X509Certificate;
import java.util.concurrent.TimeUnit;

/**
 * A {@link com.bumptech.glide.module.GlideModule} implementation to replace Glide's default
 * {@link java.net.HttpURLConnection} based {@link com.bumptech.glide.load.model.ModelLoader}
 * with an OkHttp based {@link com.bumptech.glide.load.model.ModelLoader}.
 *
 * <p> If you're using gradle, you can include this module simply by depending on the aar, the
 * module will be merged in by manifest merger. For other build systems or for more more
 * information, see {@link com.bumptech.glide.module.GlideModule}. </p>
 *
 * @deprecated Replaced by {@link OkHttpLibraryGlideModule} for Applications that use Glide's
 * annotations.
 */
@Deprecated
public class OkHttpGlideModule implements GlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    // Do nothing.
  }
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    X509TrustManager xtm = new X509TrustManager() {
      @Override
      public void checkClientTrusted(X509Certificate[] chain, String authType) {
      }

      @Override
      public void checkServerTrusted(X509Certificate[] chain, String authType) {
      }

      @Override
      public X509Certificate[] getAcceptedIssuers() {
        X509Certificate[] x509Certificates = new X509Certificate[0];
        return x509Certificates;
      }
    };

    SSLContext sslContext = null;
    try {
      sslContext = SSLContext.getInstance("SSL");

      sslContext.init(null, new TrustManager[]{xtm}, new SecureRandom());

    } catch (NoSuchAlgorithmException e) {
      e.printStackTrace();
    } catch (KeyManagementException e) {
      e.printStackTrace();
    }
    HostnameVerifier DO_NOT_VERIFY = new HostnameVerifier() {
      @Override
      public boolean verify(String hostname, SSLSession session) {
        return true;
      }
    };

    OkHttpClient.Builder builder = new OkHttpClient.Builder();
    builder.sslSocketFactory(sslContext.getSocketFactory());
    builder.hostnameVerifier(DO_NOT_VERIFY);
    builder.hostnameVerifier(new HostnameVerifier() {
      @Override
      public boolean verify(String hostname, SSLSession session) {
        return true;
      }
    });

    builder.connectTimeout(20, TimeUnit.SECONDS);
    builder.readTimeout(20,TimeUnit.SECONDS);

    registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory(builder.build()));
  }
}

```

OkHttpLibraryGlideModule：
```java
import android.content.Context;
import android.support.annotation.NonNull;
import com.bumptech.glide.Glide;
import com.bumptech.glide.Registry;
import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.load.model.GlideUrl;
import com.bumptech.glide.module.AppGlideModule;
import com.bumptech.glide.module.LibraryGlideModule;

import java.io.InputStream;

/**
 * Registers OkHttp related classes via Glide's annotation processor.
 *
 * <p>For Applications that depend on this library and include an
 * {@link AppGlideModule} and Glide's annotation processor, this class
 * will be automatically included.
 */
@GlideModule
public final class OkHttpLibraryGlideModule extends LibraryGlideModule {
  @Override
  public void registerComponents(@NonNull Context context, @NonNull Glide glide,
      @NonNull Registry registry) {
    //registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory());
    registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory());
  }
}

```

OkHttpStreamFetcher：
```java
import android.support.annotation.NonNull;
import android.util.Log;
import com.bumptech.glide.Priority;
import com.bumptech.glide.load.DataSource;
import com.bumptech.glide.load.HttpException;
import com.bumptech.glide.load.data.DataFetcher;
import com.bumptech.glide.load.model.GlideUrl;
import com.bumptech.glide.util.ContentLengthInputStream;
import com.bumptech.glide.util.Preconditions;
import okhttp3.Call;
import okhttp3.Request;
import okhttp3.Response;
import okhttp3.ResponseBody;

import java.io.IOException;
import java.io.InputStream;
import java.util.Map;

/**
 * Fetches an {@link InputStream} using the okhttp library.
 */
public class OkHttpStreamFetcher implements DataFetcher<InputStream>, okhttp3.Callback {
  private static final String TAG = "OkHttpFetcher";
  private final Call.Factory client;
  private final GlideUrl url;
  private InputStream stream;
  private ResponseBody responseBody;
  private DataCallback<? super InputStream> callback;
  // call may be accessed on the main thread while the object is in use on other threads. All other
  // accesses to variables may occur on different threads, but only one at a time.
  private volatile Call call;

  // Public API.
  @SuppressWarnings("WeakerAccess")
  public OkHttpStreamFetcher(Call.Factory client, GlideUrl url) {
    this.client = client;
    this.url = url;
  }

  @Override
  public void loadData(@NonNull Priority priority,
      @NonNull final DataCallback<? super InputStream> callback) {
    Request.Builder requestBuilder = new Request.Builder().url(url.toStringUrl());
    for (Map.Entry<String, String> headerEntry : url.getHeaders().entrySet()) {
      String key = headerEntry.getKey();
      requestBuilder.addHeader(key, headerEntry.getValue());
    }
    Request request = requestBuilder.build();
    this.callback = callback;

    call = client.newCall(request);
    call.enqueue(this);
  }

  @Override
  public void onFailure(@NonNull Call call, @NonNull IOException e) {
    if (Log.isLoggable(TAG, Log.DEBUG)) {
      Log.d(TAG, "OkHttp failed to obtain result", e);
    }

    callback.onLoadFailed(e);
  }

  @Override
  public void onResponse(@NonNull Call call, @NonNull Response response) {
    responseBody = response.body();
    if (response.isSuccessful()) {
      long contentLength = Preconditions.checkNotNull(responseBody).contentLength();
      stream = ContentLengthInputStream.obtain(responseBody.byteStream(), contentLength);
      callback.onDataReady(stream);
    } else {
      callback.onLoadFailed(new HttpException(response.message(), response.code()));
    }
  }

  @Override
  public void cleanup() {
    try {
      if (stream != null) {
        stream.close();
      }
    } catch (IOException e) {
      // Ignored
    }
    if (responseBody != null) {
      responseBody.close();
    }
    callback = null;
  }

  @Override
  public void cancel() {
    Call local = call;
    if (local != null) {
      local.cancel();
    }
  }

  @NonNull
  @Override
  public Class<InputStream> getDataClass() {
    return InputStream.class;
  }

  @NonNull
  @Override
  public DataSource getDataSource() {
    return DataSource.REMOTE;
  }
}

```

OkHttpUrlLoader：
```java
import android.support.annotation.NonNull;
import com.bumptech.glide.load.Options;
import com.bumptech.glide.load.model.GlideUrl;
import com.bumptech.glide.load.model.ModelLoader;
import com.bumptech.glide.load.model.ModelLoaderFactory;
import com.bumptech.glide.load.model.MultiModelLoaderFactory;
import okhttp3.Call;
import okhttp3.OkHttpClient;

import java.io.InputStream;

/**
 * A simple model loader for fetching media over http/https using OkHttp.
 */
public class OkHttpUrlLoader implements ModelLoader<GlideUrl, InputStream> {

  private final Call.Factory client;

  // Public API.
  @SuppressWarnings("WeakerAccess")
  public OkHttpUrlLoader(@NonNull Call.Factory client) {
    this.client = client;
  }

  @Override
  public boolean handles(@NonNull GlideUrl url) {
    return true;
  }

  @Override
  public LoadData<InputStream> buildLoadData(@NonNull GlideUrl model, int width, int height,
      @NonNull Options options) {
    return new LoadData<>(model, new OkHttpStreamFetcher(client, model));
  }

  /**
   * The default factory for {@link OkHttpUrlLoader}s.
   */
  // Public API.
  @SuppressWarnings("WeakerAccess")
  public static class Factory implements ModelLoaderFactory<GlideUrl, InputStream> {
    private static volatile Call.Factory internalClient;
    private final Call.Factory client;

    private static Call.Factory getInternalClient() {
      if (internalClient == null) {
        synchronized (Factory.class) {
          if (internalClient == null) {
            internalClient = new OkHttpClient();
          }
        }
      }
      return internalClient;
    }

    /**
     * Constructor for a new Factory that runs requests using a static singleton client.
     */
    public Factory() {
      this(getInternalClient());
    }

    /**
     * Constructor for a new Factory that runs requests using given client.
     *
     * @param client this is typically an instance of {@code OkHttpClient}.
     */
    public Factory(@NonNull Call.Factory client) {
      this.client = client;
    }

    @NonNull
    @Override
    public ModelLoader<GlideUrl, InputStream> build(MultiModelLoaderFactory multiFactory) {
      return new OkHttpUrlLoader(client);
    }

    @Override
    public void teardown() {
      // Do nothing, this instance doesn't own the client.
    }
  }
}

```
 - 在AndroidManifest.xml中配置
```xml
<Application>
    <meta-data
        android:name="com.xxx.OkHttpGlideModule"
        android:value="GlideModule"/>
</Application>
```

> 参考：[Glide4.0 加载 Https 图片:https://www.jianshu.com/p/af293c2a1d2f](https://www.jianshu.com/p/af293c2a1d2f)

> 参考：[Glide4 使用教程: http://rc.interlib.com.cn:8088/rcrobotsite//web/api/rcrobot/machine/tips.html?libcode=P3GD0755006&deviceId=1](http://rc.interlib.com.cn:8088/rcrobotsite//web/api/rcrobot/machine/tips.html?libcode=P3GD0755006&deviceId=1)


##### 2019.07.19，（）NDK构建遇到的问题

**No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-androi**
```
"No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-android"

```

 - NDK说明

```
This version of the NDK is incompatible with the Android Gradle plugin
   version 3.0 or older. If you see an error like
   `No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-android`,
   update your project file to [use plugin version 3.1 or newer]. You will also
   need to upgrade to Android Studio 3.1 or newer.

```

 - 解决办法

其实解决方法很简单，就是修改build.gradle中的红字部分，改为3.1以上版本即可

```
dependencies {
    classpath 'com.android.tools.build:gradle:3.2.0'

    // NOTE: Do not place your application dependencies here; they belong
    // in the individual module build.gradle files
}
```

##### 2019.07.17，（） 8.0悬浮窗问题

```
//解决8.0 悬浮窗崩溃的问题
if (Build.VERSION.SDK_INT >= 26) {
    floatWindowViewParams?.type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY
} else {
    floatWindowViewParams?.type = WindowManager.LayoutParams.TYPE_PHONE
}
```
 
##### 2019.07.17，（）7.0 Uri权限问题

res->xml: filepath.xml

```
<!--FileProvider提供的共享目录配置文件-->
<!-- 基本覆盖了手机中所有目录的权限， 外置SD卡对应共享目录的权限需要配置自定义的root-path中。 -->
<paths>
    <!--内置SD卡 Environment.getExternalStorageDirectory() .表示共享所有的目录，也可以指定共享的目录-->
    <external-path
        name="external-path"
        path="."/>
    <!--内置SD卡 Context.getExternalCacheDir() .表示共享所有的目录，也可以指定共享的目录-->
    <external-cache-path
        name="external-cache-path"
        path="."/>
    <!--内置SD卡 Context.getExternalFilesDir(null) .表示共享所有的目录，也可以指定共享的目录-->
    <external-files-path
        name="external-files-path"
        path="."/>
    <!--data目录下 Context.getFilesDir() .表示共享所有的目录，也可以指定共享的目录-->
    <files-path
        name="files_path"
        path="."/>
    <!--data缓存目录 Context.getCacheDir() .表示共享所有的目录，也可以指定共享的目录-->
    <cache-path
        name="cache-path"
        path="."/>
    <!--这个标签Android官方文档中是没有提及，Android设备的根目录，该目录下包含着手机内部存储器，外置SD卡等所有文件的目录-->
    <root-path
        name="name"
        path="."/>
</paths>
```

AndroidManifest.xml

```
<provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/filepaths" />
```

获取Uri

```
 val fileUri: Uri
fileUri = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
//解决调用相册不显示图片的问题
albumIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
    FileProvider.getUriForFile(
    this@PictureActivity,
    BuildConfig.APPLICATION_ID + ".fileprovider",
    file)
} else {
    Uri.fromFile(file)
}
```


##### 2019.07.16，（）编译问题

**Error: Program type already present: android.support.v4.app.FragmentTransitionCompat21$1**
 
 - 原因：没有导入v4包
 
 
##### 2019.06.24，（）异或加密算法

> [XOR 加密简介: http://www.ruanyifeng.com/blog/2017/05/xor.html](http://www.ruanyifeng.com/blog/2017/05/xor.html)

相同返回0， 不同返回1
```
1 ^ 1 // 0
0 ^ 0 // 0
1 ^ 0 // 1
0 ^ 1 // 1
```

XOR 运算有一个很奇妙的特点：如果对一个值连续做两次 XOR，会返回这个值本身。
```
// 第一次 XOR
1010 ^ 1111 // 0101

// 第二次 XOR
0101 ^ 1111 // 1010
```


##### 2019.06.04，（） adb connect 连接失败

> unable to connect 192.168.x.x

解决办法：
1. 下载termux 谷歌商店可以下
2. 执行下列命令
3. 重新连接

```
su 
setprop service.adb.tcp.port 5555 
stop adbd 
start adbd
```

>  [参考: adb connect 连接失败问题unable to connect to](https://blog.csdn.net/qq_24505485/article/details/52737523)

##### 2019.06.04，（） gradle 打包命令

```
./gradlew assembleRelease
```


##### 2019.05.16，（） 找不到Could not find manifest-merger.jar

将jcenter()和google()调换
因为在jcenter()找不到就不会到google()中找
> 参考: [解决Could not find manifest-merger.jar](https://www.jianshu.com/p/d719984b0ac8)


##### 2015.05.07，（） 配置打包

在app模块下的build.gradle中android闭包中添加
```
    signingConfigs {
        release {
            storeFile file("${rootProject.ext.signingStoreFile}")
            storePassword "${rootProject.ext.signingStorePassword}"
            keyAlias "${rootProject.ext.signingKeyAlias}"
            keyPassword "${rootProject.ext.signingKeyPassword}"
        }
    }
```

上面的值可以直接使用字符串,也可以在项目的build.gradle中一级闭包添加
```
ext{
    signingStoreFile = '../xiaoxuekey.jks'
    signingKeyAlias = 'fullmedia'
    signingStorePassword = '111111'
    signingKeyPassword = '111111'
}
```
配置打包apk的文件名, 在app模块下的build.gradle中的buildType中添加
```
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }

        applicationVariants.all {
            variant ->
                if (variant.buildType.name.equals('release')) {
                    variant.outputs.each { output ->
                        def outputFile = output.outputFile
                        if (outputFile != null && outputFile.name.endsWith('.apk')) {
                            def fileName = "${defaultConfig.applicationId}_(Release)_V${defaultConfig.versionName}_C${defaultConfig.versionCode}_(Build${releaseTime()}).apk"
                            output.outputFileName = new File(fileName)
                            //com.tcqmt.xiaoxue_(Release)_V1.0_C1_(Build201905070905).apk
                        }
                    }
                }
        }
    }
```

然后在一级目录创建方法
```
def releaseTime() {
    return new Date().format("yyyyMMddHHmm", TimeZone.getTimeZone("Asia/Chongqing"))
}
```

> [参考：Android studio 通过build.gradle 配置打包签名文件，生成 xxx.apk](https://blog.csdn.net/sinat_26710701/article/details/63262652)


> [参考：android studio 命令行打包](https://blog.csdn.net/li530893850/article/details/70889763)


##### 2019.05.05，（） 实现点击n次监听

> [参考: Android双击，连续点击5次](https://blog.csdn.net/iblade/article/details/72676229)


java实现
```
findViewById(R.id.id_test_btn).setOnClickListener(new View.OnClickListener() {
            final static int COUNTS = 5;//点击次数
            final static long DURATION = 3 * 1000;//规定有效时间
            long[] mHits = new long[COUNTS];

            @Override
            public void onClick(View v) {
                /**
                 * 实现双击方法
                 * src 拷贝的源数组
                 * srcPos 从源数组的那个位置开始拷贝.
                 * dst 目标数组
                 * dstPos 从目标数组的那个位子开始写数据
                 * length 拷贝的元素的个数
                 */
                System.arraycopy(mHits, 1, mHits, 0, mHits.length - 1);
                //实现左移，然后最后一个位置更新距离开机的时间，如果最后一个时间和最开始时间小于DURATION，即连续5次点击
                mHits[mHits.length - 1] = SystemClock.uptimeMillis();
                if (mHits[0] >= (SystemClock.uptimeMillis() - DURATION)) {
                    String tips = "您已在[" + DURATION + "]ms内连续点击【" + mHits.length + "】次了！！！";
                    Toast.makeText(TestActivity.this, tips, Toast.LENGTH_SHORT).show();
                }
            }
        });
```

kotlin实现
```
logo.setOnClickListener(object : View.OnClickListener {
            val COUNTS = 5//点击次数
            val DURATION = (3 * 1000).toLong()//规定有效时间
            var mHits = LongArray(COUNTS)

            override fun onClick(v: View) {
                /**
                 * src 拷贝的源数组
                 * srcPos 从源数组的那个位置开始拷贝.
                 * dst 目标数组
                 * dstPos 从目标数组的那个位子开始写数据
                 * length 拷贝的元素的个数
                 */
                System.arraycopy(mHits, 1, mHits, 0, mHits.size - 1)
                //然后最后一个位置更新距离首次点击的时间，如果最后一个时间和最开始时间小于DURATION，即连续5次点击
                mHits[mHits.size - 1] = SystemClock.uptimeMillis()
                if (mHits[0] >= SystemClock.uptimeMillis() - DURATION) {
                    //连续点击5次的逻辑
                    RobotServiceHelper.instance(this@HomeActivity).setData(getData())
                }
            }
        })
```


##### 2019.03.01，（）播放视频时无法调整播放位置seekTo(ms)无效

现象：重不正确的位置开始播放，或从头开始播放
原因：视频源没有关键帧

seekTo(ms)

seekTo 跳转的位置其实并不是参数所带的 position，而是离 position 最近的视频关键帧。  

```
关于视频关键帧建议大家可以去了解一下相关知识，大致上就是视频播放时需要从一个关键帧的位置开始。  
```

所以当视频在跳转到相应的 position 位置缺少关键帧的情况下，调用 seekTo 方法是无法在当前位置开始播放。这时会寻找离指定 position 最近的关键帧位置开始播放。  

> 参考：[关于Android VideoView seekTo不准确的解决方案:https://www.jianshu.com/p/f51b2febcfd2](https://www.jianshu.com/p/f51b2febcfd2)

8.0 解决方案  
seekTo(ms, mode)

> 参考: [https://blog.csdn.net/u012510322/article/details/79803433](https://blog.csdn.net/u012510322/article/details/79803433)

##### 2019.02.19，（）Java深拷贝和浅拷贝

实则浅拷贝和深拷贝只是相对的

 - 如果一个对象内部只有基本数据类型，那用 clone() 方法获取到的就是这个对象的深拷贝。
 - 如果其内部还有引用数据类型，那用 clone() 方法就是一次浅拷贝的操作。

> [参考：细说 Java 的深拷贝和浅拷贝 https://www.cnblogs.com/plokmju/p/7357205.html](https://www.cnblogs.com/plokmju/p/7357205.html)


##### 2019.02.19，（） ViewPager设置 缓存个数

```
mViewPager.setOffscreenPageLimit(2);//设置缓存view 的个数（实际有3个，缓存2个+正在显示的1个）
```
> 参考[ViewPager设置 缓存个数、页卡间距、数据更新](https://blog.csdn.net/jia4525036/article/details/18982197)

##### 2019.02.19，（） Kotlin 在 Android 开发中的 16 个建议

 - 1.延时加载：只在它被调用时才将资源加载进内存。
```
 val purchasingApi: PurchasingApi by lazy {
    val retrofit: Retrofit = Retrofit.Builder()
            .baseUrl(API_URL)
            .addConverterFactory(MoshiConverterFactory.create())
            .build()
    retrofit.create(PurchasingApi::class.java)
}
```
通过使用像这样的延迟加载，如果用户根本没有想要在应用中发生购买行为，你的应用将不会加载 PurchasingApi，因此不会消耗它可能会占用的资源。
延迟加载也是封装初始化逻辑的好方法：
```
// bounds is created as soon as the first call to bounds is made
val bounds: RectF by lazy { 
    RectF(0f, 0f, width.toFloat(), height.toFloat()) 
}
```
只有当 bounds 变量第一次被引用时，将会使用 view 的当前宽和高的值来创建 RectF，这样我们就不需要一开始显式的创建 RectF，然后把它设置给 bounds。

 - 2.

> [Kotlin 在 Android 开发中的 16 个建议：https://juejin.im/entry/5958907c6fb9a06b9e118ee5](https://juejin.im/entry/5958907c6fb9a06b9e118ee5)


##### 2019.02.19，（） android绘图canvas.clipRect()方法的作用

只显示裁剪的部分

> 参考[android绘图canvas.clipRect()方法的作用](https://blog.csdn.net/lovexieyuan520/article/details/50698320)


###### 2019.02.19，（）Canvas的save和restore

 - save：用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作。
 - restore：用来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。
 
> 参考[Canvas的save和restore](https://www.cnblogs.com/xirihanlin/archive/2009/07/24/1530246.html)

##### 2019.02.19，（） viewpager + fragment+FragmentStatePagerAdapter中用List存放多个Fragment 造成的内存泄漏

 - 原因：List里一直有Fragment的引用，Fragment无法回收造成内存泄漏
 - 解决办法：重写的PagerAdapter的getItem()方法中，return new yourFragment()

> 参考[viewpager + fragment+FragmentStatePagerAdapter中用List存放多个Fragment 造成的内存泄漏](https://blog.csdn.net/K_Hello/article/details/82996162)


##### 2019.02.18，（）在Android中记忆设计模式

 - 单例模式：获取系统服务
 - Builder模式：Dialog、Notification
 - 观察者模式：Adapter的notifyDataSetChanged()
 - 模板方法模式：Activity中的生命周期回调、AsyncTask的回调

##### 2019.01.18，（）通过Flag启动Activity

 - FLAG_ACTIVITY_CLEAR_TOP : 栈内复用，销毁Activity以上的活动
 
> 参考：[Activity的启动方式和flag详解: https://blog.csdn.net/singwhatiwanna/article/details/9294285](https://blog.csdn.net/singwhatiwanna/article/details/9294285)


##### 2019.01.18，（）VideoView适应宽高

 - 设置match_parent
 - 设置模式
 
```
VIDEO_SCALING_MODE_SCALE_TO_FIT：适应屏幕显示
VIDEO_SCALING_MODE_SCALE_TO_FIT_WITH_CROPPING：充满屏幕显示，保持比例，如果屏幕比例不对，则进行裁剪
```

如
```java
    override fun onPrepared(it: MediaPlayer?) {

        //适应屏幕显示
        it?.setVideoScalingMode(MediaPlayer.VIDEO_SCALING_MODE_SCALE_TO_FIT)

        //显示第一帧
        mVideoView?.seekTo(100)
        it?.setOnInfoListener { mp, what, extra ->
            if (what == MediaPlayer.MEDIA_INFO_VIDEO_RENDERING_START) {
                mVideoView?.setBackgroundColor(Color.TRANSPARENT)
            }
            return@setOnInfoListener true
        }
    }
```


##### 2019.01.18，（）使用迭代器遍历列表并删除其中元素

```
    Iterator<Student> iterator = list.iterator();
    while (iterator.hasNext()) {
        Student student = iterator.next();
        if ("male".equals(student.getGender()){
               iterator.remove();//使用迭代器的删除方法删除
            }
        }

```
 
 
##### 2019.01.18，（）判断视频

> https://github.com/kevinma2010/jfinal-api-scaffold/blob/master/src/main/java/com/mlongbo/jfinal/common/utils/FileUtils.java


> https://rainsilence.iteye.com/blog/842338


##### 2019.01.18，（）刷新媒体库


 - sendBroadcast(new Intent(Intent.ACTION_MEDIA_MOUNTED, dirUri));  
 - sendBroadcast(new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE,fileUri)); //刷新单个文件

> 刷新sd卡在23以上报错， 刷新单个文件可以 
 
> https://blog.csdn.net/GH_HOME/article/details/53585760
 
 
##### 2018.12.11，（）Android~获取view在屏幕中的位置

 - getLocalVisibleRect ， 返回一个填充的Rect对象， 感觉是这个View的Rect大小，left，top取到的都是0

 - getGlobalVisibleRect ， 获取全局坐标系的一个视图区域， 返回一个填充的Rect对象；该Rect是基于总整个屏幕的

 - getLocationOnScreen ，计算该视图在全局坐标系中的x，y值，（注意这个值是要从屏幕顶端算起，也就是索包括了通知栏的高度）//获取在当前屏幕内的绝对坐标 

 - getLocationInWindow ，计算该视图在它所在的widnow的坐标x，y值，//获取在整个窗口内的绝对坐标 (不是很理解= =、)

 - getLeft , getTop, getBottom, getRight,  这一组是获取相对在它父亲里的坐标


> 注意：如果在Activity的OnCreate()事件输出那些参数，是全为0，要等UI控件都加载完了才能获取到这些

example:

```
    int[] location = new int[2];
    v.getLocationOnScreen(location);
    int x = location[0];
    int y = location[1];

```

> 参考：[](> https://blog.csdn.net/centralperk/article/details/7949900) https://blog.csdn.net/centralperk/article/details/7949900


##### 2018.12.08，（） ScrollView嵌套子view的滑动冲突处理
 - 场景：使用scrollerview作为外部，内布局时ViewGroup(如RelativeLayout), ViewGoup有个子View(如ImageView), 子View设置了onTouchListener, 触摸子View时，onTouch方法会先调用，返回true，但是，事件会传递到ScrollerView的onTouchEvent，然后子View就不能继续滑动，其实子View先滑动了一丢丢距离，然后才停止滑动.  
 - 解决办法：
在子View的onTouch方法中添加
```java
//屏蔽父控件拦截onTouch事件
getParent().requestDisallowInterceptTouchEvent(true);
```

> 参考：https://thoreau.iteye.com/blog/2002272

##### 2018.12.08，（） LinearLayout子View层叠覆盖效果

设置“layout_marginTop”属性
```
android:layout_marginTop="-30dp"
```

> 参考：https://blog.csdn.net/ming_csdn_/article/details/54290244


##### 2018.12.06，（） 父view超出子view
```
android:clipChildren="false"
android:clipToPadding="false"
```
> 参考：[https://www.jianshu.com/p/e5708a23fb90](https://www.jianshu.com/p/e5708a23fb90)

##### 2018.12.06，（） Android 防止同时按下两个按钮触发两个事件，连续点击事件

 在这两个按钮 或其他控件 的父控件上加上
 ```
 android:splitMotionEvents="false"
 ```
>  参考：[https://blog.csdn.net/EvloutionPLUS/article/details/80541022](https://blog.csdn.net/EvloutionPLUS/article/details/80541022)

##### 2018.12.06，（）Android中判断当前API的版本号

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {...}
```

> 参考：[https://blog.csdn.net/wangsf1112/article/details/51545101](https://blog.csdn.net/wangsf1112/article/details/51545101)


##### 2018.11.19，（）给Grid方式排列的RecyclerView添加间距

```
        mRecyclerView.addItemDecoration(object : RecyclerView.ItemDecoration() {
            override fun getItemOffsets(
                outRect: Rect, view: View,
                parent: RecyclerView, state: RecyclerView.State
            ) {
                val pos = parent.getChildLayoutPosition(view)
                if (pos / MAX_COL == 0) {
                    //设置第一行
                    outRect.top = firstTopSpacing
                }

                outRect.bottom = bottomSpacing
            }
        })
```

> 参考：[给Grid方式排列的RecyclerView添加间距](https://blog.csdn.net/laxian2009/article/details/72824594)


##### 2018.11.19，（）Android Studio怎么把查看代码的左箭头、右箭头图标加到右边的快捷工具栏
[Android Studio怎么把查看代码的左箭头、右箭头图标加到右边的快捷工具栏](https://blog.csdn.net/dearlaoyuan/article/details/80158013)


##### 2018.11.16，（） viewPager异常

> java.lang.IllegalStateException: Can't change tag of fragment ShowImageFragment{d178bfd id=0x7f0f0104 android:switcher:2131689732:0}: was android:switcher:2131689732:0 now android:switcher:2131689732:1

 - 原因：List中会有可能出现存在两个指针指向相同的Fragment

> 参考：[https://blog.csdn.net/github_36287370/article/details/52717256](https://blog.csdn.net/github_36287370/article/details/52717256)


##### 2018.10.09，（）Fragment: getContext() NoSuchMethodError

Fragment getContext() 在Android低版本（API<=22）中出现NoSuchMethodError，Log日志

```
Exception java.lang.NoSuchMethodError: No virtual method getContext()Landroid/content/Context; in class Lcom/android/browser/preferences/MainPreferenceFragment; or its super classes (declaration of 'com.android.browser.preferences.MainPreferenceFragment' appears in /system/priv-app/Browser/Browser.apk)

```

Fragment中的getContext()方法是在Android6.0中才出现的，在低于该版本中使用会报NoSuchMethodError，可以使用getActivity()来替换。

>  参考：[Fragment getContext() NoSuchMethodError](https://blog.csdn.net/lwq573384928/article/details/79697340)



##### 2018.10.08，（） NDK 版本问题

r17 不支持armeabi

报错
```
The armeabi ABI should have exactly one architecture definitions.
```

> [参考：ndk项目编译异常---The armeabi ABI should have exactly one architecture definitions.](https://www.jianshu.com/p/dd34cb501c81)

##### 2018.10.08，（） Volley不支持大文件上传和下载
```
原因：Volley将整个response加载到内存并进行操作（可以是解析等操作）大文件可能会引起OOM
```
> [参考：Volley简介](https://juejin.im/entry/58b650de2f301e006c473a26)

##### 2018.10.08，（）Retrofit拦截日志
```
implementation 'com.squareup.okhttp3:logging-interceptor:3.10.0'
```

```kotlin
val httpClientBuilder = OkHttpClient.Builder()
if (BuildConfig.DEBUG){
    val logging = HttpLoggingInterceptor()
    //设置日志Level
    logging.level = HttpLoggingInterceptor.Level.BODY
    //添加拦截器到OkHttp，这是最关键的
    httpClientBuilder.addInterceptor(logging)
}

val retrofit = Retrofit.Builder()
    .baseUrl("https://raw.githubusercontent.com/devallever/DataProject/")
    .client(httpClientBuilder.build())
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

> [参考：Retrofit 之日志拦截](https://juejin.im/entry/58d49ae71b69e6006ba474d5)

##### 2018.10.08，（） Retrofit封装
> [参考: Retrofit--合理封装回调能让你的项目高逼格](https://blog.csdn.net/lyhhj/article/details/51720296)


##### 2018.09.29，（） 模拟点击
```
view.performClick()
```

> [参考:Android中performClick方法---代码调用点击事件（模拟去触摸控件）](https://blog.csdn.net/lplj717/article/details/76177429)

##### 2018.09.29，（）DialogFragment的返回键处理

```java
dialog.setOnKeyListener(new DialogInterface.OnKeyListener() { 
                @Override 
                public boolean onKey(DialogInterface anInterface, int keyCode, KeyEvent event) { 
                    if (keyCode == KeyEvent.KEYCODE_BACK && event.getAction() == KeyEvent.ACTION_DOWN) { 
               ／／自己业务处理，用true拦截 
                        return true; 
                    } 
                    return false; 
                } 
            }); 

```
> [参考：DialogFragment的返回键处理](https://blog.csdn.net/daoxiaomianzi/article/details/66973189)

##### 2018.09.29，（） 改变Dialog 背景透明度

```xml
<style name="CustomDialog" parent="@android:style/Theme.Dialog">
    <item name="android:backgroundDimAmount">0.2</item> 
</style>

```

```
//设置背景透明度，0~1.0  默认0.5
dialog.getWindow().setDimAmount(0.1f);

```

> [参考：android- 改变Dialog 背景透明度](https://blog.csdn.net/u013372185/article/details/45645665)


##### 2018.09.27，（）  修改纯色图标颜色

```
xml
android:tint="#7f7f7f"
代码
setColorFilter(R.color.black)
```

##### 2018.09.20，（）  需要运行时申请的权限

```
group:android.permission-group.CONTACTS
  permission:android.permission.WRITE_CONTACTS
  permission:android.permission.GET_ACCOUNTS
  permission:android.permission.READ_CONTACTS

group:android.permission-group.PHONE
  permission:android.permission.READ_CALL_LOG
  permission:android.permission.READ_PHONE_STATE
  permission:android.permission.CALL_PHONE
  permission:android.permission.WRITE_CALL_LOG
  permission:android.permission.USE_SIP
  permission:android.permission.PROCESS_OUTGOING_CALLS
  permission:com.android.voicemail.permission.ADD_VOICEMAIL

group:android.permission-group.CALENDAR
  permission:android.permission.READ_CALENDAR
  permission:android.permission.WRITE_CALENDAR

group:android.permission-group.CAMERA
  permission:android.permission.CAMERA

group:android.permission-group.SENSORS
  permission:android.permission.BODY_SENSORS

group:android.permission-group.LOCATION
  permission:android.permission.ACCESS_FINE_LOCATION
  permission:android.permission.ACCESS_COARSE_LOCATION

group:android.permission-group.STORAGE
  permission:android.permission.READ_EXTERNAL_STORAGE
  permission:android.permission.WRITE_EXTERNAL_STORAGE

group:android.permission-group.MICROPHONE
  permission:android.permission.RECORD_AUDIO

group:android.permission-group.SMS
  permission:android.permission.READ_SMS
  permission:android.permission.RECEIVE_WAP_PUSH

```


##### 2018.09.18，（）List数值排序

```
Collections.sort(list);
```
> [list集合的排序方法：](https://blog.csdn.net/tiankongcheng6/article/details/54744994)

##### 2018.09.18，（）Calendar中的小时

```
Calendar.HOUR 12小时制
Calendar.HOUR_OF_DAY 24小时制
```

##### 2018.09.18，（）TextView 文本过长省略号

```
在xml中：
android:ellipsize="end"　　   省略号在结尾
android:ellipsize="start" 　　省略号在开头
android:ellipsize="middle"   省略号在中间
android:ellipsize="marquee"  跑马灯
最好加一个TextView显示行数的约束，例如：
android:singleline="true"或者android:lines="2"

在java文件中：
tv.setEllipsize(TextUtils.TruncateAt.valueOf("END"));
tv.setEllipsize(TextUtils.TruncateAt.valueOf("START"));
tv.setEllipsize(TextUtils.TruncateAt.valueOf("MIDDLE"));
tv.setEllipsize(TextUtils.TruncateAt.valueOf("MARQUEE"));
```
> [参考：Android TextView内容过长加省略号，点击显示全部内容](https://blog.csdn.net/wwzqj/article/details/8731859)

##### 2018.09.18，（）跳转飞行模式设置界面和其他界面

```
ACTION_AIRPLANE_MODE_SETTINGS： // 飞行模式，无线网和网络设置界面

Intent intent =  new Intent(Settings.ACTION_AIRPLANE_MODE_SETTINGS);
startActivity(intent);

```

> [参考：Android通过代码跳转到系统设置的相关界面](https://www.jianshu.com/p/57c034f38bcb)


##### 2018.09.15，（）自定义radio button样式
```

<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:drawable="@color/green_16" android:state_checked="true"></item>
    <item android:drawable="@color/grey_b3" android:state_checked="false"></item>

</selector>
```

```
        <RadioGroup
            android:id="@+id/id_dialog_h_w_rg_weight_unit_container"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:layout_alignParentBottom="true"
            android:orientation="horizontal">

            <RadioButton
                android:id="@+id/id_dialog_h_w_rb_unit_kg"
                style="@style/action_finish_rb_unit"
                android:checked="true"
                android:text="@string/unit_kg"/>

            <RadioButton
                android:id="@+id/id_dialog_h_w_rb_unit_lb"
                style="@style/action_finish_rb_unit"
                android:checked="false"
                android:text="@string/unit_lb"/>

        </RadioGroup>
```

```
    <style name="action_finish_rb_unit">
        <item name="android:layout_width">30dp</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:gravity">center</item>
        <item name="android:background">@drawable/rb_finish_unit_selector</item>
        <item name="android:button">@null</item>
        <item name="android:layout_margin">5dp</item>
        <item name="android:textColor">@color/white</item>
        <item name="android:padding">5dp</item>
    </style>
```


##### 2018.09.11，（）EditText取消下划线
```xml
android:background="@null"
```

##### 2018.09.11，（）底部弹窗DialogFragment

 - 继承DialogFragment

```
package com.allever.kotlinweather.alarm

import android.app.Dialog
import android.graphics.Color
import android.graphics.drawable.ColorDrawable
import android.os.Bundle
import android.view.Gravity
import android.view.LayoutInflater
import android.view.View
import android.view.WindowManager
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatDialogFragment
import androidx.fragment.app.FragmentManager
import com.allever.kotlinweather.R

class SceneEditFragment : AppCompatDialogFragment() {

    var mView: View? = null

    override fun onStart() {
        super.onStart()

        val window = dialog.window
        if (window != null) {
            // 必须设置，否则无法全屏
            window.setBackgroundDrawable(ColorDrawable(Color.TRANSPARENT))
            //设置dialog在屏幕底部
            window.setGravity(Gravity.BOTTOM)
            //设置dialog弹出时的动画效果，从屏幕底部向上弹出
            window.setWindowAnimations(R.style.DialogStyle)
            //获得window窗口的属性
            val lp = window.attributes
            //设置窗口宽度为充满全屏
            lp.width = WindowManager.LayoutParams.MATCH_PARENT
            //设置窗口高度为包裹内容
            lp.height = WindowManager.LayoutParams.WRAP_CONTENT
            //将设置好的属性set回去
            window.attributes = lp
        }
    }

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        mView = LayoutInflater.from(activity).inflate(R.layout.fragment_sence_edit,  null)
        return AlertDialog.Builder(activity!!)
                .setView(mView)
                .create()
    }

    override fun show(fragmentManager: FragmentManager?, tag: String?) {
        try {
            fragmentManager?.beginTransaction()?.remove(this)?.commit()
            super.show(fragmentManager, tag)
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```
 - Style

```xml
    <!-- 弹出框动画 由下至上 -->
    <style name="DialogStyle" parent="@android:style/Animation.Dialog">
        <item name="@android:windowEnterAnimation">@anim/dialog_enter</item>
        <!-- 进入时的动画 -->
        <item name="@android:windowExitAnimation">@anim/dialog_exit</item>
        <!-- 退出时的动画 -->
    </style>
```

 - anim
 
```
//dialog_enter
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="100"
        android:fromYDelta="100%" />
</set>

//dialog_exit
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="100"
        android:toYDelta="100%" />
</set>


```

##### 2018.09.11，（）NestedScrollerView嵌套RecyclerView卡顿

```java
LinearLayoutManager layoutManager = new LinearLayoutManager(this);
//layoutManager.setSmoothScrollbarEnabled(true);
//layoutManager.setAutoMeasureEnabled(true);

recyclerView.setLayoutManager(layoutManager);
//recyclerView.setHasFixedSize(true);
recyclerView.setNestedScrollingEnabled(false);`
```


##### 2018.09.10，（）RecyclerView实现的格子布局，单个item夸列

```java
RecyclerView recyclerView = new RecyclerView(this);
GridLayoutManager gridLayoutManager = new GridLayoutManager(this, 2);
recyclerView.setLayoutManager(gridLayoutManager);
gridLayoutManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
    @Override
    public int getSpanSize(int position) {
        if (position == 0){
            //第一列占据两个位置
            return 2;
        }else {
            return 1;
        }
    }
});
```
> 参考：[recyclerView中GridLayoutManager实现每一行不同布局的问题](https://blog.csdn.net/yearningseeker/article/details/79709802)

##### 2018.09.10，（） 默认单选列表对话框

```
final String[] items = {"Mon, Tue, Web, Thur, Fri, Sat, Sun"};
AlertDialog.Builder builder = new AlertDialog.Builder(this);
builder.setSingleChoiceItems(items, android.R.layout.simple_list_item_1, new DialogInterface.OnClickListener() {
    @Override
    public void onClick(DialogInterface dialogInterface, int position) {
        Log.d(TAG, "onClick: " + items[position]);
    }
});
builder.create().show();
```
> [参考：Android中的普通对话框、单选对话框、多选对话框、带Icon的对话框、以及自定义Adapter和自定义View对话框详解](https://blog.csdn.net/u012702547/article/details/50676606?tdsourcetag=s_pctim_aiomsg)

##### 2018.09.10，（）修改View的背景色

```java
RelativeLayout relativeLayout = new RelativeLayout(this);
GradientDrawable drawable = (GradientDrawable) recyclerView.getBackground();
//这样不会修改背景drawable(列如使用圆角的drawable)
drawable.setColor(getResources().getColor(R.color.colorAccent));
```

##### 2018.08.29，（）PermissionsUtil

 - PermissionsUtil.java
```
import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.net.Uri;
import android.os.Build;
import android.provider.Settings;
import android.support.annotation.NonNull;
import android.support.v4.app.Fragment;
import android.support.v4.content.PermissionChecker;
import android.util.Log;

import java.util.Arrays;
import java.util.List;

/**
 * Created by dfqin on 2017/1/20.
 */

public final class PermissionsUtil {

    public static final String TAG = PermissionsUtil.class.getSimpleName();

    private static PermissionListener sListener;

    private PermissionsUtil(){}

    /**
     * 申请授权，当用户拒绝时，会显示默认一个默认的Dialog提示用户
     * @param context
     * @param listener
     * @param permission 要申请的权限
     */
    public static void requestPermission(@NonNull Context context, PermissionListener listener, String... permission) {
        sListener = listener;

        if (listener == null) {
            Log.e(TAG, "listener is null");
            return;
        }

        if (PermissionsUtil.hasPermission(context, permission)) {
            listener.permissionGranted(permission);
        } else {
            if (Build.VERSION.SDK_INT < 23) {
                listener.permissionDenied(permission);
            } else {
                String key = String.valueOf(System.currentTimeMillis());
                Intent intent = new Intent(context, PermissionActivity.class);
                intent.putExtra("permission", permission);
                intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                context.startActivity(intent);
            }
        }
    }

    /**
     *  申请授权，当用户拒绝时，可以设置是否显示Dialog提示用户，也可以设置提示用户的文本内容
     * @param context
     * @param listener
     * @param permission 需要申请授权的权限
     */


    /**
     * 判断权限是否授权
     * @param context
     * @param permissions
     * @return
     */
    public static boolean hasPermission(@NonNull Context context, @NonNull String... permissions) {

        if (permissions.length == 0) {
            return false;
        }

        for (String per : permissions ) {
            int result = PermissionChecker.checkSelfPermission(context, per);
            if ( result != PermissionChecker.PERMISSION_GRANTED) {
                return false;
            }
        }

        return true;
    }

    /**
     * 判断一组授权结果是否为授权通过
     * @param grantResult
     * @return
     */
    public static boolean isGranted(@NonNull int... grantResult) {

        if (grantResult.length == 0) {
            return false;
        }

        for (int result : grantResult) {
            if (result != PackageManager.PERMISSION_GRANTED) {
                return false;
            }
        }
        return true;
    }

    /**
     * 跳转到当前应用对应的设置页面
     * @param
     */
    public static void gotoSetting(@NonNull Activity activity, int requestCode) {
        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
        intent.setData(Uri.parse("package:" + activity.getPackageName()));
        activity.startActivityForResult(intent, requestCode);
    }

    public static void gotoSetting(@NonNull Fragment fragment, int requestCode) {
        if (fragment.getActivity() != null){
            gotoSetting(fragment.getActivity(), requestCode);
        }
    }

    public static boolean hasAlwaysDeniedPermission(@NonNull Activity activity, String... deniedPermissions) {
        return hasAlwaysDeniedPermission(activity, Arrays.asList(deniedPermissions));
    }

    public static boolean hasAlwaysDeniedPermission(@NonNull Activity activity, List<String> deniedPermissions) {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M){
            return false;
        }

        if (deniedPermissions.size() == 0) {
            return false;
        }

        for (String permission : deniedPermissions) {
            boolean rationale = activity.shouldShowRequestPermissionRationale(permission);
            if (!rationale) {
                return true;
            }
        }
        return false;
    }

    public static boolean hasAlwaysDeniedPermission(@NonNull Activity activity, String deniedPermissions) {
        return hasAlwaysDeniedPermission(activity, deniedPermissions);
    }

    public static PermissionListener getListener(){
        return sListener;
    }

    public static void removeListener(){
        sListener = null;
    }
}
```

 - PermissionListener
```
import android.support.annotation.NonNull;

/**
 * Created by dfqin on 2017/1/20.
 */

public interface PermissionListener {

    /**
     * 通过授权
     * @param permission
     */
    void permissionGranted(@NonNull String[] permission);

    /**
     * 拒绝授权
     * @param permission
     */
    void permissionDenied(@NonNull String[] permission);

    void rationableCancel();
}
```

 - PermissionActivity
 
```
/**
 * Created by dfqin on 2017/1/22.
 */

import android.content.DialogInterface;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.support.v4.app.ActivityCompat;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;

public class PermissionActivity extends AppCompatActivity {


    private static final int PERMISSION_REQUEST_CODE = 64;

    private String[] permission;

    private final String defaultTitle = "Help";
    private final String defaultContent = "This App require some important permission.\n \n Click \"Setting\"-\"Permission\"-Open Require Permission。";
    private final String defaultCancel = "Cancel";
    private final String defaultEnsure = "Go";

    private Handler mHandler;

    @Override protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getIntent() == null || !getIntent().hasExtra("permission")) {
            finish();
            return;
        }

        mHandler = new Handler();

        permission = getIntent().getStringArrayExtra("permission");

        if (PermissionsUtil.hasPermission(this, permission)) {
            permissionsGranted();
        } else {
            showRationableDialog();
        }

    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (PermissionsUtil.hasPermission(this, permission)) {
            permissionsGranted();
        } else {
            permissionsDenied();
        }
    }

    // 请求权限兼容低版本
    private void requestPermissions(String[] permission) {
        ActivityCompat.requestPermissions(this, permission, PERMISSION_REQUEST_CODE);
    }


    /**
     * 用户权限处理,
     * 如果全部获取, 则直接过.
     * 如果权限缺失, 则提示Dialog.
     *
     * @param requestCode  请求码
     * @param permissions  权限
     * @param grantResults 结果
     */
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {

        //部分厂商手机系统返回授权成功时，厂商可以拒绝权限，所以要用PermissionChecker二次判断
        if (requestCode == PERMISSION_REQUEST_CODE && PermissionsUtil.isGranted(grantResults)
                && PermissionsUtil.hasPermission(this, permissions)) {
            permissionsGranted();
        } else if (PermissionsUtil.hasAlwaysDeniedPermission(this, permissions)){
            showMissingPermissionDialog();
        } else { //不需要提示用户
            permissionsDenied();
        }
    }

    // 显示缺失权限提示
    private void showMissingPermissionDialog() {

        AlertDialog.Builder builder = new AlertDialog.Builder(PermissionActivity.this);

        builder.setTitle(defaultTitle);
        builder.setMessage(defaultContent);

        builder.setNegativeButton(defaultCancel, new DialogInterface.OnClickListener(){
            @Override public void onClick(DialogInterface dialog, int which) {
                permissionsDenied();
            }
        });

        builder.setPositiveButton(defaultEnsure , new DialogInterface.OnClickListener() {
            @Override public void onClick(DialogInterface dialog, int which) {
                PermissionsUtil.gotoSetting(PermissionActivity.this, 0);
            }
        });

        builder.setCancelable(false);
        builder.show();
    }

    // 显示缺失权限提示
    private void showRationableDialog() {

        AlertDialog.Builder builder = new AlertDialog.Builder(PermissionActivity.this);

        builder.setTitle("Title");
        builder.setMessage("We need this permission to...");

        builder.setNegativeButton("Not agree", new DialogInterface.OnClickListener(){
            @Override public void onClick(DialogInterface dialog, int which) {
                //permissionsDenied();
                final PermissionListener listener = PermissionsUtil.getListener();
                if (listener != null) {
                    dialog.cancel();
                    mHandler.postDelayed(new Runnable() {
                        @Override
                        public void run() {
                            listener.rationableCancel();
                        }
                    }, 300);

                    finish();
                }
            }
        });

        builder.setPositiveButton("Agree", new DialogInterface.OnClickListener() {
            @Override public void onClick(DialogInterface dialog, int which) {
                requestPermissions(permission);
            }
        });

        builder.setCancelable(false);
        builder.show();
    }

    private void permissionsDenied() {
        PermissionListener listener = PermissionsUtil.getListener();
        if (listener != null) {
            listener.permissionDenied(permission);
        }
        finish();
    }

    // 全部权限均已获取
    private void permissionsGranted() {
        PermissionListener listener = PermissionsUtil.getListener();
        if (listener != null) {
            listener.permissionGranted(permission);
        }
        finish();
    }

    protected void onDestroy() {
        PermissionsUtil.removeListener();
        super.onDestroy();
    }

}


```

 - theme
 
```

    <style name="GrantorNoDisplay" parent="Theme.AppCompat.Light.NoActionBar"> >
        <item name="android:windowBackground">@android:color/transparent</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:colorBackgroundCacheHint">@null</item>
        <item name="android:windowAnimationStyle">@android:style/Animation</item>
        <item name="android:windowContentOverlay">@null</item>
        <item name="android:windowTranslucentStatus" tools:targetApi="kitkat">true</item>
    </style>
```


 - Usage 
 
```
        findViewById<View>(R.id.id_btn_camera).setOnClickListener {
            val hasPerms = PermissionsUtil.hasPermission(this@MainActivity, Manifest.permission.CAMERA)
            if (hasPerms) {
                Toast.makeText(this@MainActivity, "has permission", Toast.LENGTH_SHORT).show()
            } else {
                PermissionsUtil.requestPermission(this@MainActivity, object : PermissionListener {
                    override fun permissionGranted(permission: Array<String>) {

                    }

                    override fun permissionDenied(permission: Array<String>) {

                    }

                    override fun rationableCancel() {
                        Log.d("MainActi", "cancelRationable")
                        finish()
                    }
                }, Manifest.permission.CAMERA, Manifest.permission.READ_CONTACTS)
            }
        }
```


##### 2018.08.25，（）EditText相关
###### 修改光标颜色
 - android:textCursorDrawable=”@null” 表示光标的颜色和字体的颜色一样
 - 通过xml修改，

 -  新建xml的drawable，如下内容

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle" >
    <size android:width="2dp" />
    <solid android:color="#ff7200" />
</shape>
```
 -  在EditText中使用

```
android:textCursorDrawable=”@drawable/edit_cursor_color”
```

> [参考：Android 修改EditText的光标颜色和背景色](https://blog.csdn.net/u012301841/article/details/51151745)

###### 修改下滑线颜色

 - 在style中新建这样的Style

```xml
<style name="EditTextUnderLineStyle" parent="Theme.AppCompat.Light">
    <item name="colorControlNormal">@color/message_list_line</item>
    <item name="colorControlActivated">@color/message_list_line</item>
</style>
```

 - EditText控件中使用theme属性

```
android:theme="@style/EditTextUnderLineStyle"
```

> [参考：EditText修改光标和下划线颜色](https://blog.csdn.net/qq_24252589/article/details/78133011)


> [参考：Android更改EditText下划线颜色样式的方法](https://www.jb51.net/article/101967.htm)

###### InputType属性

 - 常用
 

类型 | 值
---|---
text     | TYPE_CLASS_TEXT
number   | TYPE_CLASS_NUMBER



> [参考：TextView | Android Developers – android:inputType](http://developer.android.com/reference/android/widget/TextView.html#attr_android:inputType)

> [参考：Android中EditText（或TextView）中的InputType类型含义与如何定义](https://blog.csdn.net/ysh06201418/article/details/71123273)


###### EditText + ListPopWindow 实现带弹出列表的EditText

 - 思路：设置editText的onTouchListener，判断触摸位置，如果是右边的话就弹出ListPopWindow，如果是其他位置，则跟正常一样，即输入

 - 创建ListPopWindow对象，并设置相关属性

```java
ListPopupWindow lpw = new ListPopupWindow(this);
lpw.setAdapter(new ArrayAdapter<String>(this,android.R.layout.simple_list_item_1, list));
lpw.setAnchorView(etTest);
lpw.setModal(true);
lpw.setOnItemClickListener(this);

```
 - 设置EditText的onTouchListener, 当点击右边区域时弹出ListPopupWindow

```java
editText.setOnTouchListener(new OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        final int DRAWABLE_LEFT = 0;
        final int DRAWABLE_TOP = 1;
        final int DRAWABLE_RIGHT = 2;
        final int DRAWABLE_BOTTOM = 3;

        // Check if touch point is in the area of the right button
        if(event.getAction() == MotionEvent.ACTION_UP) {
            if(event.getX() >= (etTest.getWidth() - etTest
                .getCompoundDrawables()[DRAWABLE_RIGHT].getBounds().width())) {
                // your action here
                lpw.show();
                return true;
            }
        }
        return false;
    }
});

```

 - 监听ListPopupWindow选项,选中后将选项中的数值显示到EditText

```
@Override
public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
    String item = list[position];
    etTest.setText(item);
    lpw.dismiss();
}
```

> [参考：带弹出列表的EditText](https://www.jianshu.com/p/031fc60a66d0)

##### 2018.08.25，（）自定义控件之组合控件

 - 创建布局

```xml
<LinearLayout >
    <EditText .../>
    <TextView .../>
</LinearLayout>
```

 - 创建自定义属性,在attrs中添加下面代码
 - 添加flag标签后，当使用属性是会自动弹出, 获取属性值是做相应处理


```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="UnderTipEditText">
        <attr name="text" format="reference|string" />
        <attr name="inputType" format="integer" >
            <flag name="text" value=0/>
            <flag name="number" value=1/>
        </attr>
        <attr name="maxLength" format="integer" />
    </declare-styleable>
</resources>

```

 - 创建控件类，继承父布局控件

```
class UnderTipEditText extends LinearLayout{

    private static final int INPUT_TYPE_TEXT = 0;
    private static finel int INPUT_TYPE_NUMBER = 1;

    private Context mContext;
    
    public UnderTipEditText(Context context){
        this(context, null);
    }
    public UnderTipEditText(Context context, AttributeSet attrs){
        this(context, attrs, 0);
    }
    public UnderTipEditText(Context context, AttributeSet sattrset, int defStyle){
        super(context, attrs, defStyle);
        mContext = context;
        
        //自定义属性
        TypedArray typedArray = context.obtainStyledAttributes(attrs,R.styleable.UnderTipEditText);
        mStrContent = typedArray.getString(R.styleable.UnderTipEditText_text);
        mInputType = typedArray.getInt(R.styleable.UnderTipEditText_inputType, 0);
        mMaxLength =  typedArray.getInt(R.styleable.UnderTipEditText_maxLength, 20);
        
        //获取资源后要及时回收
        typedArray.recycle();
        
        initView();
    }
    
    private voie initView(){
        View view  = LayoutInflater.from(mContext).inflate(R.layout.under_tip_edit_text, this, ture);
        mEtContext = view.findViewById();
        mTvTip = view.findViewById();
        
        mTvTip.setText(mEtContent.getText().length() + "/" + mMaxLength);
        mEtContent.setText(mStrContent);
        siwtch(inputType){
            case INPUT_TYPE_TEXT:
                mEtContent.setInputType(InputType.XXX_TEXT);
                break;
            case INPUT_TYPE_TEXT:
                mEtContent.setInputType(InputType.XXX_NUMBER);
                break;
        }
        
        //省略其他逻辑
        //...
    }
}
```


##### 2018.08.25，（）WifiManager
```
WifiManager wifi = (WifiManager) getSystemService(Context.WIFI_SERVICE);
WifiInfo info = wifi.getConnectionInfo();
viewDelegate.setTextViewText(R.id.wifi_name_tv, info.getSSID().replace("\"", ""));
```


##### 2018.08.21，（）Android Studio设置shadowsocks代理
> [参考：Android Studio设置shadowsocks代理](https://blog.csdn.net/u013495603/article/details/50970067)

配置好shadowsocks后，不用开启全局代理服务，就这样挂着，然后，switchOmiga设置proxy，
访问外网时提示，有资源没能加载，添加限制后就可以了

##### 2018.08.21，（）ViewStub用法
两个xml布局文件, 根布局设置了id属性

```xml
<!-- stub_view_layout_1.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/id_stub_view_1">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Stub View 1"/>

</LinearLayout>


<!-- stub_view_layout_2.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/id_stub_view_2">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Stub View 2"/>

</LinearLayout>
```

使用ViewStub

 -  id属性指定viewstub控件的引用
 - inflatedId属性指定 包含的布局的根元素的id
 - layou属性指定:待加载的布局


```xml
    <ViewStub
        android:id="@+id/stub_test_view_1"
        android:inflatedId="@id/id_stub_view_1"
        android:layout="@layout/stub_view_layout_1"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <ViewStub
        android:id="@+id/stub_test_view_2"
        android:inflatedId="@id/id_stub_view_2"
        android:layout="@layout/stub_view_layout_2"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

代码中加载布局

```java
ViewStub viewStub1 = findViewById(R.id.stub_test_view_1);
viewStub1.inflate();

//或者

ViewStub viewStub2 = findViewById(R.id.stub_test_view_2);
viewStub2.setVisibility(View.VISIBLE);

```

##### 2018.08.21，（）Spinner使用

根ListView差不多,可以使用系统内部布局,可以自定义Item布局  
可以用ArrayAdapter,或者自定义Adapter,根ListView的adapter一样

xml中使用Spinner

```xml
    <Spinner
        android:id="@+id/id_spinner_weekday"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
```

代码中

```java
        mSpinnerWeeday = findViewById(R.id.id_spinner_weekday);

        mySpinnerAdapter = new MySpinnerAdapter(this, mSpinnerItemList);
        //mArrayAdapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1,mArrWeekday);

        //mSpinnerWeeday.setAdapter(mArrayAdapter);
        mSpinnerWeeday.setAdapter(mySpinnerAdapter);
        
        mSpinnerWeeday.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> adapterView, View view, int i, long l) {
                Toast.makeText(SpannerTestActivity.this, mArrWeekday[i], Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onNothingSelected(AdapterView<?> adapterView) {

            }
        });
```


##### 2018.06.12，（）DialogFragment调用show();引发的bug

java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState

```
private void checkStateLoss() {
        if (mStateSaved) {
            throw new IllegalStateException(
                    "Can not perform this action after onSaveInstanceState");
        }
        if (mNoTransactionsBecause != null) {
            throw new IllegalStateException(
                    "Can not perform this action inside of " + mNoTransactionsBecause);
        }
    }
```

只有在saveAllState()的时候，才会将它置为true。

```
Parcelable saveAllState() {
        // ...
        if (HONEYCOMB) {
            // As of Honeycomb, we save state after pausing.  Prior to that
            // it is before pausing.  With fragments this is an issue, since
            // there are many things you may do after pausing but before
            // stopping that change the fragment state.  For those older
            // devices, we will not at this point say that we have saved
            // the state, so we will allow them to continue doing fragment
            // transactions.  This retains the same semantics as Honeycomb,
            // though you do have the risk of losing the very most recent state
            // if the process is killed...  we'll live with that.
            mStateSaved = true;
        }
        // ...
    }
```
在FragmentActivity的onSaveInstanceState()的时候，会去调用FragmentManager.saveAllState()方法去保存当前所有的Fragment的状态，以便下次进行恢复。而在被销毁到恢复期间的时候，去做有关Fragment状态的操作，就会引起IllegalStateException。
```
/**
     * Save all appropriate fragment state.
     */
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        Parcelable p = mFragments.saveAllState();
        if (p != null) {
            outState.putParcelable(FRAGMENTS_TAG, p);
        }
    }

```


```
解决方案
既然已经找到了问题的症结。那么就有办法解决了。

try.catch住
已经很明朗是因为mStateSaved出现的错误，那么在继承DialogFragment之后，从写show()方法，然后把super.show()用try.catch包裹住即可。这样就可以忽略此Bug了。

    @Override
    public void show(FragmentManager manager, String tag) {
        try{
            super.show(manager,tag);
        }catch (IllegalStateException ignore){
        }
    }
重写DialogFragment
使用try.catch的方式明显不够优雅。那么就可以考虑第二种方案。

既然DialogFragment是继承于Fragment，那么可以把它完整的代码全部拷贝过来，然后重写show()方法，把commit()替换为commitAllowingStateLoss()即可。

不过如果Fragment的继承有包的限制，可以在自己的项目中，新建一个android.support.v4.app的Package，然后在其中新建一个PDialogFragment.java，将DialogFragment的代码全部copy过来，重写对应的方法即可。

    public void show(FragmentManager manager, String tag) {
        mDismissed = false;
        mShownByMe = true;
        FragmentTransaction ft = manager.beginTransaction();
        ft.add(this, tag);
        ft.commitAllowingStateLoss();
    }
虽然用继承DialogFragment的方式，自己去写show()，不去调用super.show()应该也可以，但是show()中其实是有一些状态的置换的，最好不要用这种方式，用这种方式在调试的时候可能没有问题，但是发布出去可能会导致未知问题。

```


##### 2018.06.07，（）动态添加控件
```
        //添加imageView
        LinearLayout linearLayout = findViewById(R.id.id_ll_ad_container);
        LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        recommendImageView.setLayoutParams(params);
        DisplayUtils.setMargins(recommendImageView,
                DisplayUtils.dip2px(this, 5),
                DisplayUtils.dip2px(this, 5),
                DisplayUtils.dip2px(this, 5),
                DisplayUtils.dip2px(this, 5));
        linearLayout.addView(recommendImageView);
```
> [参考：]()

##### 2018.06.07，（）dp与px互转
```
    public static int px2dip(Context context, float pxValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (pxValue / scale + 0.5f);
    }

    public static int dip2px(Context context, float dipValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dipValue * scale + 0.5f);
    }
```
> [参考:Android 为什么 dp2px 或 px2dp 公式需要加 0.5f](https://blog.csdn.net/changcsw/article/details/52440543)

这两个公式网上很多，但为什么 最后都要加上0.5f 呢？
 按正常的推理应该是  dip = pxValue / scale 和 px = dipValue * scale ，

实际上准确的值就应该是 咱们推理出来的，之所以后面加上0.5f是因为 咱们要的只不是那么精准，根据推理算出来的是个浮点数，而咱们程序中一般使用int类型就够了，这里涉及到一个类型转换精准度问题，熟悉java特效的同学应该知道

float 类型的 4.1 和4.9 强转成int类型后，会失去精准度变成 int类型的4， 而如果咱们想四舍五入的话，把他们都加上0.5f，这样转换出来的结果就是：

4.4 + 0.5 = 4.9 转为int 还是4，而4.5 + 0.5 = 5.0 转换成int后就是5，正好是四舍五入，这样就保证了咱们算出来的值相对精准。


##### 2018.06.06，（）DialogFragment监听返回键
实现DialogInterface.OnKeyListener接口

```
    @Override
    public boolean onKey(DialogInterface dialog, int keyCode, KeyEvent event) {
        //如果按下的是返回键
        if(keyCode == KeyEvent.KEYCODE_BACK){
            //TO DO 
            return true;
        }else {
            //这里注意当不是返回键时需将事件扩散，否则无法处理其他点击事件
            return false;
        }
    }
```
> [参考：dialogfragment监听返回键](https://blog.csdn.net/u011421608/article/details/51542761)


##### 2018.05.30，（）代码修改控件位置

```
params =  (RelativeLayout.LayoutParams)mLlLeftTop.getLayoutParams();
params.setMargins(left, top,right,bottom);
mLlLeftTop.setLayoutParams(params);
```

> [参考：android在java代码中修改控件的位置](https://blog.csdn.net/qq_33335724/article/details/78500377)

##### 2018.05.30，（）Android图片资源放置姿势

> [Android drawable微技巧，你所不知道的drawable的那些细节](https://blog.csdn.net/guolin_blog/article/details/50727753)


##### 2018.05.25，（）选择谷歌照片裁剪的一些坑
```
/**
* 如何用户选择了图片，则打开剪裁界面
* 注意：打开剪裁界面后，该Activity不能销毁，否则从谷歌照片选择图片后的Uri是没有权限
* SecurityException: Permission Denial +
* * com.google.android.apps.photos.contentprovider.impl.MediaContentProvider* requires the provider be exported, or grantUriPermission()
*/
```

##### 2018.05.21，（）onCreateOptionMenu()不执行的可能原因
没有设置Toolbar
例如下面代码
```
if(getSupportActionBar() != null){
    setSupportActionBar(mToolbar);
}
```

##### 2018.05.21，（）TextView加图标
```
drawableLeft="@drawable/xxx"
```

##### 2018.05.21，（）AppBarLayout注意
AppBarLayout要放在Coordinlayout布局里面


##### 2018.05.16，（）ViewPager画廊效果
> [官方文档：https://developer.android.com/training/animation/screen-slide](官方文档：https://developer.android.com/training/animation/screen-slide)

> [Android超高仿QQ附近的人搜索展示（一）](https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650820073&idx=1&sn=9e084723624180f7ab28e54f2aef132c&scene=23&srcid=0506b08maFirw2pBvnewcDsp#rd)

> [hongyangAndroid/MagicViewPager](https://github.com/hongyangAndroid/MagicViewPager)

> [ViewPager的切换动画](https://www.jianshu.com/p/4c07241292c0)

> [Android开发之ViewPager切换动画（只有你想不到，没有做不到）](https://www.jianshu.com/p/c9e8448472ed)

> [ViewPager系列之ViewPager一屏显示多个子页面](https://blog.csdn.net/JM_beizi/article/details/51297605)

##### 2018.05.16，（）调用系统相册
最常规的方式
```
Intent albumIntent = new Intent(Intent.ACTION_VIEW);
albumIntent.setDataAndType(Uri.fromFile(file), "image/*");
startActivity(albumIntent);
```

调用系统相册查看一张图片, 7.0
```
Intent albumIntent = new Intent(Intent.ACTION_VIEW);
File file;
file = new File(imagePath);
albumIntent.setDataAndType(Uri.fromFile(file), "image/*");
startActivity(albumIntent);
```

试图打开指定目录的相册(指定某个相册)是不行的
一个做法是，先打开app的一个相册界面，类似照片墙效果，然后再点击某一张图片，使用系统相册打开，

> 7.0 Uri权限问题

##### 2018.05.03，（）屏幕适配
480p:   480x800     hdpi    values-hdpi
720p:   720x1280    xhdpi
1080p:  1080x1920   xxhdpi
2k:     1440x2560   xxxhdpi
/res/value/dimens.xml

##### 2018.05.03，（）代码中改变dialog所占屏幕比例
> [参考:Android自定义Dialog大小控制](https://blog.csdn.net/true100/article/details/43982763)

```
        //在Dialog类调用
        Window dialogWindow = getWindow();
        WindowManager m = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
        Display d = m.getDefaultDisplay(); // 获取屏幕宽、高度
        WindowManager.LayoutParams p = dialogWindow.getAttributes(); // 获取对话框当前的参数值
        if (checkDeviceHasNavigationBar(context)){
            //有导航栏加上导航栏高度
            p.height = (int) ((d.getHeight() + getNavigationBarHeight() ) * 0.5);
        }else {
            p.height = (int) (d.getHeight() * 0.5);
        }
        p.width = (int) (d.getWidth() * 0.9);
        getWindow().setAttributes(p);
```

##### 2018.05.03，（）判断是否存在导航栏

```
    /**
     * 获取是否存在NavigationBar
     * @param context
     * @return
     */
    public boolean checkDeviceHasNavigationBar(Context context) {
        boolean hasNavigationBar = false;
        Resources rs = context.getResources();
        int id = rs.getIdentifier("config_showNavigationBar", "bool", "android");
        if (id > 0) {
            hasNavigationBar = rs.getBoolean(id);
        }
        try {
            Class systemPropertiesClass = Class.forName("android.os.SystemProperties");
            Method m = systemPropertiesClass.getMethod("get", String.class);
            String navBarOverride = (String) m.invoke(systemPropertiesClass, "qemu.hw.mainkeys");
            if ("1".equals(navBarOverride)) {
                hasNavigationBar = false;
            } else if ("0".equals(navBarOverride)) {
                hasNavigationBar = true;
            }
        } catch (Exception e) {

        }
        return hasNavigationBar;
    }
```
> [参考:判断有无虚拟按键（导航栏）](https://blog.csdn.net/z_x_Qiang/article/details/75911755)

##### 2018.05.03，（）获取导航栏高度

```
    private int getNavigationBarHeight() {
        Resources resources = context.getResources();
        int resourceId = resources.getIdentifier("navigation_bar_height","dimen", "android");
        int height = resources.getDimensionPixelSize(resourceId);
        Log.d(TAG, "Navi height:" + height);
        return height;
    }
```
> [参考:获取系统顶部状态栏(Status Bar)与底部导航栏(Navigation Bar)的高度](http://www.cnblogs.com/rossoneri/p/4142962.html)


##### 2018.04.09，（）简单自定义circle_progress_bar
> [参考：Android ProgressBar 详解 改变 ProgressBar 颜色](https://blog.csdn.net/chen930724/article/details/49807821)

主要修改：android:indeterminateDrawable这个属性
indeterminate是不确定进度的风格
```
    <style name="circle_progress_load">
        <item name="android:indeterminateDrawable">@drawable/progress_circle_shape</item>
        <item name="android:minWidth">80dp</item>
        <item name="android:minHeight">80dp</item>
        <item name="android:maxWidth">100dp</item>
        <item name="android:maxHeight">100dp</item>
    </style>
```
progress_circle_shape.xml
```
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromDegrees="0"
    android:pivotX="50%"
    android:pivotY="50%"
    android:toDegrees="360" >
    <shape
        android:innerRadiusRatio="3"
        android:shape="ring"
        android:thicknessRatio="14"
        android:useLevel="false" >
        <gradient
            android:centerY="0.50"
            android:endColor="#ffffff"
            android:startColor="#9e9e9e"
            android:type="sweep"
            android:useLevel="false" />
    </shape>
</rotate>
```

##### 2018.04.09，（）原生progressBar的转速
修改：android:indeterminateDuration


##### 2018.04.08，（）git强制推送当前分支,即使有冲突
```
git push [remote] --force
git push origin master --force
```
> [参考：常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)

##### 2018.04.08，（）合并指定分支到当前分支
```
git merge [branch]
当前master 合并dev到master
git merge dev
```

##### 2018.04.08，（）gradle依赖库冲突
其中一种情况：compile与implementation不一致
```
把compile方式改成implementation
```

##### 2018.04.08，（）64k限制
> [参考:配置方法数超过 64K 的应用](https://developer.android.com/studio/build/multidex.html)

如果您的 minSdkVersion 设置为 21 或更高值，您只需在模块级 build.gradle 文件中将 multiDexEnabled 设置为 true
```
android {
    defaultConfig {
        ...
        minSdkVersion 21 
        targetSdkVersion 26
        multiDexEnabled true
    }
    ...
}
```

但是，如果您的 minSdkVersion 设置为 20 或更低值，则您必须按如下方式使用 Dalvik 可执行文件分包支持库
```
android {
    defaultConfig {
        ...
        minSdkVersion 15 
        targetSdkVersion 26
        multiDexEnabled true
    }
    ...
}

dependencies {
  compile 'com.android.support:multidex:1.0.1'
}
```

如果您没有替换 Application 类，请编辑清单文件，按如下方式设置 <application> 标记中的 android:name
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    <application
            android:name="android.support.multidex.MultiDexApplication" >
        ...
    </application>
</manifest>
```

如果您替换了 Application 类，请按如下方式对其进行更改以扩展 MultiDexApplication（如果可能）
```
public class MyApplication extends MultiDexApplication { ... }
```

如果您替换了 Application 类，但无法更改基本类，则可以改为替换 attachBaseContext() 方法并调用 MultiDex.install(this) 来启用 Dalvik 可执行文件分包：
```
public class MyApplication extends SomeOtherApplication {
  @Override
  protected void attachBaseContext(Context base) {
     super.attachBaseContext(context);
     Multidex.install(this);
  }
}
```




