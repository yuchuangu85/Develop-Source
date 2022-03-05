<h1 align="center">Android架构师之路</h1>

[toc]

## 架构师必备技能

### 深入Java泛型

* 泛型的作用与定义
* 通配符与嵌套
* RxJava中泛型的使用分析
* Json解析泛型与Bean强转原理实践

### 注解深入浅出

* 自定义注解
  * 自定义注解和元注解
  * 注解参数和默认值
* 注解的使用
  * APT，编译注解处理器
  * 插桩，编译后处理筛选
  * 反射，运行时动态获取注解信息
* Retrofit中的注解原理
* 自定义注解实现ButterKnife项目架构

### 并发编程

* 线程共享和协作
  * CPU核心数，线程数，时间片轮转机制解读
  * synchronized、lock、volatile、ThreadLocal如何实现线程共享
  * Wait，Notify/NotifyAll，Join方法如何实现线程间协作
* 站在巨人肩上操作CAS
  * CAS的原理
  * CAS带来的ABA问题，
    * 原子操作类的正确使用实践
* 仅会用线程池是不够的
   * Callbale、Future和FutureTask源码解读
   * 线程池底层实现分析
   * 线程池排队机制
   * 手写线程池实战
   * Executor框架解读实战
* Android AnsyncTask原理解析

### 数据传输与序列化

* Serializable原理
* Parcelable接口原理解析
* Json

### Java虚拟机原理

* 垃圾回收机制
   * 对象存活及强、弱等各引用辨析
   * 快速解读GC算法之标记-清除、复制及标记-整理算法
   * 正确解读GC日志
* 内存分配策略
   * JVM栈桢及方法调用详解
   * JMM、Java Memory Model
* Dalvik虚拟机

### 反射与类加载

* 反射的基本概念与Class
   * 三种获取Class对象的方式
   * 获取构造器实例化对象与属性信息
   * 包信息和方法
   * Hook技术动态编程
* ClassLoader类加载器
   * 动态代理模式
   * Android Davilk与ART
   * PathClassLoader、DexClassLoader与BootClassLoader
   * 双亲委托机制
* 热修复类加载项目实战

### 动态代理

* 动态代理的基本原理
* 动态代理值Rxjava和Retrofit实战

### 高效IO

* Java IO体系
   * 装饰者模式
   * InputStream与OutputStream
   * Reader与Writer
* File文件操作
   * FileChannel
   * 内存映射
* IO操作Dex文件加密，APK加固项目实战

### 实战项目：QZone超级布丁

* ClassLoader
* 反射
* 注解
* 动态代理
* FileIO

### Kotlin实战

* Kotlin版本的WanAndroid项目
   * Kotlin基础语法（与Java不同的地方）
   * Kotlin的类和对象
   * Kotlin的集合
   * 高阶函数与Lambda
   * Kotlin泛型与注解
   * Kotlin协程框架基础
* Kotlin与Java互调原理
* Kotlin协程原理分析
* Kotlin语法糖总结

## Android高级UI与Framework源码

### 高级UI晋升

* 自定义流式布局（ViewGroup）
   * Android的UI基础
      * 坐标系，角度（弧度），颜色
   * View与ViewGroup绘制原理解析
      * 绘制流程
      * MeasureSpec
      * LayoutParams
   * 事件分发机制
* 灵动的锦鲤自定义View项目实战
   * Canvas与Paint基础
   * Canvas绘制点、线、面与几何图形
   * Canvas画布与图层
   * Path，PathMeasure，Matrix与贝塞尔曲线
* 今日头条文字渐变特效项目
   * 文字绘制
   * 视图动画与属性动画
* 自定义PhotoView事件分发项目
   * MotionEvent与多点触控
   * 手势（GestureDetector ScaleGestureDetector）
* RecyclerView实现吸顶效果项目
   * 自定义ItemDecoration与LayoutManager
   * ViewHolder与回收复用机制
   * 布局管理器LayoutManager
   * RecyclerView原理解析
* 自定义Banner高级项目
   * ViewPager
      * 加载机制与优化
      * 与Fragment的结合
   * ViewPager2原理解析
* 布局ViewGroup原理解析
   * ConstraintLayout
   * LinearLayout
   * RelativeLayout
   * FrameLayout
   * GridLayout
* Material Design设计App
   * Material Design常见控件的使用
      * Toolbar
      * FloatingActionButton
      * Snackbar
      * CoordinatorLayout
         * CoordinatorLayout原理解析
         * 自定义Behavior
      * AppbarLayout
      * NavigationView，BottomNavigationView，BottomSheet
      * DrawLayout
      * CardView
      * CollapsingToolbarLayout
   * NestScrollView原理解析
   * 自定义TabLayout
* WebView交互架构项目
   * 使用与原理
   * JS与Java交互
   * 多进程WebView使用实践
   * WebView与Native的通信框架

### Framework内核解析

* Binder
   * Linux预备知识
      * 进程隔离
      * 进程空间划分-用户空间、内核空间
      * 系统调用-用户态、内核态
      * Linux的IPC机制
         * 共享内存
         * Socket/管道/消息队列/信号量
         * Binder
   * 传统Linux中IPC通信原理
      * copy_from_user/copy_to_user
   * Binder IPC通信原理
      * Binder跨进程通信原理
         * 动态内核加载模块
         * 内存映射mmap原理解析
         * Binder IPC实现原理
      * Binder通信模型
         * Client/Server/ServiceManager/驱动
      * Binder Driver浅析
         * Binder线程池
      * 启动ServerManager
      * 获取ServerManager
      * 注册服务（addService）
      * 获取服务（getService）
      * Binder通信中的代理模式
   * Binder Java层实现
      * IBinder/IInterface/Binder/Stub
      * AIDL使用以及原理分析
   * Binder相关面试总结
      * 为什么Android采用Binder作为IPC机制
      * Binder到底是什么
      * Binder机制是如何跨进程的
      * 一次Binder通信的基本流程是什么
      * 为什么Activity之间对象需要序列化
      * 四大组件底层的通信机制是怎么样的
      * AIDL内部的实现原理是什么
* Handler消息机制
   * Linux的epoll机制
   * 一个线程几个Handler
   * 一个线程几个Looper？如何保证
   * 子线程可以创建Handler吗
   * 多个Handler往MessageQueue中添加数据，内部是如何确保线程安全的
   * looper.loop()为什么不会阻塞主线程
   * Message的数据结构是什么样子
   * Handler内存泄漏的场景有哪些，如何避免
   * IntentService源码解析
   * HandlerThread源码解析
* Dalvik VM进程系统
   * Zygote基础
   * 分析Zygote的启动过程
   * 启动SystemServer进程
      * 启动各种系统服务
   * 应用程序进程
      * 创建应用程序
      * 启动线程池
      * 创建消息循环
* 增量更新项目实战
   * dex文件结构
      * dex文件结构
      * dex文件加载基本原理
      * dex文件差分包生成
* 插件化实战
   * AMS的启动过程
   * AMS重要的数据结构解析
      * ActivityRecord
      * TaskRecord
      * ActivityStack
   * Activity栈管理
      * Activity任务栈模型
      * Launch Mode
      * Intent的Flag与taskAffinity
   * Activity启动流程
      * Hook实现启动未注册Activity
   * Activity管理
      * Activity运行机制
      * adj内存管理机制
      * Activity内核管理方案详细讲解
* 深入理解WMS
   * Window的创建过程
   * Dialog与Toast源码解析
   * 剖析Activity，View，Window之间的关系
   * 从WMS的角度分析Activity启动流程
   * WMS
      * Window的添加过程
      * Window的删除过程
* PackageManager Service
   * app的安装流程分析
* 网易插件化换肤
   * LayoutInflater加载布局分析
   * Android资源的加载机制
   * Resource与AssetManager

### Android组件内核

* Activity与调用栈
   * 四大启动模式与Intent Flag
   * APK启动流程与ActivityThread解析
   * Activity生命周期源码解析
   * 实战Splash广告载入与延时跳转
* Fragment的管理与内核
   * Fragment事物管理机制
   * Fragment转场动画
   * 嵌套后处理，ChildFragmentManager
* Service内核原理
   * start与bind区别与原理
   * 自带工作线程的IntentService
   * 前台服务与Notify
* 组件间通信方案
   * Activity和Fragment低耦合通信设计
   * Android与Service通信
   * Intent数据传输与限制
   * ViewModel通信方案
   * 事件总线EventBus源码解析
   * 实战：自动感知生命周期事件总线LiveDataBus

### 数据持久化

* Android文件系统
   * sdcard与内部存储
* 轻量级kv持久化
   * Shared Preference原理
   * 微信MMKV原理与实现
      * MMAP内存映射
      * 文件数据结构
      * 增量更新与全量更新
* 嵌入式Sqlite数据库
   * SqliteOpenHelper
   * Sqlite升级与数据迁移方案
   * 实战注解ORM数据库框架

## 360度全方面性能调优

### 设计思想与代码质量优化

* 六大原则
   * 单一职责原则
   * 开闭原则
   * 里世替换原则
   * 依赖倒置原则
   * 接口隔离原则
   * 迪米特法则
* 设计模式
   * 结构型模式
      * 桥接模式
      * 适配器模式
      * 装饰者模式
      * 代理模式
      * 外观（门面）模式
      * 组合模式
   * 创建型模式
      * 建造者模式
      * 单例模式
      * 抽象工厂模式
      * 工厂方法模式
      * 静态工厂模式
   * 行为型模式
      * 模板方法模式
      * 策略模式
      * 观察者模式
      * 责任链模式
      * 命令模式
      * 访问者模式
   * 实战设计模式结构项目网络框架
* 数据结构
   * 线性表ArrayList
   * 链表LinkedList
   * 栈Stack
   * 队列
      * Queue
      * Deque
      * 阻塞队列
   * Tree
      * 平衡二叉树
      * 红黑树
   * 映射表
      * HashTable
      * HashMap
      * SparseArray
      * ArrayMap
* 算法
   * 排序算法
      * 冒泡排序
      * 选择排序
      * 插入排序
      * 快速排序
      * 堆排序
      * 基数排序
   * 查找算法
      * 折半查找
      * 二分查找
      * 树形查找
      * hash查找

### 程序性能优化

* OOM问题原理解析
   * adj内存管理机制
   * JVM内存回收机制与GC算法解析
   * 生命周期相关问题总结
   * Bitmap压缩方案总结
* ANR问题解析
   * AMS系统时间调节原理
   * 程序等待原理分析
   * ANR问题解决方案
* Crash监控方案
   * Java层监控方案
   * Native层监控方案
* 启动速度与执行效率优化
   * 冷、热启动耗时检测与分析
   * 启动黑白屏解决
   * 卡顿分析
   * StickMode严苛模式
   * Systrace与TraceView工具
* 布局检测与优化
   * 布局层级优化
   * 过度渲染检测
   * Hierarchy Viewer与Layout Inspactor工具
* 内存优化
   * 内存抖动和内存泄漏
   * 内存大户，Bitmap内存优化
   * Profile内存监测工具
   * Mat大对象与泄漏检测
* 耗电优化
   * Doze&Standby
   * Battery Historian
   * JobScheduler，WorkManager
* 网络传输与数据存储优化
   * google序列化工具protobuf
   * 7z极限压缩
   * 使用webp图片
* APK大小优化
   * APK瘦身
   * 微信资源混淆原理
* 屏幕适配
   * 屏幕适配方案总结
   * hook技术实现屏幕完全适配

### 开发效率 优化

* 分布式版本控制系统Git
* 自动化构建系统Gradle
   * Groovy基础
      * Groovy开发环境搭建
      * Groovy变量与字符串
      * Groovy接口与闭包
      * 常用数据结构
      * 面向对象
      * Json与Xml解析
      * 文件操作
   * Gradle与Android插件
      * Gradle是什么
      * Gradle生命周期探索
      * Project与Task
   * Transform API
      * Transform执行机制与配置
      * 字节码增强技术
      * 修改无源码第三方sdk代码（Bug）
   * 自定义插件开发
      * build Script脚本
      * buildSrc目录
      * 独立项目开发插件
      * 上传本地仓库与jcenter仓库
      * Artifactory私服仓库搭建
   * 插件实战
      * 自动化加固插件
      * AOP编程字节码插桩
      * 多渠道打包
      * 发版自动钉钉

## 解读开源框架设计思想

### 热修复设计

* AOT/JIT、dexopt与dex2oat
* CLASS_ISPREVERIFIED问题与解决
* 及时生效与重启生效热修复原理
* Gradle自动补丁包生成
* 手写热修复架构
* 热修复藐视总结

### 插件化框架解读

* Class文件加载Dex原理
* Android资源加载与管理
* so库的加载原理
* Android系统服务的运行原理
* 手写插件化框架
* 面试总结

### 组件化框架设计

* 组件化之集中式路由-ARouter原理
* APT技术自动生成代码与动态类加载
* Java SPI机制实现组件服务调用
* 拦截器AOP编程（跳转前预处理--登录），路由参数传递与IOC注入
* 手写组件化式路由
* 面试总结

### 图片加载框架

* 图片加载框架选型
   * Universal ImageLoader
   * Glide
   * Picasso
   * Fresco
* Glide原理分析
   * Fragment感知生命周期原理
   * 自动图片大小计算
   * 图片解码
   * 优先级请求队列
   * ModelLoader与Registry机制
   * 内存缓存原理
      * LRU内存缓存
      * 引用计数器与弱引用活跃缓存
      * Bitmap复用池
      * 缓存大小配置
   * 磁盘文件缓存
      * 原始图像文件缓存
      * 解码图像文件缓存
* 手写图片加载框架实战

### 网络访问框架设计

* 网络通信必备基础
   * Restful URL
   * HTTP协议&TCP/IP协议
   * SSL握手与加密
   * DNS解析
   * Socket通信原则
      * SOCKS代理
      * HTTP普通代理与隧道代理
* Okhttp源码解读
   * Socket连接池复用机制
   * HTTP协议重定向与缓存处理
   * 高并发请求队列：任务分发
   * 责任链模式拦截器设计
* Retrofit源码解读
* 手写高性能网络通讯框架
* BAT网络面试模拟

### RxJava响应式编程框架设计

* 链式调用
* 扩展的观察者模式
* 事件变换设计
* Scheduler线程控制

### IOC架构设计

* 依赖注入与控制反转
* ButterKnife原理
* Dagger架构设计核心解密

### Android架构组件Jetpack

* LiveData原理
* Navigation如何解决tabLayout问题
* ViewModel如何感知View的生命周期及内核原理
* Room架构方式方法
* dataBinding为什么能够支持MVVM
* WorkManager内核揭秘
* Lifecycles生命周期

## NDK模块开发

### NDK基础知识体系

* C与C++
   * 数据类型
   * 内存结构与管理
   * 预处理指令、Typedef别名
   * 结构体与共用体
   * 指针、智能指针、方法指针
   * 线程
   * 类
      * 函数、虚函数、纯虚函数与析构函数
      * 初始化列表
* JNI开发
   * 静态与动态注册
   * 方法签名、与Java通信
   * 本地引用全局引用
* Native开发工具
   * 编译器、打包工具与分析器
   * 静态库与动态库
   * CPU架构与注意事项
   * 构建脚本与构建工具
      * Cmake
      * Makefile
   * 交叉编译移植
      * FFmpeg交叉编译
      * X264、FAAC交叉编译
      * 解决所有移植问题
   * AS构建NDK项目
* Linux编程
   * Linux环境搭建，系统管理，权限系统和工具使用（Vim等）
   * Shell脚本编程

### 底层图片处理

* PNG、JPEG、WEBP图像处理与压缩
* 微信图片压缩
* 源码都在用的giflib：GIF合成原理与实现

### 音视频开发

* 直播app（用户端与主播端）
   * 多媒体系统
      * Camera与手机屏幕采集
      * CameraX摄像头数据提取
      * 图像原始数据格式YUV420（NV21与YV12等）
      * 音频采集和播放系统
      * 编解码器MediaCodec
      * MediaMuxer复用与MediaExtractor
   * FFmpeg
      * ffmpeg模块介绍
      * 音视频解码，音视频同步
      * I帧，B帧，P帧解码原理
      * x264视频编码与faac音频编码
      * NativeWindow绘制
   * 流媒体协议
      * RTMP协议
      * 音视频通话P2P WebRtc
* OpenCV
   * 图像预处理
      * 灰度化、二值花
      * 模糊、高斯模糊
      * 图像形态学操作：腐蚀、膨胀与开闭操作
      * 轮廓查找
   * 人脸识别
      * haar模型
      * lbp特征提取
      * 不止是人脸：物体检测模型训练
   * 身份证识别
   * 车牌号识别
      * 车牌定位：sobel因子与hsv颜色模型定位
      * SVM分类候选车牌平分
      * 人工神经网络训练与车牌字符识别
* 抖音视频app
   * OpenGL与EGL
   * Android OpenGL ES OES扩展纹理处理摄像头数据
   * OpenGL ES FBO帧数据缓存
   * Seetaface2人脸关键点定位
   * 视觉效果处理：美颜大眼
   * 视觉效果处理：双分屏，三分屏，四分屏等
   * 滤镜层责任链设计
   * 音视频变速原理

## 架构师炼成实战

### 架构设计

* MVC、MVP与MVVM
* 模块化与组件化架构

### Gradle自动化项目实战



## Flutter

### Flutter基础

* Flutter环境配置
* Flutter编码语言Dart详解系列
   * 一切皆对象，Dart面向对象的原理解析
   * Dart中变量，函数，操作符，异常等语法与Java原理对比
   * 类的机制
   * 初始化列表规则
   * 命名构造方法
   * 常量构造方式
   * 工厂构造特征
   * Mixin
* Flutter框架原理与使用技巧
   * widget控件详解：text，image，button
   * 布局分析：Linear布局，弹性布局，流水布局
   * 自定义View
   * 动画、手势交互
   * 多线程开发原理
   * 网络请求原理
   * Flutter架构与原生代码交互
   * 实战发布自己的Flutter库

### Flutter进阶

* Flutter Framework架构浅析
   * Flutter启动引擎
   * TaskRunner工作原理
   * Dart虚拟机
   * Widget架构
* Flutter应用启动分析
* Flutter的Platform Channel机制
* Flutter异步Future机制
* Flutter的Isolate创建过程
* Flutter渲染机制
* setState更新截止
* Flutter动画原理 