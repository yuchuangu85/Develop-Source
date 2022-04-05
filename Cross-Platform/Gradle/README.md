<h1 align="center">Gradle</h1>

[toc]

## Gradle官方

* [User Guide](https://docs.gradle.org/current/userguide/userguide.html)
* [Gradle - Plugins](https://plugins.gradle.org/): Gadle plugin search

## Gradle源码分析

* [Gradle源码分析（一） - 简书 (jianshu.com)](https://www.jianshu.com/p/625bc82003d7)
* [Gradle源码分析（二） - 简书 (jianshu.com)](https://www.jianshu.com/p/d934b3a28c33)
* [Gradle源码分析（三） - 简书 (jianshu.com)](https://www.jianshu.com/p/7121ce8c4932)
* [Gradle源码分析（四） - 简书 (jianshu.com)](https://www.jianshu.com/p/10e14aabbfbd)
* [Gradle源码分析（五） - 简书 (jianshu.com)](https://www.jianshu.com/p/c257d3b338fe)
* [Gradle源码分析（六） - 简书 (jianshu.com)](https://www.jianshu.com/p/66dee8db16e4)
* 

## Gradle Kotlin

* [Migrating build logic from Groovy to Kotlin (gradle.org)](https://docs.gradle.org/current/userguide/migrating_from_groovy_to_kotlin_dsl.html)
* [Gradle Kotlin DSL , 你知道它吗？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/162073022)



## Composite build

* [jraska/github-client](https://github.com/jraska/github-client)--代码实例
* [Stop using Gradle buildSrc. Use composite builds instead ](https://proandroiddev.com/stop-using-gradle-buildsrc-use-composite-builds-instead-3c38ac7a2ab3)--教程



## Gradle使用

* [Android Studio手动配置Gradle的方法](https://blog.csdn.net/fuchaosz/article/details/51567808)
* [使用新 Android Gradle 插件加速您的应用构建](https://mp.weixin.qq.com/s/j6dmdiWV-tUb3aO7mYc3oQ)



## Gradle详解

* [彻底搞懂Gradle、Gradle Wrapper与Android Plugin for Gradle的区别和联系](https://www.cnblogs.com/jiangxinnju/p/8229129.html)
* Gradle: https://docs.gradle.org/current/userguide/userguide_single.html
* Gradle Wrapper: https://docs.gradle.org/current/userguide/gradle_wrapper.html
* Android Plugin for Gradle: https://developer.android.com/studio/build/index.html
* [深度探索 Gradle 自动化构建技术（一、Gradle 核心配置篇）](深度探索 Gradle 自动化构建技术（一、Gradle 核心配置篇）)
* [深度探索 Gradle 自动化构建技术（二、Groovy 筑基篇）](https://juejin.cn/post/6844904128594853902)
* [深度探索 Gradle 自动化构建技术（三、Gradle 核心解密）](https://juejin.cn/post/6844904132092903437)
* [深度探索 Gradle 自动化构建技术（四、自定义 Gradle 插件）](https://juejin.cn/post/6844904135314128903)
* [深度探索 Gradle 自动化构建技术（五、Gradle 插件架构实现原理剖析 — 下）](https://juejin.cn/post/6844904142717075469)
* [深度探索 Gradle 自动化构建技术（五、Gradle 插件架构实现原理剖析 — 上）](https://juejin.cn/post/6844904142725447687)

## Gralde原理

* [详解Android Gradle生成字节码流程](http://www.androidchina.net/10264.html)
* [深入理解Android之Gradle](https://blog.csdn.net/innost/article/details/48228651)
* [【Android 修炼手册】Gradle 篇 -- Gradle 的基本使用](https://zhuanlan.zhihu.com/p/65249493)
* [【Android 修炼手册】Gradle 篇 -- Android Gradle Plugin 插件主要流程](https://zhuanlan.zhihu.com/p/66052867)
* [【Android 修炼手册】Gradle 篇 -- Android Gradle Plugin 主要 Task 分析](https://zhuanlan.zhihu.com/p/67049158)
* [【Android 修炼手册】Gradle 篇 -- Gradle 源码分析](https://zhuanlan.zhihu.com/p/67842670)
* [动画讲解 Gradle 原理](https://link.zhihu.com/?target=https%3A//link.juejin.im/%3Ftarget%3Dhttps%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav55941638%2F)
* [【Android 修炼手册】Gradle 篇 -- Gradle 源码分析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/67842670)

## Gradle结构

### 结构

```
GradleDemo/
|---app/
|------build.gradle
|---model/
|------build.gradle
|---gradle/
|------wrapper/
|------------gradle-wrapper.jar
|------------gradle-wrapper.properties
|---build.gradle
|---gradlew
|---gradlew.bat
|---settings.gradle
   
```

### Android

- applicationVariants（只适用于 application plugin）---> 返回ApplicationVariant
- libraryVariants（只适用于 library plugin）---> 返回LibraryVariant
- testVariants（app、library plugin 均适用）---> 返回TestVariant

注：上面三个对象都是[DomainObjectCollection](http://www.gradle.org/docs/current/javadoc/org/gradle/api/DomainObjectCollection.html)对象。`DomainObjectCollection` 可以直接访问所有对象，或者通过过滤器进行筛选。

```Groovy
android.applicationVariants.each { variant ->
    ....
}
```

这三个 variant 类都拥有下面的属性：

| 属性名               | 属性类型                | 说明                                                         |
| -------------------- | ----------------------- | ------------------------------------------------------------ |
| name                 | String                  | Variant 的名字，唯一                                         |
| description          | String                  | Variant 的描述说明                                           |
| dirName              | String                  | Variant 的子文件夹名，唯一。可能有不止一个子文件夹，例如 “debug/flavor1” |
| baseName             | String                  | Variant 输出的基础名字，必须唯一                             |
| outputFile           | File                    | Variant 的输出，该属性可读可写                               |
| processManifest      | ProcessManifest         | 处理 Manifest 的 task                                        |
| aidlCompile          | AidlCompile             | 编译 AIDL 文件的 task                                        |
| renderscriptCompile  | RenderscriptCompile     | 编译 Renderscript 文件的 task                                |
| mergeResources       | MergeResources          | 合并资源文件的 task                                          |
| mergeAssets          | MergeAssets             | 合并 assets 的 task                                          |
| processResources     | ProcessAndroidResources | 处理并编译资源文件的 task                                    |
| generateBuildConfig  | GenerateBuildConfig     | 生成 BuildConfig 类的 task                                   |
| javaCompile          | JavaCompile             | 编译 Java 源代码的 task                                      |
| processJavaResources | Copy                    | 处理 Java 资源的 task                                        |
| assemble             | DefaultTask             | Variant 的标志性 assemble task                               |

`ApplicationVariant` 类还有以下附加属性：

| 属性名             | 属性类型            | 说明                                                         |
| ------------------ | ------------------- | ------------------------------------------------------------ |
| buildType          | BuildType           | Variant 的 BuildType                                         |
| productFlavors     | List<ProductFlavor> | Variant 的 ProductFlavor，一般不为空但允许为空               |
| mergedFlavor       | ProductFlavor       | android.defaultConfig 和 variant.productFlavors 的组合       |
| signingConfig      | SigningConfig       | Variant 使用的 SigningConfig 对象                            |
| isSigningReady     | boolean             | 如果是 true 则表明该 Variant 已经具备了所有需要签名的信息    |
| testVariant        | BuildVariant        | 将会测试该 Variant 的 TestVariant                            |
| dex                | Dex                 | 将代码打包成 dex 的 task。如果该 Variant 是 Library，该值可为空 |
| packageApplication | PackageApplication  | 打包最终 APK 的 task。如果该 Variant 是 Library，该值可为空  |
| zipAlign           | ZipAlign            | 对 APK 进行 zipalign 的 task。如果该 Variant 是 Library 或者 APK 不能被签名时，该值可为空 |
| install            | DefaultTask         | 负责安装的 task，可为空                                      |
| uninstall          | DefaultTask         | 负责卸载的 task                                              |

`LibraryVariant` 类还有以下附加属性：

| 属性名         | 属性类型      | 说明                                                         |
| -------------- | ------------- | ------------------------------------------------------------ |
| buildType      | BuildType     | Variant 的 BuildType                                         |
| mergedFlavor   | ProductFlavor | 只有 android.defaultConfig                                   |
| testVariant    | BuildVariant  | 用于测试 Variant                                             |
| packageLibrary | Zip           | 用于打包 Library 项目的 AAR 文件。如果是 Library 项目，该值不能为空 |

`TestVariant` 类还有以下属性：

| 属性名               | 属性类型            | 说明                                                         |
| -------------------- | ------------------- | ------------------------------------------------------------ |
| buildType            | BuildType           | Variant 的 Build Type                                        |
| productFlavors       | List<ProductFlavor> | Variant 的 ProductFlavor。一般不为空但允许为空               |
| mergedFlavor         | ProductFlavor       | android.defaultConfig 和 variant.productFlavors 的组合       |
| signingConfig        | SigningConfig       | Variant 使用的 SigningConfig 对象                            |
| isSigningReady       | boolean             | 如果是 true 则表明该 Variant 已经具备了所有需要签名的信息    |
| testedVariant        | BaseVariant         | TestVariant 测试的 BaseVariant                               |
| dex                  | Dex                 | 将代码打包成 dex 的 task。如果该 Variant 是 Library，该值可为空 |
| packageApplication   | PackageApplication  | 打包最终 APK 的 task。如果该 Variant 是 Library，该值可为空  |
| zipAlign             | ZipAlign            | 对 APK 进行 zipalign 的 task。如果该 Variant 是 Library 或者 APK 不能被签名时，该值可为空 |
| install              | DefaultTask         | 负责安装的 task，可为空                                      |
| uninstall            | DefaultTask         | 负责卸载的 task                                              |
| connectedAndroidTest | DefaultTask         | 在连接设备上执行 Android 测试的 task                         |
| providerAndroidTest  | DefaultTask         | 使用扩展 API 执行 Android 测试的 task                        |

Android task 特有类型的 API：

- ProcessManifest
   - `File manifestOutputFile`
- AidlCompile
   - `File sourceOutputDir`
- RenderscriptCompile
   - `File sourceOutputDir`
   - `File resOutputDir`
- MergeResources
   - `File outputDir`
- MergeAssets
   - `File outputDir`
- ProcessAndroidResources
   - `File manifestFile`
   - `File resDir`
   - `File assetsDir`
   - `File sourceOutputDir`
   - `File textSymbolOutputDir`
   - `File packageOutputFile`
   - `File proguardOutputFile`
- GenerateBuildConfig
   - `File sourceOutputDir`
- Dex
   - `File outputFolder`
- PackageApplication
   - `File resourceFile`
   - `File dexFile`
   - `File javaResourceDir`
   - `File jniDir`
   - File outputFile
      - 直接在 Variant 对象中使用 “outputFile” 可以改变最终的输出文件夹。
- ZipAlign
   - `File inputFile`
   - File outputFile
      - 直接在 Variant 对象中使用 “outputFile” 可以改变最终的输出文件夹。

每个 task 类型的 API 都受 Gradle 的工作方式和 Android plugin 的配置方式限制。
首先，Gradle 中存在的 task 只能配置输入输出的路径以及部分可能使用的可选标识。因此，这些 task 只能定义一些输入或者输出。

其次，这里面大多数 task 的输入都不是单一的，一般都混合了 *sourceSet*、*Build Type* 和 *Product Flavor* 中的值。所以为了保持构建文件的简洁和可读性，目标自然是让开发者通过 DSL 修改这些对象来影响构建的过程，而不是深入修改输入和 task 的选项。

另外需要注意，上面的 task 中除了 ZipAlign，其它都要求设置私有数据进行工作。这意味着不能手动创建这些类型的 task 实例。

对于 Gradle 的 task（DefaultTask，JavaCompile，Copy，Zip），请参考 Gradle 文档。

## Gradle语法

* buildscript：定义了Android编译工具的类路径，这里的配置都是给build.gradle使用的，

   * repositories：jcenter、maven都是加载依赖的仓库，

   * dependencies：配置的classpath库是build.gradle依赖的依赖包。（build.gradle中的dependencies是代码中依赖的依赖包，使用关键字implementtion）

      ```
      // 添加classpath: groupId:artifactId:version（参看customplugin中的build.gradle）
      classpath "com.codemx.plugin:customplugin:1.0.0"
      ```

      插件中build.gradle配置：

      ```
      // 发布插件
      uploadArchives { //当前项目可以发布到本地文件夹中
          repositories {
              mavenDeployer {
                  // 下面三个变量组成项目依赖中buildscript中dependencies中的classpath路径
                  // 格式：classpath groupId:artifactId:version
                  pom.groupId = 'com.codemx.plugin'   //groupId
                  pom.artifactId = 'customplugin'     //artifactId
                  pom.version = '1.0.0'               //version
                  repository(url: uri('./repo')) //定义本地maven仓库的地址(也可以设置远程地址)
              }
          }
      }
      ```

      

* allprojects：里面定义的属性是给整个项目使用的

* apply plugin: 'customplugin'：这里是customplugin插件项目中的src/main/resources/META_INF/gradle-plugins/customplugin.properties的文件名字

## 参考

*  [操作task之applicationVariants](https://www.cnblogs.com/gamesky/p/12978621.html)

