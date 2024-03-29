<h1 align="center">Android应用优化：界面优化</h1>



## 一 顿检测

我们可以利用BlockCanary去检查造成UI卡顿的地方，如下所示：

BlockCanary：https://github.com/markzhai/AndroidPerformanceMonitor

BlockCanary检查UI卡顿的原理如下图所示：

<img src="../../../art/practice/performance/blockcanary_structure.png"/>

## 二 卡顿优化

Android界面优化主要解决界面卡顿的问题，Android系统每隔16ms就会发送一个VSYNC信号，触发UI渲染，如果绘制操作超过了16ms，就会引起掉帧，也就是会导致姐们卡顿。

导致界面卡顿的原因主要是过度绘制，绘制了多余的UI，开发者选项里有检测过度绘制的工具，如下所示：

<img src="../../../art/practice/performance/overdraw_level.png" width="250"/>

1. 移除不必要的backgroud。
2. 自定义View的时候clipReact减少重叠区域的绘制。
3. 利用<merge>等标签减少View的层级。
4. 利用<ViewStub>在需要的时候再去加载View。