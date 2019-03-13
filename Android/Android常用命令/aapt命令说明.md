AAPT定义：

aapt即Android Asset Packaging Tool, 在SDK的build-tools目录下。该工具可以查看、创建、更新ZIP格式的文档附件（zip,jar,apk）。也可将资源文件编译成二进制文件，尽管你可能没有直接使用aapt工具，但是build scripts和IDE插件会使用这个工具打包apk文件构成一个Android应用程序。在使用aapt之前需要在环境变量里面配置SDK-tools路径，或者是路径+aapt的方式进入aapt。

### 一.列出apk包的内容


```
aapt l[ist] [-v] [-a] <你的应用>
```

-v 以table形式列出来

-a 详细列出内容

例如：aapt l <你的apk文件>，这个命令就是查看apk内容

 

### 二. 查看apk一些信息

```
aapt d[ump] [选项] <你的应用>

这里可以输入全称dump，也可以直接用d代替。
```

| 选项           | 说明                                                         | 例如                                               |
| -------------- | :----------------------------------------------------------- | :------------------------------------------------- |
| badging        | 查看apk包的packageName、versionCode、applicationLabel、launcherActivity、permission等各种详细信息 | aapt dump badging <file_path.apk>                  |
| permissions    | 查看权限                                                     | aapt dump resources <file_path.apk>                |
| resources      | 查看资源列表                                                 | aapt dump resources <file_path.apk>   > sodino.txt |
| configurations | 查看apk配置信息                                              | aapt dump configurations <file_path.apk>           |
| xmltree        | 以树形结构输出的xml信息。                                    | aapt dump xmltree <file_path.apk> res/***.xml      |
| xmlstrings     | 查看指定apk的指定xml文件。                                   | aapt dump xmlstrings <file_path.apk> res/***.xml   |


#### TIP：由于我们工作中需要使用badging参数来查看versioncode，因此可以使用命令aapt dump badging <file_path.apk> | findstr “versionCode”来查看


### 三.编译android资源

```
aapt p[ackage] [-d][-f][-m][-u][-v][-x][-z][-M AndroidManifest.xml] / 
        [-0 extension [-0 extension ...]] [-g tolerance] [-j jarfile] / 
        [--debug-mode] [--min-sdk-version VAL] [--target-sdk-version VAL] / 
        [--app-version VAL] [--app-version-name TEXT] [--custom-package VAL] / 
        [--rename-manifest-package PACKAGE] / 
        [--rename-instrumentation-target-package PACKAGE] / 
        [--utf16] [--auto-add-overlay] / 
        [--max-res-version VAL] / 
        [-I base-package [-I base-package ...]] / 
        [-A asset-source-dir]  [-G class-list-file] [-P public-definitions-file] / 
        [-S resource-sources [-S resource-sources ...]]         [-F apk-file] [-J R-file-dir] / 
        [--product product1,product2,...] / 
        [raw-files-dir [raw-files-dir] ...]
```

这个比较复杂，只解释几个关键参数。

-f 如果编译出来的文件已经存在，强制覆盖。

-m 使生成的包的目录放在-J参数指定的目录。

-J 指定生成的R.java的输出目录

-S res文件夹路径

-A assert文件夹的路径

-M AndroidManifest.xml的路径

-I 某个版本平台的android.jar的路径

-F 具体指定apk文件的输出

### 四.打包好的apk中移除文件

```
aapt r[emove] [-v] file.{zip,jar,apk} file1 [file2 ...]
```

例如：aapt r <你的apk文件> AndroidManifest.xml， 这个就是将apk中的AndroidManifest移除掉

### 五. 添加文件到打包好的apk中

```
aapt a[dd] [-v] file.{zip,jar,apk} file1 [file2 ...]
```

例如：aapt a <你的apk文件> <要添加的文件路径>， 这个就是将文件添加到打包好的apk文件中

### 六.显示aapt的版本

```
aapt v[ersion]
```

例如：aapt v， 就是打印这个结果 Android Asset Packaging Tool, v0.2