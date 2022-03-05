<h1 align="center">Android知识梳理</h1>
>包含：四大组件，Binder机制，消息机制，View绘制流程，事件分发，性能优化，视图优化，以及安全等知识点。

更新：2020-12-09

[toc]

## Android官方

* [Android Review](https://android-review.googlesource.com/admin/repos)
* [Android Code](https://android.googlesource.com/?format=HTML)
* [Android Code Search](https://cs.android.com/)
* [Android API](https://developer.android.com/reference/packages.html)

## Android学习路线

* [developer-roadmap](https://github.com/kamranahmedse/developer-roadmap)
* [android-developer-roadmap](https://github.com/anacoimbrag/android-developer-roadmap)
* [andorid-developer-roadmap 2022](https://github.com/skydoves/android-developer-roadmap)
* [Medium](https://medium.com/)
* [Android优秀博客](https://medium.com/androiddevelopers)
* [Android-ReadTheFuckingSourceCode](https://github.com/jeanboydev/Android-ReadTheFuckingSourceCode)

## Android四大组件

### Activity

* [Android Training - 详解Activity生命周期(Lesson 1 - 启动与销毁Activity)](http://hukai.me/android-training-managing-the-activity-lifecycle-lesson-1/)
* [Android Training - 详解Activity生命周期(Lesson 2 - 暂停与恢复activity)](http://hukai.me/android-training-managing-the-activity-lifecycle-lesson-2/)
* [Android Training - 详解Activity生命周期(Lesson 3 - 停止与重启activity)](http://hukai.me/android-training-managing-the-activity-lifecycle-lesson-3/)
* [Android Training - 详解Activity生命周期(Lesson 4 - 重新创建销毁的activity)](http://hukai.me/android-training-managing-the-activity-lifecycle-lesson-4/)
* [Android开发之InstanceState详解](http://www.cnblogs.com/hanyonglu/archive/2012/03/28/2420515.html)

### Broadcast

* [解析BroadcastReceiver之你需要了解的一些东东](http://www.cnblogs.com/net168/p/3980068.html)
* [Android应用程序注册广播接收器（registerReceiver）的过程分析](http://blog.csdn.net/luoshengyang/article/details/6737352)
* [Android应用程序发送广播（sendBroadcast）的过程分析](http://blog.csdn.net/luoshengyang/article/details/6744448)

### Service

* [Android中bindService的使用及Service生命周期](http://blog.csdn.net/iispring/article/details/48169339)
* [Android中bindService的使用及Service生命周期](http://blog.csdn.net/iispring/article/details/48169339)
* [Android通过startService实现批量下载示例](http://blog.csdn.net/iispring/article/details/48015475)
* [Android应用程序绑定服务（bindService）的过程源代码分析](http://blog.csdn.net/luoshengyang/article/details/6745181)
* [Android中IntentService的使用及其源码解析](http://blog.csdn.net/iispring/article/details/48046861)
* [Building Accessibility Services(建立可访问性服务)](http://www.android-doc.com/guide/topics/ui/accessibility/services.html)
* [Android Accessibility(辅助功能) --实现Android应用自动安装、卸载](http://blog.csdn.net/androidsecurity/article/details/41890369?utm_source=tuicool)
* [使用Android Accessibility实现免Root自动批量安装功能](http://www.infoq.com/cn/articles/android-accessibility-installing?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global)

### ContentProvider

* [Android ContentProvider和Uri详解 (绝对全面)](http://blog.sina.com.cn/s/blog_9f233c070101euqx.html)
* [Android应用程序组件Content Provider应用实例](http://blog.csdn.net/luoshengyang/article/details/6950440)
* [Android应用程序组件Content Provider的启动过程源代码分析](http://blog.csdn.net/luoshengyang/article/details/6963418)
* [Android应用程序组件Content Provider在应用程序之间共享数据的原理分析](http://blog.csdn.net/luoshengyang/article/details/6967204)
* [Android应用程序组件Content Provider的共享数据更新通知机制分析](http://blog.csdn.net/luoshengyang/article/details/6985171)
* [android 应用的启动过程分析](http://www.jianshu.com/p/a1f40b39b3de)(ContentProvider的onCreate方法在Application的onCreate方法前面)

## Android系统机制

### Binder机制

* [Android跨进程通信：图文详解 Binder机制 原理](https://blog.csdn.net/carson_ho/article/details/73560642)
* [操作系统：图文详解 内存映射](https://www.jianshu.com/p/719fc4758813)
* [Android Bander设计与实现 - 设计篇](https://blog.csdn.net/universus/article/details/6211589)
* [写给 Android 应用工程师的 Binder 原理剖析](https://mp.weixin.qq.com/s/NBm5lh8_ZLfodOXT8Ph5iA)

### Handler消息机制

* [Android系统源码分析--消息循环机制](http://codemx.cn/2017/07/13/AndroidOS004-HandleMessageLooper/)
* [android的消息处理机制（图+源码分析）——Looper,Handler,Message](https://www.cnblogs.com/codingmyworld/archive/2011/09/14/2174255.html)
* [Android 异步消息处理机制 让你深入理解 Looper、Handler、Message三者关系](https://blog.csdn.net/lmj623565791/article/details/38377229)
* [深入源码解析Android中的Handler,Message,MessageQueue,Looper](https://blog.csdn.net/iispring/article/details/47180325)
* [This Handler class should be static or leaks might occur: IncomingHandler](https://stackoverflow.com/questions/11407943/this-handler-class-should-be-static-or-leaks-might-occur-incominghandler?noredirect=1&lq=1)
* [Handlers and memory leaks in Android](http://stackoverflow.com/questions/11278875/handlers-and-memory-leaks-in-android)

### View绘制流程

* [MeasureSpec详解](/View/MeasureSpec.md)
* [Android View绘制流程](http://blog.csdn.net/wangjinyu501/article/details/9008271)
* [公共技术点之 View 绘制流程](http://a.codekk.com/detail/Android/lightSky/公共技术点之%20View%20绘制流程)
* [Android中measure过程、WRAP_CONTENT详解以及xml布局文件解析流程浅析(上)](http://blog.csdn.net/qinjuning/article/details/8051811)
* [Android中measure过程、WRAP_CONTENT详解以及xml布局文件解析流程浅析(下)](http://blog.csdn.net/qinjuning/article/details/8074262)
* [Android中View(视图)绘制不同状态背景图片原理深入分析以及StateListDrawable使用详解](http://blog.csdn.net/qinjuning/article/details/7474827)
* [Android中将布局文件/View添加至窗口过程分析 ---- 从setContentView()谈起](http://blog.csdn.net/qinjuning/article/details/7226787)
* [MeasureSpec介绍及使用](http://www.cnblogs.com/nanxiaojue/p/3536381.html?utm_source=tuicool&utm_medium=referral)

### View事件分发

* [图解 Android 事件分发机制](http://www.jianshu.com/p/e99b5e8bd67b#)
* [Android 中Touch（触屏）事件传递机制](http://blog.csdn.net/wangjinyu501/article/details/22584465)
* [Android 编程下 Touch 事件的分发和消费机制](http://www.cnblogs.com/sunzn/archive/2013/05/10/3064129.html)
* [Android-onInterceptTouchEvent()和onTouchEvent()总结](http://blog.csdn.net/lvxiangan/article/details/9309927)（注：这篇文章没找到原创，连接是转载的，如果谁找到原创可以提供给我。）
* [Android中View的量算、布局及绘图机制](http://blog.csdn.net/iispring/article/details/49203945)
* [源码解析Android中View的measure量算过程](http://blog.csdn.net/iispring/article/details/49403315)
* [源码解析Android中View的layout布局过程](http://blog.csdn.net/iispring/article/details/50366021)

### Intent详解

* [Android中Intent概述及使用](http://blog.csdn.net/iispring/article/details/48417779)
* [Android中Intent对象与Intent Filter过滤匹配过程详解](http://blog.csdn.net/iispring/article/details/48481793)
* [Android中常见Intent习惯用法-上篇(附源码下载) ](http://blog.csdn.net/iispring/article/details/48578295)
* [Android权限和动作大全](http://blog.csdn.net/github_25928675/article/details/46460417)

### Vsync信号

* [Android Graphic专栏](https://www.zhihu.com/column/c_1121774384913735680)
* [Android VSYNC （Choreographer）与UI刷新原理分析](https://cloud.tencent.com/developer/article/1582588)
* [Android VSYNC与图形系统中的撕裂、双缓冲、三缓冲浅析](https://cloud.tencent.com/developer/article/1586225)
* [Android 基于 Choreographer 的渲染机制详解](https://androidperformance.com/2019/10/22/Android-Choreographer/)

### [Systrace机制](https://androidperformance.com/2019/12/01/Android-Systrace-Vsync/)

* [Systrace 简介](https://www.androidperformance.com/2019/05/28/Android-Systrace-About/)
* [Systrace 基础知识 - Systrace 预备知识](https://www.androidperformance.com/2019/07/23/Android-Systrace-Pre/)
* [Systrace 基础知识 - Why 60 fps ？](https://www.androidperformance.com/2019/05/27/why-60-fps/)
* [Systrace 基础知识 - SystemServer 解读](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/)
* [Systrace 基础知识 - SurfaceFlinger 解读](https://www.androidperformance.com/2020/02/14/Android-Systrace-SurfaceFlinger/)
* [Systrace 基础知识 - Input 解读](https://www.androidperformance.com/2019/11/04/Android-Systrace-Input/)
* [Systrace 基础知识 - Vsync 解读](https://www.androidperformance.com/2019/12/01/Android-Systrace-Vsync/)
* [Systrace 基础知识 - Vsync-App ：基于 Choreographer 的渲染机制详解](https://androidperformance.com/2019/10/22/Android-Choreographer/)
* [Systrace 基础知识 - MainThread 和 RenderThread 解读](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/)
* [Systrace 基础知识 - Binder 和锁竞争解读](https://www.androidperformance.com/2019/12/06/Android-Systrace-Binder/)
* [Systrace 基础知识 - Triple Buffer 解读](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer)
* [Systrace 基础知识 - CPU Info 解读](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU)

## Android性能优化

* [性能优化-0.概论](./Performance/0.概论.md)
* [性能优化-1.内存模型管理及GC](./Performance/1.内存模型管理及GC.md)
* [性能优化-2.内存问题及解决方案](./Performance/2.内存问题及解决方案.md)
* [性能优化-3.启动优化](./Performance/3.启动优化.md)
* [性能优化-4.布局优化](./Performance/4.布局优化.md)
* [性能优化-5.Apk包大小优化](./Performance/5.Apk包大小优化.md)
* [性能优化-6.其他优化](./Performance/6.其他优化.md)
* [性能优化-7.内存性能测试](./Performance/7.内存性能测试.md)
* [性能优化-8.内存优化工具](./Performance/8.内存优化工具.md)
* [性能优化-9.Bug分析](./Performance/9.Bug分析.md)
* [性能优化-10.电量优化](./Performance/10.电量优化.md)
* [性能优化-11.Android优化大全](./Performance/11.Android优化大全.md)
* [性能优化-12.优化参考资料](./Performance/12.优化参考资料.md)

##  Android开源

- [开源库](./OpenSource/Library.md)：快速开发
- [图片压缩](./OpenSource/BitmapZip.md)：压缩图片大小
- [UI控件](./OpenSource/SmartDevelop.md): UI控件、功能控件、控件集合等
- [开源公司](./OpenSource/GroupSource.md#开源公司)
- [开源集合](./OpenSource/GroupSource.md#开源集合)

## Android安全

* [理解Android安全机制](https://www.cnblogs.com/lao-liang/p/5089336.html)

## Android命令

* [aapt命令说明](/adb/aapt.md)
* [adb-shell常用命令](/adb/adb-shell.md)
* [adb常用命令](/adb/adb.md)
* [Android-am命令使用](/adb/am.md)
* [Android-dumpsys命令详细使用](/adb/dumpsys.md)
* [Android-logcat命令详解](/adb/logcat.md)
* [Android-pm命令详解](/adb/pm.md)
* [adb卸载系统应用](/adb/adb-plus.md)

## Android架构

* [Guide to app architecture  | Android Developers (google.cn)](https://developer.android.google.cn/jetpack/guide)：官方最新架构--20220212
* [android-architecture-components](https://github.com/googlesamples/android-architecture-components)
* [architecture-components-samples](https://github.com/android/architecture-components-samples)
* [android-architecture](https://github.com/googlesamples/android-architecture)
* [Android-Clean-Boilerplate](https://github.com/dmilicic/Android-Clean-Boilerplate)
* [Android源码设计模式分析项目](https://github.com/simple-android-framework/android_design_patterns_analysis)
* [Android Architecture Pattern | by Akshay yadav | Team Pratilipi | Feb, 2022 | Medium](https://medium.com/team-pratilipi/android-architecture-pattern-82aa0cf7e236)

## Android系统下载及编译

* [AndroidOS编译](./OS/AndroidSystemCompile.md)
* [AndroidOS下载编译](./OS/AndroidOSDownload.md)

## Android反编译

* [Android反编译Apk](./OS/Decompilation.md)
* [jadx](https://github.com/skylot/jadx)：Android源码反编译工具（教程：[Android 反编译利器，jadx 的高级技巧](https://www.jianshu.com/p/e5b021df2170)）
* [bytecode-viewer](https://github.com/Konloch/bytecode-viewer)：A Java 8+ Jar & Android APK Reverse Engineering Suite (Decompiler, Editor, Debugger & More) [https://bytecodeviewer.com](https://bytecodeviewer.com)

## Android单元测试

* [Android单元测试只看这一篇就够了](https://www.jianshu.com/p/aa51a3e007e2)

## 其他

跳转支付宝付款码：alipays://platformapi/startapp?appId=20000056&chInfo=ch_jinliLeft
