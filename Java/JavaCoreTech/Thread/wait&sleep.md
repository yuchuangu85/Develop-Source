<h1 align="center">wait sleep 区别</h1>

[TOC]

## wait&sleep区别

* sleep是线程中的方法，但是wait是Object中的方法。
* sleep方法不会释放lock，但是wait会释放，而且会加入到等待队列中。
* sleep方法不依赖于同步器synchronized，但是wait需要依赖synchronized关键字。
* sleep不需要被唤醒（休眠之后退出阻塞），但是wait需要（不指定时间需要被别人中断）。

## notify运行过程

当线程A（消费者）调用wait()方法后，线程A让出锁，自己进入等待状态，同时加入锁对象的等待队列。 线程B（生产者）获取锁后，调用notify方法通知锁对象的等待队列，使得线程A从等待队列进入阻塞队列。 线程A进入阻塞队列后，直至线程B释放锁后，线程A竞争得到锁继续从wait()方法后执行。

