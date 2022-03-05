<h1 align="center">adb 常用命令</h1>

[toc]

## 基本指令

- `adb -s serialNumber shell` //进入指定设备
- `adb version` //查看版本
- `adb logcat` //查看日志
- `adb devices` //查看设备
- `adb get-state` //连接状态
- `adb start-server` //启动ADB服务
- `adb kill-server` //停止ADB服务
- `adb push local remote` //电脑推送到手机
- `adb pull remote local` //手机拉取到电脑
- `adb shell pm path` 包名 //adb 查找apk路径
- `adb shell ps` //查看所有进程
- `adb shell ps | grep <package-name>` //查看package_name程序进程
- `adb shell kill [PID]` //杀死进程
- `adb shell ps -x [PID]` //查看PID进程状态
- `adb shell top|grep <package_name>` //实时监听程序进程的变化
- `adb shell dumpsys package <package_name>` // 查看应用信息
- `adb connect 192.168.1.8` // 无线连接手机
- adb shell am start -N com.android.setting/com.android.setting.Settings // 启动设置Activity
- adb logcat -G 1M // 设置logcat缓存大小
- `adb backup -nosystem -noshared -apk -f 文件名 包名` // 导出文件（manifest文件要配置allowBackup=true）



## am与pm

- `am start -n {packageName}/.{activityName}` 启动app
- `am kill <packageName>` 杀app的进程
- `am force-stop <packageName>` 强制停止一切
- `am startservice` 启动服务
- `am stopservice` 停止服务
- `am start -a android.intent.action.VIEW -d http://www.12306.cn/ `打开12306网站
- `am start -a android.intent.action.CALL -d tel:10086` 拨打10086
- `pm list packages` 列出手机所有的包名
- `pm install/uninstall` 安装/卸载



## dumpsys

* `adb shell dumpsys window displays` //adb 显示屏幕信息
* `adb shell dumpsys activity` //adb 显示任务栈信息
* `adb shell dumpsys activity activities` //adb 显示任务栈信息(过滤掉其他信息)
* `adb shell dumpsys activity activities | sed -En -e '/Running activities/,/Run #0/p'` // 输出Running Activities列表
* `adb shell dumpsys batterystats --enable full-wake-history` // 清楚已有的耗电量数据
* `adb shell dumpsys batterystats --reset` // 设备耗电量数据重置
* `adb shell dumpsys window | grep -i mCur` // 查看当前显示的Activity



## 模拟用户事件

- **文本输入:** `adb shell input text <string>` 例手机端输出demo字符串，相应指令：`adb shell input "demo"`.
- **键盘事件：** `input keyevent <KEYCODE>`，其中`KEYCODE`见本文结尾的附表 例点击返回键，相应指令： `input keyevent 4`.
- **点击事件：** `input tap <x> <y>` 例点击坐标（500，500），相应指令： `input tap 500 500`.
- **滑动事件：** `input swipe <x1> <y1> <x2> <y2> <time>` 例从坐标(300，500)滑动到(100，500)，相应指令： `input swipe 300 500 100 500`. 例200ms时间从坐标(300，500)滑动到(100，500)，相应指令： `input swipe 300 500 100 500 200`.

## logcat

---

- `logcat \| grep <str>` 显示包含的logcat

- `logcat \| grep -i <str>` 显示包含，并忽略大小写的logcat

- logcat -s “ActivityManager” 显示该标签的log

- `logcat -d` 读完所有log后返回，而不会一直等待

- `logcat -c` 清空log并退出

- adb logcat | grep 关键字  `或者` adb logcat findStr 关键字

- `logcat -t <count>` 打印最近的count

   ```
   logcat -v <format>
   ```

   ， 格式化输出Log，其中

   ```plaintext
   format
   ```

   有如下可选值：

   - brief — 显示优先级/标记和原始进程的PID (默认格式)
   - process — 仅显示进程PID
   - tag — 仅显示优先级/标记
   - thread — 仅显示进程：线程和优先级/标记
   - raw — 显示原始的日志信息，没有其他的元数据字段
   - time — 显示日期，调用时间，优先级/标记，PID
   - long —显示所有的元数据字段并且用空行分隔消息内容

## 常用节点

----

> 查看节点值，例如：cat /sys/class/leds/lcd-backlight/brightness 修改节点值，例如：echo 128 > sys/class/leds/lcd-backlight/brightness

- LPM: echo N > /sys/modue/lpm_levels/parameters/sleep_disabled
- 亮度： /sys/class/leds/lcd-backlight/brightness
- CPU: /sys/devices/system/cpu/cpu0/cpufreq
- GPU: /sys/class/ kgsl/kgsl-3d0/gpuclk
- 限频： cat /data/pmlist.config
- 电流： cat /sys/class/power_supply/battery/current_now
- 查看Power： dumpsys power
- WIFI data/misc/wifi/wpa_supplicant.conf
- 持有wake_lock: echo a> sys/power/wake_lock 释放wake_lock: echo a> sys/power/wake_unlock
- 查看Wakeup_source: cat sys/kernel/debug/wakeup_sources
- SurfaceFlinger dumpsys SurfaceFlinger
- Display(关闭AD): mv /data/misc/display/calib.cfg /data/misc/display/calib.cfg.bak 重启
- 关闭cabc： echo 0 > /sys/device/virtual/graphics/fb0/cabc_onoff 打开cabc：echo 3 > /sys/device/virtual/graphics/fb0/cabc_onoff
- systrace： sdk/tools/monitor
- 限频： echo /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq 1497600
- 当出现read-only 且 remount命令不管用时： adb shell mount -o rw,remount /
- 进入9008模式： adb reboot edl
- 查看高通gpio：sys/class/private/tlmm 或者 sys/private/tlmm
- 查看gpio占用情况：sys/kernle/debug/gpio

## 无线ADB

----

> 为避免使用数据线，可通过wifi通信，前提是**手机与PC处于同一局域网**

**启动方法**：

```
adb tcpip 5555  //这一步，必须通过数据线把手机与PC连接后再执行
adb connect <手机IP>
```

**停止方法**：

```
adb disconnect //断开wifi连接
adb usb //切换到usb模式
```

## Keycode表

----

```
0 -->  "KEYCODE_UNKNOWN"
1 -->  "KEYCODE_MENU"
2 -->  "KEYCODE_SOFT_RIGHT"
3 -->  "KEYCODE_HOME"     //Home键
4 -->  "KEYCODE_BACK"     //返回键
5 -->  "KEYCODE_CALL"
6 -->  "KEYCODE_ENDCALL"
7 -->  "KEYCODE_0"       //数字键0
8 -->  "KEYCODE_1"
9 -->  "KEYCODE_2"
10 -->  "KEYCODE_3"
11 -->  "KEYCODE_4"
12 -->  "KEYCODE_5"
13 -->  "KEYCODE_6"
14 -->  "KEYCODE_7"
15 -->  "KEYCODE_8"
16 -->  "KEYCODE_9"
17 -->  "KEYCODE_STAR"
18 -->  "KEYCODE_POUND"
19 -->  "KEYCODE_DPAD_UP"
20 -->  "KEYCODE_DPAD_DOWN"
21 -->  "KEYCODE_DPAD_LEFT"
22 -->  "KEYCODE_DPAD_RIGHT"
23 -->  "KEYCODE_DPAD_CENTER"
24 -->  "KEYCODE_VOLUME_UP"    //音量键+
25 -->  "KEYCODE_VOLUME_DOWN"  //音量键-
26 -->  "KEYCODE_POWER"   //Power键
27 -->  "KEYCODE_CAMERA"
28 -->  "KEYCODE_CLEAR"
29 -->  "KEYCODE_A"    //字母键A
30 -->  "KEYCODE_B"
31 -->  "KEYCODE_C"
32 -->  "KEYCODE_D"
33 -->  "KEYCODE_E"
34 -->  "KEYCODE_F"
35 -->  "KEYCODE_G"
36 -->  "KEYCODE_H"
37 -->  "KEYCODE_I"
38 -->  "KEYCODE_J"
39 -->  "KEYCODE_K"
40 -->  "KEYCODE_L"
41 -->  "KEYCODE_M"
42 -->  "KEYCODE_N"
43 -->  "KEYCODE_O"
44 -->  "KEYCODE_P"
45 -->  "KEYCODE_Q"
46 -->  "KEYCODE_R"
47 -->  "KEYCODE_S"
48 -->  "KEYCODE_T"
49 -->  "KEYCODE_U"
50 -->  "KEYCODE_V"
51 -->  "KEYCODE_W"
52 -->  "KEYCODE_X"
53 -->  "KEYCODE_Y"
54 -->  "KEYCODE_Z"
55 -->  "KEYCODE_COMMA"
56 -->  "KEYCODE_PERIOD"
57 -->  "KEYCODE_ALT_LEFT"
58 -->  "KEYCODE_ALT_RIGHT"
59 -->  "KEYCODE_SHIFT_LEFT"
60 -->  "KEYCODE_SHIFT_RIGHT"
61 ->  "KEYCODE_TAB"
62 -->  "KEYCODE_SPACE"
63 -->  "KEYCODE_SYM"
64 -->  "KEYCODE_EXPLORER"
65 -->  "KEYCODE_ENVELOPE"
66 -->  "KEYCODE_ENTER"  //回车键
67 -->  "KEYCODE_DEL"
68 -->  "KEYCODE_GRAVE"
69 -->  "KEYCODE_MINUS"
70 -->  "KEYCODE_EQUALS"
71 -->  "KEYCODE_LEFT_BRACKET"
72 -->  "KEYCODE_RIGHT_BRACKET"
73 -->  "KEYCODE_BACKSLASH"
74 -->  "KEYCODE_SEMICOLON"
75 -->  "KEYCODE_APOSTROPHE"
76 -->  "KEYCODE_SLASH"
77 -->  "KEYCODE_AT"
78 -->  "KEYCODE_NUM"
79 -->  "KEYCODE_HEADSETHOOK"
80 -->  "KEYCODE_FOCUS"
81 -->  "KEYCODE_PLUS"
82 -->  "KEYCODE_MENU"
83 -->  "KEYCODE_NOTIFICATION"
84 -->  "KEYCODE_SEARCH"
```