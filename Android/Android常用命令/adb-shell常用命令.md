### 一、文件操作相关命令

#### 1.文件操作命令：

| 子命令 | 参数                                                       | 说明                                                         |
| ------ | :--------------------------------------------------------- | :----------------------------------------------------------- |
| cd     | 无                                                         | 进入目录                                                     |
| cat    | [-beflnstuv] [-B bsize] [file...]                          | 查看文件内容<br>-n：显示行号<br>-b：显示行号，但会忽略空行<br>-s：显示行号，连续空行标记为一行 |
| df     | 无                                                         | 列出分区列表                                                 |
| du     | [-H] [-L] [-P] [-a] [-d depth] [-s] [-cghikmnrx] [file...] | 查询文件或目录的磁盘使用空间                                 |
|ls	|[-a] [-i] [-l] [-n] [-s]	|列出目录内容<br>-a：列出所有文件，包括隐藏文件<br>-i：输出文件的i节点的索引信息<br>-l列出文件的详细信息<br>-n：用数字的GUID代替名称<br>-s：输出该文件的大小
|grep	|[-abcDEFGHhliJLlmnOoPqRSsUVvwxZz] <br>[-A num]<br>[-B num]<br>[-C[num] <br>[-e pattern]<br>[-f file]<br>[--binary-files=value]<br>[--color=when]<br>[--context=num] <br>[--directories=action]<br>[--lable]<br>[--line-buffered]<br>[pattern]                             [file...]|指定文件中搜索特定的内容，并将含有这些内容的行标准输出|
|mkdir	|-p，-parents|	创建目录<br>-p，--parents：递归创建目录|
|touch	|touch [-alm] [-t YYYYMMDD [.HHMMSS]] < file >|创建文件|
|rm |	rm [-f\|-i][-dPRrvWx]file|	删除文件<br>-f：强制删除文件，系统不提示<br>-i：交互式删除，删除前提示<br>-d：改变硬连接数据删成0，删除该文件<br>-r:强制删除文件夹包括里面的文件|
|mv|	mv[-fiv]source target|	移动文件（相当于剪切）<br>-f：强制移动，若文件已经存在目标则直接覆盖<br>-i：若目标文件已经存在，会询问是否覆盖|
|rmdir	|rmdir[-p] directory|	删除目录<br>-p：递归删除目录，只能删除空目录|
|dd	|dd[operand...]<br>dd if =source of=targe|复制文件|

#### 2.文件权限命令与其他文件命令：

| 子命令 | 参数                                                      | 说明                                                         |
| ------ | :-------------------------------------------------------- | :----------------------------------------------------------- |
| chomd  | chomd[OPTION]< MODE > < FILE >                            | 文件权限修改<br>-R：递归改变文件和目录<br>-h：不遵循符号连接 |
| chown  | chown[-R[-H\|-L\|-P]] [-fhv]<br>owner : group             | owner                                                        |
| md5    | md5 file...                                               | 查询文件的MD5值                                              |
| mount  | mount [-r] [-w] [-o options] [-t type] device directory   | 挂载设备信息                                                 |
| umount | umount < path >                                           | 卸载分区挂载                                                 |
| cmp    | cmp[-b][-l][-n count] file1 file2                         | 要指出两个文件是否存在差异                                   |
| ln     | ln [-fhinsv] file1 file2<br>ln [-fhinsv] file...directory | 用来在文件之间创建连接，创建连接后两个文件中任意一个文件改变文件内容另一文件都会相应进行同步改变 |

#### 3.命令使用实例：

```
//进入设备
adb shell
//进入指定目录"/data/local/tmp"
cd /data/local/tmp
//查看目录
ls
//进入根目录
cd /
//进入指定目录"/data/local/tmp"
cd /data/local/tmp
//查看分区列表
df
//在当前目录下创建名为1的.txt文件（再创建个两个，命名为2和3，方便后面继续学习使用）
touch 1.txt
//列出所有文件（包括隐藏文件）的详细信息，此时可以查看刚刚的1.txt是否创建成功
ls -al
//在当前目录下创建一个名为1的文件夹
mkdir1
//列出所有文件（包括隐藏文件）的详细信息，此时可以查看刚刚的目录文件夹是否创建成功
ls -al
//在当前目录下创建递归目录，2下面包含3，3下面包含4
mkdir -p 2/3/4
//回到上一级目录，连续操作两次让他回到cd /data/local/tmp目录下
cd ..
//将1.txt文件移动到1目录中（剪切效果）
mv 1.txt 1
//进入1目录cd 1
//查看1.txt是否移动进去了
ls
//返回上一级目录
cd ..
//将当前目录下的2.txt文件复制到名为2的目录下并命名为2.txt
dd if=2.txtof=2/22.txt
//进入到目录2中
cd 2
//查看上个文件操作是否操作成功
ls
//回到上一级目录
cd ..
//进入1目录
cd 1
//删除当前目录下的1.txt文件
rm 1.txt
//回到上一级目
cd ..
//删除名为1的目录
rmdir 1
//查看删除操作是否删除成功
ls
//查看文件权限信息
la -al
//修改2.txt的文件权限为最高
chomd 777 2.txt
//查看刚刚修改的文件权限信息是否成功
la -al
//查看2.txt文件的md5
md5 2.txt
//查看挂载设备信息
mount
//将system分区变成可读可写"mount -o [option] devices directory"
mount -o remount,rw /dev/block/sda6 /system
//查看刚刚的修改是否成功
mount
//查看分区列表
df
//卸载掉"/storage/sdcard"分区挂载
umount /storage/sdcard
//查看刚刚的卸载是否成功
df
//输入点内容到2.txt中
echo 333 >>2.txt
echo 222 >>2.txt
//查看2.txt文件
cat 2.txt
//指出两个文件是否存在差异
cmp 2.txt 3.txt
//复制2.txt文件夹并粘贴到当前目录中，命名为22.txt
dd if=2.txt of=22.txt
//指出两个文件是否存在差异
cmp 2.txt 3.txt
cd
//在2目录下创建一个名为2o.txt的2的硬连接文件（因为2的目录下已经存在2.txt文件，不然使用"ln 2.txt 2"命令就可以了）
ln  2.txt 2/2o.txt
//进入到2目录
cd 2
//查看2o.txt文件内容
cat 2o.txt
//在2o.txt文件中加入内容
echo >>2o.txt
//返回上一级目录
cd ..
//查看连接文件2.txt的文件内容是否与2o.txt一致
cat 2.txt
```

### 二、信息查询相关命令

#### 1.log 相关命令：

| 子命令    | 参数                                            | 说明                                                         |
| --------- | :---------------------------------------------- | :----------------------------------------------------------- |
| dumpstate | -                                               | 系统状态信息（需要root权限）<br>包括手机当前的内存信息、CPU信息、logcat缓存，kenel缓存等等<br>adb bugreport包含这个信息 |
| bugreport | -                                               | 里面含有dmesg，dumpstate和dumpsysy                           |
| demsg     | -                                               | kenel的log                                                   |
| logcat    | 参数较多                                        | 打印日志缓冲区日志                                           |
| dumpsys   | meminfo [processName]<br>activity [processName] | 获取系统各项服务信息                                         |

#### 2.获取系统信息相关命令：

| 子命令   | 参数                                                         | 说明                                   |
| -------- | :----------------------------------------------------------- | :------------------------------------- |
| getevent | -                                                            | 获取按键信息                           |
| getprop  | -                                                            | 获取系统属性                           |
| setprop  | -                                                            | 设置系统属性（需要root权限）           |
| pm       | -                                                            | 安装包管理，查询安装包的各种信息       |
| ps       | -                                                            | 查看进程信息                           |
| top      | -m num 最大显示条数<br>-n num 更新次数<br>-d num 两者更新时间<br>-s col按哪列排序（cpu，vss，rss，thr）<br>-t显示线程信息而不是进程<br>-h显示帮助文档 | 获取CPU使用情况                        |
| procrank | -                                                            | 查询各进行内存消耗情况（需要root权限） |
| wm       | size                                                         | 获取屏幕分辨率                         |

#### 3.命令使用实例：

```
//输出系统状态信息至F:\test\dumpstate.txt，由于需要root权限，所以没root的过的手机输出为空
adb shell dumptate >F:\test\dumpstate.txt
//输出过去系统的状态，log，一般操作过程中未抓取log的时候一旦出现问题就使用这个命令来查看
adb shell bugreport >F:\test\bugreport.txt
//输出内核信息
adb shell dmesg
//输出当前缓冲区日志 并保存
adb shell logcat    >F:\test\bugreport.txt
//输出内存信息
adb shell dumpsys meminfo
//输出当前CPU使用情况信息
adb shell dumpsys cpuinfo
//输出当前activity使用情况信息
adb shell dumpsys activity
//相当于过滤，只找名为"mF"的activity使用情况信息
adb shell dumpsys activity | find "mF"
//获取按键信息，在手机没有按键 信息的情况下会先提示你每个设备的ID代表的设备信息，按键过程中会实时刷新
adb shell getevent
//获取系统属性
adb shell getprop
//查看pm帮助信息
adb shell pm
//查看手机内的安装包列表
adb shell pm list packages
//查看当前手机进程信息
adb shell ps
//获取cpu使用情况，只查看一次，不实时刷新
adb shell top -n 1
//获取前十的cpu使用情况，只查看一次，不实时刷新
adb shell top   -n 1 -m 10
//查询各进行内存消耗情况
adb shell procrank
//详细查询某个包的内存使用情况
adb shell dumpsys meminfo packageName
```

### 三、操作手机相关命令

#### 1.相关命令：

| 子命令 | 参数                                                         | 说明                                                         |
| ------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| bmgr   | [backup                                                      | &#124; restore &#124; list &#124; transport &#124; run]<br>bmgr backup PACKAGE<br>bmgr restore<br>...... |
| kill   | kill [-s signame &#124; -signu &#124; -signame]{job &#124; pid &#124; pgrp}...<br>kill -l [exit_status...] | 结束进程                                                     |
| reboot | 无                                                           | 重启手机                                                     |
| svc    | power 控制电源管理<br>data 控制数据连接<br>控制wifi管理<br>控制USB状态 | 控制电源、网络、USB                                          |
| wipe   | wipe system &#124; data &#124; all                           | 擦除分区，恢复出厂设置                                       |
| am     | am [subcommand] [options]<br>am start<br>......              | antivyty管理器<br>用于开启应用，广播，服务等功能             |

#### 2.命令使用实例：

```
//查询已安装包名列表
adb shell pm list package
//对com.tencent.mm包使用monkey命令
adb shell monkey -p com.tencent.mm --throttle 200 50000
//查找monkey进程信息
adb shell ps | find "monkey"
//杀掉monkey进程，例子中的数字是monkey的PID进程号
adb shell kill 23770
//重启手机
adb shell reboot
//打开svc帮助界面
adb shell svc
//查询wifi操作帮助
adb shell svc wifi
//关闭wifi
adb shell svc wifi disable
//打开wifi
adb shell svc wifi enable
//擦除data，即恢复出厂设置
adb shell wipe data
//指定查询"mF"的activity信息
adb shell dumpsys activity | find "mF"
//启动指定activity
adb shell am start -n com.android.browser/.BrowserActivyty
//查看am命令的帮助信息
adb shell am
```

### 四、测试用途相关命令

#### 1.测试信息相关命令：

| 子命令       | 参数                                                | 说明                                     |
| ------------ | :-------------------------------------------------- | :--------------------------------------- |
| iftop        | iftop [-r repeats] [-d delay]                       | 列出网络传输包情况                       |
| Monkey       | Monkey [options] count                              | 执行Monkey命令                           |
| netstat      | -                                                   | 显示各种网络相关信息                     |
| ping         | ping [option] ipv4                                  | 因特网包探测器，用于测试网络连接量的程序 |
| ping6        | ping6 [option] ipv6                                 | 因特网包探测器，用于测试网络连接量的程序 |
| screenrecord | screenrecord [options] < filename >                 | 屏幕录像(只支持android4.4以上的设备)     |
| screencap    | [-hp] [-d display-id] [FILENAME]<br>-p 文件保存路径 | 屏幕截图                                 |
| uiautomator  | uiautomator [options]                               | 执行uiautomator脚本                      |

* 命令使用实例:

```
//进入交互模式
adb shell
//列出网络传输包情况
iftop
//显示各种网络相关信息
netstat
//实时查看网络连接量
ping www.baidu.com
//实时查看网络连接量
ping6 www.baidu.com
//屏幕录像，保存路径为/mnt/sdcard/1.mp4，需要注意的是屏幕录像只支持android4.4以上的设备使用ctrl+c停止录像
screenrecord /mnt/sdcard/1.mp4
//将sd卡路径下的1.mp4导出到F盘(先退出交互模式)
adb pull /mnt/sdcard/1.mp4 f:\
//截图
screencap /mnt/sdcard/1.png
```

#### 2.输入信息命令：

1）input

```
作用：模拟硬件设备的输入格式：input []  [...]参数：test(Defalt;touchscreen)keyevent [--longpress] ...(Default:keyboard)tap (Default:touchscreen)swipe [duration(ms)] (Default:touchscreen)press (Default:trackball)roll (Default:trackball)
```

2）命令使用实例

```
//进入交互模式
adb shell
//输入文本123456
input text 123456
//使用keycode num输入，keycode表可百度查询
input keyevent 7
//使用keycode name输入1
input keyevent KEYCODE_1
//使用keycode name按空格键
input keyevent KEYCODE_HOME
//点击坐标367  1277
input tap 367 1277
//从（1024，945）滑动到（134，968）200毫秒内
input swipe 1024 945 134 968 200
```



原文：[https://www.cnblogs.com/JianXu/category/782865.html](https://www.cnblogs.com/JianXu/category/782865.html)