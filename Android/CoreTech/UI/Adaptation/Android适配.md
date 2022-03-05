<h1 align="center">Android特性适配</h1>

[toc]

## Android开发对应Java适配

[agp-java-support](https://github.com/JakeWharton/agp-java-support)

## Android 9.0/P http 网络请求的问题

有以下三种解决方案

1. APP改用https请求
2. targetSdkVersion 降到27以下
3. 在 res 下新增一个 xml 目录，然后创建一个名为：network_security_config.xml 文件（名字自定） ，内容如下，大概意思就是允许开启http请求

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <network-security-config>
        <base-config cleartextTrafficPermitted="true" />
    </network-security-config>
    ```
    
    然后在APP的AndroidManifest.xml文件下的application标签增加以下属性
    
    ```xml
    <application
        ...
         android:networkSecurityConfig="@xml/network_security_config"
        ...
    />
    ```
    
## Android7.0以后下载问题

在AndroidManifest.xml中配置（provider的name属性默认）
```xml
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.CyDownloadFileProvider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/app_file_paths" />
        </provider>
```

**说明：**
android:grantUriPermissions="true"为临时授权

app_file_paths.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <!--代表外部存储区域的根目录下的文件 Environment.getExternalStorageDirectory()/DCIM/camerademo目录-->
    <external-path
        name="MX_DCIM"
        path="DCIM/camera" />
    <!--代表外部存储区域的根目录下的文件 Environment.getExternalStorageDirectory()/Pictures/camerademo目录-->
    <external-path
        name="mx_files"
        path="Download" />
    <!--代表app 私有的存储区域 Context.getFilesDir()目录下的images目录 /data/user/0/com.codemx.androidmx/files/images-->
    <files-path
        name="mx_private_files"
        path="images" />
    <!--代表app 私有的存储区域 Context.getCacheDir()目录下的images目录 /data/user/0/com.codemx.androidmx/cache/images-->
    <cache-path
        name="mx_private_cache"
        path="images" />
    <!--代表app 外部存储区域根目录下的文件 Context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)目录下的Pictures目录-->
    <!--/storage/emulated/0/Android/data/com.codemx.androidmx/files/Pictures-->
    <external-files-path
        name="mx_external_files"
        path="Pictures" />
    <!--代表app 外部存储区域根目录下的文件 Context.getExternalCacheDir目录下的images目录-->
    <!--/storage/emulated/0/Android/data/com.codemx.androidmx/cache/images-->
    <external-cache-path
        name="mx_cache_files"
        path="Download" />
</paths>
```

上面的name:隐藏真实目录后的显示名字；path:真实的下载文件的路径，例如下面代码中的PATH，要和这个一样。

例如：
```java
// 设置的下载路径：/storage/emulated/0/Download/
final String PATH = "Download";
String downloadPath = Environment.getExternalStorageDirectory() + File.separator + PATH;
// 原始地址：
String filePath = “/storage/emulated/0/Download/wechat.apk”
// 7.0之前地址
String filePath = "file:///storage/emulated/0/Download/wechat.apk";
// 7.0之后隐藏地址
String filePath = “content://com.codemx.androidmx.MxDownloadFileProvider/mx_files/wechat.apk”;
```

安装代码：
```java
    // filePath:文件真实路径
    public void installApk(Context context, String filePath) {
        File file = new File(filePath);
        if (!file.exists()) {
            return;
        }
        Intent intent = new Intent(Intent.ACTION_VIEW);
        Uri uri;// 隐藏后的地址
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            uri = FileProvider.getUriForFile(context, context.getPackageName() + ".CyDownloadFileProvider", file);
            intent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        } else {
            uri = Uri.fromFile(file);
        }
        intent.setDataAndType(uri, "application/vnd.android.package-archive");
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.getApplicationContext().startActivity(intent);
    }
```

## 接收系统广播问题

在Android O（8.0及以后）接收系统广播要动态注册，在动态注册广播接收系统广播如：

```java
    void register(Context context) {
            IntentFilter filter = new IntentFilter();
            filter.addAction(Intent.ACTION_PACKAGE_ADDED);
            filter.addAction(Intent.ACTION_PACKAGE_REPLACED);
            filter.addAction(Intent.ACTION_PACKAGE_REMOVED);
            filter.addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED);
            filter.addDataScheme("package");
            context.registerReceiver(this, filter);
        }
```

需要加入：

```java
filter.addDataScheme("package");
```

否则接收不到广播，接收到广播后在获取包名时：

```java
void onReceive(Context context, Intent intent) {
   String pkg = intent.getDataString();
   // 这里获取到的数据格式是："package:com.liulishuo.engzo"，前面会有个前缀：“package:”，使用时要去除
}
```

