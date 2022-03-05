<h1 align="center">Android跨进程通信：图文详解 Binder机制 原理</h1>

[toc]

## 前言

- 如果你接触过 跨进程通信 （`IPC`），那么你对`Binder`一定不陌生
- 虽然 网上有很多介绍 `Binder`的文章，可是存在一些问题：**浅显的讨论`Binder`机制 或 一味讲解 `Binder`源码、逻辑不清楚，最终导致的是读者们还是无法形成一个完整的`Binder`概念**
- 本文采用清晰的图文讲解方式，按照大角度 -> 小角度去分析Binder，即：
   1. 先从 **机制、模型的角度** 去分析 整个`Binder`跨进程通信机制的模型
   2. 再 从**源码实现角度**，分析 `Binder`在 `Android`中的具体实现

从而全方位地介绍 `Binder`，希望你们会喜欢。

> 请尽量在PC端而不要在移动端看，否则图片可能看不清。

## 目录

![目录](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1jOWVhMTE2MTFiZjhhODYzLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

## 1. Binder到底是什么？

- 中文即 粘合剂，意思为粘合了两个不同的进程
- 网上有很多对`Binder`的定义，但都说不清楚：`Binder`是跨进程通信方式、它实现了`IBinder`接口，是连接 `ServiceManager`的桥梁blabla，估计大家都看晕了，没法很好的理解
- 我认为：对于`Binder`的定义，在不同场景下其定义不同

![定义](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS00NWRiNGRmMzM5MzQ4YjliLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

在本文的讲解中，按照 **大角度 -> 小角度** 去分析`Binder`，即：

- 先从 **机制、模型的角度** 去分析 整个`Binder`跨进程通信机制的模型

> 其中，会详细分析模型组成中的 `Binder`驱动

- 再 从源码实现角度，分析 `Binder`在 `Android`中的具体实现

从而全方位地介绍 `Binder`，希望你们会喜欢。

## 2. 知识储备

在讲解`Binder`前，我们先了解一些`Linux`的基础知识

### 2.1 进程空间划分

- 一个进程空间分为 用户空间 & 内核空间（`Kernel`），即把进程内 用户 & 内核 隔离开来
- 二者区别：
   1. 进程间，用户空间的数据不可共享，所以用户空间 = 不可共享空间
   2. 进程间，内核空间的数据可共享，所以内核空间 = 可共享空间

> 所有进程共用1个内核空间

- 进程内 用户空间 & 内核空间 进行交互 需通过 **系统调用**，主要通过函数：

> 1. copy_from_user（）：将用户空间的数据拷贝到内核空间
> 2. copy_to_user（）：将内核空间的数据拷贝到用户空间

![示意图](media/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDQzNjUtMTNkNTkwNThkNGUwY2JhMS5wbmc_aW1hZ2VNb2dyMi9hdXRvLW9yaWVudC9zdHJpcCU3Q2ltYWdlVmlldzIvMi93LzEyNDA.png)

### 2.2 进程隔离 & 跨进程通信（ IPC ）

- 进程隔离
   为了保证 安全性 & 独立性，一个进程 不能直接操作或者访问另一个进程，即`Android`的进程是**相互独立、隔离的**
- 跨进程通信（ `IPC` ）
   即进程间需进行数据交互、通信
- 跨进程通信的基本原理

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0xMjkzNTY4NGU4ZWMxMDdjLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

> a. 而`Binder`的作用则是：连接 两个进程，实现了mmap()系统调用，主要负责 创建数据接收的缓存空间 & 管理数据接收缓存
> b. 注：传统的跨进程通信需拷贝数据2次，但`Binder`机制只需1次，主要是使用到了内存映射，具体下面会详细说明

### 2.5 内存映射

具体请看文章：[操作系统：图文详解 内存映射](https://www.jianshu.com/p/719fc4758813)

## 3. Binder 跨进程通信机制 模型

### 3.1 模型原理图

`Binder` 跨进程通信机制 模型 基于 `Client - Server` 模式
![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1jMTBkNjAzMmY5MWExMDNmLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

### 3.2 模型组成角色说明

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0xMzU1NjBjODdjOTgzZTQzLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

此处重点讲解 `Binder`驱动的作用 & 原理：

- 简介

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS04MmQ2YTA2NThlNTVlOWQ3LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

- 跨进程通信的核心原理

> 关于其核心原理：内存映射，具体请看文章：[操作系统：图文详解 内存映射](https://www.jianshu.com/p/719fc4758813)

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS02NWE1YjE3NDI2YWVkNDI0LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

### 3.3 模型原理步骤说明

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1kM2M3OGIxOTNjM2U4YTM4LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

### 3.4 额外说明

##### 说明1：`Client`进程、`Server`进程 & `Service Manager` 进程之间的交互 都必须通过`Binder`驱动（使用 `open` 和 `ioctl`文件操作函数），而非直接交互

原因：

1. `Client`进程、`Server`进程 & `Service Manager`进程属于进程空间的用户空间，不可进行进程间交互
2. `Binder`驱动 属于 进程空间的 内核空间，可进行进程间 & 进程内交互

所以，原理图可表示为以下：

> 虚线表示并非直接交互

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1iNDcwMDhhMDkyNjViOWM2LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

##### 说明2： `Binder`驱动 & `Service Manager`进程 属于 `Android`基础架构（即系统已经实现好了）；而`Client` 进程 和 `Server` 进程 属于`Android`应用层（需要开发者自己实现）

所以，在进行跨进程通信时，开发者只需自定义`Client` & `Server` 进程 并 显式使用上述3个步骤，最终借助 `Android`的基本架构功能就可完成进程间通信

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0xNjMwYzY5ZTQ4Y2IxZGViLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

##### 说明3：Binder请求的线程管理

- `Server`进程会创建很多线程来处理`Binder`请求
- `Binder`模型的线程管理 采用`Binder`驱动的线程池，并由`Binder`驱动自身进行管理

> 而不是由`Server`进程来管理的

- 一个进程的`Binder`线程数默认最大是16，超过的请求会被阻塞等待空闲的Binder线程。

> 所以，在进程间通信时处理并发问题时，如使用`ContentProvider`时，它的`CRUD`（创建、检索、更新和删除）方法只能同时有16个线程同时工作

- 至此，我相信大家对`Binder` 跨进程通信机制 模型 已经有了一个非常清晰的定性认识
- 下面，我将通过一个实例，分析`Binder`跨进程通信机制 模型在 `Android`中的具体代码实现方式

> 即分析 上述步骤在`Android`中具体是用代码如何实现的

## 4. Binder机制 在Android中的具体实现原理

- `Binder`机制在 `Android`中的实现主要依靠 `Binder`类，其实现了`IBinder` 接口

> 下面会详细说明

- 实例说明：`Client`进程 需要调用 `Server`进程的加法函数（将整数a和b相加）

> 即：
>
> 1. `Client`进程 需要传两个整数给 `Server`进程
> 2. `Server`进程 需要把相加后的结果 返回给`Client`进程

- 具体步骤
   下面，我会根据`Binder` 跨进程通信机制 模型的步骤进行分析

### 步骤1：注册服务

- 过程描述
   `Server`进程 通过`Binder`驱动 向 `Service Manager`进程 注册服务
- 代码实现
   `Server`进程 创建 一个 `Binder` 对象

> 1. `Binder` 实体是 `Server`进程 在 `Binder` 驱动中的存在形式
> 2. 该对象保存 `Server` 和 `ServiceManager` 的信息（保存在内核空间中）
> 3. `Binder` 驱动通过 内核空间的`Binder` 实体 找到用户空间的`Server`对象

- 代码分析

```java
    Binder binder = new Stub();
    // 步骤1：创建Binder对象 ->>分析1

    // 步骤2：创建 IInterface 接口类 的匿名类
    // 创建前，需要预先定义 继承了IInterface 接口的接口 -->分析3
    IInterface plus = new IPlus(){

          // 确定Client进程需要调用的方法
          public int add(int a,int b) {
               return a+b;
         }

          // 实现IInterface接口中唯一的方法
          public IBinder asBinder（）{ 
                return null ;
           }
};
          // 步骤3
          binder.attachInterface(plus，"add two int");
         // 1. 将（add two int，plus）作为（key,value）对存入到Binder对象中的一个Map<String,IInterface>对象中
         // 2. 之后，Binder对象 可根据add two int通过queryLocalIInterface（）获得对应IInterface对象（即plus）的引用，可依靠该引用完成对请求方法的调用
        // 分析完毕，跳出


<-- 分析1：Stub类 -->
    public class Stub extends Binder {
    // 继承自Binder类 ->>分析2

          // 复写onTransact（）
          @Override
          boolean onTransact(int code, Parcel data, Parcel reply, int flags){
          // 具体逻辑等到步骤3再具体讲解，此处先跳过
          switch (code) { 
                case Stub.add： { 

                       data.enforceInterface("add two int"); 

                       int  arg0  = data.readInt();
                       int  arg1  = data.readInt();

                       int  result = this.queryLocalIInterface("add two int") .add( arg0,  arg1); 

                        reply.writeInt(result); 

                        return true; 
                  }
           } 
      return super.onTransact(code, data, reply, flags); 

}
// 回到上面的步骤1，继续看步骤2

<-- 分析2：Binder 类 -->
 public class Binder implement IBinder{
    // Binder机制在Android中的实现主要依靠的是Binder类，其实现了IBinder接口
    // IBinder接口：定义了远程操作对象的基本接口，代表了一种跨进程传输的能力
    // 系统会为每个实现了IBinder接口的对象提供跨进程传输能力
    // 即Binder类对象具备了跨进程传输的能力

        void attachInterface(IInterface plus, String descriptor)；
        // 作用：
          // 1. 将（descriptor，plus）作为（key,value）对存入到Binder对象中的一个Map<String,IInterface>对象中
          // 2. 之后，Binder对象 可根据descriptor通过queryLocalIInterface（）获得对应IInterface对象（即plus）的引用，可依靠该引用完成对请求方法的调用

        IInterface queryLocalInterface(Stringdescriptor) ；
        // 作用：根据 参数 descriptor 查找相应的IInterface对象（即plus引用）

        boolean onTransact(int code, Parcel data, Parcel reply, int flags)；
        // 定义：继承自IBinder接口的
        // 作用：执行Client进程所请求的目标方法（子类需要复写）
        // 参数说明：
        // code：Client进程请求方法标识符。即Server进程根据该标识确定所请求的目标方法
        // data：目标方法的参数。（Client进程传进来的，此处就是整数a和b）
        // reply：目标方法执行后的结果（返回给Client进程）
         // 注：运行在Server进程的Binder线程池中；当Client进程发起远程请求时，远程请求会要求系统底层执行回调该方法

        final class BinderProxy implements IBinder {
         // 即Server进程创建的Binder对象的代理对象类
         // 该类属于Binder的内部类
        }
        // 回到分析1原处
}

<-- 分析3：IInterface接口实现类 -->
public interface IPlus extends IInterface {
          // 继承自IInterface接口->>分析4
          // 定义需要实现的接口方法，即Client进程需要调用的方法
         public int add(int a,int b);
// 返回步骤2
}

<-- 分析4：IInterface接口类 -->
// 进程间通信定义的通用接口
// 通过定义接口，然后再服务端实现接口、客户端调用接口，就可实现跨进程通信。
public interface IInterface
{
    // 只有一个方法：返回当前接口关联的 Binder 对象。
    public IBinder asBinder();
}
// 回到分析3原处
```

**注册服务后，`Binder`驱动持有 `Server`进程创建的`Binder`实体**

### 步骤2：获取服务

- `Client`进程 使用 某个 `service`前（此处是 **相加函数**），须 通过`Binder`驱动 向 `ServiceManager`进程 获取相应的`Service`信息
- 具体代码实现过程如下：

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS05YTJjN2I5ZTU5NDMzMmFlLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

**此时，`Client`进程与 `Server`进程已经建立了连接**

### 步骤3：使用服务

`Client`进程 根据获取到的 `Service`信息（`Binder`代理对象），通过`Binder`驱动 建立与 该`Service`所在`Server`进程通信的链路，并开始使用服务

- 过程描述
   1. `Client`进程 将参数（整数a和b）发送到`Server`进程
   2. `Server`进程 根据`Client`进程要求调用 目标方法（即加法函数）
   3. `Server`进程 将目标方法的结果（即加法后的结果）返回给`Client`进程
- 代码实现过程

**步骤1： `Client`进程 将参数（整数a和b）发送到`Server`进程**

```java
// 1. Client进程 将需要传送的数据写入到Parcel对象中
// data = 数据 = 目标方法的参数（Client进程传进来的，此处就是整数a和b） + IInterface接口对象的标识符descriptor
  android.os.Parcel data = android.os.Parcel.obtain();
  data.writeInt(a); 
  data.writeInt(b); 

  data.writeInterfaceToken("add two int");；
  // 方法对象标识符让Server进程在Binder对象中根据"add two int"通过queryLocalIInterface（）查找相应的IInterface对象（即Server创建的plus），Client进程需要调用的相加方法就在该对象中

  android.os.Parcel reply = android.os.Parcel.obtain();
  // reply：目标方法执行后的结果（此处是相加后的结果）

// 2. 通过 调用代理对象的transact（） 将 上述数据发送到Binder驱动
  binderproxy.transact(Stub.add, data, reply, 0)
  // 参数说明：
    // 1. Stub.add：目标方法的标识符（Client进程 和 Server进程 自身约定，可为任意）
    // 2. data ：上述的Parcel对象
    // 3. reply：返回结果
    // 0：可不管

// 注：在发送数据后，Client进程的该线程会暂时被挂起
// 所以，若Server进程执行的耗时操作，请不要使用主线程，以防止ANR

// 3. Binder驱动根据 代理对象 找到对应的真身Binder对象所在的Server 进程（系统自动执行）
// 4. Binder驱动把 数据 发送到Server 进程中，并通知Server 进程执行解包（系统自动执行）

```

**步骤2：`Server`进程根据`Client`进要求 调用 目标方法（即加法函数）**

```java
// 1. 收到Binder驱动通知后，Server 进程通过回调Binder对象onTransact（）进行数据解包 & 调用目标方法
  public class Stub extends Binder {

          // 复写onTransact（）
          @Override
          boolean onTransact(int code, Parcel data, Parcel reply, int flags){
          // code即在transact（）中约定的目标方法的标识符

          switch (code) { 
                case Stub.add： { 
                  // a. 解包Parcel中的数据
                       data.enforceInterface("add two int"); 
                        // a1. 解析目标方法对象的标识符

                       int  arg0  = data.readInt();
                       int  arg1  = data.readInt();
                       // a2. 获得目标方法的参数
                      
                       // b. 根据"add two int"通过queryLocalIInterface（）获取相应的IInterface对象（即Server创建的plus）的引用，通过该对象引用调用方法
                       int  result = this.queryLocalIInterface("add two int") .add( arg0,  arg1); 
                      
                        // c. 将计算结果写入到reply
                        reply.writeInt(result); 
                        
                        return true; 
                  }
           } 
      return super.onTransact(code, data, reply, flags); 
      // 2. 将结算结果返回 到Binder驱动

```

**步骤3：`Server`进程 将目标方法的结果（即加法后的结果）返回给`Client`进程**

```java
  // 1. Binder驱动根据 代理对象 沿原路 将结果返回 并通知Client进程获取返回结果
  // 2. 通过代理对象 接收结果（之前被挂起的线程被唤醒）

    binderproxy.transact(Stub.ADD, data, reply, 0)；
    reply.readException();；
    result = reply.readInt()；
          }
}
```

- 总结
   下面，我用一个原理图 & 流程图来总结步骤3的内容

![原理图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS02MmZkYWI5MDVlN2QyNzA2LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

![流程图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0yZjUzMGU5NjRmZmFiOGQ3LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

## 5. 优点

对比 `Linux` （`Android`基于`Linux`）上的其他进程通信方式（管道、消息队列、共享内存、
信号量、`Socket`），`Binder` 机制的优点有：
![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1jMzIxMTYxYmZlYTdlNzhkLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

## 6. 总结

- 本文主要详细讲解 跨进程通信模型 `Binder`机制 ，总结如下：

![定义](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS00NWRiNGRmMzM5MzQ4YjliLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

特别地，对于从模型结构组成的Binder驱动来说：

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS04MmQ2YTA2NThlNTVlOWQ3LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

- 整个`Binder`模型的原理步骤 & 源码分析

![示意图](media/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1kNDY4MDdjMDA4OWE3ZjQ0LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA.png)

- 下面我将继续对 `Android`中的知识进行讲解 ，感兴趣的同学可以继续关注**carson_ho的微信公众号**

## 来源：

https://blog.csdn.net/carson_ho/article/details/73560642