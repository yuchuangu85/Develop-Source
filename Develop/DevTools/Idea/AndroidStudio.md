<h1 align="center">Android Studio常见用法、设置、和问题</h1>

[toc]



## android studio的右侧gradle只有dependens，没有tasks任务栏

如图：

![img](media/webp-20210904224137827)


 出现该问题，是因为升级了 `android studio` 到 `4.2.1`，默认关闭了，打开即可

![img](media/webp-20210904224202011)


**转载自：**
[android studio的右侧gradle只有dependens，没有tasks任务栏](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Ffeijian_%2Farticle%2Fdetails%2F117573009)



如果：右侧的`gradle` 面板中没有内容，显示“Nothing to show”,可以尝试一下 `android studio` 的 `file` 菜单下的 `Sync Project with Gradle Files`

![img](media/webp)

