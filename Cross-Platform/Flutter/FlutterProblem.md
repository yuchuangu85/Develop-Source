<h1 align="center">Flutter问题总结</h1>

[toc]

## 1. it is taking an unexpectedly long time

Please try steps below:
a. delete all contents under /home/nima/.gradle
b. cd to the android folder for your flutter project and run `./gradlew sync` command, it will download the gradle-xxx.zip automatically. It this gradle process succeeds, re-run the `flutter run` command.

## 2. Resolving dependencies…

​	配置本地gradle

​	路径：android⁩ ▸ gradle⁩ ▸ wrapper⁩ ▸ gradle-wrapper.properties

```
distributionUrl=https\://services.gradle.org/distributions/gradle-5.6.4-bin.zip
改成（下载gradle压缩文件路径）：
distributionUrl=../../../../../Gradle/gradle-5.6.2-all.zip
```

## 3. kotlin问题

```
[+1023004 ms] FAILURE: Build failed with an exception.
[   +3 ms] * What went wrong:
[        ] A problem occurred configuring root project 'android'.
[        ] > Could not resolve all artifacts for configuration ':classpath'.
[        ]    > Could not download kotlin-gradle-plugin.jar (org.jetbrains.kotlin:kotlin-gradle-plugin:1.3.50)
[        ]       > Could not get resource
'https://jcenter.bintray.com/org/jetbrains/kotlin/kotlin-gradle-plugin/1.3.50/kotlin-gradle-plugin-1.3.50.jar'.
[   +1 ms]          > Connection reset
[        ] * Try:
[        ] Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to
get full insights.
[        ] * Get more help at https://help.gradle.org
[        ] BUILD FAILED in 17m 13s
```

把Android项目中build.gradle中的

```
classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
```

改为常用的版本，例如：

```
ext.kotlin_version = '1.3.71'
```

## 4. Could not find io.flutter:flutter_embedding_debug:1.0.0-5b952f286fc070e99cf192775fa5c9dfe858b692

```
flutter channel stable
flutter upgrade
flutter clean
flutter run
```



## 5.Running CocoaPods on Apple Silicon (M1)

问题：I have a Flutter project that I'm trying to run on iOS. It runs normally on my Intel-based Mac, but on my new Apple Silicon-based M1 Mac it fails to install pods.

解决：[ios - Running CocoaPods on Apple Silicon (M1) - Stack Overflow](https://stackoverflow.com/questions/64901180/running-cocoapods-on-apple-silicon-m1)

```
EDIT: I recently disabled Rosetta, and Cocoapods runs just fine with the addition of the ffi gem.

For anyone else struggling with this issue, I just found a way to solve it. In addition to running terminal in Rosetta:

Right-click on Terminal in Finder
Get Info
Open with Rosetta
I installed a gem that seems to be related to the symbol not found in the error:
```



## 6.[Error Regarding undefined method `map' for nil:NilClass for Flutter App / CocoaPod Error](https://stackoverflow.com/questions/67443265/error-regarding-undefined-method-map-for-nilnilclass-for-flutter-app-cocoap)

```
Are you using Apple M1? I had this issue as well and after some research I find that it might be something to do with Rosetta. You can refer to Running CocoaPods on Apple Silicon (M1).

I managed to solve this issue on my MacBook Air M1 by typing this in the terminal:

sudo arch -x86_64 gem install ffi

from here https://stackoverflow.com/a/65334677/13814270.
```



