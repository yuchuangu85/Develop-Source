<h1 align="center">AIDL中的in、out、inout、oneway</h1>

[TOC]

## in、out、inout

AIDL中的定向 tag 表示了在跨进程通信中数据的流向，其中：

* in 表示数据只能由客户端流向服务端， 
* out 表示数据只能由服务端流向客户端，
* inout 则表示数据可在服务端与客户端之间双向流通。

其中，数据流向是针对在客户端中的那个传入方法的对象而言的。

* in 为定向 tag 的话表现为服务端将会接收到一个那个对象的完整数据，但是客户端的那个对象不会因为服务端对传参的修改而发生变动；
* out 的话表现为服务端将会接收到那个对象的的空对象，但是在服务端对接收到的空对象有任何修改之后客户端将会同步变动；
* inout 为定向 tag 的情况下，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。

Java 中的基本类型和 String ，CharSequence 的定向 tag 默认且只能是 in



## oneway







## 参考

[你真的理解AIDL中的in，out，inout么？ - 简书 (jianshu.com)](https://www.jianshu.com/p/ddbb40c7a251)

[Android-你真的懂AIDL的oneway嘛？_嵌入式Linux-CSDN博客](https://blog.csdn.net/weiqifa0/article/details/104284978)

