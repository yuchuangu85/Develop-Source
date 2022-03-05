<h1 align="center">Intent</h1>

[toc]

## Intent 属性

Intent 的属性有：component(组件)、action、category、data、type、extras、flags；所有的属性也是各显神通，满足开发者的各种需要满足不同场景； **component:** 显然就是设置四大组件的，将直接使用它指定的组件，借助这一属性可以实现不同应用组件之间通讯；

**action：** 是一个可以指定目标组件行为的字符串，开发人员可以自定义 action 通过匹配 action 实现组件之间的隐士跳转，当然 Android 系统也已经预定部分 String 作为系统应用 Action，例如打开系统设置页面等等；

**data：**通常是 URI 类型或者 MIME 类型格式定义的操作数据；表示与动作要操纵的数据

* data中常见的MIME类型数据；

   tel: 号码数据格式，后跟电话号码

   mailto: 邮件数据格式，后跟邮件收件人地址

   smsto: 短信数据格式，后跟短信接收号码

   content: 内容数据格式，后跟需要读取的内容

   file: 文件数据格式，后跟文件路径

   market: search?q=pname:pkgname: 市场数据格式，在Google Market里搜

* data中URI

   data元素组成的URI模型: scheme://host:port/path 例如：file://com.android.jony.test:520/mnt/sdcard

**Category：** 属性用于指定当前动作（Action）被执行的环境；

**type：** 对于 data 范例的描写；

**extras 和 flags** 这两个太熟悉了就不在重复；

 

