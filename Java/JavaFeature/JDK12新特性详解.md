<h1 align="center">JDK12新特性详解</h1>

#### 1、JDK12之Shenandoah低暂停时间垃圾收集器（实验性）

   定义：

```
    添加一个名为Shenandoah的新垃圾收集（GC）算法，通过与正在运行的Java线程同时进行疏散工作
来减少GC暂停时间。使用Shenandoah的暂停时间与堆大小无关，这意味着无论堆是200MB还是200GB，都
将具有相同的一致暂停时间。
```

   非目标：

```
    这不是一个统治所有人的GC。还有其他垃圾收集算法可以优先考虑吞吐量或内存占用而不是响应性。
Shenandoah是适用于评估响应性和可预测的短暂停顿的应用程序的算法。目标不是解决所有JVM暂停问
题。由于GC之外的其他原因（例如安全时间点（TTSP）发布或监控通胀）而暂停时间超出了此JEP的范围。
```

   构建和调用：

```
    作为实验性功能，Shenandoah将-XX:+UnlockExperimentalVMOptions在命令行中要求。Shenandoah构建系统会自动禁用不受
支持的配置。下游建设者可以选择--with-jvm-features=-shenandoahgc在其他支持的平台上禁用构建Shenandoah。
要启用/使用Shenandoah GC，需要以下JVM选项：-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC
```



#### 2、JDK12之Microbenchmark Suite

   定义：

```
    在JDK源代码中添加一套基本的微基准测试，使开发人员可以轻松运行现有的微基准测试并创
建新的基准测试。
```

   目标：

```
1、基于[Java Microbenchmark线束（JMH）] [1]
2、稳定且经过调整的基准测试，针对持续性能测试
    2.1、在功能发布的功能完成里程碑之后，以及非功能版本之后的稳定且不移动的套件
    2.2、支持与先前JDK版本的适用测试比较
3、简单
    3.1 轻松添加新基准
    3.2 在API和选项更改，不推荐使用或在开发期间删除时，可以轻松更新测试
    3.3 易于构建
    3.4 易于查找和运行基准
    3.5 支持JMH更新
    3.6 在套件中包含大约一百个基准的初始集
```



#### 3、JDK12之Switch表达式（预览版）

   预览功能，如果该jdk12的switch效果不好的话，会被下一版本jdk13直接移除该功能，并不是之后每个版本必须的。

   许多break使它不必要地冗长，如果忘记写break，则会产生意料之外的结果或者异常，所以针对于此jdk12在这里进行了优化升级。

   JDK12之前的版本：  

```
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        System.out.println(6);
        break;
    case TUESDAY:
        System.out.println(7);
        break;
    case THURSDAY:
    case SATURDAY:
        System.out.println(8);
        break;
    case WEDNESDAY:
        System.out.println(9);
        break;
}
```

   JDK12:我们建议引入一种新形式的开关标签，写成“case L ->”表示如果标签匹配，则只执行标签右侧的代码。

```
switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
    case TUESDAY                -> System.out.println(7);
    case THURSDAY, SATURDAY     -> System.out.println(8);
    case WEDNESDAY              -> System.out.println(9);
}
```

   许多Switch表达式，每个case都会赋值给一个变量或者执行某种操作，如下是赋值给numLetters变量具体值。

   JDK12之前：

```
int numLetters;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        numLetters = 6;
        break;
    case TUESDAY:
        numLetters = 7;
        break;
    case THURSDAY:
    case SATURDAY:
        numLetters = 8;
        break;
    case WEDNESDAY:
        numLetters = 9;
        break;
    default:
        throw new IllegalStateException("Wat: " + day);
}
```

   JDK12:将此表达为一种陈述是迂回的，重复的，并且容易出错。意味着我们应该计算numLetters每一天的价值。应该可以直接说，使用更清晰，更安全Switch表达式

```
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};
```

   如果将它用在方法上，则可以为：

```
static void howMany(int k) {
    switch (k) {
        case 1 -> System.out.println("one");
        case 2 -> System.out.println("two");
        case 3 -> System.out.println("many");
    }
}
```

   或者类上，我想到的这个switch这个功能可以用在抽象工厂类方面。

```
T result = switch (arg) {
    case L1 -> e1;
    case L2 -> e2;
    default -> e3;
};
```



#### 4、JDK12之JVM常量API

   摘要：

```
引入API来模拟关键类文件和运行时工件的名义描述，特别是可从常量池加载的常量。
```

   动机：

```
    每个Java类文件都有一个常量池，用于存储类中字节码指令的操作数。从广义上讲，常量池中的条目描述了运行
时工件（如类和方法）或简单值（如字符串和整数）。所有这些条目都称为可加载常量，因为它们可以作为ldc指令的
操作数（“加载常量”）。
它们也可能出现在invokedynamic指令的bootstrap方法的静态参数列表中。执行一个ldc或invokedynamic指令
导致加载常数被解析成一个标准的Java类，如“活”的值Class，String或int。
    操作class文件的程序需要对字节码指令进行建模，并依次对可加载的常量进行建模。但是，使用标准Java类型
来模拟可加载常量是不合适的。
    描述字符串（CONSTANT_String_info条目）的可加载常量可能是可以接受的，因为生成“实时” String对象
很简单，但是对于描述类（CONSTANT_Class_info条目）的可加载常量是有问题的，因为产生“实时”ClassObject
依赖于类加载的正确性和一致性。
    不幸的是，类加载有许多环境依赖和失败模式：所需的类不存在或者请求者可能无法访问; 类加载的结果因上下文
而异; 装载类有副作用;有时类加载可能根本不可能（例如当所描述的类尚不存在或者无法加载时，如在编译那些相同类
或在jlink转换期间）。
    因此，处理可加载常量的程序如果能够以纯粹的名义符号形式操作类和方法，以及不太知名的工件（如方法句柄和动
态计算常量），则会更简单：
    1、字节码解析和生成库必须以符号形式描述类和方法句柄。如果没有标准机制，它们必须采用ad-hoc机制，无论是
ASM的描述符类型Handle，还是字符串元组（方法所有者，方法名称，方法描述符），或者这些机制的特殊（并且容易出
错）编码单个字符串。
    2、如果它们可以在符号域中工作而不是使用“实时”类和方法句柄，invokedynamic那么通过旋转字节码（例如LambdaMetafactory）来操作的Bootstraps 将更简单。
    3、编译器和脱机转换器（例如jlink插件）需要描述无法加载到正在运行的VM的类的类和成员。编译器插件（例如注
释处理器）同样需要用符号术语来描述程序元素。
```



#### 5、JDK12之一个AArch64端口，而不是两个

   摘要：

```
arm64在保留32位ARM端口和64位aarch64端口的同时，删除与端口相关的所有源。
```

   动机：

```
删除此端口将允许所有贡献者将他们的精力集中在单个64位ARM实现上，并消除维护两个端口所需的重复工作。
```



#### 6、JDK12之默认CDS档案

   摘要：

```
在64位平台上使用默认类列表增强JDK构建过程以生成类数据共享（CDS）归档。
```

   目标：

```
1、改善开箱即用的启动时间
2、消除用户运行-Xshare:dump以从CDS中受益的需要
```



#### 7、JDK12之G1的可流动混合收集

   摘要：

```
如果G1混合集合可能超过暂停目标，则使其可以中止。
```

   动机：

```
    G1的目标之一是满足用户提供的暂停时间目标以暂停其收集暂停。G1使用高级分析引擎来选择在集合期间
要完成的工作量（这部分基于应用程序行为）。此选择的结果是一组称为集合集的区域。一旦确定了集合集并且
已经开始集合，则G1必须在不停止的情况下收集集合集的所有区域中的所有活动对象。如果启发式选择过大的收
集，则此行为可导致G1超过暂停时间目标，例如，如果应用程序的行为发生变化，以致启发式工作在“陈旧”数据
上，则可能发生这种情况。特别是在混合集合期间可以观察到这种情况，其中集合集通常可以包含太多旧区域。
需要一种机制来检测启发式方法何时反复为集合选择错误的工作量，如果是，则让G1逐步递增地执行收集工作，
其中集合可以在每个步骤之后中止。这种机制将允许G1更频繁地满足暂停时间目标。
```



#### 8、JDK12之从G1中立即返回未使用的已提交内存

   摘要：

```
增强G1垃圾收集器，以便在空闲时自动将Java堆内存返回给操作系统。
```

   成功指标：

```
如果应用程序活动非常低，G1应该在合理的时间段内释放未使用的Java堆内存。
```

   动机：

```
    目前G1垃圾收集器可能无法及时将已提交的Java堆内存返回给操作系统。G1仅在完整GC或并发周期内
从Java堆返回内存。由于G1很难完全避免完整的GC，并且只触发基于Java堆占用和分配活动的并发周期，
因此在许多情况下它不会返回Java堆内存，除非在外部强制执行此操作。
    在使用资源支付的容器环境中，这种行为特别不利。即使在VM由于不活动而仅使用其分配的内存资源的
一小部分的阶段，G1也将保留所有Java堆。这导致客户始终为所有资源付费，云提供商无法充分利用其硬件。
    如果VM能够检测到Java堆的利用率不足（“空闲”阶段），并在此期间自动减少其堆使用量，则两者都
将受益。
    Shenandoah和OpenJ9的GenCon收集器已经提供了类似的功能。
```



#### 9、核心库java.lang中支持Unicode11

  JDK 12版本包括对Unicode 11.0.0的支持。在发布支持Unicode 10.0.0的JDK 11之后，Unicode 11.0.0引入了以下JDK 12中包含的新功能：

```
1、684个新角色
    1.1、66个表情符号字符
    1.2、Copyleft符号
    1.3、评级系统的半星
    1.4、额外的占星符号
    1.5、象棋中国象棋符号
2、11个新区块
    2.1、格鲁吉亚扩展
    2.2、玛雅数字
    2.3、印度Siyaq数字
    2.4、国际象棋符号
3、7个新脚本
    3.1、Hanifi Rohingya
    3.2、Old Sogdian
    3.3、Sogdian
    3.4、Dogra
    3.5、Gunjala Gondi
    3.6、Makasar
    3.7、Medefaidrin
```



#### 10、核心库java.text支持压缩数字格式

```
    NumberFormat添加了对以紧凑形式格式化数字的支持。紧凑数字格式是指以简短或人类可
读形式表示的数字。例如，在en_US语言环境中，1000可以格式化为“1K”，1000000可以格式化
为“1M”，具体取决于指定的样式NumberFormat.Style。紧凑数字格式由LDML的Compact 
Number格式规范定义。要获取实例，请使用NumberFormat紧凑数字格式所给出的工厂方法之一。    例如：NumberFormat fmt = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);
String result = fmt.format(1000);     上面的例子导致“1K”。
```



#### 12、安全库java.security

   禁止并允许java.security.manager系统属性的选项 

```
    新的“disallow”和“allow”令牌选项已添加到java.security.manager系统属性中。
在JDK实现中，如果Java虚拟机以系统属性java.security.manager设置为“disallow”开始，
则该System.setSecurityManager方法不能用于设置安全管理器并将抛出
UnsupportedOperationException。“disallow”选项可以提高从未设置安全管理器的应用程
序的运行时性能。
```

   groupname选项添加到keytool密钥对生成 

```
-groupname添加了一个新选项，keytool -genkeypair以便用户在生成密钥对时可以指
定命名组。例如，keytool -genkeypair -keyalg EC -groupname secp384r1将使用
secp384r1曲线生成EC密钥对。由于可能存在多个具有相同大小的曲线，因此使用该-groupname
选项优先于该-keysize选项。
```

   新Java飞行记录器（JFR）安全事件

```
略过...
```

   自定义PKCS12密钥库生成 

```
    添加了新的系统和安全属性，使用户能够自定义PKCS＃12密钥库的生成。这包括用于密钥保护，
证书保护和MacData的算法和参数。可以在文件的“PKCS12 KeyStore属性”部分中找到这些属性的
详细说明和可能的值java.security。
```



#### 13、安全库javax.net.ssl

   ChaCha20和Poly1305 TLS密码

```
    JSSE中添加了使用ChaCha20-Poly1305算法的新TLS密码套件。默认情况下启用这些密码套件。
TLS_CHACHA20_POLY1305_SHA256密码套件适用于TLS 1.3。
```



#### 14、安全库/ org.ietf.jgss

   在krb5.conf中支持dns_canonicalize_hostname

```
    该dns_canonicalize_hostname标志中krb5.conf的配置文件现在是由JDK Kerberos实现支持。
设置为“true”时，服务主体名称中的短主机名将被规范化为完全限定的域名（如果可用）。否则，不执行规
范化。默认值是true”。这也是JDK 12之前的行为。 
```



#### 15、删除项

   核心库/ java.util.jar中，删除java.util.ZipFile / Inflator / Deflator中的finalize方法 

```
    该finalize方法在java.util.ZipFile，java.util.Inflator和java.util.Deflator是在JDK9
弃用用于移除及其执行了更新，是一个空操作。该finalize在方法java.util.ZipFile，java.util.Inflator
以及java.util.Deflator在此版本中被删除。finalize应修改为执行清理而重写的子类，以使用备用清理机制并
删除写finalize方法。
    finalize方法，去除将暴露Object.finalize到的子类ZipFile，Deflater和Inflater。finalize由于
声明的异常发生更改，可能会在覆盖时发生编译错误。Object.finalize现在宣布投掷java.lang.Throwable。
以前，只有java.io.IOException宣布。
```

   核心库/ java.io，从FileInputStream和FileOutputStream中删除finalize方法 

```
    该finalize方法FileInputStream和FileOutputStream被弃用去除JDK 9.他们已经在此版本中被删除。
所述java.lang.ref.Cleaner，自JDK9的主要机制已被实施，以关闭文件描述符不再从可到达的FileInputStream和FileOutputStream。关闭文件的推荐方法是显式调用close或使用try-with-resources。
```

   工具/ javac的删除javac支持6 / 1.6源，目标和发布值 

```
为的javac的6/1.6的参数值的支持-source，-target以及--release标志已被删除。
```

source : https://my.oschina.net/mdxlcj/blog/3102739