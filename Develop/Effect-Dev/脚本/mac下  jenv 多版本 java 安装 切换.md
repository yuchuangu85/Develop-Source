<h1 align="center">mac下  jenv 多版本 java 安装 切换</h1>

[TOC]

## 1、下载安装 jenv

```
curl -s get.jenv.io | bash
```

## 2、jenv参考（关键是方便别的java工具管理）： 

[参考该链接](https://github.com/linux-china/jenv/wiki/Chinese-Introduction)


## 3、进入jenv目录,然后建相关目录：

```
mkdir ~/.jenv/candidates/java/1.6
mkdir ~/.jenv/candidates/java/1.7
mkdir ~/.jenv/candidates/java/1.8

ls ~/.jenv/candidates/java/
```

## 4、安装不同版本的 java

1.6从官网下载


## 5、执行以下命令

```
ln -s /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home/bin ~/.jenv/candidates/java/1.6/
ln -s /Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home/bin ~/.jenv/candidates/java/1.7/
ln -s /Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home/bin ~/.jenv/candidates/java/1.8/
```

大功告成： 
* 最先默认的jdk一般是你最后安装的那jdk。 
* 切换版本：jenv use java 1.8 
* 设置缺少版本：jenv default java 1.6