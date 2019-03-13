# 内存问题分类
* 内存泄漏
* 内存溢出
* 内存优化

## 内存泄漏原因、分类以及解决办法

### 内存泄漏定义：
    如果一个对象从根节点开始是可达的有引用的，但实际上它已经没有再使用了，是无用的，这样的对象就是内存泄漏的对象。

### 原因：
    在内存回收的时候，应该被回收的对象，还存在引用指向该对象导致系统对该对象回收失败。

### 分类：
* 非静态内部类默认持有外部类的引用会导致内存泄漏
* Handler持有当前类的Context对象，导致对象无法释放
* 静态对象引用，无法释放对象导致内存泄漏
* 线程内引用有生命周期的外部对象
* 资源未关闭造成的内存泄漏
* 单例引用Context导致内存泄漏
* 广播未及时注销造成内存泄漏
* 集合类泄漏


### 内存泄漏举例及解决办法：

#### 1.非静态内部类默认持有外部类的引用会导致内存泄漏
静态内部类与非静态内部类之间存在一个最大的区别，就是非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。

代码：
```java
public class Outer {
    
    private void outerDo() {}
    
    class Inter {
        
        private void innerDo() {
            // 内部类可以直接访问外部类成员，原因在于隐式持有了一个外部类引用
            outerDo();
            // Outer.this 就是内部类隐式持有的外部类引用
            Outer.this.outerDo();
        }
    }
}
```

如果Inter的实例为静态的会导致内存泄漏。

解决方法：将Inter改成静态内部类


#### 2.Handler持有当前类的Context对象，导致对象无法释放

引用自：[Android内存优化：常见内存泄露及优化方案](https://www.jianshu.com/p/8fac127433ce)

例如：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        start();
    }

    private void start() {
        Message msg = Message.obtain();
        msg.what = 1;
        mHandler.sendMessage(msg);
    }

    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            if (msg.what == 1) {
                // 做相应逻辑
            }
        }
    };
}
```

熟悉Handler消息机制的都知道，mHandler会作为成员变量保存在发送的消息msg中，即msg持有mHandler的引用，而mHandler是Activity的非静态内部类实例，即mHandler持有Activity的引用，那么我们就可以理解为msg间接持有Activity的引用。msg被发送后先放到消息队列MessageQueue中，然后等待Looper的轮询处理（MessageQueue和Looper都是与线程相关联的，MessageQueue是Looper引用的成员变量，而Looper是保存在ThreadLocal中的）。那么当Activity退出后，msg可能仍然存在于消息对列MessageQueue中未处理或者正在处理，那么这样就会导致Activity无法被回收，以致发生Activity的内存泄露。

解决办法：静态内部类+弱引用

例如：

```java
public class MainActivity extends AppCompatActivity {

    private static class ParseHandler extends XyHandler<MainActivity> {

        private ParseHandler(MainActivity activity) {
            super(activity);
        }

        @Override
        protected void handleMessage(Message msg, MainActivity activity) {
            switch (msg.what) {
                case 0:
                    activity.doSomething();
                    break;
                default:
                    break;
            }
        }
    }

    private Handler mHandler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mHandler = new ParseHandler(this);
        start();
    }

    private void start() {
        Message msg = Message.obtain();
        msg.what = 1;
        mHandler.sendMessage(msg);
    }
    
    public void doSomething(){
        
    }
}

public abstract class XyHandler<T> extends Handler {

    private WeakReference<T> mWeak;

    public XyHandler(T t) {
        mWeak = new WeakReference<>(t);
    }

    @Override
    public void handleMessage(Message msg) {
        if (mWeak == null || mWeak.get() == null) {
            return;
        }
        handleMessage(msg, mWeak.get());
        super.handleMessage(msg);
    }

    protected abstract void handleMessage(Message msg, T t);

}

```

mHandler通过弱引用持有Activity时，在GC操作时，Activity就会被正常回收。

关于消息发送机制看博客：[Android系统源码分析--消息循环机制](http://codemx.cn/2017/07/13/AndroidOS004-HandleMessageLooper/)，上面的解决办法也可以在进一步封装，可以看博客最后的Handler正确使用方法。

关于强引用、软引用、弱引用、虚引用可以看文章：[内存泄漏：使用弱应用处理外部类引用](https://www.jianshu.com/p/26ca58b490f0)

#### 3.静态对象引用，无法释放对象导致内存泄漏
这种情况不常遇到，一般有些新手会犯这个错误。这种就是一些对象的引用是静态的，导致无法回收。

例如：

```java
public class MainActivity extends AppCompatActivity {

    privite static MainActivity mMainActivity;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mMainActivity = this;
    }
}
```
还有自定义View或者ViewGroup中也是出现这种状况，导致内存无法释放，出现内存溢出。对于这种Context不能用静态引用，如果需要调用这些对象内的方法可以将对应方法代码提取出来。


#### 4.线程内引用有生命周期的外部对象
线程一般是指Thread和AsyncTask

Thread，例如：
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 模拟相应耗时逻辑
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```

AsyncTask，例如：
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                // 模拟相应耗时逻辑
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return null;
            }
        }.execute();
    }
}
```

我们在刚开始写代码的时候都是像上面一样写线程和异步任务，但是这种方式使用Thread和AsyncTask都是匿名内部类对象，默认持有外部类Activity的引用，在Activity回收时可能会有Thread和AsyncTask没有执行完的情况，所以可能会造成内存泄漏。

#### 5.资源未关闭或者未释放造成的内存泄漏

IO流、File流或者数据库、Cursor等资源在使用完成后要及时关闭，如果没有及时关闭，会导致缓冲对象一直被占用，不能得到释放，发生内存泄漏。Bitmap未回收。


#### 6.单例引用Context导致内存泄漏

例如：
```java
public class AppSettings {

    private static AppSettings sInstance;
    private Context mContext;

    private AppSettings(Context context) {
        this.mContext = context;
    }

    public static AppSettings getInstance(Context context) {
        if (sInstance == null) {
            sInstance = new AppSettings(context);
        }
        return sInstance;
    }
}
```
单例模式的生命周期会和整个应用的生命周期一样，如果在使用单例模式时传入的是Activity或者Service等声明周期短于Application的声明周期时，都会造成内存泄漏。因此我们传入的Context应该是Application的Context，这样生命周期就和Application的声明周期一样，避免造成内存泄漏


#### 7.广播未及时注销造成内存泄漏

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        this.registerReceiver(mReceiver, new IntentFilter());
    }

    private BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            // 接收到广播需要做的逻辑
        }
    };

    @Override
    protected void onDestroy() {
        super.onDestroy();
        this.unregisterReceiver(mReceiver);
    }
}
```

广播是非静态内部类，会持有Activity引用，而广播注册会将广播对象注册到系统内部，如果没有取消注册，那么系统中会存在Activity的应用，因此不能释放Activity，造成内存泄漏。

解决办法：就是在OnDestroy方法中取消注册广播。

#### 8.集合类泄漏
集合类如果仅仅有添加元素的方法，而没有相应的删除机制，导致内存被占用。如果这个集合类是全局性的变量 (比如类中的静态属性，全局性的 map 等即有静态引用或 final 一直指向它)，那么没有相应的删除机制，很可能导致集合所占用的内存只增不减。


## 内存溢出

### 原因：
    内存占用不断增大不能及时释放，导致内存值大于系统给与单个程序的内存极限从而导致内存溢出

### 导致内存溢出常见情况

* 上面内存泄漏问题导致内存不能释放从而导致内存溢出
* Android系统内存杀手Bitmap
* 多次重复创建对象
* 无限循环创建对象

### 内存溢出解决办法

#### 1.上面内存泄漏问题导致内存不能释放从而导致内存溢出：

这个主要针对上面内存泄漏的解决办法进行处理。

#### 2.Android系统内存杀手Bitmap：

第一：及时销毁；第二：采用小分辨率加载图片，减小内存占用；第三：采用软引用，可以在内存不足时被系统回收。

#### 3.多次创建对象：

在onDraw或者dispatchDraw方法中创建对象，导致对象多次创建，大量占用内存。

解决办法：将对象的创建放到构造函数中进行

#### 4.无限循环创建对象：

方法嵌套导致无限循环，或者刷新View视图导致无限刷新，在无限循环过程中创建对象，导致内存溢出。



## 内存优化

* 内存泄漏优化
* 内存溢出优化
* xml布局层级优化：通过HierarchyViewer进行查看View层级通过Layoutopt优化布局
* 资源占用优化
* 视图刷新优化
* 避免内部调用Getter和Setter方法
* 减少不必要的全局变量
* 避免使用枚举：枚举变量非常方便，但不幸的是它会牺牲执行的速度和并大幅增加文件体积。
* 合理利用缓存
* 关闭无用的资源：SQLite,Cursor,File,IO等在操作
* 优化Bitmap
* 合理使用ViewStub、merge
* 针对ListView进行复用优化
* 使用9patch
* 减少网络请求
* 数据压缩
* 合理使用线程池


## 内存分析工具
* leakcanary：[连接](https://github.com/square/leakcanary)
* LeakCanary原理分析：[连接](https://blog.csdn.net/aptentity/article/details/71308257)
* BlockCanary — 轻松找出Android App界面卡顿元凶：[连接](http://blog.zhaiyifan.cn/2016/01/16/BlockCanaryTransparentPerformanceMonitor/)
* Android Profiler：[连接](https://www.jianshu.com/p/e75680772375)



## 相关文章

* Android内存优化之OOM：[连接](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0920/3478.html)
* Android探究oom内幕：[连接](http://www.360doc.com/content/14/0526/11/9462341_381066152.shtml)
* Android内存管理原理：[连接](http://www.cnblogs.com/killmyday/archive/2013/06/12/3132518.html)
* Android性能调优：[连接](http://www.trinea.cn/android/android-performance-demo/)
* 性能优化之Java(Android)代码优化：[连接](http://www.trinea.cn/android/java-android-performance/)
* 性能优化之布局优化：[连接](http://www.trinea.cn/android/layout-performance/)
* 性能优化之数据库优化：[连接](http://www.trinea.cn/android/database-performance/)




## 参考
[常见的内存泄漏原因及解决方法](https://www.jianshu.com/p/90caf813682d)

[Android进程的内存管理分析](https://blog.csdn.net/linghu_java/article/details/39480761)

[Android内存的全面分析-让你吃透](https://www.jianshu.com/p/63f9846ae1f1)

[Android内存优化：常见内存泄露及优化方案](https://www.jianshu.com/p/8fac127433ce)

[Android内存泄漏终极解决篇](https://www.jianshu.com/p/c54e18376d09)