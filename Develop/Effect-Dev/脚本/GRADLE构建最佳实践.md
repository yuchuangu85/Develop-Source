# GRADLE构建最佳实践
>用GRADLE构建安卓项目已是大势所趋，具体实战中姿势啥的很重要，结合具体应用场景，最佳实践给你最佳的体验

随着谷歌对Eclipse的无情抛弃和对Android Studio的日趋完善，使用gradle构建Android项目已经成为开发者的一项必会良技。那么，问题来了，采用什么样的姿势才能让项目开发构建过程高潮迭起，精彩不断呢？

其实网上有很多关于gradle的文章，gradle官方和谷歌也提供了详细的文档和教程，可素，当你在构建过程中遇到一些问题或者有特殊的爱好（需求）的时候，这些东西未必能帮（mei）上（shen）什（me）么（niao）忙（yong），然后就是一顿翻墙找谷歌蜀黍约约约，去stackoverflow上各种搜刮问大神，最后解决了。即使没有真的解决那么就忍了。


那怎么行？是可忍孰不可忍，奇技淫巧必须有。所以就会有这样一篇文章，我在这里不讲原理，因为我知道很多人明白辣么多的底层原理，仍然撸不上好代码，做不成好项目，出不了好产品，自然也就过不好这一生咯。

我们先从GRADLE构建的时间花销开始谈起。

## 加速篇
GRADLE的构建过程通常会比较漫长，一个中等项目，10M左右大小的app，一次完整构建大概在5分钟左右，是不是很吓人，当然，如果是在调试阶段，采用Android Studuo 2.0，默认提供的Instant Run方式，每次修改都不会重新构建项目，从而加快了构建过程。恩，这是另一个故事，这里，我们先谈谈GRADLE脚本的加速姿势。

一般来说，GRADLE一次完整的构建过程通常分成三个部分，初始化，配置和执行任务，那么我们可以考虑从这三个部分分别尝试优化。

### 使用daemon
构建初始化的很多工作是关于java虚拟机的启动，加载虚拟机环境，加载class文件等，如果这些动作交给一个单独的后台进程去做，那么，第一次初始化之后的修改代码再构建是不是可以节省很多时间呢？答案是肯定的，通过在gradle.properties加入这样一句来开启，如果想让修改全局所有项目都生效，那么修改这个文件~/.gradle/gradle.properties

```java
org.gradle.daemon=true
```

### 按需配置
配置有一种方式是按需配置，目前还在试验孵化阶段，所以默认是关闭的，可以通过在gradle.properties加入这样一句来开启

```java
org.gradle.configureondemand=true
```

### 依赖库使用固定版本
项目开发过程中，难免需要用到三方库，这就形成了项目之间的依赖关系，GRADLE提供了多种集成三方库的方式，提供了很方便的项目依赖管理，本地库，库工程，maven库全支持。既然用到库，就会遇到库版本的问题和升级问题，其中maven库的依赖管理支持一种动态版本的方式，也就是说，GRADLE可以做到不依赖具体某个版本的库，而是每次从repo拉取最新的库到本地做编译。具体使用是这样的：

拿gson库举例，如果依赖2.2.1这个版本，可以在build.gradle文件里这样写

```java
dependencies {
	compile 'com.google.code.gson:gson:2.2.1'
}
```

如果不想依赖具体的库，想每次从maven repo中拉取最新的库，那么，可以写成这样：

```java
dependencies {
	compile 'com.google.code.gson:gson:2.2.＋'
}
```

也可以写成这样

```java
dependencies {
	compile 'com.google.code.gson:gson:2.＋'
}
```

甚至可以这样

```java
dependencies {
	compile 'com.google.code.gson:gson:＋'
}
```

其中含义相信不用我解释，大家也看得明白吧。

用”+”来通配一个版本族，这样有个好处是maven上有新库了，不用你操心升级，GRADLE编译的时候自动升级了，但是带来了两个坏处，一是，有可能新版库的接口改了，导致编译失败，这个时候需要修改代码做升级适配；更大的坏处是，每次GRADLE编译完整的项目，都会去maven上试图拉取最新的库，这样，拖慢了编译速度，尤其在网络非常差的时候，所以，为了构建速度，建议写死依赖库的版本号。

### 升级到最新的GRADLE和JDK
有一个很通俗的道理是，发展的东西会越来越好，最新版的GRADLE和JDK往往是性能最好，运行最流畅最快的，所以，升级吧，JDK的升级这里不说了，具体看Oracle的官方文档。这里说说GRADLE的版本升级，GRALDE采用了一种叫做wrapper的方式，可以做到每个项目独立使用其自己的GRADLE版本，这样做的好处不言而喻，每个项目的构建环境独立，互不影响。但为什么会出现这个东西，我的猜想是因为GRADLE发展太快，新旧版本之间很难兼容。如果你有多个项目都采用GRADLE构建，假设都用同一个全局的GRADLE，那么当这个GRADLE升级后，所有的项目可能都会编译失败，你得一个一个改配置，那么，下次再升级，同样的流程的再走一遍，是不是很烦。采用wrapper的方式很好的解决了这个问题，每个项目采用独立的GRADLE版本，互不影响，如果你只想升级其中一个，你改这一个项目的GRADLE wrapper就好了。在你的项目目录下找到这个文件gradle/wrapper/gradle-wrapper.properties并修改distributionUrl=https://services.gradle.org/distributions/gradle-2.11-all.zip到你想升级的版本就可以了。

### 减少编译脚本中的I/O操作
有时候，编译脚本中会有一些代码做动态信息的获取，比如想从git中获取一个数字作为版本号

```java
def cmd = 'git rev-list HEAD --first-parent --count'
def gitVersion = cmd.execute().text.trim().toInteger()
android {
  defaultConfig {
    versionCode gitVersion
  }
}
```

其实这个操作主要是为了在构建的机器上为了发布版本而做的，日常环境研发调试无需这样，所以可以优化成如下方式：

```java
def gitVersion() {
  if (!System.getenv('CI_BUILD')) {
    // don't care
    return 1
  }
  def cmd = 'git rev-list HEAD --first-parent --count'
  cmd.execute().text.trim().toInteger()
}
android {
  defaultConfig {
    versionCode gitVersion()
  }
}
    
```

### 并行构建模块化项目
将你的项目拆分成多个子项目并开启并行构建也是一个不错的主意，比如将相对独立的模块拆分成独立的库工程(Library projects)，主工程(Application project)依赖这些库工程，这样的话，开启并行构建才会发挥作用。并行构建开启方式是修改文件gradle.properties，加入如下行：

```java
org.gradle.parallel=true
```

## 基础配置篇
全局基础配置管理

### 设置全局编码
如果导入一个windows下编写的项目，而代码中有中文注释，采用GBK, GB18030等编码方式时，编译会报错，可以采用如下方式统一项目的编码

```java
allprojects {
    repositories {
        jcenter()
    }

    tasks.withType(JavaCompile) {
        options.encoding = "UTF-8"
    }
}
```

### 设置全局编译器的版本
如果编程过程中采用了新版JDK（比如1.7）才支持的特性(比如new HashMap<>这样的写法)，而编译的时候默认是旧版的JDK(比如1.6)，这个时候编译会报错，采用如下方式可以指定用哪个版本的编译器编译，前提是JAVA_HOME指定的JDK是大于等于新版JDK的哦^o^，其他和java编译器相关的也可以在这里配置

```java
allprojects {
    repositories {
        jcenter()
    }
    tasks.withType(JavaCompile) {
        sourceCompatibility = JavaVersion.VERSION_1_7
        targetCompatibility = JavaVersion.VERSION_1_7
    }
}
```

如果不想全局生效，可以将tasks.withType(JavaCompile)放入某个子项目中。

### 配置签名信息
签名信息属于敏感信息，建议不要写死放到gradle脚本中，而是写到一个单独的配置文件里，而且这个配置文件不要同步到版本管理系统上，而是由本地维护，防止在版本管理平台上泄漏敏感信息。建议签名信息内容写到gradle.properties或者local.properties文件里，这样，gradle脚本可以直接引用，如果是放在一个自定义的文件中，gradle脚本需要提供相应的代码来读取文件的内容。 文件内容参考如下：

```java
RELEASE_KEY_PASSWORD=android
RELEASE_KEY_ALIAS=androidreleasekey
RELEASE_STORE_PASSWORD=android
RELEASE_STORE_FILE=../resources/release.keystore
DEBUG_KEY_PASSWORD=android
DEBUG_KEY_ALIAS=androiddebugkey
DEBUG_STORE_PASSWORD=android
DEBUG_STORE_FILE=../resources/debug.keystore
```

gradle脚本引用代码参考：

```java
android {
    signingConfigs {
        debug {
            storeFile file(DEBUG_STORE_FILE)
            storePassword DEBUG_STORE_PASSWORD
            keyAlias DEBUG_KEY_ALIAS
            keyPassword DEBUG_KEY_PASSWORD
        }

        release {
            storeFile file(RELEASE_STORE_FILE)
            storePassword RELEASE_STORE_PASSWORD
            keyAlias RELEASE_KEY_ALIAS
            keyPassword RELEASE_KEY_PASSWORD
        }
    }
}
```

如果签名信息没有放到gradle.properties或者local.properties文件里，那就需要自己写代码读取咯，假设签名信息放在signing.properties文件中, 文件内容可以参考上面，读取文件的代码放入gradle脚本中就可以了，参考代码如下

```java
def File propFile = new File('signing.properties')
if (propFile.canRead()) {
    def Properties props = new Properties()
    props.load(new FileInputStream(propFile))

    if (props != null && props.containsKey('RELEASE_STORE_FILE') && props.containsKey('RELEASE_STORE_PASSWORD') &&
            props.containsKey('RELEASE_KEY_ALIAS') && props.containsKey('RELEASE_KEY_PASSWORD')) {

        android.signingConfigs.release.storeFile = file(props['RELEASE_STORE_FILE'])
        android.signingConfigs.release.storePassword = props['RELEASE_STORE_PASSWORD']
        android.signingConfigs.release.keyAlias = props['RELEASE_KEY_ALIAS']
        android.signingConfigs.release.keyPassword = props['RELEASE_KEY_PASSWORD']
        println 'all good to go'
    } else {
        android.buildTypes.release.signingConfig = null
        println 'signing.properties found but some entries are missing'
    }
} else {
    println 'signing.properties not found'
    android.buildTypes.release.signingConfig = null
}
```

**特别注意**

如果在构建类型(buildTypes)或者产品种类(productFlavors)需要引用到上述的签名类型，请注意一定要把签名类型的定义放在引用它的对象之前，否则会报错找不到签名配置。其实整个GRADLE语法都是先定义后引用的，这个让写惯JAVA的我确实不习惯。

### 设置第三方maven地址
其中name和credentials是可选项，视具体情况而定

```java
allprojects {
    repositories {
        maven {
            url 'url'
            name 'maven name'
            credentials {
                username = 'username'
                password = 'password'
            }
        }
    }
}
```

### GRADLE脚本拆分以及引用
如果一个gradle脚本太大，可以按照具体任务的类型拆分成几个子脚本，然后引入到主脚本中

```java
apply from:"../resource/config.gradle"
```

### 全局变量定义及引用
可以在顶层build.gradle脚本中定义一些全局变量，提供给子脚本引用

```java
ext {
    // global variables definition
    compileSdkVersion = 'Google Inc.:Google APIs:23'
    buildToolsVersion = "23.0.3"
    minSdkVersion = 14
    targetSdkVersion = 23
}
```

子脚本引用

```java
android {
    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion

    defaultConfig {
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
    }
}
```

## 构建参数篇
构建参数设置

### AndroidManifest占位符，BuildConfig以及资源配置
根据版本类型和构建变种定义不同的变量值供程序引用

```java
manifestPlaceholders = [APP_KEY:"release"]
buildConfigField "String", "EMAIL", "\"release@android.studio.com\""
resValue "string", "content_main", "Hello world from release!"
```

>buildConfigField支持Java中基本数据类型，如果是字符串，记得加转义后的双引号 resValue支持res/values下的资源定义，字符串无需加转义后的双引号

### 设置支持的语言
利用这个配置可以去掉三方库中无用的语言（人类的自然语言，非计算机编程语言）

```java
android {
    defaultConfig {
        resConfigs "zh"
    }
}
```

### 重命名产出的文件
需要将产出的aar/apk移到另外一个地方的时候

```java
android.libraryVariants.all { variant ->
    variant.outputs.each { output ->
        if (output.outputFile != null && output.outputFile.name.endsWith('.aar')) {
            def name = "${rootDir}/demo/libs/library.aar"
            output.outputFile = file(name)
        }
    }
}
```

### 删除unaligned apk
删除无用的apk中间产物

```java
android.libraryVariants.all { variant ->
    variant.outputs.each { output ->
        if (output.zipAlign != null) {
            output.zipAlign.doLast {
                output.zipAlign.inputFile.delete()
            }
        }
    }
}
```

### 将自定义的任务加入构建流程
有时候编写了一些自定义的任务，希望加入到构建流程中对输入做预处理或者对输出做后处理

```java
project.tasks.whenTaskAdded { task ->
    android.applicationVariants.all { variant ->
        if (task.name == "prepare${variant.name.capitalize()}Dependencies") {
            task.dependsOn ":library:assemble${variant.name.capitalize()}"
        }
    }

}
```

比如这里app工程依赖library的构建，可以这样手工指定依赖关系

### 打包选项
有时候引用的三方库会带有一些配置文件xxxx.properties,或者license信息，打包的时候想去掉这些信息，就可以这样做

```java
android {
    packagingOptions {
        exclude 'proguard-project.txt'
        exclude 'project.properties'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/NOTICE.txt'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/DEPENDENCIES.txt'
        exclude 'META-INF/DEPENDENCIES'
    }
}
```
### lint选项开关
lint会按默认选项会做严格检查，遇到包错误会终止构建过程，可以用如下开关关掉这个选项，不过最好是重视下lint的输出，有问题及时修复，避免潜在问题

```java
android {
    lintOptions {
        abortOnError false
    }
}
```

## 依赖库篇
三方库（本地，maven）的依赖和工程库依赖关系

### 依赖库按构建目标定制
不同的依赖库可以按构建目标做定制，比如freemium的变种集成了广告，就可以这样写

```java
dependencies {
    freemiumCompile 'com.google.android.gms:ads:7.8.0'
｝
```

### aar本地库依赖
jar本地库的依赖很容易写，aar本地库的依赖稍微麻烦些

```java
allprojects {
   repositories {
      jcenter()
      flatDir {
        dirs 'libs'
      }
   }
}

dependencies {
    compile(name:'本地库aar的名字,不用加后缀', ext:'aar')
}
```

## NDK篇
NDK配置

### 只保留某一个abi，比如armeabi
为了包大小的考虑，去掉多余的本地库

```java
android {
 defaultConfig {
        ndk {
            abiFilters 'armeabi'
        }
    }
}
```

## 特殊任务篇
本篇主要是讲述一些原本GRADLE默认构建脚本不支持的任务，如果通过自定义task来完成

### 如何产生Jar文件
用GRADLE构建SDK项目的时候，有时候需要提供给集成方Jar包。如果项目是安卓库工程的方式，那么默认的产出是aar库，其实aar内部放了一个Jar（classes.jar），我们要做的，其实只要把这个jar文件拷贝出来并且重新命名就可以了。脚本参考如下：

```java
android.libraryVariants.all { variant ->
    variant.outputs.each { output ->
        def file = output.outputFile
        def fileName = 'classes.jar'
        def name = variant.buildType.name

        task "makeJar${variant.name.capitalize()}" << {
            copy {
                from("${projectDir}/build/intermediates/bundles/"+"${name}") {
                    include(fileName)
                }
                into(file.parent)
                rename (fileName, "${project.name}"+"-${name}.jar")
            }
        }
    }

}

project.tasks.whenTaskAdded { task ->
    android.libraryVariants.all { variant ->
        if (task.name == "bundle${variant.name.capitalize()}") {
            task.finalizedBy "makeJar${variant.name.capitalize()}"
        }
    }

}
```

这个办法是不是很讨巧，😄，合适的解决问题就好！

### 让public.xml工作起来吧
有人会问public.xml是做啥子的，关于这个问题，这里不赘述了，感兴趣的可以自行搜补。可惜的是，Gradle plugin 1.3已经不再支持这个功能，解决方案参考了ceabie/AndroidPublicXmlCompat，这里摘录如下：

```java
afterEvaluate {
    for (variant in android.applicationVariants) {
        def scope = variant.getVariantData().getScope()
        String mergeTaskName = scope.getMergeResourcesTask().name
        def mergeTask = tasks.getByName(mergeTaskName)

        mergeTask.doLast {
            copy {
                int i=0
                from(android.sourceSets.main.res.srcDirs) {
                    include 'values/public.xml'
                    rename 'public.xml', (i++ == 0? "public.xml": "public_${i}.xml")
                }

                into(mergeTask.outputDir)
            }
        }
    }
}
```

## 总结篇
**只有更好，木有最好；**

**只有总结，木有完结；**

笔者从软件时代开始使用构建工具和系统，一路从mk, make, cmake, qmake, 再到ant，maven, ivy, 到如今互联网时代的gradle，sbt，构建配置越来越抽水马桶化的人性体验和更多的功能，让开发者更专注于自己的业务开发，每个人都在自己的岗位专注的精耕细作专业的事，这样才会有更高效的产出和成果，用好手头的每一个工具，掌握各种姿势和适用场景，这会是高效率高质量开发的良好的开端。

**希望大家的姿势都用对了，妥妥的，新姿势，新场景，也请反馈给我，谢谢！**

## 感谢篇
[加速GRADLE构建的6个技巧](https://medium.com/@shelajev/6-tips-to-speed-up-your-gradle-build-3d98791d3df9#.hhvknc20d)

[安卓新的构建系统](http://tools.android.com/tech-docs/new-build-system)

[GRADLE官网](http://gradle.org/)

[【原文】](http://www.figotan.org/2016/04/01/gradle-on-android-best-practise/)