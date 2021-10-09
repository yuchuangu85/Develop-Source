<h1 align="center">Android性能优化</h1>

[toc]

## Java内存模型

* [Java 中对象的生命周期](https://juejin.im/post/6844903817897574407)
* [Java 对象的生命周期](https://blog.csdn.net/sodino/article/details/38387049)
* [JVM系列-01-JVM内存模型](https://www.jianshu.com/p/eab3a814010b)
* [JVM（二）Java虚拟机组成详解](https://segmentfault.com/a/1190000017878834)
* [Java内存管理-程序运行过程（一）](https://segmentfault.com/a/1190000019991674)
* [Java内存管理-JVM内存模型以及JDK7和JDK8内存模型对比总结（三）](https://segmentfault.com/a/1190000020028333)
* [深入详解JVM内存模型与JVM参数详细配置](https://www.cnblogs.com/rinack/p/9888692.html)
* [JVM内存模型详解](https://www.cnblogs.com/KevinStark/p/10925666.html)
* [JVM内存模型详解](https://blog.csdn.net/ocean_fan/article/details/79298076)
* [深入理解JVM-内存模型（jmm）和GC](https://www.jianshu.com/p/76959115d486)
* [JVM源码分析之临门一脚的OutOfMemoryError完全解读](http://lovestblog.cn/blog/2016/08/29/oom/)
* [什么是Java内存模型](https://www.jianshu.com/p/bf158fbb2432)

## ART vs Dalvik

* [Android ART vs Dalvik 差別](https://medium.com/@chloe.thhsu/android-art-vs-dalvik-%E5%B7%AE%E5%88%A5-c69df49f0b31)

## Android内存回收机制

* [Android基础之Java内存模型](https://www.zybuluo.com/TryLoveCatch/note/882064)
* [Android内存管理分析总结](https://www.jianshu.com/p/8b1d9c86fa84)
* [Android 内存回收](https://www.jianshu.com/p/10a78fd67b88)

## Low Memory Killer机制

* [Android LowMemoryKiller原理分析](http://gityuan.com/2016/09/17/android-lowmemorykiller/)
* [Android LowMemoryKiller 简介](https://juejin.im/post/6844903750474137613)

## 内存工具

* [leakcanary](https://github.com/square/leakcanary): LeakCanary is a memory leak detection library for Android. https://square.github.io/leakcanary
* [AndroidPerformanceMonitor](https://github.com/markzhai/AndroidPerformanceMonitor):A transparent ui-block detection library for Android. (known as BlockCanary)
* [AndroidGodEye](https://github.com/Kyson/AndroidGodEye): An app performance monitor(APM) , like "Android Studio profiler", you can easily monitor the performance of your app real time in browser
* [bugsnag-android](https://github.com/bugsnag/bugsnag-android): Bugsnag crash monitoring and reporting tool for Android apps
* [AnotherMonitor](https://github.com/AntonioRedondo/AnotherMonitor): Monitors and records the CPU and memory usage of Android devices
* [AndroidMonitor](https://github.com/jackuhan/AndroidMonitor): Android开发辅助工具fps,topActivity,activity启动耗时,电量cpu内存分析。适配全机型悬浮窗权限。
* [DoraemonKit](https://github.com/didi/DoraemonKit): A full-featured App (iOS & Android) development assistant. You deserve it. 简称 "DoKit" 。一款功能齐全的客户端（ iOS 、Android、微信小程序 ）研发助手，你值得拥有。https://www.dokit.cn/
* [matrix](https://github.com/Tencent/matrix)：**Matrix** 是一款微信研发并日常使用的应用性能接入框架，支持iOS, macOS和Android。 Matrix 通过接入各种性能监控方案，对性能监控项的异常数据进行采集和分析，输出相应的问题分析、定位与优化建议，从而帮助开发者开发出更高质量的应用。
* [iqiyi/xCrash](https://github.com/iqiyi/xCrash)：xCrash provides the Android app with the ability to capture java crash, native crash and ANR. No root permission or any system permissions are required.
* [AppMethodOrder](https://github.com/zjw-swun/AppMethodOrder)：Android代码方法执行时间监控工具
* [scalpel](https://github.com/JakeWharton/scalpel)：A surgical debugging tool to uncover the layers under your app.
* [Cockroach](https://github.com/android-notes/Cockroach)：降低Android非必要crash
* [Takt](https://github.com/wasabeef/Takt)：Takt is Android library for measuring the FPS using Choreographer.
* [SoloPi](https://github.com/alipay/SoloPi)：Soloπ是一个无线化、非侵入式的Android自动化工具，公测版拥有录制回放、性能测试、一机多控三项主要功能，能为测试开发人员节省宝贵时间。
* [AabResGuard](https://github.com/bytedance/AabResGuard)：The tool of obfuscated aab resources.(Android app bundle资源混淆工具
* [bytedance/tailor](https://github.com/bytedance/tailor): Tailor是西瓜Android团队开发的一款通用内存快照裁剪压缩工具，通过它可以在异常时直接dump出一个迷你内存快照。快照中没 有任何敏感信息，更重要的是文件非常小的同时数据也相对完整，非常适合离线分析OOM及其他类型异常的调查定位。
  * [Android Camera内存问题剖析](https://mp.weixin.qq.com/s/-oaN-bOqHDjN30UP1FMpgA)
  * [西瓜视频稳定性治理体系建设一：Tailor 原理及实践](https://mp.weixin.qq.com/s/DWOQ9MSTkKSCBFQjPswPIQ)
  * [西瓜视频稳定性治理体系建设二：Raphael 原理及实践](https://mp.weixin.qq.com/s/RF3m9_v5bYTYbwY-d1RloQ)

## 常见内存问题及解决方案

* [Android App解决卡顿慢之内存抖动及内存泄漏（发现和定位）](https://www.cnblogs.com/xgjblog/p/9042458.html)
* [性能优化1 - 内存抖动](https://myliupengcheng.github.io/2019/01/09/android-Performance-optimization-Memory-jitter-md/)
* [Android 内存优化一 内存抖动的定位及优化](https://blog.csdn.net/qiyei2009/article/details/89789585)
* [Android OOM案例分析](https://tech.meituan.com/2017/04/14/oom-analysis.html)
* [OOM分析之问题定位（一）](https://zhuanlan.zhihu.com/p/58919881)
* [OOM分析之ThreadPoolExecutor（二）](https://zhuanlan.zhihu.com/p/58920682)

## ANR

* [今日头条 ANR 优化实践系列 - 设计原理及影响因素 - 掘金 (juejin.cn)](https://juejin.cn/post/6940061649348853796)
* [今日头条 ANR 优化实践系列 - 监控工具与分析思路 - 掘金 (juejin.cn)](https://juejin.cn/post/6942665216781975582)
* [今日头条 ANR 优化实践系列分享 - 实例剖析集锦 - 掘金 (juejin.cn)](https://juejin.cn/post/6945267342671052807)
* [今日头条 ANR 优化实践系列 - Barrier 导致主线程假死 - 掘金 (juejin.cn)](https://juejin.cn/post/6947986170135445535)
* [今日头条 ANR 优化实践系列 - 告别 SharedPreference 等待 (qq.com)](https://mp.weixin.qq.com/s/kfF83UmsGM5w43rDCH544g)

## 启动优化

* [几乎包含了市面上所有启动优化方案](https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650829658&idx=1&sn=a228e6e9699f34ea8c7abc0f6d4a5f3a&chksm=80b7a7c4b7c02ed2cc0c45a4777b39b86c4930b579cba1276b05095292855f754497707908c0&scene=21#wechat_redirect)
* [抖音BoostMultiDex优化实践：Android低版本上APP首次启动时间减少80%（一）](https://mp.weixin.qq.com/s/jINCbTJ5qMaD6NdeGBHEwQ)
* [抖音BoostMultiDex优化实践：Android低版本上APP首次启动时间减少80%（二）](https://mp.weixin.qq.com/s/ILDTykAwR0xIkW-d1YzRHw)
* [你知道android的MessageQueue.IdleHandler吗？](https://wetest.qq.com/lab/view/352.html)
* [测试应用启动性能](https://mp.weixin.qq.com/s/ohKG7i5ttLh5hNH7fuKiUw)
* [Testing App Startup Performance](https://medium.com/androiddevelopers/testing-app-startup-performance-36169c27ee55)

## 布局及流畅度优化

* [如何监测Android应用卡顿？这篇就够了](https://zhuanlan.zhihu.com/p/90042663)
* [Android性能优化之布局优化实战](https://zhuanlan.zhihu.com/p/89488794)
* [测试界面性能](https://developer.android.com/training/testing/performance?hl=zh-cn)
* [Android流畅度之帧率](https://www.cnblogs.com/matric/p/6942281.html)
* [安卓帧率获取方法总结](https://blog.csdn.net/qq_42174669/article/details/107836614)
* [Android 两种实时获取FPS的方法](https://wizzie.top/Blog/2020/03/31/2020/200330_android_getFPS/)
* [fpsviewer—实时显示fps，监控Android卡顿的可视化工具](https://www.wanandroid.com/blog/show/2583)
* [Android App流畅度FPS测试方法总结](https://zhuanlan.zhihu.com/p/67056913)

## Bitmap优化

* [Android Bitmap优化](https://juejin.im/post/6844903919479422984)
* [Android性能优化（五）之细说Bitmap](https://juejin.im/post/6844903470030389255)
* [Android性能优化系列之Bitmap图片优化](https://blog.csdn.net/u012124438/article/details/66087785)
* [Bitmap优化详谈](https://www.jianshu.com/p/4c661166ff2a?tdsourcetag=s_pctim_aiomsg)
* [高效加载大型位图](https://developer.android.com/topic/performance/graphics/load-bitmap?hl=zh-cn)
* [Android性能优化之UI卡顿优化实例分析](https://juejin.cn/post/6870389004387385352)

## 线程优化

* [探索 Android 多线程优化方法](https://juejin.im/post/6844903909178212359)
* [【Bugly干货分享】Android性能优化典范之多线程篇](https://segmentfault.com/a/1190000005181461)
* [通过线程提升性能](https://developer.android.com/topic/performance/threads?hl=zh-cn)

## Apk包大小优化

* [Android App包瘦身优化实践](https://tech.meituan.com/2017/04/07/android-shrink-overall-solution.html)
* [APK包大小优化总结](https://felixzhang00.github.io/2020/03/24/APK%E5%8C%85%E5%A4%A7%E5%B0%8F%E4%BC%98%E5%8C%96%E6%80%BB%E7%BB%93/)
* [抖音包大小优化-资源优化](https://juejin.im/post/6844904106696376334)
* [抖音BoostMultiDex优化实践：Android低版本上APP首次启动时间减少80%（二）](https://mp.weixin.qq.com/s/ILDTykAwR0xIkW-d1YzRHw)
* [抖音BoostMultiDex优化实践：Android低版本上APP首次启动时间减少80%（一）](https://mp.weixin.qq.com/s/jINCbTJ5qMaD6NdeGBHEwQ)
* [开源 | AabResGuard: AAB 资源混淆工具](https://mp.weixin.qq.com/s/4hBhaS_0uwHsJdwUHk1ZOQ)

## LRUCache

* [彻底解析Android缓存机制——LruCache](https://www.jianshu.com/p/b49a111147ee)
* [彻底解析Android缓存机制——LruCache](https://www.jianshu.com/p/b49a111147ee)
* [Android DiskLruCache完全解析，硬盘缓存的最佳方案](https://yuanfentiank789.github.io/2016/06/13/disklrucache/)
* [DiskLruCache源码](https://github.com/JakeWharton/DiskLruCache)

## WebView优化

* [H5 缓存机制浅析 - 移动端 Web 加载性能优化](https://segmentfault.com/a/1190000004132566#articleHeader8)
* [Android H5秒开方案调研—今日头条H5秒开方案详解](https://yuweiguocn.github.io/android-h5/)
* [腾讯祭出大招VasSonic，让你的H5页面首屏秒开](https://segmentfault.com/a/1190000010711024)
* [Android WebView缓存机制和性能优化](https://juejin.im/post/6844903934004297736)

## 电量优化

* [Android电量优化全解析](https://juejin.im/post/6844903779268034574)
* [深入探索 Android 电量优化](https://juejin.im/post/6844904195523346439)
* [Android 电量测试以及电量优化](https://www.cnblogs.com/CharlesGrant/p/9284181.html)
* [针对低电耗模式和应用待机模式进行优化](https://developer.android.com/training/monitoring-device-state/doze-standby?hl=zh-cn)

## IO优化

* [IO优化是怎么做的，使用 SharedPreferences为什么这么卡，mmkv原理是什么](https://segmentfault.com/a/1190000022360556)

## 稳定性优化

* [深入探索Android稳定性优化 (juejin.cn)](https://juejin.cn/post/6844903972587716621)
* [深入探索Android启动速度优化](https://jsonchao.github.io/2019/11/10/深入探索Android启动速度优化/)
* [Android性能优化之内存优化](https://jsonchao.github.io/2019/08/18/Android性能优化之内存优化/)
* [深入探索Android内存优化](https://jsonchao.github.io/2019/12/29/深入探索Android内存优化/)
* [Android性能优化之绘制优化](https://juejin.im/post/5e5f090de51d4526e4190980)
* [深入探索Android布局优化（上）](https://juejin.im/post/5e1d15a851882536ca666a49)
* [深入探索Android布局优化（下）](https://juejin.im/post/5e1e6cf66fb9a0301828ca0a)
* [深入探索Android卡顿优化（上）](https://juejin.im/post/5e41fb7de51d4526c80e9108)
* [深入探索Android卡顿优化（下）](https://juejin.im/post/5e49fc29e51d4526d326b056)

