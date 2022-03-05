<h1 align="center">Java基础知识整理</h1>

[toc]

## 一、基础篇

### JVM

* JVM内存模型和结构
* GC原理，性能调优
* 调优：Thread Dump，分析内存结构
* class二进制字节码结构，class loader体系，class加载过程，实例创建过程，双亲委派（破坏双亲委派）
* 方法执行过程
* 堆、栈、方法区、直接内存、堆和栈区别
   Java内存模型
* 内存可见性、重排序、顺序一致性、volatile、锁、final
   垃圾回收
* 内存分配策略、垃圾收集器（G1）、GC算法、GC参数、对象存活的判定 
   JVM参数及调优Java对象模型



### Java运行

* javac编译java文件为class文件
* java命令的使用，带package的java类如何在命令行中启动
* java程序涉及到的各个路径（classpath，java、library path，java运行的主目录等）



### 对象和实例

​	**Class和Instance的概念**

​	Instance的创建过程：

* 无继承：分配内存空间，初始化变量，调用构造函数
* 有继承：处理静态动作，分配内存空间，变量定义为初始值，从基类->子类，处理定义处的初始化，执行构造函数

### 访问控制

* public，projected，default，private对于class，method，field的修饰作用

### 流程控制

* if，switch，loop，for，while

### 面向对象编程概念

* 面向对象三大特性：封装，继承，多态
* 各自的定义概念，有哪些特性体现出来，各自的使用场景
* 静态多分派，动态单分派的概念
* 重载的概念和使用
* 继承：接口多实现，基类单继承
* 抽象，抽象类，接口
* 多态：方法覆盖的概念和使用
* 接口回调

### static，abstract，final，finally，finalize

* 静态属性的定义，使用，以及类加载时如何初始化
* 静态方法的定义和作用
* 静态类的定义和使用
* 静态代码块的定义和初始化时机

### 阅读源代码

* String、Integer、Long、Enum、BigDecimal、ThreadLocal、ClassLoader & URLClassLoader、ArrayList & LinkedList、 HashMap & LinkedHashMap & TreeMap & CouncurrentHashMap、HashSet & LinkedHashSet & TreeSet
   Java中各种变量类型熟悉Java String的使用，熟悉String的各种函数

* JDK 6和JDK 7中substring的原理及区别、replaceFirst、replaceAll、replace区别、String对“+”的重载、String.valueOf和Integer.toString的区别、字符串的不可变性自动拆装箱

* Integer的缓存机制
   熟悉Java中各种关键字

* transient、instanceof、volatile、synchronized、final、static、const 原理及用法。

* Object类型：equals，hashcode

   


### 集合类

* 常用集合类的使用
   ArrayList和LinkedList和Vector的区别，SynchronizedList和Vector的区别
   ，HashMap、HashTable、ConcurrentHashMap区别，Java 8中stream相关用法，不同版本的JDK中HashMap的实现的区别以及原因
   枚举
* 集合框架体系：Collection，Map

### 异常

* 异常类型：Throwable，Exception，RuntimeException，Error
* RuntimeException和一般Exception的区别和具体处理方法
* 异常是否可以跨线程抓取
* 正确处理异常、自定义异常

### 枚举

* 枚举的用法
* 枚举与单例
* Enum类

### Java IO

* InputStream，OutputStream，Reader/Writer，文件读取，各种流的读取以及各使用场景
* bio、nio和aio的区别、三种IO的用法与原理、netty
   Java反射与javassist

### 反射

* 反射与工厂模式、 java.lang.reflect.*

### Java序列化

* 什么是序列化与反序列化、为什么序列化
* 序列化底层原理
* 序列化与单例模式
* protobuf
* 为什么说序列化并不安全
   注解

### 注解

* 元注解、自定义注解、Java中常用注解使用、注解与反射的结合
   JMS

### 泛型

* 泛型与继承
* 类型擦除
* 泛型中K，T，V，E， Object等的含义、泛型各种用法
* [java 泛型详解-绝对是对泛型方法讲解最详细的，没有之一](https://blog.csdn.net/s10461/article/details/53941091)

### 语法糖

* Java中语法糖原理、解语法糖

### Java并发编程

#### 1. 线程

* 什么是线程，与进程的区别阅读源代码，并学会使用
* Thread、Runnable、Callable、ReentrantLock、ReentrantReadWriteLock、Atomic*、Semaphore、CountDownLatch、、ConcurrentHashMap、Executors
   线程池
* 自己设计线程池、submit() 和 execute()
* [Java中的多线程你只要看这一篇就够了](https://www.cnblogs.com/wxd0108/p/5479442.html)

#### 2. 线程安全

* 死锁、死锁如何排查、Java线程调度、线程安全和内存模型的关系
   锁
* CAS、乐观锁与悲观锁、数据库相关锁机制、分布式锁、偏向锁、轻量级锁、重量级锁、monitor、锁优化、锁消除、锁粗化、自旋锁、可重入锁、阻塞锁、死锁
   死锁volatile

#### 3. synchronized

* synchronized是如何实现的？
* synchronized和lock之间关系
* 不使用synchronized如何实现一个线程安全的单例
* sleep 和 waitwait 和 notifynotify 和 notifyAllThreadLocal写一个死锁的程序写代码来解决生产者消费者问题守护线程
* 守护线程和非守护线程的区别以及用法

#### 4. 多线程的实现和启动

#### 5. callable和runnable的区别

#### 6. synchronized，reentrantLock各自特点和对比

#### 7. 线程池

#### 8. future异步方式获取执行结果

#### 9. concurrent包

#### 10. lock



## 二、 进阶篇

Java底层知识
字节码、class文件格式CPU缓存，L1，L2，L3和伪共享尾递归位运算

用位运算实现加、减、乘、除、取余
设计模式
了解23种设计模式会使用常用设计模式

单例、策略、工厂、适配器、责任链。
实现AOP实现IOC不用synchronized和lock，实现线程安全的单例模式nio和reactor设计模式

网络编程
tcp、udp、http、https等常用协议

三次握手与四次关闭、流量控制和拥塞控制、OSI七层模型、tcp粘包与拆包
http/1.0 http/1.1 http/2之前的区别Java RMI，Socket，HttpClientcookie 与 session



## 三、 高级篇

### 1. 新技术

* Java 8：lambda表达式、Stream API、
* Java 9：Jigsaw、Jshell、Reactive Streams
* Java 10：局部变量类型推断、G1的并行Full GC、ThreadLocal握手机制

### 2. 响应式编程

### 3. 性能优化

* 使用单例、使用Future模式、使用线程池、选择就绪、减少上下文切换、减少锁粒度、数据压缩、结果缓存

### 4. 线上问题分析

* dump获取，线程Dump、内存Dump、gc情况
   dump分析
* 分析死锁、分析内存泄露
   自己编写各种outofmemory，stackoverflow程序
* HeapOutOfMemory、 Young OutOfMemory、MethodArea OutOfMemory、ConstantPool OutOfMemory、DirectMemory OutOfMemory、Stack OutOfMemory Stack OverFlow
   常见问题解决思路

### 5. 内存问题

* 内存溢出、线程死锁、类加载冲突

### 6. 数据结构与算法知识

* 简单的数据结构
* 栈、队列、链表、数组、哈希表、
   树
* 二叉树、字典树、平衡树、排序树、B树、B+树、R树、多路树、红黑树
   排序算法
* 各种排序算法和时间复杂度 深度优先和广度优先搜索 全排列、贪心算法、KMP算法、hash算法、海量数据处理

 

## 四、Java优化

* [Effective Java for Android](/Java/Effect/EffectiveJava4Android.md)
* [Java 性能优化：35 个小细节，让你提升 Java 代码的运行效率](/Java/Effect/EffectJava35.md)

## 五、Java工具

* [jadx](https://github.com/skylot/jadx)
* [反编译工具-dex2jar](https://github.com/pxb1988/dex2jar)
* [easyexcel](https://github.com/alibaba/easyexcel)：快速、简单避免OOM的java处理Excel工具





