<h1 align="center">Android界面开发：View自定义实践布局篇</h1>

- [01Android界面开发：View自定义实践概览](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android应用开发实践篇/Android界面开发/01Android界面开发：View自定义实践概览.md)
- [02Android界面开发：View自定义实践布局篇](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android应用开发实践篇/Android界面开发/02Android界面开发：View自定义实践布局篇.md)
- [03Android界面开发：View自定义实践绘制篇](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android应用开发实践篇/Android界面开发/03Android界面开发：View自定义实践绘制篇.md)
- [04Android界面开发：View自定义实践交互篇](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android应用开发实践篇/Android界面开发/04Android界面开发：View自定义实践交互篇.md)

在文章[02Android显示框架：Android应用视图的载体View](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android系统应用框架篇/Android显示框架/02Android显示框架：Android应用视图载体View.md)中我们理解了
View的测量、布局、绘制、触摸事件处理等内容，今天我们开始我们View自定义实践的内容。

View自定义是开发中最常见的需求，图表等各种复杂的ui以及产品经理各种奇怪的需求😤都要通过View自定义来完成。

View自定义有三个关键点：

- 布局：决定View的摆放位置
- 绘制：决定View的具体内容
- 交互：决定View与用户的交互体验

这篇文章我们就来分析关于View自定义的布局问题。要想彻底掌握View自定义的布局，就要理解View的布局实现，这个我们在前面的文章分析过源码，这里再来整体总结一下。



