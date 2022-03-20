<h1 align="center">Android权限总结</h1>

[toc]



Android开发权限总结，动态权限需要API>=23,该权限为Android 9.0 整理最新权限列表

#### 一、PROTECTION_NORMAL：Android 9.0 (API28)--普通权限
|权限|名称|API|
|---|---|---|
|android.permission.ACCESS_LOCATION_EXTRA_COMMANDS||API 1|
|android.permission.ACCESS_NETWORK_STATE||API 1|
|android.permission.ACCESS_NOTIFICATION_POLICY||API 23|
|android.permission.ACCESS_WIFI_STATE||API 1|
|android.permission.BLUETOOTH||API 1|
|android.permission.BLUETOOTH_ADMIN||API 1|
|android.permission.BROADCAST_STICKY||API 1|
|android.permission.CHANGE_NETWORK_STATE||API 1|
|android.permission.CHANGE_WIFI_MULTICAST_STATE||API 4|
|android.permission.CHANGE_WIFI_STATE||API 1|
|android.permission.DISABLE_KEYGUARD||API 1|
|android.permission.EXPAND_STATUS_BAR||API 1|
|android.permission.FOREGROUND_SERVICE||API 28|
|android.permission.GET_PACKAGE_SIZE||API 1|
|android.permission.INSTALL_SHORTCUT||API 19|
|android.permission.INTERNET||API 1|
|android.permission.KILL_BACKGROUND_PROCESSES||API 8|
|android.permission.MANAGE_OWN_CALLS||API 26|
|android.permission.MODIFY_AUDIO_SETTINGS||API 1|
|android.permission.NFC||API 9|
|android.permission.READ_SYNC_SETTINGS||API 1|
|android.permission.READ_SYNC_STATS||API 1|
|android.permission.RECEIVE_BOOT_COMPLETED||API 1|
|android.permission.READ_SYNC_STATS||API 1|
|android.permission.RECEIVE_BOOT_COMPLETED||API 1|
|android.permission.REORDER_TASKS||API 1|
|android.permission.REQUEST_COMPANION_RUN_IN_BACKGROUND||API 26|
|android.permission.REQUEST_COMPANION_USE_DATA_IN_BACKGROUND||API 26|
|android.permission.REQUEST_DELETE_PACKAGES||API 26|
|android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS||API 23|
|android.permission.SET_ALARM||API 9|
|android.permission.SET_WALLPAPER||API 1|
|android.permission.SET_WALLPAPER_HINTS||API 1|
|android.permission.TRANSMIT_IR||API 19|
|android.permission.USE_FINGERPRINT||API 23|
|android.permission.VIBRATE||API 1|
|android.permission.WAKE_LOCK||API 1|
|android.permission.WRITE_SYNC_SETTINGS||API 1|

#### 二、Signature permissions：Android 9.0 (API28)--签名权限
|权限|名称|API|
|---|---|---|
|android.permission.BIND_ACCESSIBILITY_SERVICE||API 16|
|android.permission.BIND_AUTOFILL_SERVICE||API 26|
|android.permission.BIND_CARRIER_SERVICES||API 23|
|android.permission.BIND_CHOOSER_TARGET_SERVICE||API 23|
|android.permission.BIND_CONDITION_PROVIDER_SERVICE||API 24|
|android.permission.BIND_DEVICE_ADMIN||API 8|
|android.permission.BIND_DREAM_SERVICE||API 21|
|android.permission.BIND_INCALL_SERVICE||API 23|
|android.permission.BIND_INPUT_METHOD||API 3|
|android.permission.BIND_MIDI_DEVICE_SERVICE||API 23|
|android.permission.BIND_NFC_SERVICE||API 19|
|android.permission.BIND_NOTIFICATION_LISTENER_SERVICE||API 18|
|android.permission.BIND_PRINT_SERVICE||API 19|
|android.permission.BIND_SCREENING_SERVICE||API 24|
|android.permission.BIND_TELECOM_CONNECTION_SERVICE||API 23|
|android.permission.BIND_TEXT_SERVICE||API 14|
|android.permission.BIND_TV_INPUT||API 21|
|android.permission.BIND_VISUAL_VOICEMAIL_SERVICE||API 26|
|android.permission.BIND_VOICE_INTERACTION||API 21|
|android.permission.BIND_VPN_SERVICE||API 14|
|android.permission.BIND_VR_LISTENER_SERVICE||API 24|
|android.permission.BIND_WALLPAPER||API 8|
|android.permission.CLEAR_APP_CACHE||API 1|
|android.permission.MANAGE_DOCUMENTS||API 19|
|android.permission.READ_VOICEMAIL||API 21|
|android.permission.REQUEST_INSTALL_PACKAGES||API 23|
|android.permission.SYSTEM_ALERT_WINDOW||API 1|
|android.permission.WRITE_SETTINGS||API 1|
|android.permission.WRITE_VOICEMAIL||API 21|

#### 三、Dangerous permissions：Android 9.0 (API28)--危险权限
|权限组名|权限名称|API|
|---|---|---|
|android.permission-group.CALENDAR| * READ_CALENDAR <br> * WRITE_CALENDAR|API 1|
|android.permission-group.CALL_LOG| * READ_CALL_LOG <br>  * WRITE_CALL_LOG <br>  * PROCESS_OUTGOING_CALLS|API 16 <br> API 16 <br> API 1|
|android.permission-group.CAMERA| * CAMERA|API 1|
|android.permission-group.CONTACTS| * READ_CONTACTS <br>  * WRITE_CONTACTS <br>  * GET_ACCOUNTS|API 1|
|android.permission-group.LOCATION| * ACCESS_FINE_LOCATION <br>  * ACCESS_COARSE_LOCATION|API 1|
|android.permission-group.MICROPHONE| * RECORD_AUDIO|API 1|
|android.permission-group.PHONE| * READ_PHONE_STATE <br>  * READ_PHONE_NUMBERS <br>  * CALL_PHONE <br>  * ANSWER_PHONE_CALLS <br>  * ADD_VOICEMAIL <br>  * USE_SIP|API 1 <br> API 26 <br> API 1 <br> API 26 <br> API 14 <br> API 9|
|android.permission-group.SENSORS| * BODY_SENSORS|API 20|
|android.permission-group.SMS| * SEND_SMS <br>  * RECEIVE_SMS <br>  * READ_SMS <br>  * RECEIVE_WAP_PUSH <br>  * RECEIVE_MMS|API 1|
|android.permission-group.STORAGE|READ_EXTERNAL_STORAGE <br> WRITE_EXTERNAL_STORAGE|API 16 <br> API 4|


#### 四、权限使用代码
```java
// 权限判断
int permissionCheck = ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE);
if(permissionCheck == PackageManager.PERMISSION_GRANTED){
    // 有权限
} else {
    // 无权限
}

// 权限申请
ActivityCompat.shouldShowRequestPermissionRationale(Activity activity, String permission);

// 权限结果
@Override
public void onRequestPermissionResult(int requestCode, String permissions[], int[] grantResults){

}
```

#### 五、特殊权限
##### 1、悬浮窗权限：
(1)AndroidManifest配置权限 
```xml
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
```

WindowManager.LayoutParams.type在8.0上有所不同，需要如下配置：
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    mLayoutParams.type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY;
} else {
    mLayoutParams.type = WindowManager.LayoutParams.TYPE_PHONE;
}
```

> WindowManager.LayoutParams.TYPE_TOAST只在部分少量机型可用，尽可能少用。

(2)判断是否有悬浮窗权限
对于4.3及以下系统，悬浮窗权限均为true。
```java
if (Build.VERSION.SDK_INT < Build.VERSION_CODES.KITKAT) {
    return true;
} 
```

4.4～6.0以下系统，可以通过反射AppOpsManager获取悬浮窗权限（AppOpsManager是API 19新增的类）。
```java
if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
    try {
        Class cls = Class.forName("android.content.Context");
        Field declaredField = cls.getDeclaredField("APP_OPS_SERVICE");
        declaredField.setAccessible(true);
        Object obj = declaredField.get(cls);
        if (!(obj instanceof String)) {
            return false;
        }
        String str2 = (String) obj;
        obj = cls.getMethod("getSystemService", String.class).invoke(context, str2);
        cls = Class.forName("android.app.AppOpsManager");
        Field declaredField2 = cls.getDeclaredField("MODE_ALLOWED");
        declaredField2.setAccessible(true);
        Method checkOp = cls.getMethod("checkOp", Integer.TYPE, Integer.TYPE, String.class);
        int result = (Integer) checkOp.invoke(obj, 24, Binder.getCallingUid(), context.getPackageName());
        return result == declaredField2.getInt(cls);
    } catch (Exception e) {
        return false;
    }
}
```

6.0 Android系统提供了对应的方法获取悬浮窗权限。
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    return Settings.canDrawOverlays(context);
}
// 实测Settings.canDrawOverlays(context)兼容至Android最新版(当前9.0)。
```

(3)跳转开启悬浮窗权限的页面
Android在6.0及以上同样提供了系统级的悬浮窗权限开启页面。如下跳转：
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    try {
        Class clazz = Settings.class;
        Field field = clazz.getDeclaredField("ACTION_MANAGE_OVERLAY_PERMISSION");
        Intent intent = new Intent(field.get(null).toString());
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setData(Uri.parse("package:" + context.getPackageName()));
        context.startActivity(intent);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

但是在6.0之下，悬浮窗权限开启页面真是金屋藏娇，该页面不仅区别于不同rom，甚至区别于相同rom的不同版本，更有甚者只将其藏在了手机管家App里，真是找的辛苦。

考虑到现今4.x、5.x手机占比不高且只会越来越少，这里只提供前往app设置页面的代码。如果各位有需要可以根据手机rom自行适配。
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT && Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
    Intent intent = new Intent();
    intent.setAction(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
    intent.setData(Uri.fromParts("package", context.getPackageName(), null));
    context.startActivity(intent);
}
```
4.4以下版本建议忽略。

##### 2.无障碍辅助权限
判断是否有无障碍辅助权限
```java
    public static boolean isSettingsOn(Context mContext, Class serviceClass) {
        int accessibilityEnabled = 0;
        final String service = mContext.getPackageName() + "/" + serviceClass.getCanonicalName();

        try {
            accessibilityEnabled = Settings.Secure.getInt(mContext.getApplicationContext().getContentResolver(),
                    android.provider.Settings.Secure.ACCESSIBILITY_ENABLED);
        } catch (Settings.SettingNotFoundException e) {
            e.printStackTrace();
        }
        TextUtils.SimpleStringSplitter mStringColonSplitter = new TextUtils.SimpleStringSplitter(':');
        if (accessibilityEnabled == 1) {
            String settingValue = Settings.Secure.getString(mContext.getApplicationContext().getContentResolver(),
                    Settings.Secure.ENABLED_ACCESSIBILITY_SERVICES);
            if (settingValue != null) {
                mStringColonSplitter.setString(settingValue);
                while (mStringColonSplitter.hasNext()) {
                    String accessibilityService = mStringColonSplitter.next();
                    if (accessibilityService.equalsIgnoreCase(service)) {
                        return true;
                    }
                }
            }
        }
        return false;
    }
```

其中class为继承自AccessibilityService的无障碍功能类。
上述代码兼容至Android最新版本（9.x）。

跳转至无障碍辅助页面
通常情况下下面代码会跳转至需要无障碍辅助的应用列表页，但是不排除个别rom修改了无障碍辅助的页面层级关系，这种情况如有需要应单独针对rom做适配。

```java
    public static void jumpToSetting(final Context context) {
        try {
            Intent intent = new Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS);
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            context.startActivity(intent);
        } catch (Throwable e) {//出现异常 可能和rom删除了该页面有关
            e.printStackTrace();
        }
    }
```

##### 3.sdk>=28网络权限

```
    <!-- 注意：在targetSdkVersion>=28时,在9.0设备中默认禁用http请求 -->
    <!-- 需要设置usesCleartextTraffic="true"解除限制,否则会影响广告的请求 -->
    <!-- 或者通过xml/network_security_config.xml配置usesCleartextTraffic="true"解除限制 -->
    <!-- targetSdkVersion>28时,设置requestLegacyExternalStorage="true",可以停用分区存储 -->
```

xml/network_security_config.xml

```xml
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

application

```xml
<?xml version="1.0" encoding="utf-8"?>
   <manifest xmlns:android="http://schemas.android.com/apk/res/android"
     package="com.example.myapplication">

    <uses-permission android:name="android.permission.INTERNET"></uses-permission>
       <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:networkSecurityConfig="@xml/network_security_config"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
          <intent-filter>
             <action android:name="android.intent.action.MAIN" />

              <category android:name="android.intent.category.LAUNCHER" />
          </intent-filter>
        </activity>
      </application>

   </manifest>
```

