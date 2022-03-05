<h1 align="center">Android API检测</h1>

[TOC]

## 工具

工具：veridex

地址：https://android.googlesource.com/platform/prebuilts/runtime/+/master/appcompat

## 使用方法

>Linux or Mac

```
// 在veridex工具目录下（直接在命令工具输出）
./appcompat.sh --dex-file=test.apk

// 保存文件
./appcompat.sh --dex-file=test.apk >> veridex.txt

// 显示更详细的信息：加 --imprecise 参数
./appcompat.sh --dex-file=test.apk --imprecise >> veridex.txt

// 过滤信息(只留下blacklist的)
./appcompat.sh --dex-file=test.apk -i 3 | grep blacklist
```

## 信息解读

* whitelist  白名单  可随意调用
* greylist 灰名单  警告
* greylist-max-o  targetSDK >= O 时异常
* greylist-max-p  targetSDK >= P 时异常
* blacklist  黑名单  不允许调用


参考：https://juejin.im/post/5d1623e1e51d45775964873c