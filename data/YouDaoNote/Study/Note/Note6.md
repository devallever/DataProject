**笔记本6**

 - 记录日常的知识点
 - 按时间倒序记录


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

