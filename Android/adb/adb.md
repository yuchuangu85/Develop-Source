### 一、adb介绍与环境配置

adb：Android调试桥接Android Debug Bridge，是一个C/S架构的命令行工具

#### 1. adb组成：


* 客户端（Client）：可以通过它对android应用进行安装、卸载及调试

* 服务（Server）：管理客户端到android设备上adb后台进程的连接

* 守护进程（adb daemon）：运行在android设备上的adb后台进程

#### 2. 下载安装：


* 下载Android SDK，点击Android官网 下载

* 环境配置：将tools和platform-tools目录配置到系统变量PATH(如何配置详情请参考http://blog.csdn.net/qq_26967883/article/details/49337043的android配置环境)

* 验证是否配置成功：在cmd窗口输入adb

#### 3. adb命令格式

```
adb[-e|-d|-s<设备序列号>]<子命令>
```

| 参数     | 说明            |
| -------- | :-------------- |
| -e       | 只运行在模拟器  |
| -s       | 运行指定的设备  |
| -help    | 列出adb帮助文件 |
| -version | 列出adb版本     |

* 例如：

使用adb devices命令后发现有两个设备一个模拟器，一个真机

1.使用adb -e shell命令进入到模拟器中

2.使用adb -d shell命令进入到真机中

3.使用adb -s <设备ID> shell命令进入到指定设备中

```
List of devices attached
192.168.213.101:5555device
//上面那串数字就是设备ID
```

### 二、adb基本命令

#### 1.文件传输与安装命令

|子命令|	参数	|说明|
|----|:-----|:----|
|devices |	[-l] |	列出所有已经连接的设备，有三种状态：device/offline/device not found
|push	| < local > < remote >|	复制一个文件或者目录到设备中|
|pull	| < remote > < local >	|从手机复制文件到本地|
|install	| [-l   -r   -t   -s -d]< file >|安装apk <br> -l:表示应用为受限应用 <br>  -r:替换已经存在的应用  <br> -t:运行安装测试包 <br> -s:安装到SD卡中 <br> -d:允许安装到sd卡中|
|install-multiple	| [-l  -r  -t  -s  -d  -p]< file... >|批量安装 <br> -p:部分应用程序安装|
|uninstall	|[-k]< package >	|-k:保持data和cache下的文件|

#### 2.获取信息命令

| 子命令          | 参数 | 说明                                     |
| --------------- | :--- | :--------------------------------------- |
| wait-for-device | 无   | 等待设备连接(设备未连接之前使用)         |
| start-server    | 无   | 开启adb服务                              |
| kill-server     | 无   | 杀掉adb服务                              |
| get-state       | 无   | 获取adb服务状态offline/bootloader/device |
| get-serialno    | 无   | 获取SN号                                 |
| get-devpath     | 无   | 获取device-path                          |
| status-window   | 无   | 连续打印指定设备的设备状态               |

#### 3.Log与重启相关命令

| 子命令            | 参数                         | 说明                                                         |
| ----------------- | :--------------------------- | :----------------------------------------------------------- |
| bugreport         | 无                           |                                                              |
| logcat            | 命令较多                     | 输出android系统日志                                          |
| shell             | 命令较多                     | 进入远程shell端                                              |
| remount           | 无                           | 重新挂载系统分区，使系统分区重新可写(需要root权限)           |
| reboot            | [bootloader &#124; recovery] | 重启 <br> Bootloader：重启设备到bootloader状态 <br> recovery：重启设备到recovery状态 |
| reboot-bootloader | 无                           | 重启到bootloader                                             |
| root              | 无                           | 重新启动adbd获取root身份                                     |
| usb               | 无                           | 重新启动adbd来监听USB                                        |
| tcpip             | < port >                     | 重新启动adbd来监听指定TCP端口                                |

* 针对Logcat相关命令补充
  我本人是做测试的，所以对于应用的log这块比较看重，因此总结了一些指向性的实例：

```
//使用该命令可以查看指定应用的实时日志
adb logcat | find "packageName"
//使用该命令后指定的应用的相关日志会导出到相应位置
adb logcat | find "packageName" >F:\test\test.txt
```

#### 4.实例演示

```
1)devices
//列出已连接的设备
adb devices
//列出已连接的设备，并显示状态
adb devices -l
2)push
//将C盘目录下的apktool.log复制到设备的/mnt/sdcard/目录中
adb push C:\apktool.log /mnt/sdcard/
//将C盘目录下的apktool.log复制到虚拟设备的/mnt/sdcard/目录中
adb -e push C:\apktool.log /mnt/sdcard/
3)pull
//查看模拟器设备/data/app目录下的所有文件
adb -e shell ls /data/app
//将模拟器设备/data/app目录下的test.txt文件复制到本地c盘根目录
adb -e pull /data/app/test.txt c:\
  4)remount、pull、root
//在模拟器设备中重新挂载系统分区，使系统分区重新可写(需要root权限)
adb -e remount
//重新获取一下模拟器设备的root身份
adb -e root
//复制I：\com.android.cts.uiautomator.apk到/system/app中
adb push I：\com.android.cts.uiautomator.apk /system/app
5)install、uninstall
//将本地.apk文件安装到模拟器设备中
adb -e install I：\com.android.cts.uiautomator.apk
//替换掉模拟器设备中的.apk文件然后重新安装一次
adb -e install -r I：\com.android.cts.uiautomator.apk
//卸载包名为com.android.cts.uiautomator的应用，但是保留保持data和cache下的文件(可以使用"adb -e shell pm list packages "命令查看包名)
adb -e uninstall -k com.android.cts.uiautomator
//完全卸载包名为com.android.cts.uiautomator的应用。用到这个位置的话是删除data和cache下的文件的用意
adb -e uninstall com.android.cts.uiautomator
  6)servers
//杀掉adb服务
adb kill-server
//启动adb服务
adb start-server
//获取真机的连接状态
adb -d  get-state
//获取真机SN号
adb -d get-serialno
//获取真机的path
adb -d get-devpath
//不断获取真机的连接状态
adb -d status-window
//会列出许多真机的当前信息
adb -d bugreport
//重启真机
adb -d reboot
//重启USB，相当于重新插了一下USB设备的效果
adb -d usb
//在不插入设备的情况下输入该命令刚开始会提示找不到设备，但那是等插入设备后就可以正常安装了
adb -d install -r I：\com.android.cts.uiautomator.apk
//他会先等待你连接上设备后再进行替换安装
adb -d wait-for-device install -r I：\com.android.cts.uiautomator.apk
//等待设备连接后输出日志
adb logcat wait-for-device
```

### 三、adb备份与恢复命令

|子命令	| 参数	|说明|
|------|:----|:-----|
|backup|	无	|将应用的数据文件写入到指定的文件，在不指定-f输出目录的情况下，保持在当前目录的"backup.ab"
|backup|[-f < file >]	|指定备份目录
|backup|[-apk &#124; -noapk]	|是否备份apk文件，默认是noapk
|backup|[-obb &#124; -noobb]	|是否备份obb数据包，默认是noobb
|backup|[-shared &#124; -noshared]	|是否备份SD卡共享内容，默认是noshared
|backup|[-all]	|备份所有已安装的应用
|backup|[-system &#124; -nosystem]	|是否备份系统应用，-all默认是包括系统应用
|backup|< packages... >	|备份指定的应用列表|
|restore	|< file >|	将备份文件恢复到手机中|

* 例如：

```
//数据备份在你本地的当前目录，比如："C:\Users\test>adb -apk -all"里的C:\Users\test路径就是当前目录
adb -apk -all 
//将当前目录的备份文件恢复到设备
adb shell -restore back.ab 
```

### 四、adb重定向端口命令

#### 1.端口映射概念
比如将PC上的端口（1314）重定向到设备的端口（5200）上，这样所有发往PC端口（1314）的数据都会被转发到设备的端口（5200）上。这个机制可以实现远程控制Android设备应用。

#### 2.端口映射命令

| 子命令               | 参数                               | 说明                                      |
| -------------------- | :--------------------------------- | :---------------------------------------- |
| forward --list       | 无                                 | 列出所有套接字连接列表                    |
| forward              | < local > < remote >               | 重定向端口                                |
| forward --no-rebind  | < local > < remote >               | 重定向端口，例如local端口已经被占用则失败 |
| forward --remove     | < local >                          | 移除本地已经连接的套接字                  |
| forward --remove-all | 无	移除本地已经连接的所有套接字 |                                           |
| reverse --list       | 无	列出所有连接设备反向的套接字 |                                           |
| reverse              | < remove > < local >               | 反向连接套接字                            |
| reverse --norebind   | < remove > < local >               | 反向连接，加入端口已经被占用则连接失败    |
| reverse --remove     | < remove >                         | 删除一个特定的逆转套接字连接              |
| reverse --remove-all | 无                                 | 删除所有逆转的套接字连接设备              |

* 注：

```
a. foward系列的命令是PC端发出的
b. reverse系列的命令是设备发出的
```

#### 3.实例：

* 正向连接的例子：

```
//给设备上的monkey开辟端口1080
adb shell monkey --port 1080
//PC上的1080端口映射到设备上的1080端口（需要再打开一个新的cmd窗口
adb forward tcp:1080 tcp1080
//连接1080端口，连接好后会弹出一个新的窗口，此时可以发送一些按键消息比如"press 3"使用完毕关掉该窗口
telent localgost 1080
//查看刚刚映射的端口是否还在
   adb forward --list
//移除所有映射的端口
adb forward --remove-all
```

* 说明:由于反向连接貌似比较复杂，并且我个人在工作中暂时没有需求，就没有进行深入研究，有兴趣的朋友可以自己去看看

### 五、adb无线连接与文件同步
无线连接可以实现不用USB进行调试应用，文件同步可将修改的文件自动快速的push到手机对应的目录中

#### 1.相关命令
| 子命令     | 参数                | 说明                                                         |
| ---------- | :------------------ | :----------------------------------------------------------- |
| connect    | < host >[:< port >] | 通过TCP/IP连接到设备 <br>如果没有指定端口号则使用5555作为默认端口 |
| disconnect | < host >[:< port >] | 断开TCP/IP设备 <br> 如果没有指定端口号则使用5555作为默认端口<br>使用这个命令没有附加参数，将断开所有连接的TCP/IP设备 |
| sync       | [< directory >]     | 只要文件发生改变时就会自动从主机拷贝到设备需要指定环境变量ANDROID_PRODUCT_OUT为同步目录 |

#### 2.无线连接步骤

* 将安卓设备root掉
* 手机端安装wireless adb工具（一个命令行工具可以在手中使用命令）
* PC和Android设备连接到同一网络，手机上查看android设备IP可以使用命令"netcfg"
* PC端输入命令`adb connect IP地址:端口`（默认端口为5555）来通过TCP/IP连接到设备

```
 adb connect 192.168.1.104:5555
//这里的地址就是在手机端使用"netcfg"命令后显示出来的IP，5555为默认端口
//然后就可以使用adb的其他命令对手机进行操作了
```

#### 3.文件同步步骤

* sync如果没有指定更新目录，则会自动更新这些目录"system"、"vendor"、"data"

1.在PC上新建一个目录，目录中新建三个文件夹分别命名为"system"、"vendor"、"data"

2.为新建目录（就是三个新建文件夹的父文件夹所在的位置）设置系统环境变量ANDROID_PRODUCT_OUT

3.使用命令 `adb sync` 进行同步

* 注意

1.如果不是特别常用文件同步功能的话可以设置临时环境变量，就是在cmd窗口设置临时环境变量，只能在当前窗口可用，窗口关闭则变量失效

2.临时环境变量设置方式，打开一个cmd窗口然后在命令行输入`set ANDROID_PRODUCT_OUT=I:\sync`其实sync为父文件夹的名子，这样在本cmd窗口内就可以使用该变量了，关闭本cmd窗口则临时变量消失

3.如果要在设备的目录的子目录下同步文件需要在PC端创建与手机端同名的文件夹，比如说要在手机的/data/data目录中同步PC端的文件，那么只需在PC端的data目录中再创建一个data文件夹即可


原文：[https://www.cnblogs.com/JianXu/category/782865.html](https://www.cnblogs.com/JianXu/category/782865.html)