<h1 align="center">Android组件管理框架：Android后台服务Service</h1>

Content Provider用于提供数据统一访问的格式，封装底层的具体实现，对于调用者而言，无需失效数据的来源，例如：数据库、文件或者
网络，只需要使用Content Provider提供的接口就可以进行数据的增删改查操作。

Content Provider作为Android四大组件之一，没有复杂的生命周期，只有简单的onCreate()过程。底层基于Binder实现，可以用来实现
跨进程数据交互与共享。

