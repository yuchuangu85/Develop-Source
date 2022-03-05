<h1 align="center">Proguard</h1>

[toc]

## Proguard

Proguard 具有以下三个功能：

- 压缩（Shrink）: 检测和删除没有使用的类，字段，方法和特性
- 优化（Optimize） : 分析和优化Java字节码
- 混淆（Obfuscate）: 使用简短的无意义的名称，对类，字段和方法进行重命名

## 规则

- 关键字

| 关键字                     | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| keep                       | 保留类和类中的成员，防止被混淆或移除                         |
| keepnames                  | 保留类和类中的成员，防止被混淆，成员没有被引用会被移除       |
| keepclassmembers           | 只保留类中的成员，防止被混淆或移除                           |
| keepclassmembernames       | 只保留类中的成员，防止被混淆，成员没有引用会被移除           |
| keepclasseswithmembers     | 保留类和类中的成员，防止被混淆或移除，保留指明的成员         |
| keepclasseswithmembernames | 保留类和类中的成员，防止被混淆，保留指明的成员，成员没有引用会被移除 |

 - 通配符

| 通配符     | 描述                                  |
| ---------- | ------------------------------------- |
| \<field\>  | 匹配类中的所有字段                    |
| \<method\> | 匹配类中所有的方法                    |
| \<init\>   | 匹配类中所有的构造函数                |
| *          | 匹配任意长度字符，不包含包名分隔符(.) |
| **         | 匹配任意长度字符，包含包名分隔符(.)   |
| ***        | 匹配任意参数类型                      |

 - 指定混淆时可使用字典

```
-applymapping filename 指定重用一个已经写好了的map文件作为新旧元素名的映射。
-obfuscationdictionary filename 指定一个文本文件用来生成混淆后的名字。
-classobfuscationdictionary filename 指定一个混淆类名的字典
-packageobfuscationdictionary filename 指定一个混淆包名的字典
-overloadaggressively 混淆的时候大量使用重载，多个方法名使用同一个混淆名（慎用）
```

## 公共模板

```
#############################################
#
# 对于一些基本指令的添加
#
#############################################
# 代码混淆压缩比，在 0~7 之间，默认为 5，一般不做修改
-optimizationpasses 5

# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames

# 指定不去忽略非公共库的类
-dontskipnonpubliclibraryclasses

# 这句话能够使我们的项目混淆后产生映射文件
# 包含有类名->混淆后类名的映射关系
-verbose

# 指定不去忽略非公共库的类成员
-dontskipnonpubliclibraryclassmembers

# 不做预校验，preverify 是 proguard 的四个步骤之一，Android 不需要 preverify，去掉这一步能够加快混淆速度。
-dontpreverify

# 保留 Annotation 不混淆
-keepattributes *Annotation*,InnerClasses

# 避免混淆泛型
-keepattributes Signature

# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable

# 指定混淆是采用的算法，后面的参数是一个过滤器
# 这个过滤器是谷歌推荐的算法，一般不做更改
-optimizations !code/simplification/cast,!field/*,!class/merging/*


#############################################
#
# Android开发中一些需要保留的公共部分
#
#############################################

# 保留我们使用的四大组件，自定义的 Application 等等这些类不被混淆
# 因为这些子类都有可能被外部调用
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Appliction
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View
-keep public class com.android.vending.licensing.ILicensingService


# 保留 support 下的所有类及其内部类
-keep class android.support.** { *; }

# 保留继承的
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**

# 保留 R 下面的资源
-keep class **.R$* { *; }

# 保留本地 native 方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留在 Activity 中的方法参数是view的方法，
# 这样以来我们在 layout 中写的 onClick 就不会被影响
-keepclassmembers class * extends android.app.Activity {
    public void *(android.view.View);
}

# 保留枚举类不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保留我们自定义控件（继承自 View）不被混淆
-keep public class * extends android.view.View {
    *** get*();
    void set*(***);
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 保留 Parcelable 序列化类不被混淆
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# 保留 Serializable 序列化的类不被混淆
-keepnames class * implements java.io.Serializable
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# 对于带有回调函数的 onXXEvent、**On*Listener 的，不能被混淆
-keepclassmembers class * {
    void *(**On*Event);
    void *(**On*Listener);
}

# webView 处理，项目中没有使用到 webView 忽略即可
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
    public *;
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.webView, java.lang.String);
}

# js
-keepattributes JavascriptInterface
-keep class android.webkit.JavascriptInterface { *; }
-keepclassmembers class * {
    @android.webkit.JavascriptInterface <methods>;
}

# @Keep
-keep,allowobfuscation @interface android.support.annotation.Keep
-keep @android.support.annotation.Keep class *
-keepclassmembers class * {
    @android.support.annotation.Keep *;
}
```

##  常用的自定义混淆规则

```xml
# 通配符*，匹配任意长度字符，但不含包名分隔符(.)
# 通配符**，匹配任意长度字符，并且包含包名分隔符(.)

# 不混淆某个类
-keep public class com.jasonwu.demo.Test { *; }

# 不混淆某个包所有的类
-keep class com.jasonwu.demo.test.** { *; }

# 不混淆某个类的子类
-keep public class * com.jasonwu.demo.Test { *; }

# 不混淆所有类名中包含了 ``model`` 的类及其成员
-keep public class **.*model*.** {*;}

# 不混淆某个接口的实现
-keep class * implements com.jasonwu.demo.TestInterface { *; }

# 不混淆某个类的构造方法
-keepclassmembers class com.jasonwu.demo.Test { 
  public <init>(); 
}

# 不混淆某个类的特定的方法
-keepclassmembers class com.jasonwu.demo.Test { 
  public void test(java.lang.String); 
}
```


## aar中增加独立的混淆配置

``build.gralde``

```gradle
android {
    ···
    defaultConfig {
        ···
        consumerProguardFile 'proguard-rules.pro'
    }
    ···
}
```

## 检查混淆和追踪异常

开启 Proguard 功能，则每次构建时 ProGuard 都会输出下列文件：

- dump.txt  
  说明 APK 中所有类文件的内部结构。

- mapping.txt  
  提供原始与混淆过的类、方法和字段名称之间的转换。

- seeds.txt  
  列出未进行混淆的类和成员。

- usage.txt  
  列出从 APK 移除的代码。

这些文件保存在 /build/outputs/mapping/release/ 中。我们可以查看 seeds.txt 里面是否是我们需要保留的，以及 usage.txt 里查看是否有误删除的代码。 mapping.txt 文件很重要，由于我们的部分代码是经过重命名的，如果该部分出现 bug，对应的异常堆栈信息里的类或成员也是经过重命名的，难以定位问题。我们可以用 retrace 脚本（在 Windows 上为 retrace.bat；在 Mac/Linux 上为 retrace.sh）。它位于 /tools/proguard/ 目录中。该脚本利用 mapping.txt 文件和你的异常堆栈文件生成没有经过混淆的异常堆栈文件,这样就可以看清是哪里出问题了。使用 retrace 工具的语法如下：

```shell
retrace.bat|retrace.sh [-verbose] mapping.txt [<stacktrace_file>]
```
