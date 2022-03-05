<h1 align="center">Handler中的Looper无限循环，为什么没有阻塞UI主线程？</h1>

[toc]

## 1.循环问题

作者：Gityuan  链接：https://www.zhihu.com/question/34652589/answer/90344494

**(1) Android中为什么主线程不会因为Looper.loop()里的死循环卡死？** 

这里涉及线程，先说说进程/线程，**进程：**每个app运行时前首先创建一个进程，该进程是由Zygote fork出来的，用于承载App上运行的各种Activity/Service等组件。进程对于上层应用来说是完全透明的，这也是google有意为之，让App程序都是运行在Android Runtime。大多数情况一个App就运行在一个进程中，除非在AndroidManifest.xml中配置Android:process属性，或通过native代码fork进程。

**线程：**线程对应用来说非常常见，比如每次new Thread().start都会创建一个新的线程。该线程与App所在进程之间资源共享，从Linux角度来说进程与线程除了是否共享资源外，并没有本质的区别，都是一个task_struct结构体**，在CPU看来进程或线程无非就是一段可执行的代码，CPU采用CFS调度算法，保证每个task都尽可能公平的享有CPU时间片**。

有了这个准备，再说说死循环问题：

对于线程既然是一段可执行的代码，当可执行代码执行完成后，线程生命周期便该终止了，线程退出。而对于主线程，我们是绝不希望会被运行一段时间，自己就退出，那么如何保证能一直存活呢？**简单做法就是可执行代码是能一直执行下去的，死循环便能保证不会被退出，**例如，binder线程也是采用死循环的方法，通过循环方式不同与Binder驱动进行读写操作，当然并非简单地死循环，无消息时会休眠。但这里可能又引发了另一个问题，既然是死循环又如何去处理其他事务呢？通过创建新线程的方式。

真正会卡死主线程的操作是在回调方法onCreate/onStart/onResume等操作时间过长，会导致掉帧，甚至发生ANR，looper.loop本身不会导致应用卡死。


**(2) 没看见哪里有相关代码为这个死循环准备了一个新线程去运转？** 

事实上，会在进入死循环之前便创建了新binder线程，在代码ActivityThread.main()中：

```java
public static void main(String[] args) {
        ....

        //创建Looper和MessageQueue对象，用于处理主线程的消息
        Looper.prepareMainLooper();

        //创建ActivityThread对象
        ActivityThread thread = new ActivityThread(); 

        //建立Binder通道 (创建新线程)
        thread.attach(false);

        Looper.loop(); //消息循环运行
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

**thread.attach(false)；便会创建一个Binder线程（具体是指ApplicationThread，Binder的服务端，用于接收系统服务AMS发送来的事件），该Binder线程通过Handler将Message发送给主线程**，具体过程可查看 [startService流程分析](https://link.zhihu.com/?target=http%3A//gityuan.com/2016/03/06/start-service/)，这里不展开说，简单说Binder用于进程间通信，采用C/S架构。关于binder感兴趣的朋友，可查看我回答的另一个知乎问题：
[为什么Android要采用Binder作为IPC机制？ - Gityuan的回答](https://www.zhihu.com/question/39440766/answer/89210950)

另外，**ActivityThread实际上并非线程**，不像HandlerThread类，ActivityThread并没有真正继承Thread类，只是往往运行在主线程，给人以线程的感觉，其实承载ActivityThread的主线程就是由Zygote fork而创建的进程。

**主线程的死循环一直运行是不是特别消耗CPU资源呢？** 其实不然，这里就涉及到**Linux pipe/epoll机制**，简单说就是在主线程的MessageQueue没有消息时，便阻塞在loop的queue.next()中的nativePollOnce()方法里，详情见[Android消息机制1-Handler(Java层)](https://link.zhihu.com/?target=http%3A//www.yuanhh.com/2015/12/26/handler-message-framework/%23next)，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作。这里采用的epoll机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。 **所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。**

**(3) Activity的生命周期是怎么实现在死循环体外能够执行起来的？**

ActivityThread的内部类H继承于Handler，通过handler消息机制，简单说Handler机制用于同一个进程的线程间通信。

**Activity的生命周期都是依靠主线程的Looper.loop，当收到不同Message时则采用相应措施：**
在H.handleMessage(msg)方法中，根据接收到不同的msg，执行相应的生命周期。

​    比如收到msg=H.LAUNCH_ACTIVITY，则调用ActivityThread.handleLaunchActivity()方法，最终会通过反射机制，创建Activity实例，然后再执行Activity.onCreate()等方法；
​    再比如收到msg=H.PAUSE_ACTIVITY，则调用ActivityThread.handlePauseActivity()方法，最终会执行Activity.onPause()等方法。 上述过程，我只挑核心逻辑讲，真正该过程远比这复杂。

**主线程的消息又是哪来的呢？**当然是App进程中的其他线程通过Handler发送给主线程，请看接下来的内容：

\-----------------------------------------------------
**最后，从进程与线程间通信的角度，通过一张图加深大家对App运行过程的理解：**

<img src="../media/15851976928352.jpg" style="zoom:67%;" />

**system_server进程是系统进程**，java framework框架的核心载体，里面运行了大量的系统服务，比如这里提供ApplicationThreadProxy（简称ATP），ActivityManagerService（简称AMS），这个两个服务都运行在system_server进程的不同线程中，由于ATP和AMS都是基于IBinder接口，都是binder线程，binder线程的创建与销毁都是由binder驱动来决定的。

**App进程则是我们常说的应用程序**，主线程主要负责Activity/Service等组件的生命周期以及UI相关操作都运行在这个线程； 另外，每个App进程中至少会有两个binder线程 ApplicationThread(简称AT)和ActivityManagerProxy（简称AMP），除了图中画的线程，其中还有很多线程，比如signal catcher线程等，这里就不一一列举。

Binder用于不同进程之间通信，由一个进程的Binder客户端向另一个进程的服务端发送事务，比如图中线程2向线程4发送事务；而handler用于同一个进程中不同线程的通信，比如图中线程4向主线程发送消息。

**结合图说说Activity生命周期，比如暂停Activity，流程如下：**

1. 线程1的AMS中调用线程2的ATP；（由于同一个进程的线程间资源共享，可以相互直接调用，但需要注意多线程并发问题）
2. 线程2通过binder传输到App进程的线程4；
3. 线程4通过handler消息机制，将暂停Activity的消息发送给主线程；
4. 主线程在looper.loop()中循环遍历消息，当收到暂停Activity的消息时，便将消息分发给ActivityThread.H.handleMessage()方法，再经过方法的调用，最后便会调用到Activity.onPause()，当onPause()处理完后，继续循环loop下去。

## 2.Looper休眠和唤醒问题

作者：Luckie stone  链接：https://blog.csdn.net/suyimin2010/article/details/82469548

C++类Looper中的睡眠和唤醒机制是通过pollOnce和wake函数提供的，它们又是利用操作系统（Linux内核）的epoll机制来完成的。当被监控的文件（通过epoll_ctl的EPOLL_CTL_ADD添加进去）可I/O时，epoll_wait调用会从睡眠中醒来，这时，可以检查是哪个（或哪些）文件描述符对应的文件可以进行I/O读写了，从而做出进一步处理。使用者利用它们就可以拥有睡眠等待和唤醒机制。下面详述。

在Looper的构造函数中，会创建一个管道（下面的行73），然后调用epoll_create获取一个epoll的实例的描述符（行88），最后将管道读端描述符作为一个事件报告项添加给epoll（行95）。这样，当管道读端有数据可读时，将会得到报告。Looper的构造函数如下（见文件Looper.cpp）:

![](../media/15851977915931.jpg)


Looper的pollOnce函数将最终调用到其pollInner函数。在后者里面，将调用epoll_wait睡眠等待其监控的文件描述符是否有可I/O事件的到来，若有（哪怕只有一个），epoll_wait将会醒来，然后可检查是哪个文件描述符上的可I/O事件。pollInner函数中的相关代码如下（见文件Looper.cpp）:

![](../media/15851978256465.jpg)

![](../media/15851978365246.jpg)

可见，在线程循环中调用了Looper的pollOnce函数，将导致睡眠等待在上面的行218处的epoll_wait上。当向消息队列发送消息并进行唤醒时，行218将被唤醒，因此从pollOnce函数中返回，可以从消息队列中取出消息进行处理。

![](../media/15851978501232.jpg)


Looper的wake函数用于向管道中写入字符（下面的行367），以唤醒pollOnce：


下面来看一下Java层的MessageQueue如何利用这种机制。

前面提到在android.os.MessageQueue的next函数中取出下一个消息时，会调用到native层实现的函数nativePollOnce时，实际调用到了如下native实现（见文件android_os_MessageQueue.cpp）：

![](../media/15851978614762.jpg)

上面行157的pollOnce函数代码是（见文件android_os_MessageQueue.cpp）：

![](../media/15851978729735.jpg)

这样，它们就通过Looper的pollOnce实现了在Looper中的管道上的读端上的睡眠等待。

当android.os.MessageQueue的enqueueMessage函数往队列上添加了一个新消息或removeSyncBarrier移除了同步屏障后，可能需要调用nativeWake唤醒，其native实现为：（见文件android_os_MessageQueue.cpp）：

![](../media/15851978816875.jpg)

上面的行162调用的又是下面的函数，代码如下（见文件android_os_MessageQueue.cpp）：

![](../media/15851978916375.jpg)

这样，Looper将向管道写端写入字符，唤醒其在管道读端上的睡眠等待。

因此，通过借助于Looper的wake和pollOnce函数，可以让别的消息队列（如Java层的消息队列）拥有睡眠唤醒机制：没有消息时pollOnce调用者将睡眠等待，有消息时让wake函数去唤醒睡眠等待。



