## adb 卸载系统自动应用

```
// 查看当前应用包名
adb shell dumpsys window | grep mCurrentFocus

// 查看全部应用列表
adb shell pm list packages --user 0

// 卸载应用（shell后面的都用双引号引起来）
adb shell pm uninstall [-k] [--user USER_ID] 包名
```

说明：

* -k 卸载应用保留数据与缓存，如果不加-k则全部删除
* --user 指定用户id，Android系统支持多个用户，默认用户只有一个，id=0

实例：

```
adb shell "pm uninstall -k --user 0 com.qihoo.brower"
```



shell脚本：

```shell
r = $(adb shell dumpsys window | grep 'mCurrentFocus=')
r = ${r##*}
r = ${r%%/*}
echo "packageName is $r"
adb shell pm uninstall --user 0 ${r}
```

