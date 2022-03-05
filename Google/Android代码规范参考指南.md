# Android代码规范参考指南

代码规范对于一个软件项目来说非常重要，当然Android项目也不例外，一个优秀的Android项目不仅需要严谨的业务逻辑和架构设计，更需要一套统一优雅的代码规范标准，才可以让整个项目团队更加高效。

**包命名规范**

包（packages）：采用反域名命名规则，全部使用小写字母。一级包名为com，二级包名为xxx（可以是公司域名或者个人命名），三级包名根据应用进行命名，四级包名为模块名或层级名。

| 包名                                                    | 说明                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| com.xxx.应用名称缩写.activities                         | 页面用到的Activity类（activities层级用户界面）               |
| com.xxx.应用名称缩写.fragment                           | 页面用到的Fragment类                                         |
| com.xxx.应用名称缩写.base                               | 页面中每个Activity类共享的可以写成一个BaseActivity类（基础共享的类） |
| com.xxx.应用名称缩写.adapter                            | 页面用到的Adapter类（适配器的类）                            |
| com.xxx.应用名称缩写.utils                              | 此包中包含：公共工具方法类（包含日期、网络、存储、日志等工具类） |
| com.xxx.应用名称缩写.bean（model/domain均可，个人喜好） | 实体类                                                       |
| com.xxx.应用名称缩写.db                                 | 数据库操作                                                   |
| com.xxx.应用名称缩写.view（或者.ui）                    | 自定义的View类等                                             |
| com.xxx.应用名称缩写.service                            | Service服务                                                  |
| com.xxx.应用名称缩写.broadcast                          | Broadcast服务                                                |

**类命名规范**

类（classes）：名词，采用大驼峰命名法，尽量避免缩写，除非该缩写是众所周知的，比如HTML，URL,如果类名称包含单词缩写，则单词缩写的每个字母均应大写。

| 类                  | 描述                     | 例如                                                |
| ------------------- | ------------------------ | --------------------------------------------------- |
| Application类       | Application为后缀标识    | XXXApplication                                      |
| Activity类          | Activity为后缀标识       | 闪屏页面类SplashActivity                            |
| 解析类              | Handler为后缀标识        |                                                     |
| 公共方法类          | Utils或Manager为后缀标识 | 线程池管理类：ThreadPoolManager日志工具类：LogUtils |
| 数据库类            | 以DBHelper后缀标识       | MySQLiteDBHelper                                    |
| Service类           | 以Service为后缀标识      | 播放服务：PlayService                               |
| BroadcastReceiver类 | 以Broadcast为后缀标识    | 时间通知：TimeBroadcast                             |
| ContentProvider类   | 以Provider为后缀标识     | 单词内容提供者：DictProvider                        |
| 直接写的共享基础类  | 以Base为前缀             | BaseActivity,BaseFragment                           |

**变量命名规范**

变量（variables）采用小驼峰命名法。类中控件名称必须与xml布局id保持一致。

公开的常量：定义为静态final，名称全部大写。eg: public staticfinal String ACTION_MAIN=”android.intent.action.MAIN”;

静态变量：名称以s开头 eg：private static long sInstanceCount = 0;

非静态的私有变量、protected的变量：以m开头，eg：private Intent mItent;

**接口命名规范**

接口（interface）：命名规则与类一样采用大驼峰命名法，多以able或ible结尾，eg：interface Runable; interface Accessible;

**方法命名规范**

方法（methods）：动词或动名词，采用小驼峰命名法，eg：onCreate(),run();

| 方法        | 说明                                                       |
| ----------- | ---------------------------------------------------------- |
| initXX()    | 初始化相关方法，使用init为前缀标识，如初始化布局initView() |
| isXX()      | checkXX()方法返回值为boolean型的请使用is或check为前缀标识  |
| getXX()     | 返回某个值的方法，使用get为前缀标识                        |
| processXX() | 对数据进行处理的方法，尽量使用process为前缀标识            |
| displayXX() | 弹出提示框和提示信息，使用display为前缀标识                |
| saveXX()    | 与保存数据相关的，使用save为前缀标识                       |
| resetXX()   | 对数据重组的，使用reset前缀标识                            |
| clearXX()   | 清除数据相关的                                             |
| removeXX()  | 清除数据相关的                                             |
| drawXXX()   | 绘制数据或效果相关的，使用draw前缀标识                     |

**布局文件命名规范**

全部小写，采用下划线命名法

1)．contentview命名, Activity默认布局，以去掉后缀的Activity类进行命名。不加后缀：

功能模块.xml

eg：main.xml、more.xml、settings.xml

或者：activity_功能模块.xml

eg：activity_main.xml、activity_more.xml

2)．Dialog命名：dialog_描述.xml

eg：dlg_hint.xml

3)．PopupWindow命名：ppw_描述.xml

eg：ppw_info.xml

4). 列表项命名listitem_描述.xml

eg：listitem_city.xml

5)．包含项：include_模块.xml

eg：include_head.xml、include_bottom.xml

6)．adapter的子布局：功能模块_item.xml

eg：main_item.xml、

**资源id命名规范**

命名模式为：view缩写_模块名称_view的逻辑名称

view的缩写详情如下：

| 控件                 | 缩写       |
| -------------------- | ---------- |
| LayoutView           | lv         |
| RelativeView         | rv         |
| TextView             | tv         |
| Button               | btn        |
| ImageButton          | imgBtn     |
| ImageView            | iv         |
| CheckBox             | cb         |
| RadioButton          | rb         |
| analogClock          | anaClk     |
| DigtalClock          | dgtClk     |
| DatePicker           | dtPk       |
| EditText             | edtTxt     |
| TimePicker           | tmPk       |
| toggleButton         | tglBtn     |
| ProgressBar          | proBar     |
| SeekBar              | skBar      |
| AutoCompleteTextView | autoTxt    |
| ZoomControls         | zmCtl      |
| VideoView            | vdoVi      |
| WdbView              | webVi      |
| RantingBar           | ratBar     |
| Tab                  | tab        |
| Spinner              | spn        |
| Chronometer          | cmt        |
| ScollView            | sclVi      |
| TextSwitch           | txtSwt     |
| ImageSwitch          | imgSwt     |
| listView             | lVi 或则lv |
| ExpandableList       | epdLt      |
| MapView              | mapVi      |

**动画文件命名**

动画文件（anim文件夹下）：全部小写，采用下划线命名法，加前缀区分。

//前面为动画的类型，后面为方向

| 动画命名例子      | 规范写法       | 备注 |
| ----------------- | -------------- | ---- |
| fade_in           | 淡入           |      |
| fade_out          | 淡出           |      |
| push_down_in      | 从下方推入     |      |
| push_down_out     | 从下方推出     |      |
| push_left         | 推像左方       |      |
| slide_in_from_top | 从头部滑动进入 |      |
| zoom_enter        | 变形进入       |      |
| slide_in          | 滑动进入       |      |
| shrink_to_middle  | 中间缩小       |      |

**图片资源文件命名**

| 命名                   | 说明                                                       |
| ---------------------- | ---------------------------------------------------------- |
| bg_xxx                 | 这种图片一般那些比较大的图片，比如作为某个Activity的背景等 |
| btn_xxx                | 按钮，一般用于按钮，而且这种按钮没有其他状态               |
| ic_xxx                 | 图标，一般用于单个图标，比如启动图片ic_launcher            |
| bg_描述_状态1[_状态2]  | 用于控件上的不同状态                                       |
| btn_描述_状态1[_状态2] | 用于按钮上的不同状态                                       |
| chx_描述_状态1[_状态2] | 选择框，一般有2态和4态                                     |
|                        |                                                            |

**一些常见的单词缩写**

| 名称                 | 缩写                                                         |
| -------------------- | ------------------------------------------------------------ |
| icon                 | ic （主要用在app的图标）                                     |
| color                | cl（主要用于颜色值）                                         |
| divider              | di（主要用于分隔线，不仅包括Listview中的divider，还包括普通布局中的线） |
| selector             | sl（主要用于某一view多种状态，不仅包括Listview中的selector，还包括按钮的selector） |
| average              | avg                                                          |
| background           | Bg（主要用于布局和子布局的背景）                             |
| buffer               | buf                                                          |
| control              | ctrl                                                         |
| delete               | del                                                          |
| document             | doc                                                          |
| error                | err                                                          |
| escape               | esc                                                          |
| increment            | inc                                                          |
| infomation           | info                                                         |
| initial              | init                                                         |
| image                | img                                                          |
| Internationalization | I18N                                                         |
| length               | len                                                          |
| library              | lib                                                          |
| message              | msg                                                          |
| password-            | pwd                                                          |
| position             | pos                                                          |
| server               | srv                                                          |
| string               | str                                                          |
| temp                 | tmp                                                          |
| window               | wnd(win)                                                     |