<h1 align="center">Android窗口管理框架篇：Android窗口管理框架概述</h1>

<img src="../../../art/app/ui/android_ui_system.png"/>

从上图可以看出，Android的显示系统分为3层：

- UI框架层：负责管理窗口中View组件的布局与绘制以及响应用户输入事件
- WindowManagerService层：负责管理窗口Surface的布局与次序
- SurfaceFlinger层：将WindowManagerService管理的窗口按照一定的次序显示在屏幕上

在Android显示框架里有这么几个角色：

- Activity：应用视图的容器。
- Window：应用窗口的抽象表示，它的实际表现是View。
- View：实际显示的应用视图。
- WindowManagerService：用来创建、管理和销毁Window。

后续的分析思路是这样的，我们先分析最上层的View，然后依次是Window、WindowManagerService。这样可以由浅入深，便于理解。至于Activity我们会放在Android组件框架里分析。
