<h1 align="center">Android系统源码下载编译</h1>

[toc]

## 1. 准备

### 1.1 下载repo工具

```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

### 1.2 使用每月更新的初始化包

由于首次同步需要下载约 30GB 数据，过程中任何网络故障都可能造成同步失败，我们强烈建议您使用初始化包进行初始化。

下载 https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar，下载完成后记得根据 checksum.txt 的内容校验一下。

由于所有代码都是从隐藏的 `.repo` 目录中 checkout 出来的，所以我们只保留了 `.repo` 目录，下载后解压 再 `repo sync` 一遍即可得到完整的目录。

使用方法如下:

```
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
tar xf aosp-latest.tar
cd AOSP   # 解压得到的 AOSP 工程目录
# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
repo sync # 正常同步一遍即可得到完整目录
# 或 repo sync -l 仅checkout代码
```

此后，每次只需运行 `repo sync` 即可保持同步。 **我们强烈建议您保持每天同步，并尽量选择凌晨等低峰时间**

### 1.3 传统初始化方法

建立工作目录:

```
mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
```

初始化仓库:

```
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest
```

**如果提示无法连接到 gerrit.googlesource.com，请参照[git-repo的帮助页面](https://mirrors.tuna.tsinghua.edu.cn/help/git-repo)的更新一节。**

如果需要某个特定的 Android 版本([列表](https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds))：

```
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-4.0.1_r1
```

同步源码树（以后只需执行这条命令来同步）：

```
repo sync
```

## 2. 替换已有的 AOSP 源代码的 remote

如果你之前已经通过某种途径获得了 AOSP 的源码(或者你只是 init 这一步完成后)， 你希望以后通过 TUNA 同步 AOSP 部分的代码，只需要修改 `.repo/manifests.git/config`，将

```
url = https://android.googlesource.com/platform/manifest
```

更改为

```
url = https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest
```

或者可以不修改文件，而执行

```
git config --global url.https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/.insteadof https://android.googlesource.com
```

## 3. 源码下载以及切换源码分支

源码下载：

**清华大学镜像地址：**[https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)

```
## 切换分支
repo init -b android-7.1.2_r11

## 如果不能连接google源码可以先切换到下面地址：
repo init -u https://aosp.tuna.tsinghua.edu.cn//platform/manifest -b android-10.0.0_r9

## 如果不切换源码地址：
repo init -b android-10.0.0_r9  ## 切换分支
repo sync  						## 如果不需要与服务器数据一致，可以不运行该步
repo start android-10.0.0_r9 --all 
```

同步代码：

```
repo sync
```

仅从本地checkout出来代码，不从远程fetch

```bash
repo sync --local-only  
```

查看源码分支：

```
build\core\version_defaults.mk //搜索该文件中的 PLATFORM_VERSION值
```

查看可切换的分支：

```
cd .repo/manifests
git branch -a
或
git branch -a | cut -d / -f 3
```

查看切换结果

```
repo branches
```

AOSP代号、标记和细分版本号

```
https://source.android.google.cn/setup/start/build-numbers.html#source-code-tags-and-builds
```

## 4. Idea导入源码方法：

>本教程基于Mac OS X 10.12。Android系统版本为：7.1.2_r11(7.1.2最终版)。先介绍方法，后面会给出各种问题解决方案。

### 生成导入idea或者eclipse需要的文件：

* 1.首先是idea和eclipse导入项目需要的文件

```
.classpath (Eclipse)
android.ipr (IntelliJ / Android Studio)
android.iml (IntelliJ / Android Studio)
```

* 1.修改MacOS sdk 版本：

路径Android-7.1.2_r11/build/core/combo/mac_version.mk，加上你现在系统的版本：

 ```
 mac_sdk_versions_supported :=  10.8 10.9 10.10 10.11 10.12
 ```
* 2.生成idegen.jar过程：

命令：

```
$ source build/envsetup.sh
```
执行结果：

```
including device/asus/fugu/vendorsetup.sh
including device/generic/mini-emulator-arm64/vendorsetup.sh
including device/generic/mini-emulator-armv7-a-neon/vendorsetup.sh
including device/generic/mini-emulator-mips/vendorsetup.sh
including device/generic/mini-emulator-mips64/vendorsetup.sh
including device/generic/mini-emulator-x86/vendorsetup.sh
including device/generic/mini-emulator-x86_64/vendorsetup.sh
including device/google/dragon/vendorsetup.sh
including device/google/marlin/vendorsetup.sh
including device/htc/flounder/vendorsetup.sh
including device/huawei/angler/vendorsetup.sh
including device/lge/bullhead/vendorsetup.sh
including device/linaro/hikey/vendorsetup.sh
including device/moto/shamu/vendorsetup.sh
including sdk/bash_completion/adb.bash
```

命令：

```
$ mmm development/tools/idegen
```

执行结果：

```
...
date: 1496452336: No such file or directory
Starting build with ninja
ninja: Entering directory `.'
[ 25% 1/4] host Java: idegen (out/host/com...VA_LIBRARIES/idegen_intermediates/classes)
注: 某些输入文件使用或覆盖了已过时的 API。
注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
[100% 4/4] Install: out/host/darwin-x86/framework/idegen.jar

#### make completed successfully (18 seconds) ####
```

命令：

```
$ development/tools/idegen/idegen.sh
```

执行结果：

```
bash-3.2$ development/tools/idegen/idegen.sh
Read excludes: 4ms
Traversed tree: 44148ms
```

此时会在根目录生成两个文件：android.ipr和android.iml，然后打开idea软件，执行下面操作：
![](media/download1.png)
接着打开如下界面，找到Android源码位置，然后找到生成的android.iml文件，鼠标选中，然后点击open即可。
![](media/download2.png)


注：mmm命令要先执行第一条命令。

### 生成文件出现的问题：

* 1.在执行

```
$ source build/envsetup.sh
```

命令时会遇到下面问题：

```
build/envsetup.sh:630: command not found: complete
WARNING: Only bash is supported, use of other shell would lead to erroneous results
```

警告需要再bash下执行命令，我用的是zsh，临时切换回bash,直接输入bash：

```
$ bash
```

如果不切换回bash输入

```
$ mmm development/tools/idegen
```

命令则会报下面错误：

```
Couldn't locate the directory development/tools/idegen
```

* 2.由于生成该文件需要MacOS SDK，所以需要安装Xcode,最新版Xcode里面的sdk是10.12（与最新系统一样），而在Android源码里面最高到10.11，所以不支持，需要修改源码中的对sdk的支持：

打开路径Android-7.1.2_r11/build/core/combo/mac_version.mk，加上你现在系统的版本：

```
mac_sdk_versions_supported :=  10.8 10.9 10.10 10.11 10.12
```

其实这个方法也不能解决，因为后面的编译中还是不支持10.12，由于对里面不熟，所以采用了另一个方法，在Xcode中添加sdk，见下面方法。

* 3.问题：不支持Mac OS X 10.12，添加sdk

```
system/core/libcutils/threads.c:38:10: error: 'syscall' is deprecated: first deprecated in OS X 10.12 - syscall(2) is unsupported; please switch to a supported interface. For SYS_kdebug_trace use kdebug_signpost(). [-Werror,-Wdeprecated-declarations]
  return syscall(SYS_thread_selfid);
         ^
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk/usr/include/unistd.h:733:6: note: 'syscall' has been explicitly marked deprecated here
int      syscall(int, ...);
         ^
1 error generated.
ninja: build stopped: subcommand failed.
make: *** [ninja_wrapper] Error 1
```
解决办法：

```
Here is how I fixed it:

Download earlier Mac OSX SDK(10.11 worked for me) from
https://github.com/phracker/MacOSX-SDKs/releases
Unzip and copy to /Applications/XCode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs
```

* 4.Mac默认系统不区分大小问题：

报错：

```
Checking build tools versions...
build/core/main.mk:90: ************************************************************
build/core/main.mk:91: You are building on a case-insensitive filesystem.
build/core/main.mk:92: Please move your source tree to a case-sensitive filesystem.
build/core/main.mk:93: ************************************************************
build/core/main.mk:94: *** Case-insensitive filesystems not supported. Stop.
```

此时你可以建一个磁盘镜像，步骤如下：
打开磁盘工具-->文件-->新建映像-->空白映像-->弹出如下界面，填写下面框内的信息，格式选择区分大小写格式，点击存储，然后会在你的位置文件夹内生成一个Android.dmg文件，双击即可安装，然后将Android源码考入即可操作。
![](media/download3.png)

## 5. Android studio源码

```
$ mkdir studio-master-dev
$ cd studio-master-dev
$ repo init -u https://android.googlesource.com/platform/manifest -b studio-master-dev
$ repo sync
// 或者
$ repo init -u https://android.googlesource.com/platform/manifest -b studio-3.1.2
```

## 6. Androidx源码

```
$ mkdir androidx-master-dev
$ cd androidx-master-dev
$ repo init -u https://android.googlesource.com/platform/manifest -b androidx-master-dev
$ repo sync
// 或者
$ repo init -u https://android.googlesource.com/platform/manifest -b androidx-master-dev

// 清华镜像
$ repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b androidx-master-dev --partial-clone --clone-filter=blob:limit=10M
$ repo sync -j4 -c
```

