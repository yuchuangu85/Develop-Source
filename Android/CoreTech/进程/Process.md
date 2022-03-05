<h1 align="center">Process</h1>

[toc]

进程（Process） 是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。

当某个应用组件启动且该应用没有运行其他任何组件时，Android 系统会使用单个执行线程为应用启动新的 Linux 进程。默认情况下，同一应用的所有组件在相同的进程和线程（称为“主”线程）中运行。

各类组件元素的清单文件条目``<activity>``、``<service>``、``<receiver>`` 和 ``<provider>``—均支持 android:process 属性，此属性可以指定该组件应在哪个进程运行。

## 进程和线程的区别

`进程有独立的内存空间，线程没有。`

## 进程生命周期

**1、前台进程**

- 托管用户正在交互的 Activity（已调用 Activity 的 ``onResume()`` 方法）
- 托管某个 Service，后者绑定到用户正在交互的 Activity
- 托管正在“前台”运行的 Service（服务已调用 ``startForeground()``）
- 托管正执行一个生命周期回调的 Service（``onCreate()``、``onStart()`` 或 ``onDestroy()``）
- 托管正执行其 ``onReceive()`` 方法的 BroadcastReceiver

**2、可见进程**  

- 托管不在前台、但仍对用户可见的 Activity（已调用其 ``onPause()`` 方法）。例如，如果前台 Activity 启动了一个对话框，允许在其后显示上一 Activity，则有可能会发生这种情况。
- 托管绑定到可见（或前台）Activity 的 Service

**3、服务进程**  

- 正在运行已使用 startService() 方法启动的服务且不属于上述两个更高类别进程的进程。

**4、后台进程**

- 包含目前对用户不可见的 Activity 的进程（已调用 Activity 的 ``onStop()`` 方法）。通常会有很多后台进程在运行，因此它们会保存在 LRU （最近最少使用）列表中，以确保包含用户最近查看的 Activity 的进程最后一个被终止。

**5、空进程**

- 不含任何活动应用组件的进程。保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。 为使总体系统资源在进程缓存和底层内核缓存之间保持平衡，系统往往会终止这些进程。\

## 多进程

如果注册的四大组件中的任意一个组件时用到了多进程，运行该组件时，都会创建一个新的 Application 对象。对于多进程重复创建 Application 这种情况，只需要在该类中对当前进程加以判断即可。

```java
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        Log.d("MyApplication", getProcessName(android.os.Process.myPid()));
        super.onCreate();
    }

    /**
     * 根据进程 ID 获取进程名
     * @param pid 进程id
     * @return 进程名
     */
    public  String getProcessName(int pid){
        ActivityManager am = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
        List<ActivityManager.RunningAppProcessInfo> processInfoList = am.getRunningAppProcesses();
        if (processInfoList == null) {
            return null;
        }
        for (ActivityManager.RunningAppProcessInfo processInfo : processInfoList) {
            if (processInfo.pid == pid) {
                return processInfo.processName;
            }
        }
        return null;
    }
}
```

>一般来说，使用多进程会造成以下几个方面的问题：
>
>- 静态成员和单例模式完全失效
>- 线程同步机制完全失效
>- SharedPreferences 的可靠性下降
>- Application 会多次创建

## 进程存活

### OOM_ADJ

| ADJ级别                | 取值 | 解释                                                |
| ---------------------- | ---- | --------------------------------------------------- |
| UNKNOWN_ADJ            | 16   | 一般指将要会缓存进程，无法获取确定值                |
| CACHED_APP_MAX_ADJ     | 15   | 不可见进程的adj最大值                               |
| CACHED_APP_MIN_ADJ     | 9    | 不可见进程的adj最小值                               |
| SERVICE_B_AD           | 8    | B List 中的 Service（较老的、使用可能性更小）       |
| PREVIOUS_APP_ADJ       | 7    | 上一个App的进程(往往通过按返回键)                   |
| HOME_APP_ADJ           | 6    | Home进程                                            |
| SERVICE_ADJ            | 5    | 服务进程(Service process)                           |
| HEAVY_WEIGHT_APP_ADJ   | 4    | 后台的重量级进程，system/rootdir/init.rc 文件中设置 |
| BACKUP_APP_ADJ         | 3    | 备份进程                                            |
| PERCEPTIBLE_APP_ADJ    | 2    | 可感知进程，比如后台音乐播放                        |
| VISIBLE_APP_ADJ        | 1    | 可见进程(Visible process)                           |
| FOREGROUND_APP_ADJ     | 0    | 前台进程（Foreground process)                       |
| PERSISTENT_SERVICE_ADJ | -11  | 关联着系统或persistent进程                          |
| PERSISTENT_PROC_ADJ    | -12  | 系统 persistent 进程，比如telephony                 |
| SYSTEM_ADJ             | -16  | 系统进程                                            |
| NATIVE_ADJ             | -17  | native进程（不被系统管理）                          |

### 进程被杀情况

| 进程杀死场景               | 调用接口              | 可能影响范围                                                 |
| -------------------------- | --------------------- | ------------------------------------------------------------ |
| 触发系统进程管理机制       | Lowmemorykiller       | 从进程importance值由大到小一次杀死，释放内存                 |
| 被第三方应用杀死（无Root） | killBackgroundProcess | 只能杀死OOM_ADJ为4以上的进程                                 |
| 被第三方应用杀死（有Root） | force-stop或者kill    | 理论上可以杀所有进程，一般只杀非系统关键进程和非前台和可见进程 |
| 厂商杀进程功能             | force-stop或者kill    | 理论上可以杀所有进程，包括Native进程                         |
| 用户主动“强制停止”进程     | force-stop            | 只能停用第三方和非system/phone进程应用（停用system进程应用会造成Android重启） |

### 进程保活方案

- 开启一个像素的 Activity
- 使用前台服务
- 多进程相互唤醒
- JobSheduler 唤醒
- 粘性服务 & 与系统服务捆绑