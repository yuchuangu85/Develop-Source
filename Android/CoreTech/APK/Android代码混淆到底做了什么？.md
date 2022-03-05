<h1 align="center">Android | 代码混淆到底做了什么？</h1>

[TOC]

## 前言

代码混淆对于每个入门的 Android 工程师来说都不会太陌生，因为在编译正式版本时，这是一个必不可少的过程。而且使用代码混淆也相当简单，简单到只需要配置一句`minifyEnabled true`。但是你是否理解混淆的原理，如果问你代码混淆到底做了什么，你会怎么说？

------

## 目录

![img](media/b63debd6ab294db7a9f15e273ac6c3f5~tplv-k3u1fbpfcp-watermark.image)

------

## 1. 混淆编译器

如果以混淆编译器来划分的话，Android 代码混淆可以分为以下两个时期：

- ProGuard：一个通用的 Java 字节码优化工具，由比利时团队 GuardSquare 开发
- R8：ProGuard 的继承者，专为 Android 设计，编译性能和编译产物更优秀

下图梳理了它们随着 [Android Gradle Plugin](https://developer.android.google.cn/studio/releases/gradle-plugin) 版本迭代相应做出的变更：

![Android Gradle Plugin 版本迭代](media/91f70a8db8094d48abb72b75715d0eda~tplv-k3u1fbpfcp-watermark.image)

其中，混淆编译器的变更：

- 远古： ProGuard
- [3.2.0](https://developer.android.google.cn/studio/releases/gradle-plugin#3-2-0)：ProGuard（默认），R8（引入）
- [3.4.0](https://developer.android.google.cn/studio/releases/gradle-plugin#3-4-0)：R8（默认）

其中：DEX编译器的变更：

- 远古： DX
- [3.0.0](https://developer.android.google.cn/studio/releases#preview-the-new-d8-dex-compiler)：DX（默认），D8（引入）
- [3.1.0](https://developer.android.google.cn/studio/releases/gradle-plugin#3-1-0)：D8（默认）

如果需要修正 Android Gradle Plugin 的默认行为，可以在`gradle.properties`中添加配置：

- 启用与禁用 R8

  ```
  # 显式启用 R8
  android.enableR8 = true
  ```

  ```
  # 1. 只对 Android Library module 停用 R8 编译器
  android.enableR8.libraries = false
  # 2. 对所有 module 停用 R8 编译器
  android.enableR8 = false
  ```

- 启用与禁用 D8

  ```
  # 显式启用 D8
  android.enableD8 = true
  ```

  ```
  # 显式禁用 D8
  android.enableD8 = false
  ```

另外，如果在应用模块的 `build.gradle` 文件中设置`useProguard = false`，也会使用 R8 编译器代替 ProGuard。

------

## 2. 四大功能

ProGuard 与 R8 都提供了压缩（shrinker）、优化（optimizer）、混淆（obfuscator）、预校验（preverifier）四大功能：

- **压缩**（也称为摇树优化，tree shaking）：从 **应用及依赖项** 中移除 **未使用** 的类、方法和字段，有助于规避 64 方法数的瓶颈
- **优化**：通过代码 **分析** 移除更多未使用的代码，甚至重写代码
- **混淆**：使用无意义的简短名称 **重命名** 类/方法/字段，增加逆向难度
- **预校验**：对于面向 Java 6 或者 Java 7 JVM 的 class 文件，编译时可以把 **预校验信息** 添加到类文件中（StackMap 和 StackMapTable属性），从而加快类加载效率。预校验对于 Java 7 JVM 来说是必须的，但是对于 Android 平台 [无效](https://www.guardsquare.com/en/products/proguard/manual/usage#preverificationoptions)

使用 ProGuard 时，部分编译流程如下图所示：

- ProGuard 对 .class 文件执行代码压缩、优化与混淆
- D8 编译器执行脱糖，并将 .class 文件转换为 .dex文件

![img](media/7a59153f20d14944835d59ddc7aa2968~tplv-k3u1fbpfcp-watermark.image)

使用 R8 时，部分编译流程如下图所示：

- **R8 将脱糖（Desugar）、压缩、优化、混淆和 dex（D8 编译器）整合到一个步骤**
- R8 对 .class 文件执行代码压缩、优化与混淆
- D8 编译器执行脱糖，并将 .class 文件转换为 .dex文件

![img](media/2b60da6b46fd4f5894c8fb0f8c7a1e88~tplv-k3u1fbpfcp-watermark.image)

对比以下 ProGuard 与 R8 ：

- **共同点：**

1、开源

2、R8 支持所有现有 ProGuard 规则文件

3、都提供了四大功能：**压缩**、**优化**、**混淆**、**预校验**

- **不同点：**

1、ProGuard 可用于 Java 项目，而 R8 专为 Android 项目设计

2、R8 将脱糖（Desugar）、压缩、优化、混淆和 dex（D8 编译器）整合到**一个步骤**中，显着提高了编译性能

> **关于 D8 编译器**
>
> **将 .class 字节码转化为 .dex 字节码的过程被称为 DEX 编译**，最初是由DX 编译器完成。与 DX 编译器相比，新的 D8 编译器的编译速度 **更快**，输出的 .dex 文件 **更小** ，却能保持相同乃至 **更出色** 的应用运行时性能

------

## 3. 使用示例

**无论使用 R8 还是 ProGuard，默认不会启用压缩、优化和混淆功能。** 这个设计主要是出于两方面考虑：一方面是因为这些编译时任务会增加编译时间，另一方面是因为如果没有充分定义混淆保留规则，还可能会引入运行时错误。因此，最好 **只在应用的测试版本和发布版本中启用这些编译时任务**，参考使用示例：

```
// build.gradle
...
android {

  buildTypes {

    // 测试版本
    preview {
      // 启用代码压缩、优化和混淆（由R8或者ProGuard执行）
      minifyEnabled true
      // 启用资源压缩（由Android Gradle plugin执行）
      shrinkResources true
      // 指定混淆保留规则文件
      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    } 

    // 发布版本
    release {
      minifyEnabled true
      shrinkResources true
      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }

    // 开发版本
    debug{
      minifyEnabled false
    }
  }
...
}
```

- `minifyEnabled`：（默认情况下）启用代码压缩、优化、混淆与预校验

- `shrinkResources`：启用资源压缩

- ```
  proguardFiles
  ```

  、

  ```
  proguardFile
  ```

  ：指定 ProGuard 规则文件，前者可以指定多个参数。下面两段配置的作用是一样的。

  ```
  // 方式一：
  proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
  // 方式二：
  proguardFile getDefaultProguardFile('proguard-android-optimize.txt')
  proguardFile 'proguard-rules.pro'
  ```

前面提到了：无论使用R8还是ProGuard，压缩、优化和混淆功能都是 **默认关闭**的。通过以下配置可以灵活控制：

- **整体关闭**

```
minifyEnabled false
// 这行就没有效果了
proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
```

- **关闭压缩**

```
-dontshrink
```

- **关闭优化**（R8 无效）

```
-dontoptimize
```

注意：**R8 不能关闭优化，也不允许修改优化的行为**，事实上，R8 会[忽略](https://developer.android.google.cn/studio/build/shrink-code#optimization)修改默认优化行为的规则。例如设置 `-optimizations` 和 `-optimizationpasses`后会得到编译时警告：

```
AGPBI: {"kind":"warning","text":"Ignoring option: -optimizations","sources":[{"file":"省略..."}],"tool":"D8"}
AGPBI: {"kind":"warning","text":"Ignoring option: -optimizationpasses","sources":"省略..."}],"tool":"D8"}
```

- **关闭混淆**（建议在开发版本关闭混淆）

```
-dontobfuscate
```

- **关闭预校验**（对 Android 平台[无效](https://www.guardsquare.com/en/products/proguard/manual/usage#preverificationoptions)，建议关闭）

```
-dontpreverify
```

------

## 4. ProGuard 规则文件

**R8 延续了 ProGuard 使用规则文件修改默认行为的做法。\**在很多时候，规则文件也被称为\**混淆保留规则文件**，这是因为该文件内定义的绝大多数规则都是和代码混淆相关的。事实上，文件内还可以定义代码压缩、优化和预校验规则，因此称为 ProGuard 规则文件比较严谨。

在上一节里，我们提到了使用`proguardFiles`和`proguardFile`指定 ProGuard 规则文件。对于任何一个项目，它的 ProGuard 规则文件有以下三种来源：

- **1、Android Gradle 插件**

在编译时，Android Gradle 插件会生成 `proguard-android-optimize.txt`、 `proguard-android.txt`，位置在`<module-dir>/build/intermediates/proguard-files/`。这两个文件中除了注释之外，唯一的区别是前者启用了如下代码压缩，而后者关闭了代码压缩，如下所示：

```
# proguard-android-optimize.txt
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
-optimizationpasses 5
-allowaccessmodification
相同部分省略...
复制代码
# proguard-android.txt
-dontoptimize
相同部分省略...
```

其中相同的那部分混淆规则中，下面这一部分是比较特殊的：

```
-keep class android.support.annotation.Keep
-keep class androidx.annotation.Keep
// 保留@Keep注解的类，保留...TODO
-keep @android.support.annotation.Keep class * {*;}
-keep @androidx.annotation.Keep class * {*;}
// 保留@Keep修饰的方法
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}
-keepclasseswithmembers class * {
    @androidx.annotation.Keep <methods>;
}
// 保留@Keep修饰的字段
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}
-keepclasseswithmembers class * {
    @androidx.annotation.Keep <fields>;
}
// 保留@Keep修饰的构造方法
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}
-keepclasseswithmembers class * {
    @androidx.annotation.Keep <init>(...);
}
```

它指定了与`@Keep`注解相关的所有保留规则，这里就解释了为什么使用`@Keep`修饰的成员不会被混淆了吧？

- **2、Android Asset Package Tool 2 (AAPT2)**

在编译时，AAPT2 会根据对 Manifest 中的类、布局及其他应用资源的引用来生成`aapt_rules.txt`，位置在`<module-dir>/build/intermediates/proguard-rules/debug/aapt_rules.txt`。 例如，AAPT2 会为 Manifest 中注册的每个组件添加保留规则：

```
Referenced at [项目路径]/app/build/intermediates/merged_manifests/release/AndroidManifest.xml:19
-keep class com.have.a.good.time.MainActivity { <init>(); }
省略...
```

在这里，AAPT2 生成了`MainActivity`的保留规则，同时它还指出了引用出处：`AndroidManifest.xml:19`。这是因为 启动 Activity 的过程中，需要使用反射的方式实例化具体的每一个 Activity ，有兴趣可以看下 `ActivityThread#performLaunchActivity()` -> `Instrumentation#newActivity()`

- **3、Module**

创建新 Module 时，IDE 会在该模块的根目录中创建一个 `proguard-rules.pro` 文件。当然，除了这个自动生成的文件，还可以按需创建额外的规则文件。例如，下面的配置对 release 添加了额外的规则文件：

```
...
android {
  ...
  buildTypes {
    release {
      minifyEnabled true
      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
  }
  productFlavors {
    dev{
      ...
    }
    release{
      proguardFile 'release-rules.pro'
    }
  }
}
...
```

小结一下：

| 规则文件来源        | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| Android Gradle 插件 | 在编译时，由 Android Gradle 插件生成                         |
| AAPT2               | 在编译时，AAPT2 根据对应用清单中的类、布局及其他应用资源的引用来生成保留规则 |
| Module              | 创建新 Module 时，由 IDE 创建，或者另外按需创建              |

如果将 `minifyEnabled` 属性设为 `true`，**ProGuard 或 R8 会将来自上面列出的所有可用来源的规则组合在一起**。为了看到完整的规则文件，可以在`proguard-rules.pro` 中添加以下配置，输出编译项目时应用的所有规则的完整报告：

```
-printconfiguration build/intermediates/proguard-files/full-config.txt
```

------

## 5. 组件化混淆

在组件化的项目中，需要注意应用 Module 和 Library Module 的行为差异和组件化的资源汇合规则，总结为以下几个重点：

- 编译时会依次对各层 Library Module进行编译，最底层的 Base Module 会最先被编译为 aar 文件，然后上一层编译时会将依赖 Module 输出的 aar 文件/ jar 文件解压到模块的 build 中相应的文件夹中
- App Module 这一层汇总了全部的 aar 文件后，才真正开始编译操作
- 后编译的 Module 会覆盖之前编译的 Module 中的同名资源

![组件化资源汇总](media/91a52fe6f8fc430f9b001713f2fa48db~tplv-k3u1fbpfcp-watermark.image)

![Lib Module 汇总到 App Module](media/9fa4030fe34d464d821122cd91c77b57~tplv-k3u1fbpfcp-watermark.image)

使用较高版本的 Android Gradle Plugin，不会将汇总的资源放置在 `exploded-aar`文件夹。即便如此，Lib Module 的资源汇总到 App Module 的规则是一样的。

我们通过一个简单示例测试不同配置下的混淆结果：

|                      | 配置一 | 配置二 | 配置三 | 配置四 |
| -------------------- | ------ | ------ | ------ | ------ |
| App Module 开启混淆  | X      | X      | √      | √      |
| Base Module 开启混淆 | X      | √      | X      | √      |

![示例程序：App Module 依赖了 Base Module](media/ac1886a05d4a4a45b1d753b6ba85932c~tplv-k3u1fbpfcp-watermark.image)

将构建的 apk 包拖到 Android Studio 面板上即可分析 Base 类混淆结果，例如配置一的结果：

![使用配置一时，Base 类没有被混淆](media/f37198cd936d40c4974ad9f98410b2cf~tplv-k3u1fbpfcp-watermark.image)

全部测试结果如下：

|                           | 配置一 | 配置二 | 配置三 | 配置四 |
| ------------------------- | ------ | ------ | ------ | ------ |
| App Module 开启混淆       | X      | X      | √      | √      |
| Base Module 开启混淆      | X      | √      | X      | √      |
| （结果）Base 类是否被混淆 | X      | X      | √      | √      |

可以看到，**混淆开启由 App Module 决定， 与Lib Module 无关**。

现在我们分别在 Lib Module 和 App Module 的 `proguard-rules.pro`中添加 Base 类的混淆保留规则，并在 `build.gradle`中添加配置文件，测试 Base 类是否能保留：

```
-keep class com.rui.base.Base
```

测试结果如下：

| 配置位置                | Lib Module | App Module |
| ----------------------- | ---------- | ---------- |
| （结果）Base 类是否保留 | X          | √          |

可以看到：**（默认情况）混淆规则以 App Module 中的混淆规则文件为准**。

这里就引入两种主流的组件化混淆方案：

- **在 App Module 中设置混淆规则**

这种方案将混淆规则都放置到 App Module 的`proguard-rules.pro`中，最简单也最直观，缺点是移除 Lib Module 时，需要从 App Module 中移除相应的混淆规则。尽管多余的混淆规则并不会造成编译错误或者运行错误，但还是会影响编译效率。

很多的第三方 SDK，就是采用了这种组件化混淆方案。在 App Module 中添加依赖的同时，也需要在`proguard-rules.pro`中添加专属的混淆规则，这样才能保证`release`版本正常运行。

- **在 App Module 中设置公共混淆规则，在 Lib Module 中设置专属混淆规则**

这种方案将专属的混淆规则设置到 Lib Module 的`proguard-rules.pro`，但是根据前面的测试，在 Lib Module 中设置的混淆规则是不生效的。为了让规则生效，还需要在 Lib Module 的`build.gradle`中添加以下配置：

```groovy
...
android{
  defaultConfig{
    consumerProguardFiles 'consumer-rules.pro'
  }
}
```

其中`consumer-rules.pro`文件：

```
-keep class com.rui.base.Base
```

测试结果表明，Base 类已经被保留了。这种使用`consumerProguardFiles`的方式有以下几个特点：

- `consumerProguardFiles`只对 Lib Module 生效，对 App Module 无效
- `consumerProguardFiles`会将混淆规则输出为`proguard.txt`文件，并打包进 aar 文件
- App Module 会使用 aar 文件中的`proguard.txt`汇总为最终的混淆规则，这一点可以通过前面提到的`-printconfiguration`证明

------

## 6. 总结

- ProGuard 是 Java 字节码优化工具，而 R8 是专为 Android 设计的，编译性能和编译产物更优秀；
- ProGuard 与 R8 都提供了四大功能：压缩、优化、混淆和预校验。ProGuard 主要是对 .class 文件执行代码压缩、优化与混淆，再由 D8 编译器执行脱糖并转换为 .dex 文件。R8 将压缩、优化、混淆、脱糖和 dex 整合为一个步骤；
- ProGuard 规则文件有三种来源：Android Gradle 插件、AAPT2、Module；
- 默认情况下，混淆规则以 App Module 中的混淆规则文件为准，使用 consumer-rules.pro 文件可以设置 Lib Module 专属混淆规则。

------

## 参考资料

- [压缩您的应用](https://developer.android.google.cn/studio/build/shrink-code)
- [ProGuard | Office website](https://www.guardsquare.com/en/products/proguard)
- [ProGuard | Manual](https://www.guardsquare.com/en/products/proguard/manual)
- [R8 | Google Git](https://r8.googlesource.com/r8)
- [Android Gradle plugin release notes](https://developer.android.google.cn/studio/releases/gradle-plugin)
- 《深入理解Java虚拟机 — JVM高级特性与最佳实践》 周志明 著
- 《Android 组件化架构》 仓王 著
- 《Android开发高手课》 张绍文 著，极客时间 出品

## 来源

链接：https://juejin.cn/post/6930648501311242248


