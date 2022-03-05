<h1 align="center">Systrace教程</h1>

[TOC]

## Systrace

### Systrace位置

`systrace` 命令在 Android SDK 工具软件包中提供，并且可以在 `android-sdk/platform-tools/systrace/` 中找到。

### 支持系统版本

Android 4.3（API 级别 18）及以上

## 语法

如需为应用生成 HTML 报告，您需要使用以下语法通过命令行运行 `systrace`：

```bash
$ python systrace.py [options] [categories]
// macOS一般同时安装了Python3，所以命令为
$ python2.7 systrace.py [options] [categories]
或者
$ python2.7 systrace.py [options] [category1] [category2] ... [categoryN]
```

例如，以下命令会调用 `systrace` 来记录设备活动，并生成一个名为 `mynewtrace.html` 的 HTML 报告。此类别列表是大多数设备的合理默认列表。

```bash
$ python systrace.py -o mynewtrace.html sched freq idle am wm gfx view \
        binder_driver hal dalvik camera input res
// macOS一般同时安装了Python3，所以命令为
$ python2.7 systrace.py -o mynewtrace.html sched freq idle am wm gfx view \
        binder_driver hal dalvik camera input res
```

**提示**：如果要在跟踪输出中查看任务名称，必须在命令参数中添加 `sched` 类别。

如需查看已连接设备支持的类别列表，请运行以下命令：

```bash
$ python systrace.py --list-categories
// macOS一般同时安装了Python3，所以命令为
$ python2.7 systrace.py --list-categories
```

如果您未指定任何类别或选项，`systrace` 会生成包含所有可用类别的报告，并使用默认设置。可用类别取决于您所使用的已连接设备。

## 命令

### 全局选项

| 全局选项                 | 说明                               |
| :----------------------- | :--------------------------------- |
| `-h | --help`            | 显示帮助消息。                     |
| `-l | --list-categories` | 列出您的已连接设备可用的跟踪类别。 |

### 命令和命令选项

| 命令和选项                                | 说明                                                         |
| :---------------------------------------- | :----------------------------------------------------------- |
| `-o file`                                 | 将 HTML 跟踪报告写入指定的文件。如果您未指定此选项，`systrace` 会将报告保存到 `systrace.py` 所在的目录中，并将其命名为 `trace.html`。 |
| `-t N | --time=N`                         | 跟踪设备活动 N 秒。如果您未指定此选项，`systrace` 会提示您在命令行中按 Enter 键结束跟踪。 |
| `-b N | --buf-size=N`                     | 使用 N KB 的跟踪缓冲区大小。使用此选项，您可以限制跟踪期间收集到的数据的总大小。 |
| `-k functions|--ktrace=functions`         | 跟踪逗号分隔列表中指定的特定内核函数的活动。                 |
| `-a app-name|--app=app-name`              | 启用对应用的跟踪，指定为包含[进程名称](https://developer.android.com/guide/topics/manifest/application-element#proc)的逗号分隔列表。这些应用必须包含 `Trace` 类中的跟踪检测调用。您应在分析应用时指定此选项。很多库（例如 `RecyclerView`）都包括跟踪检测调用，这些调用可在您启用应用级跟踪时提供有用的信息。如需了解详情，请参阅[定义自定义事件](https://developer.android.com/topic/performance/tracing/custom-events)。如需跟踪搭载 Android 9（API 级别 28）或更高版本的设备上的所有应用，请传递用添加引号的通配符字符 `"*"`。 |
| `--from-file=file-path`                   | 根据文件（例如包含原始跟踪数据的 TXT 文件）创建交互式 HTML 报告，而不是运行实时跟踪。 |
| `-e device-serial|--serial=device-serial` | 在已连接的特定设备（由对应的[设备序列号](https://developer.android.com/studio/command-line/adb#devicestatus)标识）上进行跟踪。 |
| `categories`                              | 包含您指定的系统进程的跟踪信息，如 `gfx` 表示用于渲染图形的系统进程。您可以使用 `-l` 命令运行 `systrace`，以查看已连接设备可用的服务列表。 |

### options

| options                                        | 解释                                                |
| :--------------------------------------------- | :-------------------------------------------------- |
| -o `<FILE`>                                    | 输出的目标文件                                      |
| -t N, –time=N                                  | 执行时间，默认5s                                    |
| -b N, –buf-size=N                              | buffer大小（单位kB),用于限制trace总大小，默认无上限 |
| -k `<KFUNCS`>，–ktrace=`<KFUNCS`>              | 追踪kernel函数，用逗号分隔                          |
| -a `<APP_NAME`>,–app=`<APP_NAME`>              | 追踪应用包名，用逗号分隔                            |
| –from-file=`<FROM_FILE`>                       | 从文件中创建互动的systrace                          |
| -e `<DEVICE_SERIAL`>,–serial=`<DEVICE_SERIAL`> | 指定设备                                            |
| -l, –list-categories                           | 列举可用的tags                                      |

### category

| category   | 解释                           | 备注 |
| :--------- | :----------------------------- | ---- |
| gfx        | Graphics                       |      |
| input      | Input                          |      |
| view       | View System                    |      |
| webview    | WebView                        |      |
| wm         | Window Manager                 |      |
| am         | Activity Manager               |      |
| sm         | Sync Manager                   |      |
| audio      | Audio                          |      |
| video      | Video                          |      |
| camera     | Camera                         |      |
| hal        | Hardware Modules               |      |
| app        | Application                    |      |
| res        | Resource Loading               |      |
| dalvik     | Dalvik VM                      |      |
| rs         | RenderScript                   |      |
| bionic     | Bionic C Library               |      |
| power      | Power Management               |      |
| sched      | CPU Scheduling                 |      |
| irq        | IRQ Events                     |      |
| freq       | CPU Frequency                  |      |
| idle       | CPU Idle                       |      |
| disk       | Disk I/O                       |      |
| mmc        | eMMC commands                  |      |
| load       | CPU Load                       |      |
| sync       | Synchronization                |      |
| workq      | Kernel Workqueues              |      |
| memreclaim | Kernel Memory Reclaim          |      |
| regulators | Voltage and Current Regulators |      |

实例：

```bash
// <-o mynewtrace.html>：输出文件 
$ python2.7 systrace.py -o mynewtrace.html sched freq idle am wm gfx view \
        binder_driver hal dalvik camera input res
// <-t 10>：执行时间；
$ python2.7 systrace.py -t 10 sched gfx view wm am app webview -a <package-name>
// <-o out.html>：输出文件
$ python2.7 systrace.py -t 5 sched gfx input view webview wm am app network -a <pkg-name> -o out.html
```





## 快捷操作

### 导航操作

| 导航操作 | 作用                   |
| :------- | :--------------------- |
| w        | 放大，[+shift]速度更快 |
| s        | 缩小，[+shift]速度更快 |
| a        | 左移，[+shift]速度更快 |
| d        | 右移，[+shift]速度更快 |

### 快捷操作

| 常用操作 | 作用                                        |
| :------- | :------------------------------------------ |
| f        | **放大**当前选定区域                        |
| m        | **标记**当前选定区域                        |
| v        | 高亮**VSync**                               |
| g        | 切换是否显示**60hz**的网格线                |
| 0        | 恢复trace到**初始态**，这里是数字0而非字母o |

| 一般操作 | 作用                                |
| :------- | :---------------------------------- |
| h        | 切换是否显示详情                    |
| /        | 搜索关键字                          |
| enter    | 显示搜索结果，可通过← →定位搜索结果 |
| `        | 显示/隐藏脚本控制台                 |
| ?        | 显示帮助功能                        |

对于脚本控制台，除了能当做记事本的功能，目前还不清楚有啥功能，或许还在开发中。

### 模式切换

1. Select mode: **双击已选定区**能将所有相同的块高亮选中；（对应数字1）
2. Pan mode: 拖动平移视图（对应数字2）
3. Zoom mode:通过上/下拖动鼠标来实现放大/缩小功能；（对应数字3）
4. Timing mode:拖动来创建或移除时间窗口线。（对应数字4）

可通过按数字1~4，用于切换鼠标模式； 另外，按住alt键，再滚动鼠标滚轮能实现放大/缩小功能

## 分析工具

用**Chrome**浏览器或者**Edge**浏览器直接打开生成的systrace.html文件或者用谷歌提供的 **[Perfetto](https://ui.perfetto.dev/#!/viewer)** 工具



## Systrace教程

1. [Systrace 简介](https://www.androidperformance.com/2019/05/28/Android-Systrace-About/)
2. [Systrace 基础知识 - Systrace 预备知识](https://www.androidperformance.com/2019/07/23/Android-Systrace-Pre/)
3. [Systrace 基础知识 - Why 60 fps ？](https://www.androidperformance.com/2019/05/27/why-60-fps/)
4. [Systrace 基础知识 - SystemServer 解读](https://www.androidperformance.com/2019/06/29/Android-Systrace-SystemServer/)
5. [Systrace 基础知识 - SurfaceFlinger 解读](https://www.androidperformance.com/2020/02/14/Android-Systrace-SurfaceFlinger/)
6. [Systrace 基础知识 - Input 解读](https://www.androidperformance.com/2019/11/04/Android-Systrace-Input/)
7. [Systrace 基础知识 - Vsync 解读](https://www.androidperformance.com/2019/12/01/Android-Systrace-Vsync/)
8. [Systrace 基础知识 - Vsync-App ：基于 Choreographer 的渲染机制详解](https://androidperformance.com/2019/10/22/Android-Choreographer/)
9. [Systrace 基础知识 - MainThread 和 RenderThread 解读](https://www.androidperformance.com/2019/11/06/Android-Systrace-MainThread-And-RenderThread/)
10. [Systrace 基础知识 - Binder 和锁竞争解读](https://www.androidperformance.com/2019/12/06/Android-Systrace-Binder/)
11. [Systrace 基础知识 - Triple Buffer 解读](https://www.androidperformance.com/2019/12/15/Android-Systrace-Triple-Buffer)
12. [Systrace 基础知识 - CPU Info 解读](https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU)
13. Systrace 实战 - 分析应用冷热启动时间问题
14. Systrace 实战 - 分析卡顿问题

## 参考

* [性能工具Systrace](http://gityuan.com/2016/01/17/systrace/)
* [手把手教你使用Systrace（一）](https://zhuanlan.zhihu.com/p/27331842)
* [手把手教你使用Systrace（二）——锁优化](https://zhuanlan.zhihu.com/p/27535205)
* [在命令行上捕获系统跟踪记录](https://developer.android.com/topic/performance/tracing/command-line)--官方

