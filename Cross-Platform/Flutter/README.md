<h1 align="center">Flutter</h1>

[toc]

## Flutter官网及资源

* [Flutter 官网](https://flutter.dev/)
* [flutter源码](https://github.com/flutter/flutter)

## 优秀文章

* [总结了30个例子之后，我悟到了Flutter的布局原理](https://juejin.cn/post/6914155427651387399)
* [花两天时间做了15个例子的解析，彻底掌握Flutter的布局原理](https://juejin.cn/post/6916157915590033421)
* [【译】Flutter | 深入理解布局约束](https://juejin.cn/post/6846687593745088526)
* [美团外卖Flutter动态化实践](https://tech.meituan.com/2020/06/23/meituan-flutter-flap.html)

## 视频教程

* [Flutter Widget of the Week - YouTube](https://www.youtube.com/playlist?list=PLjxrf2q8roU23XGwz3Km7sQZFTdB996iG)

* [Flutter Tutorial for Beginners - YouTube](https://www.youtube.com/playlist?list=PL4cUxeGkcC9jLYyp2Aoh6hcWuxFDX6PBJ)

* [Flutter & Firebase App Build - YouTube](https://www.youtube.com/playlist?list=PL4cUxeGkcC9j--TKIdkb3ISfRbJeJYQwC)

  

## 环境配置

1. 下载代码：

```
git clone https://github.com/flutter/flutter.git
或者
git clone https://github.com/flutter/flutter.git -b stable --depth 1
```

2. 配置环境变量

```
export PATH="$PATH:`pwd`/flutter/bin"
```

3. 中国地区

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

4. 预加载依赖

```
flutter precache
```

5. 检测是否完成

```
flutter doctor
flutter doctor -v // 显示详细的信息
```

6. 查看路径

```
which flutter
```

7. 创建运行项目

```
flutter create my_app
cd my_app
flutter run
flutter run -v // 显示详细信息
```

8. iOS setup

```
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch
```

9. 启动iOS虚拟机

```
open -a Simulator
```

10. 部署到iOS设备

```
 sudo gem install cocoapods
 pod setup
```



## Flutter镜像

### 官方提供的国内镜像--推荐

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

### 上海交通大学提供的国内镜像

```
export FLUTTER_STORAGE_BASE_URL=https://mirrors.sjtug.sjtu.edu.cn/
expot PUB_HOSTED_URL=https://dart-pub.mirrors.sjtug.sjtu.edu.cn/
```

### 清华大学 TUNA 协会

```
export PUB_HOSTED_URL=https://mirrors.tuna.tsinghua.edu.cn/dart-pub
export FLUTTER_STORAGE_BASE_URL=https://mirrors.tuna.tsinghua.edu.cn/flutter
```

如果长期使用：

```
echo 'export FLUTTER_STORAGE_BASE_URL="https://mirrors.tuna.tsinghua.edu.cn/flutter"' >> ~/.bashrc
echo 'export PUB_HOSTED_URL="https://mirrors.tuna.tsinghua.edu.cn/dart-pub"' >> ~/.bashrc

## android项目中：
allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://mirrors.tuna.tsinghua.edu.cn/flutter/download.flutter.io' }
    }
}
```

###  CNNIC

```
export PUB_HOSTED_URL=http://mirrors.cnnic.cn/dart-pub
export FLUTTER_STORAGE_BASE_URL=http://mirrors.cnnic.cn/flutter
```

## 版本号

​	Dart采用的是加号式的版本描述方式，`+`前面是版本号，`+`后面是当前版本的build号。所以我们设置APP的版本号和build次数，在这里设置即可，例如：`version: 1.2.0+1`。

