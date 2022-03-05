<h1 align="center">jar 命令</h1>
(1)创建jar包

```
jar cf hello.jar hello  // 后面的hello是项目名称
```

利用test目录生成hello.jar包,如hello.jar存在,则覆盖

(2)创建并显示打包过程
```
jar cvf hello.jar hello
```
利用hello目录创建hello.jar包,并显示创建过程
例：
```
E:\>jar cvf hello.jar hello
```
标明清单(manifest)
增加：hello/(读入= 0) (写出= 0)(存储了 0%)
增加：hello/TestServlet2.class(读入= 1497) (写出= 818)(压缩了 45%)
增加：hello/HelloServlet.class(读入= 1344) (写出= 736)(压缩了 45%)
增加：hello/TestServlet1.class(读入= 2037) (写出= 1118)(压缩了 45%)

(3)显示jar包：
```
jar tvf hello.jar   查看hello.jar包的内容
```
指定的jar包必须真实存在，否则会发生FileNoutFoundException。

(4)解压jar包：
```
jarxvf hello.jar
```
解压hello.jar至当前目录

(5)jar中添加文件：
```
jar uf hello.jar HelloWorld.java
```
将HelloWorld.java添加到hello.jar包中

(6)创建不压缩内容jar包：
```
jar cvf0 hello.jar *.class
```
利用当前目录中所有的.class文件生成一个不压缩jar包

(7)创建带manifest.mf文件的jar包：
```
jar cvfm hello.jar manifest.mf hello
```
创建的jar包多了一个META-INF目录,META-INF止录下多了一个manifest.mf文件,至于manifest.mf的作用,后面会提到.

(8)忽略manifest.mf文件：
```
jar cvfM hello.jar hello
```
生成的jar包中不包括META-INF目录及manifest.mf文件

(9)加-C应用：
```
jar cvfm hello.jar mymanifest.mf -C hello/
```
表示在切换到hello目录下然后再执行jar命令

(10)-i为jar文件生成索引列表：
当一个jar包中的内容很好的时候，你可以给它生成一个索引文件，这样看起来很省事。
```
jar i hello.jar
```
执行完这条命令后，它会在hello.jar包的META-INF文件夹下生成一个名为INDEX.LIST的索引文件，它会生成一个列表，最上边为jar包名。

(11)导出解压列表：
jar tvf hello.jar >hello.txt   如果你想查看解压一个jar的详细过程，而这个jar包又很大，屏幕信息会一闪而过，这时你可以把列表输出到一个文件中，慢慢欣赏！

(12)
```
jar -cvf hello.jar hello/*
```
   例如原目录结构如下：
   hello
     |---com
     |---org

你本想只把com目录和org目录打包，而这时jar命令会连同hello目洋也一块打包进。这点大家要注意。jar命令生成的压缩文件会包含它后边出的目录。我们应该进入到hello目录再执行jar命令。

注意：manifest.mf这个文件名，用户可以任指定，但jar命令只认识Manifest.mf，它会对用户指定的文件名进行相应在的转换，这不需用户担心。

(13) 启动jar包

```
java -jar hello.jar &
```

