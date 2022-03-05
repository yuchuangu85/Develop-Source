<h1 align="center">Android启动</h1>

[toc]

## App启动流程

1. Launcher startActivity
2. AMS startActivity
3. Zygote fork进程
4. Activity main()
5. ActivityThread 进程loop循环
6. 开启Activity,开始生命周期回调…

## Activity启动流程

1. Activity startActivity\startActivityForResult
2. Instrumentation execStartActivity
3. AMS startActivity
4. ApplicationThread scheduleLaunchActivity
5. ActivityThread.H handleMessage -> performLaunchActivity
6. Activity attach
7. Instrumentation callActivityOnCreate

## Activity 启动

1. Activity1调用startActivity，实际会调用Instrumentation类的execStartActivity方法，Instrumentation是系统用来监控Activity运行的一个类，Activity的整个生命周期都有它的影子。（1- 4）

2. 通过跨进程的binder调用，进入到ActivityManagerService中，其内部会处理Activity栈，通知Activity1 Pause，Activity1 执行Pause 后告知AMS。（5 - 29）

3. 在ActivityManagerService中的startProcessLocked中调用了Process.start()方法。并通过连接调用Zygote的native方法forkAndSpecialize，执行fork任务。之后再通过跨进程调用进入到Activity2所在的**进程**中。（30 - 36）

4. ApplicationThread是一个binder对象，其运行在binder线程池中，内部包含一个H类，该类继承于类Handler。主线程发起bind Application，AMS 会做一些配置工作，然后让主线程 bind ApplicationThread，ApplicationThread将启动Activity2的信息通过H对象发送给**主线程**。发送的消息是EXECUTE_TRANSACTION，消息体是一个 ClientTransaction，即 LaunchActivityItem。主线程拿到Activity2的信息后，调用Instrumentation类的newActivity方法，其内通过ClassLoader创建Activity2**实例**。（37 - 40）

5. 通知Activity2去performCreate。（41 - 最后）

注：现在发送的都是EXECUTE_TRANSACTION ，通过 TransactionExecutor 来执行 ClientTransaction, ClientTransaction 中包含各种 ClientTransactionItem，如 PauseActivityItem、LaunchActivityItem、StopActivityItem、ResumeActivityItem、DestroyActivityItem 等，这些Item的execute方法来处理相应的handle，如handlePauseActivity、handleLaunchActivity等，通知相应的Activity来perform。

## 冷启动与热启动是什么，区别，如何优化，使用场景等

**app冷启动**： 当应用启动时，后台没有该应用的进程，这时系统会重新创建一个新的进程分配给该应用， 这个启动方式就叫做冷启动（后台不存在该应用进程）。冷启动因为系统会重新创建一个新的进程分配给它，所以会先创建和初始化Application类，再创建和初始化`MainActivity`类（包括一系列的测量、布局、绘制），最后显示在界面上。

**app热启动**： 当应用已经被打开， 但是被按下返回键、Home键等按键时回到桌面或者是其他程序的时候，再重新打开该app时， 这个方式叫做热启动（后台已经存在该应用进程）。热启动因为会从已有的进程中来启动，所以热启动就不会走Application这步了，而是直接走`MainActivity`（包括一系列的测量、布局、绘制），所以热启动的过程只需要创建和初始化一个`MainActivity`就行了，而不必创建和初始化`Application`

**冷启动的流程**

当点击app的启动图标时，安卓系统会从Zygote进程中fork创建出一个新的进程分配给该应用，之后会依次创建和初始化Application类、创建`MainActivity`类、加载主题样式Theme中的`windowBackground`等属性设置给`MainActivity`以及配置Activity层级上的一些属性、再inflate布局、当`onCreate`/`onStart`/`onResume`方法都走完了后最后才进行`contentView`的`measure`/`layout`/`draw`显示在界面上冷启动的生命周期简要流程：

Application构造方法 –> `attachBaseContext()`–>`onCreate` –>Activity构造方法 –> `onCreate()` –> 配置主体中的背景等操作 –>`onStart()` –> `onResume()` –> 测量、布局、绘制显示

冷启动的优化主要是视觉上的优化，解决白屏问题，提高用户体验，所以通过上面冷启动的过程。能做的优化如下：

> 1、减少onCreate()方法的工作量2、不要让Application参与业务的操作3、不要在Application进行耗时操作4、不要以静态变量的方式在Application保存数据5、减少布局的复杂度和层级6、减少主线程耗时

### 冷启动流程：

①点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；

②system_server进程接收到请求后，向zygote进程发送创建进程的请求；

③Zygote进程fork出新的子进程，即App进程；

④App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；

⑤system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；

⑥App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；

⑦主线程在收到Message后，通过发射机制创建目标Activity，并回调Activity.onCreate()等方法。

⑧到此，App便正式启动，开始进入Activity生命周期，执行完onCreate/onStart/onResume方法，UI渲染结束后便可以看到App的主界面。

