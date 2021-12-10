<h1 align="center">Android知识梳理</h1>
>包含：四大组件，Binder机制，消息机制，View绘制流程，事件分发，性能优化，视图优化，以及安全等知识点。

[toc]

## Android学习路线

* [developer-roadmap](https://github.com/kamranahmedse/developer-roadmap)：开发路线
* [android-developer-roadmap](https://github.com/anacoimbrag/android-developer-roadmap)：Android开发路线

## Android四大组件

#### Activity

* [Android Training - 详解Activity生命周期(Lesson 1 - 启动与销毁Activity)](http://hukai.me/android-training-managing-the-activity-lifecycle-lesson-1/)
* [Android Training - 详解Activity生命周期(Lesson 2 - 暂停与恢复activity)](http://hukai.me/android-training-managing-the-activity-lifecycle-lesson-2/)
* [Android Training - 详解Activity生命周期(Lesson 3 - 停止与重启activity)](http://hukai.me/android-training-managing-the-activity-lifecycle-lesson-3/)
* [Android Training - 详解Activity生命周期(Lesson 4 - 重新创建销毁的activity)](http://hukai.me/android-training-managing-the-activity-lifecycle-lesson-4/)
* [Android开发之InstanceState详解](http://www.cnblogs.com/hanyonglu/archive/2012/03/28/2420515.html)

#### Broadcast

* [解析BroadcastReceiver之你需要了解的一些东东](http://www.cnblogs.com/net168/p/3980068.html)
* [Android应用程序注册广播接收器（registerReceiver）的过程分析](http://blog.csdn.net/luoshengyang/article/details/6737352)
* [Android应用程序发送广播（sendBroadcast）的过程分析](http://blog.csdn.net/luoshengyang/article/details/6744448)

#### Service

* [Android中bindService的使用及Service生命周期](http://blog.csdn.net/iispring/article/details/48169339)
* [Android中bindService的使用及Service生命周期](http://blog.csdn.net/iispring/article/details/48169339)
* [Android通过startService实现批量下载示例](http://blog.csdn.net/iispring/article/details/48015475)
* [Android应用程序绑定服务（bindService）的过程源代码分析](http://blog.csdn.net/luoshengyang/article/details/6745181)
* [Android中IntentService的使用及其源码解析](http://blog.csdn.net/iispring/article/details/48046861)
* [Building Accessibility Services(建立可访问性服务)](http://www.android-doc.com/guide/topics/ui/accessibility/services.html)
* [Android Accessibility(辅助功能) --实现Android应用自动安装、卸载](http://blog.csdn.net/androidsecurity/article/details/41890369?utm_source=tuicool)
* [使用Android Accessibility实现免Root自动批量安装功能](http://www.infoq.com/cn/articles/android-accessibility-installing?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global)

#### ContentProvider

* [Android ContentProvider和Uri详解 (绝对全面)](http://blog.sina.com.cn/s/blog_9f233c070101euqx.html)
* [Android应用程序组件Content Provider应用实例](http://blog.csdn.net/luoshengyang/article/details/6950440)
* [Android应用程序组件Content Provider的启动过程源代码分析](http://blog.csdn.net/luoshengyang/article/details/6963418)
* [Android应用程序组件Content Provider在应用程序之间共享数据的原理分析](http://blog.csdn.net/luoshengyang/article/details/6967204)
* [Android应用程序组件Content Provider的共享数据更新通知机制分析](http://blog.csdn.net/luoshengyang/article/details/6985171)
* [android 应用的启动过程分析](http://www.jianshu.com/p/a1f40b39b3de)(ContentProvider的onCreate方法在Application的onCreate方法前面)

## Android系统机制

#### Binder机制

* [Android跨进程通信：图文详解 Binder机制 原理](https://blog.csdn.net/carson_ho/article/details/73560642)
* [操作系统：图文详解 内存映射](https://www.jianshu.com/p/719fc4758813)
* [Android Bander设计与实现 - 设计篇](https://blog.csdn.net/universus/article/details/6211589)

#### 消息机制

* [Android系统源码分析--消息循环机制](http://codemx.cn/2017/07/13/AndroidOS004-HandleMessageLooper/)
* [android的消息处理机制（图+源码分析）——Looper,Handler,Message](https://www.cnblogs.com/codingmyworld/archive/2011/09/14/2174255.html)
* [Android 异步消息处理机制 让你深入理解 Looper、Handler、Message三者关系](https://blog.csdn.net/lmj623565791/article/details/38377229)
* [深入源码解析Android中的Handler,Message,MessageQueue,Looper](https://blog.csdn.net/iispring/article/details/47180325)
* [This Handler class should be static or leaks might occur: IncomingHandler](https://stackoverflow.com/questions/11407943/this-handler-class-should-be-static-or-leaks-might-occur-incominghandler?noredirect=1&lq=1)
* [Handlers and memory leaks in Android](http://stackoverflow.com/questions/11278875/handlers-and-memory-leaks-in-android)

#### View绘制流程

* [MeasureSpec详解](/Android/View/MeasureSpec.md)
* [Android View绘制流程](http://blog.csdn.net/wangjinyu501/article/details/9008271)
* [公共技术点之 View 绘制流程](http://a.codekk.com/detail/Android/lightSky/公共技术点之%20View%20绘制流程)
* [Android中measure过程、WRAP_CONTENT详解以及xml布局文件解析流程浅析(上)](http://blog.csdn.net/qinjuning/article/details/8051811)
* [Android中measure过程、WRAP_CONTENT详解以及xml布局文件解析流程浅析(下)](http://blog.csdn.net/qinjuning/article/details/8074262)
* [Android中View(视图)绘制不同状态背景图片原理深入分析以及StateListDrawable使用详解](http://blog.csdn.net/qinjuning/article/details/7474827)
* [Android中将布局文件/View添加至窗口过程分析 ---- 从setContentView()谈起](http://blog.csdn.net/qinjuning/article/details/7226787)
* [MeasureSpec介绍及使用](http://www.cnblogs.com/nanxiaojue/p/3536381.html?utm_source=tuicool&utm_medium=referral)

#### View事件分发

* [图解 Android 事件分发机制](http://www.jianshu.com/p/e99b5e8bd67b#)
* [Android 中Touch（触屏）事件传递机制](http://blog.csdn.net/wangjinyu501/article/details/22584465)
* [Android 编程下 Touch 事件的分发和消费机制](http://www.cnblogs.com/sunzn/archive/2013/05/10/3064129.html)
* [Android-onInterceptTouchEvent()和onTouchEvent()总结](http://blog.csdn.net/lvxiangan/article/details/9309927)（注：这篇文章没找到原创，连接是转载的，如果谁找到原创可以提供给我。）
* [Android中View的量算、布局及绘图机制](http://blog.csdn.net/iispring/article/details/49203945)
* [源码解析Android中View的measure量算过程](http://blog.csdn.net/iispring/article/details/49403315)
* [源码解析Android中View的layout布局过程](http://blog.csdn.net/iispring/article/details/50366021)

#### Intent

* [Android中Intent概述及使用](http://blog.csdn.net/iispring/article/details/48417779)
* [Android中Intent对象与Intent Filter过滤匹配过程详解](http://blog.csdn.net/iispring/article/details/48481793)
* [Android中常见Intent习惯用法-上篇(附源码下载) ](http://blog.csdn.net/iispring/article/details/48578295)
* [Android权限和动作大全](http://blog.csdn.net/github_25928675/article/details/46460417)

## Android性能优化

#### 性能优化典范

* [Android性能优化典范 - 第1季](http://hukai.me/android-performance-patterns/)
* [Android性能优化典范 - 第2季](http://hukai.me/android-performance-patterns-season-2/)
* [Android性能优化典范 - 第3季](http://hukai.me/android-performance-patterns-season-3/)
* [Android性能优化典范 - 第4季](http://hukai.me/android-performance-patterns-season-4/)
* [Android性能优化典范 - 第5季](http://hukai.me/android-performance-patterns-season-5/)
* [Android性能优化典范 - 第6季](http://hukai.me/android-performance-patterns-season-6/)
* [Android性能优化之内存篇](http://hukai.me/android-performance-memory/)

#### 内存优化

* [ANDROID内存优化(大汇总——上)](https://blog.csdn.net/a396901990/article/details/37914465)
* [ANDROID内存优化(大汇总——中)](https://blog.csdn.net/a396901990/article/details/38707007)
* [ANDROID内存优化(大汇总——全)](https://blog.csdn.net/a396901990/article/details/38904543)

#### 优化总结

* [Android内存问题总结](/Android/optimize/Memory.md)
* [ANR问题总结](/Android/optimize/ANR.md)
* [Merge使用](/Android/optimize/Merge.md)
* [ViewStubs使用](/Android/optimize/ViewStubs.md)
* [5个导致主线程卡顿较鲜为人知的元凶](http://blog.nimbledroid.com/2016/03/21/ways-to-hang-main-thread-zh.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
* [Android抽象布局——include、merge 、ViewStub](http://blog.csdn.net/xyz_lmn/article/details/14524567)
* [Performance Tuning On Android](http://blog.venmo.com/hf2t3h4x98p5e13z82pl8j66ngcmry/performance-tuning-on-android)
* [BlockCanary — 轻松找出Android App界面卡顿元凶](http://blog.zhaiyifan.cn/2016/01/16/BlockCanaryTransparentPerformanceMonitor/)
* [Android 性能优化必知必会](https://www.androidperformance.com/2018/05/07/Android-performance-optimization-skills-and-tools/)
* [Android性能优化典范](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)
* [Android性能优化典范](http://hukai.me)
* [Systrace 基础知识](https://www.androidperformance.com/2019/07/23/Android-Systrace-Pre/)

#### Android性能优化工具

* [matrix](https://github.com/Tencent/matrix)：**Matrix** 是一款微信研发并日常使用的应用性能接入框架，支持iOS, macOS和Android。 Matrix 通过接入各种性能监控方案，对性能监控项的异常数据进行采集和分析，输出相应的问题分析、定位与优化建议，从而帮助开发者开发出更高质量的应用。

* [AppMethodOrder](https://github.com/zjw-swun/AppMethodOrder)：Android代码方法执行时间监控工具
* [scalpel](https://github.com/JakeWharton/scalpel)：A surgical debugging tool to uncover the layers under your app.
* [AndroidPerformanceMonitor](https://github.com/markzhai/AndroidPerformanceMonitor)：A transparent ui-block detection library for Android. (known as BlockCanary)
* [Cockroach](https://github.com/android-notes/Cockroach)：降低Android非必要crash
* [Takt](https://github.com/wasabeef/Takt)：Takt is Android library for measuring the FPS using Choreographer.
* [SoloPi](https://github.com/alipay/SoloPi)：Soloπ是一个无线化、非侵入式的Android自动化工具，公测版拥有录制回放、性能测试、一机多控三项主要功能，能为测试开发人员节省宝贵时间。

##  Android开源控件

- [UI控件](/Android/OpenSource/OpenSource.md#UI控件)
- [功能控件](/Android/OpenSource/OpenSource.md#功能控件)
- [控件集合](/Android/OpenSource/OpenSource.md#控件集合)

## 开源公司及集合

* [开源公司](/Android/OpenSource/GroupSource.md#开源公司)
* [开源集合](/Android/OpenSource/GroupSource.md#开源集合)

## Android安全

* [理解Android安全机制](https://www.cnblogs.com/lao-liang/p/5089336.html)

## Android命令

* [aapt命令说明](/Android/adb/aapt.md)
* [adb-shell常用命令](/Android/adb/adb-shell.md)
* [adb常用命令](/Android/adb/adb.md)
* [Android-am命令使用](/Android/adb/am.md)
* [Android-dumpsys命令详细使用](/Android/adb/dumpsys.md)
* [Android-logcat命令详解](/Android/adb/logcat.md)
* [Android-pm命令详解](/Android/adb/pm.md)
* [adb卸载系统应用](/Android/adb/adb-plus.md)

## Android架构

* [android-architecture-components](https://github.com/googlesamples/android-architecture-components)
* [android-architecture](https://github.com/googlesamples/android-architecture)
* [Android源码设计模式分析项目](https://github.com/simple-android-framework/android_design_patterns_analysis)

## Android开发技巧

* [Android开发技巧](/Android/OS/AndroidDevelopSkill.md)

## Android系统下载及编译

* [AndroidOS编译](/Android/OS/AndroidSystemCompile.md)
* [AndroidOS下载编译](/Android/OS/AndroidOSDownload.md)

## Android反编译

* [Android反编译Apk](/Android/Decompilation.md)
* [jadx](https://github.com/skylot/jadx)：Android源码反编译工具（教程：[Android 反编译利器，jadx 的高级技巧](https://www.jianshu.com/p/e5b021df2170)）
* [bytecode-viewer](https://github.com/Konloch/bytecode-viewer)：A Java 8+ Jar & Android APK Reverse Engineering Suite (Decompiler, Editor, Debugger & More) [https://bytecodeviewer.com](https://bytecodeviewer.com)



