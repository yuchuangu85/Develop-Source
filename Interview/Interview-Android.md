<h1 align="center">Interview Android</h1>

[toc]

## Android四大组件

### 1.Activity

* Activity的四种启动模式对比[Activity launch model](../Android/Framework/Activity/ActivityStackGoogle.md)
* 下拉状态栏是不是影响activity的生命周期：**Android下拉通知栏不会影响Activity的生命周期方法**
* Activity状态保存与恢复：参看：Android-->Android.md

### 2.Service

* Service的开启方式、生命周期：参看：Android-->Android.md
* IntentService原理及作用是什么？用完自动结束，自己调用stopService
* 如何保证一个后台服务不被杀死？（相同问题：如何保证service在后台不被kill？）比较省电的方式是什么？
   * [如何保证Service在后台不被杀死？](https://blog.csdn.net/weixin_41101173/article/details/79719617)
   * [Android 保证Service服务不被杀死的几个方法](https://blog.csdn.net/xialong_927/article/details/80262622)
   * [android六种方法保证service不被杀死](https://segmentfault.com/a/1190000011905950)
   * 设置闹铃自启。
* 

### 3.Broadcast

* BroadcastReceiver的理解，BroadcastReceiver，LocalBroadcastReceiver 区别：参看：Android-->Android.md（区别：前者是手机系统全局广播，后者是应用内广播（Handler实现））
* 

### 4.ContentProvider

* 谈谈你对ContentProvider的理解、说说ContentProvider、ContentResolver、ContentObserver 之间的关系。参看：Android-->Android.md
* 如何导入外部数据库? ContentProvider
* 请介绍下ContentProvider 是如何实现数据共享的？通过uri查找对应的ContentProvider，然后通过Binder传递数据
* ContentProvider的权限管理？(解答：读写分离，权限控制-精确到表级，URL控制)
* Android系统为什么会设计ContentProvider？数据跨进程共享。

## Fragment

* Fragment状态保存startActivityForResult是哪个类的方法，在什么情况下使用？
* 如何实现Fragment的滑动？ViewPager+Fragment
* fragment之间传递数据的方式？接口，观察者模式，Eventbus，livedata



## Window、View、ViewGroup

### Activity-Window-View三者的差别

* Activity里面实例化了一个Window，Window里面有一个DecorView(根布局)。看一下这篇文章，Android Window机制探索：https://blog.csdn.net/qian520ao/article/details/78555397

### AlertDialog、popupWindow、Activity区别

* 不是同一个Window
* Activity的attach方法,这里是为Activity实例化了一个PhoneWindow实例
* Dialog的构造方法里面也是实例化了一个PhoneWindow实例

### LinearLayout、RelativeLayout、FrameLayout的特性及对比，并介绍使用场景。

* RelativeLayout慢于LinearLayout是因为它会让子View调用**2次measure**过程，而后者只需一次，但是有weight属性存在时，后者同样会进行两次measure。
*  RelativeLayout的子View如果**高度**和RelativeLayout不同，会引发效率问题，可以使用padding代替margin以优化此问题。
* 在不响应层级深度的情况下，使用Linearlayout而不是RelativeLayout。
* 据此，说明两个常见的，暂时没有引起注意的事情：
   * ①作为顶级View的**DecorView**却是个垂直方向的LinearLayout，上面是标题栏，下面是内容栏，我们常用的setContentView()方法就是给内容栏设置布局。采用RelativeLayout并不会降低层级深度，因此这种情况下使用LinearLayout效率更高。
   * ②为开发者***\*默认新建\**RelativeLayout**是希望开发者能采用***\**\*尽量少的\*\*View层级\****，很多效果是需要多层LinearLayout的嵌套，这必然不如一层的RelativeLayout性能更好。因此我们应该尽量减少布局嵌套，减少层级结构，使用比如viewStub，include等技巧。可以进行较大的布局优化。

### 介绍下View、SurfView

- View是Android中所有控件的基类
- View适用于主动更新的情况，而SurfaceView则适用于被动更新的情况，比如频繁刷新界面。
- View在主线程中对页面进行刷新，而SurfaceView则开启一个子线程来对页面进行刷新。
- View在绘图时没有实现双缓冲机制，SurfaceView在底层机制中就实现了双缓冲机制。

### 如何优化自定义View

* 继承View，通过绘制自定义View
* 继承ViewGroup，通过设置属性自定义布局

### View刷新机制



### View绘制流程、渲染

* [死磕Android_View工作原理你需要知道的一切](https://blog.csdn.net/xfhy_/article/details/90270630)



### 自定义控件原理

* 继承View，进行绘制
* 继承ViewGroup，组合或者绘制

### 自定义View如何提供获取View属性的接口？

​	在res/values目录下新建一个attrs.xml文件

```
自定义属性中 name是自定义属性的名字，format是指自定义属性的取值。
format的取值类如下所示：

boolean 表示attr取值为true或者false
color 表示attr取值是颜色类型，例如#ff3344,或者是一个指向color的资源id，例如R.color.colorAccent.
dimension 表示 attr 取值是尺寸类型，例如例如取值16sp、16dp，也可以是一个指向dimen的资源id，例如R.dimen.dp_16
float 表示attr取值是整形或者浮点型
fraction 表示 attr取值是百分数类型，只能以%结尾，例如30%
integer 表示attr取值是整型
string 表示attr取值是String类型，或者一个指向String的资源id，例如R.string.testString
reference 表示attr取值只能是一个指向资源的id。
enum 表示attr取值只能是枚举类型。
flag 表示attr取值是flag类型。
注意：
refrence , 表示attr取值只能是一个指向资源的id。
```

### 计算一个view的嵌套层级

### 封装view的时候怎么知道view的大小

### View渲染

* [Android进阶——性能优化之布局渲染原理和底层机制机详解及卡顿根源探究（四）](https://blog.csdn.net/CrazyMo_/article/details/80038948)
* [Android绘制优化----系统显示原理](https://zhuanlan.zhihu.com/p/27344882)

### RecycleView优化

* [RecyclerView性能优化](https://zhuanlan.zhihu.com/p/49338922)

### ListView的优化

* [Android-ListView优化常见的三种方式](https://blog.csdn.net/u014657752/article/details/47379941)
* [Android性能优化之提高ListView性能的技巧](https://juejin.im/post/6844903784238284814)

### RecyclerView和ListView的性能对比和区别

* [Android ListView 与 RecyclerView 对比浅析—缓存机制](https://zhuanlan.zhihu.com/p/23339185)

## 适配

### 屏幕适配的处理技巧都有哪些?

使用相对布局，尽量相对四个边设置边距来固定布局的位置。



## 事件分发

### Touch事件传递流程

事件传递大体过程：Activity--> Window-->DecorView --> View树从上往下，传递过程中谁想拦截就拦截自己处理。MotionEvent是Android中的点击事件。主要事件类型：

- ACTION_DOWN 手机初次触摸到屏幕事件
- ACTION_MOVE 手机在屏幕上滑动时触发，会回调多次
- ACTION_UP 手指离开屏幕时触发

需要关注的几个方法。

- dispatchTouchEvent(event);
- onInterceptTouchEvent(event);
- onTouchEvent(event);

上面3个方法可以用以下伪代码来表示其关系：

```java
public boolean dispatchTouchEvent(MotionEvent ev) { 
    boolean consume = false;//事件是否被消费 
    if (onInterceptTouchEvent(ev)) {
        //调用onInterceptTouchEvent判断是否拦截事件 
        consume = onTouchEvent(ev);
        //如果拦截则调用自身的onTouchEvent方法 
    } else { 
        consume = child.dispatchTouchEvent(ev);
        //不拦截调用子View的dispatchTouchEvent方法 } 
        return consume;
        //返回值表示事件是否被消费，true事件终止，false调用父View的onTouchEvent方法
    }
}
```

事件分发中的onTouch 和onTouchEvent 有什么区别，又该如何使用？

### 点击事件被拦截，但是想传到下面的View，如何操作？



## Handler-Message-Looper

### Handler以及子线程更新UI问题

* [Handler面试知识点](../Android/CoreTech/Handler/Handler.md)
* [Handler-Looper无限循环不阻塞线程](../Android/CoreTech/Handler/AndroidHandlerLooper.md)



## 动画

* Android属性动画特性
* 差值器、估值器：[Android 动画：你真的会使用插值器与估值器吗？（含详细实例教学）](https://blog.csdn.net/carson_ho/article/details/72863901)
* 



## 性能优化

### hprof如何生成？如果分析判断OOM？





### OOM定义，原因，是否可以try-catch，如何避免？

[Android内存优化之OOM](http://hukai.me/android-performance-oom/)

**常见的OOM情况有三种：**

**1）** **java.lang.OutOfMemoryError: Java heap space**  ------>java堆内存溢出，此种情况最常见，一般由于内存泄露或者堆的大小设置不当引起。对于内存泄露，需要通过内存监控软件查找程序中的泄露代码，而堆大小可以通过虚拟机参数-Xms,-Xmx等修改。

**2**）**java.lang.OutOfMemoryError: PermGen space/** Metaspace------>java永久代（元数据）溢出，即方法区溢出了，一般出现于大量Class或者jsp页面，或者采用cglib等反射机制的情况，因为上述情况会产生大量的Class信息存储于方法区。此种情况可以通过更改方法区的大小来解决，使用类似-XX:PermSize/MetaspaceSize=64m    -XX:MaxPermSize/MaxMetaspaceSize =256m的形式修改。另外，过多的常量也会导致方法区溢出。

**3）** **java.lang.StackOverflowError -**----->不会抛OOM error，但也是比较常见的Java内存溢出。JAVA虚拟机栈溢出，一般是由于程序中存在死循环或者深度递归调用造成的，栈大小设置太小也会出现此种溢出。可以通过虚拟机参数-Xss来设置栈的大小。

**Catching java.lang.OutOfMemoryError?**

java.lang.Error的文档说：

> An Error is a subclass of Throwable that indicates serious problems that a reasonable application should not try to catch

但是由于java.lang.Error是java.lang.Throwable的子类，因此我可以捕获此类Throwable。

### Native异常捕获

* [Android 平台 Native 代码的崩溃捕获机制及实现](https://mp.weixin.qq.com/s/g-WzYF3wWAljok1XjPoo7w)
* 

### 内存泄漏

* 参看：Android-->Performance-->内存问题及解决方案

### Bitmap 使用时候注意什么？

* Bitmap的recycler()

### Bitmap如何处理大图，如一张30M的大图，如何预防OOM

[高效加载大型位图](https://developer.android.com/topic/performance/graphics/load-bitmap)

[如何加载100M的图片却不撑爆内存,一张 100M 的大图，如何预防 OOM？](https://zhuanlan.zhihu.com/p/118526873)

### Android为每个应用程序分配的内存大小是多少？

[Android为每个应用分配多少内存？如何进行性能调优](https://cloud.tencent.com/developer/article/1444860)



### 如何保持应用的稳定性

* [深入探索Android稳定性优化](6844903972587716621)
* [Use Android vitals to improve your app's performance, stability, and size](https://developer.android.google.cn/distribute/best-practices/develop/android-vitals)



### App启动崩溃异常捕捉

* [有赞 Android 崩溃保护的探索及实践](https://tech.youzan.com/android_crash_explore/)
* [理解Android Crash处理流程](http://gityuan.com/2016/06/24/app-crash/)
* [Android开发之打造永不崩溃的APP——Crash防护](https://www.jianshu.com/p/01b69d91a3a8)

### 启动优化

* Android-->Performnace-->启动优化.md

### 启动页白屏及黑屏解决？

* [Android启动页黑屏及最优解决方案](https://cloud.tencent.com/developer/article/1184907)

### 性能优化如何分析systrace？

* [手把手教你使用Systrace（一）](https://zhuanlan.zhihu.com/p/27331842)
* [手把手教你使用Systrace（二）——锁优化](https://zhuanlan.zhihu.com/p/27535205)

### 如何对Android 应用进行性能分析以及优化?（如内存优化、网络优化、布局优化、电量优化、业务优化）

* 参考：Android-->Performance

### 统计启动时长,标准



## Binder、AIDL、通信

### Binder机制

* [Android跨进程通信：图文详解 Binder机制 原理](https://blog.csdn.net/carson_ho/article/details/73560642)
* [操作系统：图文详解 内存映射](https://www.jianshu.com/p/719fc4758813)
* [Android Bander设计与实现 - 设计篇](https://blog.csdn.net/universus/article/details/6211589)

### Binder机制及底层实现

### 简述IPC？

* Android-->Framwork-->AndroidIPC.md

### 进程间通信的方式、机制？

* 方式：AIDL 、广播、文件、socket、管道

### jni原理？

* [Android JNI原理分析](http://gityuan.com/2016/05/28/android-jni/)

### 谈谈对多进程开发的理解以及多进程应用场景

### 谈谈对进程共享和线程安全的认识

### 进程调度

* 参看：Android-->FrameworkReview-->doc-->Android系统底层框架篇-->Android进程框架-->Android进程框架：进程的创建、启动与调度流程

### Android进程分类？

* 第一高：前台进程：前台进程是Android系统中最重要的进程，是与用户正在交互的进程。
* 第二高：可见进程：可见进程指部分程序界面能够被用户看见，却不在前台与用户交互。
* 第三高：服务进程：一个包含已启动服务的进程就是服务进程，服务没有用户界面，不与用户直接交互，但能够在后台长期运行，提供用户所关心的重要功能。
* 第四高：后台进程：如果一个进程不包含任何已经启动的服务，而且没有用户可见的Activity，则这个进程就是后台进程。
* 第五高：空进程：空进程是不包含任何活跃组件的进程。在系统资源紧张时会被首先清楚。

### Android 上的 Inter-Process-Communication 跨进程通信时如何工作的？

* [Android-IPC](../Android/Framework/AndroidIPC.md)

### AIDL

* 参看：Android-->FrameworkReview-->doc-->Android系统底层框架篇-->Android进程框架-->Android系统应用框架篇：AIDL



## Framework源码

### Android动画框架实现原理

### requestLayout，onlayout，onDraw，drawChild区别与联系

* requestLayout：请求刷新视图
* onLayout：重新布局复写方法
* onDraw：绘制复习方法
* drawChild：绘制子View复写方法

### invalidate和postInvalidate的区别及使用

* invalidate 主线程调用
* postInvalidate 子线程调用

### ActivityThread，AMS，WMS的工作原理

### AsyncTask 如何使用、机制，如何取消AsyncTask?

### AsyncTask：[AsyncTask](Android/AndroidAsyncTask.md)

### AndroidManifest的作用与理解

* Apk的配置信息，包含四大组件，权限信息，图标信息，名称信息等

### Android线程有没有上限？线程池有没有上限？

```
cat /proc/sys/kernel/threads-max  // 查看线程限制
ulimit -a  // 查看所有限制
```

### App启动流程，从点击桌面开始

* [Android系统源码分析--Activity启动过程](http://codemx.cn/2018/01/26/AndroidOS008-Activity/)

### 简述Activity启动全部过程

* [startActivity启动过程分析](http://gityuan.com/2016/03/12/start-activity/)

### 系统启动流程是什么？（提示：Zygote进程 –> SystemServer进程 –> 各种系统服务 –> 应用进程）

* [Android 系统启动流程](https://juejin.im/post/6844903929369608206)
* [Android 系统启动流程](https://juejin.im/post/6844903663971155982)

### Dalvik、ART虚拟机理解及区别？

* Android-->Performance-->1.内存模型管理及GC.md

### App中唤醒其他进程的实现方式

* 启动服务，发送静态广播，启动Activity

### 进程拉活、保活

* [史上最强Android保活思路：深入剖析腾讯TIM的进程永生技术](https://segmentfault.com/a/1190000021579231)
* [【腾讯Bugly干货分享】Android 进程保活招式大全](https://segmentfault.com/a/1190000006251859)
* [Android进程保活、拉活方案](https://blog.csdn.net/weixin_38203696/article/details/90232355)
* [Android进程永生技术终极揭秘：进程被杀底层原理、APP应对被杀技巧](https://cloud.tencent.com/developer/article/1591535)

## 插件化、组件化、模块化、热更新、热修复

对热修复和插件化的理解
插件化原理分析
模块化实现（好处，原因）
热修复,插件化
项目组件化的理解

### 动态加载

### 组件之间相互引用，如何解决

- 调用其他组件的对外提供的方法：之前看到过一种思路,利用"接口+实现"的方式,定义一个ComponentBase 中间层，然后里面有每个组件对外提供方法调用的Interface，每个组件在初始化的时候就把这些Interface给实现了，然后其他组件需要用的时候就从ComponentBase里面取。
- 界面跳转：ARouter

对于应用更新这块是如何做的？(解答：灰度，强制更新，分区域更新)？

## 三方库源码

### LruCache

* [LruCache的使用及原理](https://www.cnblogs.com/huhx/p/useLruCache.html)

图片库对比
图片库的源码分析
图片框架缓存实现
图片加载原理

图片加载库相关，bitmap如何处理大图，如一张30M的大图，如何预防OOM

[详细信息](../Android/CoreTech/三方库/Summary.md)

## 网络


TCP的3次握手和四次挥手
TCP与UDP的区别
TCP与UDP的应用
HTTP协议
HTTP1.0与2.0的区别
HTTP报文结构
HTTP与HTTPS的区别以及如何实现安全性
如何验证证书的合法性?
https中哪里用了对称加密，哪里用了非对称加密，对加密算法（如RSA）等是否有了解?
client如何确定自己发送的消息被server收到?
谈谈你对WebSocket的理解
WebSocket与socket的区别







## 数据

### 序列化的作用，以及Android两种序列化的区别：

* [Android中Serializable和Parcelable序列化对象详解](https://www.cnblogs.com/yezhennan/p/5527506.html)

### Android中数据存储方式

* SharedPreferences: 小东西,最终是xml文件中,key-value的形式存储的.
* 文件
* 数据库
* ContentProvider
* 网络



## 架构、设计模式

### 1.Jetpack



### 2.架构

2.1 MVC MVP MVVM原理和区别

2.2谈谈对RxJava的理解

2.3从0设计一款App整体架构，如何去做？

### 3.设计模式

谈谈你对Android设计模式的理解

你所知道的设计模式有哪些？

项目中常用的设计模式

手写生产者/消费者模式

写出观察者模式的代码

适配器模式，装饰者模式，外观模式的异同？

Fragment如果在Adapter中使用应该如何解耦？



## Gradle

### 1.描述清点击 Android Studio 的 build 按钮后发生了什么

* [Android build 过程](Android/Android-build.md)

### 2.应用程序打包、安装和加载流程

* 查看：FrameworkReview-->doc-->Android系统应用框架篇-->Android包管理框架



## 安全

谈谈你对安卓签名的理解。

请解释安卓为啥要加签名机制?

视频加密传输

App 是如何沙箱化，为什么要这么做？

权限管理系统（底层的权限是如何进行 grant 的）？

### **Android 数字签名**

校验用户身份，校验数据的完整性。



## 资料

Github同步地址：[https://github.com/AweiLoveAndroid/CommonDevKnowledge](https://github.com/AweiLoveAndroid/CommonDevKnowledge)

[Timdk857/Android-Architecture-knowledge-2-: 阿里P7移动架构师学习路线 (github.com)](https://github.com/Timdk857/Android-Architecture-knowledge-2-)

[Android-Architecture-knowledge-2-/Android高级面试题.md at master · Timdk857/Android-Architecture-knowledge-2- (github.com)](https://github.com/Timdk857/Android-Architecture-knowledge-2-/blob/master/面试题/Android高级面试题.md)

