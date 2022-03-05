<h1 align="center">Java GUI</h1>

[toc]

## Java Swing

### 资料

* [Lesson: Using Swing Components (The Java™ Tutorials > Creating a GUI With Swing) (oracle.com)](https://docs.oracle.com/javase/tutorial/uiswing/components/index.html)
* [Java Swing 图形界面开发（目录）_XTS的专栏-CSDN博客_swing](https://blog.csdn.net/xietansheng/article/details/72814492)



### 美化

* [JFormDesigner/FlatLaf](https://github.com/JFormDesigner/FlatLaf): FlatLaf - Flat Look and Feel (with Darcula/IntelliJ themes support)
* [FlatLaf](https://www.formdev.com/flatlaf/): Flat Look and Feel | FormDev
* [bulenkov/Darcula](https://github.com/bulenkov/Darcula): Darcula Look and Feel



## JavaFX

### 资料

* [JavaFX](https://openjfx.io/)--官网
* [openjdk/jfx](https://github.com/openjdk/jfx)--JavaFX mainline development
* [JavaFX China](http://javafxchina.net/main/)
* [jfreechart](https://github.com/jfree/jfreechart)
* [TestFX](https://github.com/TestFX/TestFX/): Simple and clean testing for JavaFX.
* [OpenJDK Wiki](https://wiki.openjdk.java.net/display/OpenJFX)
* [JavaFx与Jfoenix](https://www.cnblogs.com/stars-one/category/1474826.html)
* [IntelliJ IDEA and JavaFX | The IntelliJ IDEA Blog (jetbrains.com)](https://blog.jetbrains.com/idea/2021/01/intellij-idea-and-javafx/)
* [LiveStream Summary: JavaFX, the Cross-platform UI Development in Java | The IntelliJ IDEA Blog (jetbrains.com)](https://blog.jetbrains.com/idea/2021/02/livestream-summary-javafx-the-cross-platform-ui-development-in-java/)
* [JavaFX中文视频](https://space.bilibili.com/5096022/channel/seriesdetail?sid=394169):视频教程
* [Create a new JavaFX project | IntelliJ IDEA (jetbrains.com)](https://www.jetbrains.com/help/idea/javafx.html)
* [Configure JavaFX Scene Builder | IntelliJ IDEA (jetbrains.com)](https://www.jetbrains.com/help/idea/opening-fxml-files-in-javafx-scene-builder.html)
* [Package JavaFX applications | IntelliJ IDEA (jetbrains.com)](https://www.jetbrains.com/help/idea/packaging-javafx-applications.html)
* [JavaFX: Cross-platform UI development in Java on desktop, mobile and embedded clients - YouTube](https://www.youtube.com/watch?v=JbITM8xapIQ)
* [JavaFX 精选资源 | 来唧唧歪歪(Ljjyy.com) - 多读书多实践，勤思考善领悟](https://www.ljjyy.com/archives/2019/08/100573.html)



### 依赖文件

* [Central Repository: org/openjfx (maven.org)](https://repo1.maven.org/maven2/org/openjfx/)
* [Central Repository: org/openjfx/javafx-graphics/11.0.2 (maven.org)](https://repo1.maven.org/maven2/org/openjfx/javafx-graphics/11.0.2/)



### 开发工具
* [JavaFX Scene Builder 1.x Archive (oracle.com)](https://www.oracle.com/java/technologies/javafxscenebuilder-1x-archive-downloads.html) 开发JavaFX的视图工具



### Material Theme

* [sshahine/JFoenix: JavaFX Material Design Library (github.com)](https://github.com/sshahine/JFoenix)
* [palexdev/MaterialFX: A library of material components for JavaFX (github.com)](https://github.com/palexdev/MaterialFX)



### Theme

* [Maddosaurus/aerofx: JavaFX Skin for native Windows 7 look & feel (github.com)](https://github.com/Maddosaurus/aerofx)
* [JFXtras/jfxtras-styles: Java, JavaFX themes or look and feels. Currently contains JMetro theme. (github.com)](https://github.com/JFXtras/jfxtras-styles)
* [guigarage/AquaFX (github.com)](https://github.com/guigarage/AquaFX)
    * [AquaFX | GuiGarage](https://guigarage.com/aquafx/)
* [guigarage/aerofx: JavaFX Skin for native Windows 7 look & feel (github.com)](https://github.com/guigarage/aerofx)
    * [AeroFX | GuiGarage](https://guigarage.com/aerofx/)
* [dicolar/jbootx: a javafx theme of bootstrap (github.com)](https://github.com/dicolar/jbootx)



### 开源项目

* [asciidocfx/AsciidocFX: Asciidoc Editor and Toolchain written with JavaFX 16 (Build PDF, Epub, Mobi and HTML books, documents and slides) (github.com)](https://github.com/asciidocfx/AsciidocFX)



### 注意

Beginning with Java 11, JavaFX is no longer part of the JDK. It has been extracted to its own project: OpenJFX. This means, extra dependencies must be added to your project.
The easiest way to add the JavaFX libraries to your Gradle project is to use the JavaFX Gradle Plugin.
After following the README for the JavaFX Gradle Plugin you will end up with something like:
```groovy
plugins {
    id 'org.openjfx.javafxplugin' version '0.0.8'
}

javafx {
    version = '12'
    modules = [ 'javafx.controls', 'javafx.fxml' ]
}
```



### TornadoFX--JavaFX kotlin封装

* [edvin/tornadofx: Lightweight JavaFX Framework for Kotlin (github.com)](https://github.com/edvin/tornadofx)：源码
* [Introduction · TornadoFX Guide (gitbooks.io)](https://edvin.gitbooks.io/tornadofx-guide/content/)--JavaFX的kotlin版本的指导文档



## Syntax Highlighting -- 代码语法高亮

* [bobbylight/RSyntaxTextArea: A syntax highlighting, code folding text editor for Java Swing applications. (github.com)](https://github.com/bobbylight/RSyntaxTextArea): 代码高亮编辑器
* [bobbylight/AutoComplete: A code completion library for Swing text components, with special support for RSyntaxTextArea. (github.com)](https://github.com/bobbylight/AutoComplete)
* [bobbylight/RSTALanguageSupport: A library adding code completion and other advanced features for Java, JavaScript, Perl, and other languages to RSyntaxTextArea. (github.com)](https://github.com/bobbylight/RSTALanguageSupport)
* [bobbylight/SpellChecker: A spell checker add-on library for RSyntaxTextArea. (github.com)](https://github.com/bobbylight/SpellChecker)
* [bobbylight/RSTAUI: A library of common dialogs and UI elements needed by applications embedding text components such as RSyntaxTextArea. (github.com)](https://github.com/bobbylight/RSTAUI)
* [jflex-de/jflex: The fast scanner generator for Java™ with full Unicode support (github.com)](https://github.com/jflex-de/jflex)



## libgdx

### 官方资源

* [libgdx/libgdx: Desktop/Android/HTML5/iOS Java game development framework (github.com)](https://github.com/libgdx/libgdx) -- 源码
* [libGDX](http://libgdx.com/)--官网
* [Home · libgdx/libgdx Wiki (github.com)](https://github.com/libgdx/libgdx/wiki)--wiki,一些教程和资源



### 相关开源资源

* [yairm210/Unciv: Open-source Android/Desktop remake of Civ V (github.com)](https://github.com/yairm210/Unciv)
* [sivvig/ZombieBird: Zombie Bird source code (http://www.kilobolt.com/zombie-bird-tutorial-flappy-bird-remake.html) (github.com)](https://github.com/sivvig/ZombieBird)
* [collinsmith/riiablo: Diablo II remade using Java and LibGDX (github.com)](https://github.com/collinsmith/riiablo)
* [qjoy/libGDX-Android-AppEffect: Show a way of android application's super animation effects with libgdx. (github.com)](https://github.com/qjoy/libGDX-Android-AppEffect)
* [lucasnlm/antimine-android: Antimine is an open source minesweeper-like puzzle game. (github.com)](https://github.com/lucasnlm/antimine-android)
* [dingjibang/GDX-RPG: java & libgdx制作的RPG游戏！ an RPG by java and LibGDX (github.com)](https://github.com/dingjibang/GDX-RPG)
* [agateau/pixelwheels: A top-down retro racing game for PC (Linux, macOS, Windows) and Android. (github.com)](https://github.com/agateau/pixelwheels)
* 

## 编译安装文件

* [fvarrui/JavaPackager: Gradle/Maven plugin to package Java applications as native Windows, Mac OS X, or GNU/Linux executables and create installers for them. (github.com)](https://github.com/fvarrui/JavaPackager)----打包插件
* [Packaging Overview (oracle.com)](https://docs.oracle.com/en/java/javase/14/jpackage/packaging-overview.html#GUID-C1027043-587D-418D-8188-EF8F44A4C06A)--打包讲解
* [The Java Packager Tool (oracle.com)](https://docs.oracle.com/javase/8/docs/technotes/guides/deploy/packager.html)--jdk打包工具教程

