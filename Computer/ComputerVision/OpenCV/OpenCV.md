<h1 align="center">OpenCV</h1>

[TOC]

## 一、官方资源：

* [https://opencv.org/platforms/](https://opencv.org/platforms/)：平台
* [https://opencv.org/opencv-3-4-1.html](https://opencv.org/opencv-3-4-1.html)：版本以及对应平台SDK
* [https://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/O4A_SDK.html](https://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/O4A_SDK.html)：Android平台导入教程
* [https://jaist.dl.sourceforge.net/project/opencvlibrary/opencv-android/3.4.1/opencv-3.4.1-android-sdk.zip ](https://jaist.dl.sourceforge.net/project/opencvlibrary/opencv-android/3.4.1/opencv-3.4.1-android-sdk.zip )：sdk下载 
* [https://opencv.org/releases.html](https://opencv.org/releases.html)：下载列表：
* [https://docs.opencv.org/2.4/index.html](https://docs.opencv.org/2.4/index.html)：详细文档

## 二、项目实战：

检测硬件平台命令：
```
adb shell getprop ro.product.cpu.abi
```

NDK编译jni：
打开到项目文件夹，执行命令：
```
ndk-build
```
平台选择：（与.so有关的长年大坑：https://zhuanlan.zhihu.com/p/21359984  ）


## 三、参考文章：
* [https://www.jianshu.com/p/57925243c028](https://www.jianshu.com/p/57925243c028)
* [https://blog.csdn.net/qq_28931623/article/details/73528102](https://blog.csdn.net/qq_28931623/article/details/73528102)
* [https://blog.csdn.net/yanzi1225627/article/details/38098729/](https://blog.csdn.net/yanzi1225627/article/details/38098729/)
* [https://www.jianshu.com/p/3a9f6921e3d7](https://www.jianshu.com/p/3a9f6921e3d7)
* 

## 四、参考博客
* [https://www.cnblogs.com/skyfsm/category/1000207.html](https://www.cnblogs.com/skyfsm/category/1000207.html)