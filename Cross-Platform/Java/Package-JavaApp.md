<h1 align="center">package Java applications as native</h1>

[toc]

> Window下打包需要安装wix3：https://github.com/wixtoolset/wix3



## Java程序打包资料

### [javapackager](https://docs.oracle.com/javase/10/tools/javapackager.htm)

Github:[fvarrui/JavaPackager: Gradle/Maven plugin to package Java applications as native Windows, Mac OS X, or GNU/Linux executables and create installers for them. (github.com)](https://github.com/fvarrui/JavaPackager)

> The Java Packager tool compiles, packages, and prepares Java and JavaFX applications for distribution. The javapackager command is the command-line version.
>
> – Oracle's documentation

The `javapackager` utility ships with the JDK. It can generate .exe files with the `-native exe` flag, among many other things.

### [WinRun4J](http://winrun4j.sourceforge.net/)

> WinRun4j is a java launcher for windows. It is an alternative to javaw.exe and provides the following benefits:
>
> - Uses an INI file for specifying classpath, main class, vm args, program args.
> - Custom executable name that appears in task manager.
> - Additional JVM args for more flexible memory use.
> - Built-in icon replacer for custom icon.
> - *[more bullet points follow]*
>
> – WinRun4J's webpage

WinRun4J is an open source utility. It has *many* features.

### [packr](https://github.com/libgdx/packr)

> Packages your JAR, assets and a JVM for distribution on Windows, Linux and Mac OS X, adding a native executable file to make it appear like a native app. Packr is most suitable for GUI applications.
>
> – packr README

packr is another open source tool.

### [JSmooth](http://jsmooth.sourceforge.net/)

> JSmooth is a Java Executable Wrapper. It creates native Windows launchers (standard .exe) for your java applications. It makes java deployment much smoother and user-friendly, as it is able to find any installed Java VM by itself.
>
> – JSmooth's website

JSmooth is open source and has features, but it is very old. The last release was in 2007.

### [JexePack](https://www.duckware.com/jexepack/index.html)

> *JexePack* is a command line tool (great for automated scripting) that allows you to package your Java application (class files), optionally along with its resources (like GIF/JPG/TXT/etc), into a single *compressed* 32-bit Windows EXE, which runs using Sun's Java Runtime Environment. Both console and windowed applications are supported.
>
> – JexePack's website

JexePack is trialware. Payment is required for production use, and exe files created with this tool will display "reminders" without payment. Also, the last release was in 2013.

### [InstallAnywhere](https://www.flexera.com/products/installation/installanywhere.html)

> InstallAnywhere makes it easy for developers to create professional installation software for any platform. With InstallAnywhere, you’ll adapt to industry changes quickly, get to market faster and deliver an engaging customer experience. And know the vulnerability of your project’s OSS components before you ship.
>
> – InstallAnywhere's website

InstallAnywhere is a commercial/enterprise package that generates installers for Java-based programs. It's probably capable of creating .exe files.

### Executable JAR files

As an alternative to .exe files, you can create a JAR file that automatically runs when double-clicked, by [adding an entry point to the JAR manifest](https://docs.oracle.com/javase/tutorial/deployment/jar/appman.html).

### For more information

An excellent source of information on this topic is Excelsior's article "[Convert Java to EXE – Why, When, When Not and How](https://www.excelsior-usa.com/articles/java-to-exe.html)".

See also the companion article "[Best JAR to EXE Conversion Tools, Free and Commercial](https://www.excelsior-usa.com/articles/best-jar-to-exe-conversion-tools-free-commercial.html)".



## Package Tools

* [libgdx/packr: Packages your JAR, assets and a JVM for distribution on Windows, Linux and Mac OS X (github.com)](https://github.com/libgdx/packr)
* [fvarrui/JavaPackager: Gradle/Maven plugin to package Java applications as native Windows, Mac OS X, or GNU/Linux executables and create installers for them. (github.com)](https://github.com/fvarrui/JavaPackager)----打包插件
* [libgdx/packr: Packages your JAR, assets and a JVM for distribution on Windows, Linux and Mac OS X (github.com)](https://github.com/libgdx/packr)--打包工具
* [petr-panteleyev/jpackage-gradle-plugin: JPackage Gradle Plugin (github.com)](https://github.com/petr-panteleyev/jpackage-gradle-plugin)
* [FibreFoX/javafx-gradle-plugin: Gradle plugin for JavaFX (github.com)](https://github.com/FibreFoX/javafx-gradle-plugin)



## jpackage打包

* [JDK14之jpackage打包命令_u012698467的博客-CSDN博客_jpackage](https://blog.csdn.net/u012698467/article/details/104897144/)
* [Java 应用程序打包 - jpackage 使用记录 | Raven's Blog (ravenxrz.ink)](https://ravenxrz.ink/archives/421e5ad2.html)
* [Basic Packaging (oracle.com)](https://docs.oracle.com/en/java/javase/14/jpackage/basic-packaging.html#GUID-24F95A39-049C-491A-B49A-75898FAA7C57)
* [The Badass JLink Plugin (beryx.org)](https://badass-jlink-plugin.beryx.org/releases/latest/)
* [The jpackage Command (oracle.com)](https://docs.oracle.com/en/java/javase/14/docs/specs/man/jpackage.html)
* [java - Gradle jpackage creating unrunnable application - Stack Overflow](https://stackoverflow.com/questions/66517949/gradle-jpackage-creating-unrunnable-application)
* [Solved\] Java Gradle badassruntimeplugin and ProGuard Gradle Plugin - Code Redirect](https://coderedirect.com/questions/578888/gradle-badass-runtime-plugin-and-proguard-gradle-plugin)
* [Package JavaFX applications | IntelliJ IDEA (jetbrains.com)](https://www.jetbrains.com/help/idea/packaging-javafx-applications.html#java_fx_build_artifact)
* [java - What is the best way to deploy JavaFX application, create JAR and self-contained applications and native installers - Stack Overflow](https://stackoverflow.com/questions/30145772/what-is-the-best-way-to-deploy-javafx-application-create-jar-and-self-contained/30162808#30162808)
* [java - Gradle jpackage creating unrunnable application - Stack Overflow](https://stackoverflow.com/questions/66517949/gradle-jpackage-creating-unrunnable-application)
* [使用jpackage将java程序打包成exe程序(不需要安装jdk即可运行) - zeromi - 博客园 (cnblogs.com)](https://www.cnblogs.com/zeromi/p/14852323.html)



## 参考

* [Packaging Overview (oracle.com)](https://docs.oracle.com/en/java/javase/14/jpackage/packaging-overview.html#GUID-C1027043-587D-418D-8188-EF8F44A4C06A)--打包讲解
* [The Java Packager Tool (oracle.com)](https://docs.oracle.com/javase/8/docs/technotes/guides/deploy/packager.html)--jdk打包工具教程
* [Packaging Java apps for the Windows/Linux desktop - Stack Overflow](https://stackoverflow.com/questions/7720/packaging-java-apps-for-the-windows-linux-desktop)