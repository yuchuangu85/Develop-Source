### 一、pm命令介绍与包名信息查询

#### 1.pm命令介绍：

pm工具为包管理（package manager）的简称

可以使用pm工具来执行应用的安装和查询应用宝的信息、系统权限、控制应用

pm工具是Android开发与测试过程中必不可少的工具，shell命令格式如下：

```
pm <command>
```

#### 2.包名信息查询：

shell模式下：

```
pm list packages [options] [FILTER]
```

打印所有的已经安装的应用的包名，如果设置了文件过滤则值显示包含过滤文字的内容

| 参数 | 描述                               |
| ---- | :--------------------------------- |
| -f   | 显示每个包的文件位置               |
| -d   | 使用过滤器，只显示禁用的应用的包名 |
| -e   | 使用过滤器，只显示可用的应用的包名 |
| -s   | 使用过滤器，只显示系统应用的包名   |
| -3   | 使用过滤器，只显示第三方应用的包名 |
| -i   | 查看应用的安装者                   |


### 二、权限信息查询

#### 1.权限基础：

权限的组成：权限的名称，属于的权限组，保护级别

例如:

```
<permission android:description="string resource"
android:icon="drable resource"
android:label="string resource"
android:name="string"
android:permissionGroup="string"
android:protectionLevel=["normal"|"dangerous"|"signature"|"signatureOrSystem"]/>
```

| protectionLevel   | 说明                                                         |
| ----------------- | :----------------------------------------------------------- |
| normal            | 表示权限是低风险的，不会对系统，用户或其他应用程序造成危害   |
| dangerous         | 表示权限是高风险的，系统将可能要球用户输入相关信息，才会授予此权限 |
| signature         | 表示只有当应用程序所用数字签名与声明引用权限的应用程序所用签名相同时，才能将权限授予给它 |
| signatureOrSystem | 需要签名或者系统级应用（放置在/system/app目录下）才能赋予权限 |
| system            | 系统级应用（放置在/system/app目录下）才能赋予权限            |
| 自定义权限        | 应用自行定义的权限                                           |


#### 2.权限查询：

shell模式下：

```
pm list permission-groups

#打印所有已知的权限组

pm list permissions [options] [GROUP]

#打印权限
```

**参数可以组合使用例如：pm list permissions –g -d**

| 参数 | 说明                                        |
| ---- | :------------------------------------------ |
| -g   | 按组进行列出权限                            |
| -f   | 打印所有信息                                |
| -s   | 简短的摘要                                  |
| -d   | 只有危险的权限列表                          |
| -u   | 只有权限的用户将看到列表 <br>用户自定义权限 |


#### 3.授权与取消：

注意：目标apk的minSdkVersion、targetSdkVersion也必需为23及以上

| 子命令                             | 说明                                                    |
| ---------------------------------- | :------------------------------------------------------ |
| grant <package_name> <permission>  | 授予应用权限许可。必需android6.0（API级别23）以上的设备 |
| revoke <package_name> <permission> | 撤销应用权限。必需android6.0（API级别23）以上的设备     |

例如：

需要注意的是所谓的授权是指你的apk里面已有的权限进行授权，相当于启用的概念

```
adb shell pm grant <packageName> android.permission.READ_CONTACTS
#授权( 取消权限同理)
```


### 三、其他信息查询


#### 1.测试包与apk路径查询：

```
pm
```

| 子命令               | 参数             | 说明                            |
| -------------------- | :--------------- | :------------------------------ |
| list instrymentation | 无参数           | 列出所有的instrumentation测试包 |
| list instrymentation | -f               | 列出apk文件位置                 |
| list instrymentation | <target_package> | 列出某个app的测试包             |
| path <package>       | <package>        | 打印指定包名的apk路径           |

例如：

adb shell pm list instrumentation

adb shell pm list instrumentation TARGET_PACKAGE

adb shell pm path PACKAGE_NAME

#### 2.系统功能与支持库查询：

```
pm
```

| 子命令         | 说明                                        |
| -------------- | :------------------------------------------ |
| list feature   | 打印系统的所有功能 <br>列出所有硬件相关信息 |
| list libraries | 打印当前设备所支持的所有库                  |

例如：

adb shell pm list feature

#### 3.打印包的系统状态信息：

```
pm dump PACKAGE

打印给定的包的系统状态
```

| 打印内容                     | 说明                               |
| ---------------------------- | :--------------------------------- |
| DUMP OF SERVICE package      | 打印服务信息                       |
| DUMP OF SERVICE activity     | 打印activity信息                   |
| DUMP OF SERVICE meminfo      | 打印当前内存使用信息               |
| DUMP OF SERVICE procstats    | 打印系统内存使用与一段时间内存汇总 |
| DUMP OF SERVICE usagestats   | 打印服务器使用状态信息             |
| DUMP OF SERVICE batterystats | 打印电池状态信息                   |

例如：

adb shell pm dump PACKAGE_NAME

 

### 四、安装与卸载


#### 1.安装：


```
pm install [-lrtsfd] [-i PACKAGE] [PATH]

通过指定路径安装apk到手机中(与adb install不同的是adb install安装的.apk是在你的电脑上，而pm install安装的apk是存储在你的手机中)
```

| 参数                        | 说明                                                |
| --------------------------- | :-------------------------------------------------- |
| -l                          | 锁定应用程序                                        |
| -r                          | 重新安装应用，且保留应用数据                        |
| -t                          | 允许测试apk被安装                                   |
| -i <INSTALLER_PACKAGE_NAME> | 指定安装包的包名                                    |
| -s                          | 安装到sd卡                                          |
| -f                          | 安装到系统内置存储中（默认安装位置）                |
| -d                          | 允许降级安装（同一应用低级换高级）                  |
| -g                          | 授予应用程序清单中列出的所有权限（只有6.0系统可用） |

首先将test.apk文件push到手机目录中比如/data/local/tmp

adb shell pm install /data/local/tmp/test.apk           #安装

adb shell pm install –r /data/local/tmp/test.apk       #重新安装


#### 2.卸载：


```
pm uninstall [options] <PACKAGE>

#卸载应用
```

| 参数 | 说明                                             |
| ---- | :----------------------------------------------- |
| -k   | 卸载应用且保留数据与缓存（如果不加-k则全部删除） |



### 五、控制命令


#### 1.清除应用数据：


```
pm clear <PACKAGE_NAME>
```


#### 2.禁用和启用应用：

```
pm
```

**只有系统应用才可以用，第三方应用不行**

| 子命令                                          | 说明                                             |
| ----------------------------------------------- | :----------------------------------------------- |
| enable <PACKAGE_OR_COMPONENT>                   | 使package或component可用                         |
| disenable <PACKAGE_OR_COMPONENT>                | 使package或component不可用（直接就找不到应用了） |
| disenable-user [options] <PACKAGE_OR_COMPONENT> | 使package或component不可用（会显示已停用）       |


#### 3.隐藏与恢复应用：

```
pm
```

**被隐藏应用在应用管理中变得不可见，桌面图标也会消失**

| 子命令                      | 说明                       |
| --------------------------- | :------------------------- |
| hide PACKAGE_OR_COMPONENT   | 隐藏package或component     |
| unhide PACKAGE_OR_CONPONENT | 恢复可见package或component |


#### 4.控制应用的默认安装位置：

```
pm
```

**需要root权限**

| 子命令                          | 说明                                                         |
| ------------------------------- | :----------------------------------------------------------- |
| set-install-location <LOCATION> | 更改默认的安装位置： <br>0：自动-让系统决定最好的位置 <br>1：内部存储-安装在内部设备上的存储 <br>2：外部存储-安装在外部媒体 <br>**注：只用于调试，不要瞎搞** |
| get-install-localtion           | 返回当前的安装位置 <br>0 <br>1 <br>2<br> 对应上面的数字说明  |