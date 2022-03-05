# 原生launcher3 添加角标

### 原理
 使用NotificationListenerService 接收应用发出的广播，并在launcher的界面图标上绘制出通知中的消息数。
 
 ### code
 #### 1.NLServer
 
 ```java
 package com.android.launcher3;

import android.app.Notification;
import android.content.Intent;
import android.os.Build;
import android.os.Bundle;
import android.service.notification.NotificationListenerService;
import android.service.notification.StatusBarNotification;
import android.util.Log;

import com.android.slider.model.IconIndicators;
import com.google.gson.Gson;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class NLService extends NotificationListenerService {

    private String TAG = this.getClass().getSimpleName();
    public static final String NOTIFY_LISTENER_HANDLER = "com.stan.NOTIFICATION_LISTENER_EXAMPLE";

    @Override
    public void onNotificationPosted(StatusBarNotification sbn) {
        Log.i(TAG,"**********  onNotificationPosted");
        Log.i(TAG,"ID :" + sbn.getId() + "\t" + sbn.getNotification().tickerText + "\t" + sbn.getPackageName());
        sendNotifyBroadcast(sbn,true);
    }

    private void sendNotifyBroadcast(StatusBarNotification sbn,boolean received){
        Notification notification = sbn.getNotification();
        if (notification == null) {
            return;
        }
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            Bundle extras = notification.extras;
            if (extras != null) {
                if (!sbn.getPackageName().equals("com.tencent.mobileqq") &&
                        !sbn.getPackageName().equals("com.stan.d1115") &&
                        !sbn.getPackageName().equals("com.tencent.mm") &&
                        !sbn.getPackageName().equals("com.sina.weibo")) {
                    return;
                }
                // 获取通知标题
                String title = extras.getString(Notification.EXTRA_TITLE, "");
                // 获取通知内容
                String content = extras.getString(Notification.EXTRA_TEXT, "");
                Log.i(TAG,"title: "+title+ "   content ; "+content);
                Intent i = new  Intent(NOTIFY_LISTENER_HANDLER);
                int num = getNum(title);
                IconIndicators indicators = new IconIndicators(sbn.getPackageName(),num,received);
                String message = new Gson().toJson(indicators, IconIndicators.class);
                Log.e(TAG,"notify message: "+message);
                i.putExtra("notification_event",message);
                sendBroadcast(i);
            }
        }
    }

    private int getNum(String title){
        int num = 0;
        Matcher matcher = Pattern.compile("((\\d+)(?=条新消息))").matcher(title);
        if (matcher.find()){
            try {
                num = Integer.valueOf(matcher.group(1));
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return num;
    }


    @Override
    public void onNotificationRemoved(StatusBarNotification sbn) {
        Log.i(TAG,"********** onNOtificationRemoved");
        Log.i(TAG,"ID :" + sbn.getId() + "\t" + sbn.getNotification().tickerText +"\t" + sbn.getPackageName());
        sendNotifyBroadcast(sbn,false);
    }

}

 ```
 
 #### 2.manifest 中注册服务
 
 ```java
   <service android:name=".NLService"
            android:label="@string/application_name"
            android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE">
            <intent-filter>
                <action android:name="android.service.notification.NotificationListenerService" />
            </intent-filter>
        </service>
 ```
 
 #### 3.Launcher 中开启通知接收
 
 ```java

    private void initNotifyListenerSettings(){
        startService(new Intent(this,NLService.class));
        if (!NotificationManagerCompat.getEnabledListenerPackages(this).contains(getPackageName())) {
            startActivity(new Intent("android.settings.ACTION_NOTIFICATION_LISTENER_SETTINGS"));
        }
    }
 ```
 
 #### 4.LauncherAppState中注册广播
 
 ```java
        // add notify listener handler
        filter = new IntentFilter();
        filter.addAction(NLService.NOTIFY_LISTENER_HANDLER);
        sContext.registerReceiver(mModel, filter);
 ```
 
 #### 5.LauncherModel中处理广播
 
 ```java
    onReceive 方法下处理广播
 
    else if (NLService.NOTIFY_LISTENER_HANDLER.equals(action)) {
                //handle notify listener
                // TODO: 2017/11/16  handle notify listener
                String temp = intent.getStringExtra("notification_event");
                Log.e(TAG,"notify receive message: "+temp);
                IconIndicators indicators = new Gson().fromJson(temp,IconIndicators.class);
                handleIconIndicator(indicators);
            }
        
    getShortcutInfo 方法下处理icon
  
     // the db
        if (icon == null) {
            if (c != null) {
                icon = getIconFromCursor(c, iconIndex, context);
            }
        }
        // the fallback icon
        if (icon == null) {
            icon = mIconCache.getDefaultIcon(user);
            info.usingFallbackIcon = true;
        }
        
        //info.setIcon(icon);
        //reset icon
        boolean needReset = false;
        for (IconIndicators indicators : iconIndicatorses) {
            if (indicators.packageName.equals(componentName.getPackageName())) {
                needReset = true;
                if (indicators.received)
                    info.setIcon(getNumIcon(icon,indicators.num));
                else {
                    info.setIcon(getNumIcon(icon,0));
                }
            }
        }
        if (!needReset) {
            info.setIcon(icon);
        }


        // From the cache.
        if (labelCache != null) {
            info.title = labelCache.get(componentName);
        }
        
        
    //添加方法
     /**---------------------------------------------*
     * TODO 处理接收到的通知
     * ----------------------------------------------*/
    public static ArrayList<IconIndicators> iconIndicatorses = new ArrayList<>();
    private void handleIconIndicator(IconIndicators indicators) {
        boolean inIconIndicatorList = false;
        for (IconIndicators iconI : iconIndicatorses) {
            if (indicators.packageName.equals(iconI.packageName)) {
                iconI.num = indicators.num;
                iconI.received = indicators.received;
                inIconIndicatorList = true;
            }
        }
        if (!inIconIndicatorList) {
            iconIndicatorses.add(indicators);
        }
        forceReload();
    }
    
    /**----------------------------------*
     *  TODO 画右上角小图标
     * ----------------------------------*/
    private Bitmap getNumIcon(Bitmap bitmap,int num){
        Log.e(TAG, "getNumIcon: =============getNumIcon===========");
        Log.e(TAG, "getNumIcon: num: "+num);
        if (num ==0) return bitmap;
        Bitmap changed = bitmap.copy(Bitmap.Config.ARGB_8888, true);
        Canvas canvas = new Canvas(changed);
        Paint paint = new Paint();
        paint.setColor(Color.RED);
        paint.setStyle(Paint.Style.FILL);
        paint.setStrokeWidth(10);
        canvas.drawCircle(changed.getWidth()*6/8,changed.getHeight()/4,changed.getWidth()/4,paint);
        paint.setColor(Color.WHITE);
        paint.setTextSize(30);
        paint.setTextAlign(Paint.Align.CENTER);
        Paint.FontMetrics fontMetrics = paint.getFontMetrics();
        float top = fontMetrics.top;//为基线到字体上边框的距离,即上图中的top
        float bottom = fontMetrics.bottom;//为基线到字体下边框的距离,即上图中的bottom
        int baseLineY = (int) (changed.getHeight()/4 - top/2 - bottom/2);//基线中间点
        if (num<=99)
            canvas.drawText(String .valueOf(num),changed.getWidth()*6/8,baseLineY,paint);
        else
            canvas.drawText("99+",changed.getWidth()*6/8,baseLineY,paint);
        Log.e(TAG, "getNumIcon: =============reset icon num.===========");
        return changed;
    }
 
 ```
 
 #### IconIndicators
 
 ```java
 public class IconIndicators {

    public String packageName="";
    public int num = 0;
    public boolean received = false;

    public IconIndicators(String packageName, int num, boolean received) {
        this.packageName = packageName;
        this.num = num;
        this.received = received;
    }

    public String getPackageName() {
        return packageName;
    }

    public void setPackageName(String packageName) {
        this.packageName = packageName;
    }

    public int getNum() {
        return num;
    }

    public void setNum(int num) {
        this.num = num;
    }

    public boolean isReceived() {
        return received;
    }

    public void setReceived(boolean received) {
        this.received = received;
    }
}
```
 
 
 ### 参考
 http://blog.csdn.net/qq_32707879/article/details/52279058
 
 https://github.com/kpbird/NotificationListenerService-Example
 