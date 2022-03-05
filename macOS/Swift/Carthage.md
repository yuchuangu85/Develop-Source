<h1 align="center">Carthage操作指南</h1>

[TOC]

## 安装步骤：

```
// 安装Carthage
$ brew install carthage

// 查看版本命令
$ carthage version
```

## 使用Carthage
```
// 进入项目目录
$ cd /Users/cdmac/Desktop/Demos/DemoX8

// 创建一个空的carthage文件
$ touch Cartfile

// 使用xcode打开cartfile文件
$ open -a Xcode Cartfile

// 加入以下内容
github "Alamofire/Alamofire" ~> 4.0
 
github "SwiftyJSON/SwiftyJSON"

```

版本说明：
* ~> 3.0 表示使用版本3.0以上但是低于4.0的最新版本，如3.5, 3.9
* == 3.0 表示使用3.0版本
* >= 3.0表示使用3.0或更高的版本
如果你没有指明版本号，则会自动使用最新的版本

```
// 保存并关闭cart file文件，在终端执行命令
$ carthage update --platform iOS
```

carthage会为你下载和编译所需要的第三方库，当命令执行完毕，在你的项目文件夹中会创建一个名为Carthage的文件夹

在 /Users/cdmac/Desktop/Demos/DemoX8/Carthage/Build/iOS 里会出现xxx.framework文件已经为你创建好了。

当然，你也可以通过命令行进入此文件夹：

```
$ open Carthage
```

现在打开你的项目，点击project，选择target, 再选择上方的General，将需要的framework文件拖到 Linked frameworks and Binaries内

点击Build Phrase tab选项，添加相应的run script