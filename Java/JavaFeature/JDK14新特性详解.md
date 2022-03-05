<h1 align="center">JDK14新特性详解</h1>

   预览版：该功能在当前版本可以使用，如果效果不是很好的话，可能以后的其他版本就会删去该功能。

   最终版：该功能在之前版本效果很好，之后的每个版本中都会存在该功能。



#### 1、Switch（最终版）

和之前的jdk12、13功能一样，只不过确定下来为最终版

```
int numLetters = switch (day) {     
        case MONDAY, FRIDAY, SUNDAY -> 6;   
        case TUESDAY -> 7;     
        case THURSDAY, SATURDAY -> 8;
        case WEDNESDAY -> 9;
    };
```



#### 2、垃圾回收器（更新优化）

```
1、Windows的ZGC：现在可以在Windows上作为实验功能使用，要启用它，请使用JVM标志-XX:+UnlockExperimentalVMOptions 
-XX:+UseZGC。
2、Mac的ZGC：现在可作为macOS上的实验功能使用。要启用它，请使用JVM标志-XX:+UnlockExperimentalVMOptions -XX:+UseZGC。
3、并行GC的改进：并行GC已采用与其他收集器相同的任务管理机制来调度并行任务。这可能会显着提高性能。由于这一变化，以下产品标志
已过时：-XX:BindGCTaskThreadsToCPUs，-XX:UseGCTaskAffinity，和-XX:GCTaskTimeStampEntries。    
4、G1 NUMA感知内存分配：现在尝试跨垃圾收集在年轻一代的同一NUMA节点上分配并保留对象。这类似于并行GC NUMA意识。G1尝试使用
严格的交错在所有可用的NUMA节点上均匀分配Humongous和Old区域。从年轻一代复制到老一代的对象的放置是随机的。这些新的NUMA感知
内存分配试探法通过使用-XX：+UseNUNMA命令行选项自动启用。
```



#### 3、Record（预览功能）

```
@Data
@AllArgsConstructor
class Group {
    // 组名
    private String name;
    // 人数
    private int nums;
}
```

使用它可以替代构造器、equal方法、toString方法，hashCode方法

```
Point（String name,int nums）{}
```

  Java语言中一种新型的类型声明。像枚举一样enum， record是类的受限形式。它声明其表示形式，并提交与该表示形式匹配的API。记录放弃了类通常享有的自由：将API与表示分离的能力。作为回报，记录获得了很大程度的简洁性。



#### 4、**货币格式（优化）**

```
    可以通过 NumberFormat.getCurrencyInstance(Locale)使用“ u-cf-account” Unicode区域设置扩展名来获得具有记帐样式的
货币格式实例，其中金额在某些区域设置中用括号表示，例如，Locale.US,它将格式化为($3.27)而不是-$3.27。
```

   而之前的版本是前边结果为负数。

![img](media/up-e8d039b0e3b3f8718e72f532a15b038c528.png)



#### 5、NIO的Channel通道

```
    阐明ReadableByteChannel.read()的规范和规格DatagramChannel.receive(),FileChannel.read(ByteBuffer,long),Read
ableByteChannel.read(),ScatteringByteChannel.read()方法已经在此版本已经更新到指定的IllegalArgumentException,如果
（任何）缓冲区参数（S）是只读的抛出。
```



#### 6、删除功能

```
1、CMS垃圾收集器已被删除。-XX:UseConcMarkSweepGC和别名-Xconcgc，-Xnoconcgc以及所有CMS特定选项（太多，无法列出）都已废弃。

2、删除了安全库java.security.acl API
```



#### 7、**instanceof的模式匹配（预览版）**

提供模式匹配来 增强Java编程语言instanceof

```
if (obj instanceof String s) { 
    // can use s here 
    } else {
     // can't use s here 
}
```



#### **8、弃用****功能**

**线程：**

```
    不建议使用线程挂起、删除，下面的方法中涉及的线程挂起Thread,并且Thread已在本版本中晚期弃用,Thread.suspend(),Thread.
resume(),ThreadGroup.suspend(),ThreadGroup.resume(),ThreadGroup.allowThreadSuspension(boolean)这些方法将在
将来的版本中删除。
```

**垃圾回收器：**

```
    弃用ParallelScavenge + SerialOld GC组合，任何UseParallelOldGC用于启用此垃圾回收算法组合的命令行选项的使用，都会引
起弃用警告。嵌入式替换是通过-XX:+UseParallelGC在命令行上使用ParallelScavenge + ParallelOld垃圾收集器。
```

**椭圆曲线:**

```
security-libs / javax.crypto,已过时的旧椭圆曲线去除。
```



#### 9、注意点

**线程中断状态始终可用：**

```
    该规范java.lang.Thread::interrupt允许实现仅跟踪活动线程的中断状态，并且以前就是这种情况。从此版本开始，a的中断状态
Thread始终可用，并且如果您在线程t启动之前或终止之后中断线程，查询t.isInterrupted()将返回true。
```

 **DatagramSocket.send和MulticastSocket.send抛出IllegalArgumentException当套接字没有连接和数据包不包含地址：**

```
如果套接字未连接且没有套接字地址,send则由DatagramSocket和定义的方法MulticastSocket已更改为抛出。
```

**MulticastSocket getOption（IP_MULTICAST_IF）未设置传出接口时返回null：**

```
该MulticastSocket方法getOption已更改为符合中描述的行为StanndardSocketOptions.IP_MULTCAST_IF。如果没有设置接口，
MulticastSocket.getOption(StanndardSocketOptions.IP_MULTCAST_IF)现在返回null。
```

**MulticastSocket上getOption /的SetOption为IP_MULTICAST_LOOP个符合随着StandardSocketOptions.IP_MULTICAST_LOOP规范的行为：**

```
该MulticastSocket方法getOption和setOption已更改以符合所描述的行为StandardSocketOptions.IP_MULTICAST_LOOP规范,
MulticastSocket.getOption(StanndardSocketOptions.IP_MULTCAST_IF)现在，如果启用了环回模式，则返回true。
设置MulticastSocket.getOption(StanndardSocketOptions.IP_MULTCAST_IF)启用回送模式。
```

source https://my.oschina.net/mdxlcj/blog/3197478