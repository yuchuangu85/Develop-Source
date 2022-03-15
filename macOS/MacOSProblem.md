<h1 align="center">Mac OS 常见问题和常用软件</h1>

[toc]

## 1.常见问题

xxx Is Damaged and Can’t Be Opened. You Should Move It To The Trash

```
// 后面的app路径可以直接拖入自动生成
➜ codesign --force --deep --sign - /Applications/CleanMyMac.app
// 或者
xattr -cr /path/to/application.app  例如：xattr -cr /Applications/Signal.app
```

安装第三方软件提示已损坏或无法验证开发者

```
// 临时允许访问一个软件或文件
sudo xattr -d com.apple.quarantine /xxx/xxx （直接拖拉文件到终端也可以获取全路径）
```

## 2.python路径

/System/Library/Frameworks/Python.framework/Versions/2.7



## 3.Mac命令失效

在配置文件添加：

export PATH="/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin"

重启命令端



## 4. 关于Big Sur没有权限打开应用程序的解决方法

### 4.1 安装无权限修复插件

```
brew install upx
```

### 4.2 然后执行命令

```
sudo upx -d app路径
```

### 4.3 或者参考

https://jingyan.baidu.com/article/b24f6c82c815d9c7bfe5daef.html



## 5.Unknown host 'xxx: nodename nor servname provided, or not known'. You may need to adjust the proxy settings in Gradle.

设置DNS为自动获取



## 6.macOS插入移动硬盘不能识别

关闭防火墙



## 7.关于“.DS_Store”文件

> .DS_Store是Mac OS保存文件夹的自定义属性的隐藏文件，如文件的图标位置或背景色，相当于Windows的desktop.ini。

**1.禁止** “.DS_store”**生成**：打开**terminal**，复制黏贴下面的命令，回车执行，重启Mac即可生效。

```bash
defaults write com.apple.desktopservices DSDontWriteNetworkStores true
defaults read com.apple.desktopservices DSDontWriteNetworkStores
// 最新
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
```

**2.恢复** “.DS_store”**生成**：

```bash
defaults write com.apple.desktopservices DSDontWriteNetworkStores false
// 最新
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
```

**3.删除** 所有**目录**的“.DS_store”文件： 在**terminal**中输入下面命令并输入密码:

```bash
sudo find / -name ".DS_Store" -depth -exec rm {} \;
```

⭐️：删除 **当前目录**的“.DS_store”文件

```bash
find . -name '*.DS_Store' -type f -delete
```



## 8.弹出App can’t be opened because Apple cannot check it for malicious software的解决方法

```undefined
sudo spctl --master-disable
```



## 9.macOS无法联网问题

1、打开系统偏好设置—>网络—>WiFi—>高级—>WiFi—>删除首选网络框内的所有网络—>点击好—>点击应用；

2、还是在网络页面先，在边框有WiFi、蓝牙PAN、网桥等，选中WiFi，点击下面的减号删除WiFi，点击应用；

3、再次在系统偏好设置中打开网络页面，在左边框的下方点击加号，接口选择WiFi，服务名称随便写，点击创建，然后点击打开WiFi，链接你的WiFi。应该可以上网了。亲测可行。

上面试完就可以了，网上还有另外两种，未试，先留着

4、Finder—>xxx的Mac—>Macintosh HD—>资源库–>Preferences—>SystemConfiguration—>找到NerworkInterfaces.plist文件并删除；

5、终极大招：command+option+p+r，开机时按住，听到3声以上声响（屏幕闪烁3下）后松开。这个方法就是完全重置你电脑的控制器了，会将设置都恢复初始化，但是不影响硬盘数据，不必备份。
