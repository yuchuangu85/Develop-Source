# 反编译apk

#### 第一步：
下载反编译工具集，apktool、dex2jar、jd-gui，最后我会上传这些工具，解压后如下图：
 
![](/images/decompilation/decompilation1.jpg)

#### 第二步
工具集准备好之后还不能进行反编译，你在命令窗口下执行输入apktool 会提示命令不存在，需要配置一下环境变量
，怎么配置呢，如下命令：
   * 1.打开命令窗口，cd /usr/local/bin下，可能有的mac电脑不存在bin这个目录，直接在创建一个就好了，命令是:sudo mkdir bin,执行后会让你输入root权限密码，输入完后执行ls命令查看，bin目录就存在了，如图：
![](/images/decompilation/decompilation2.jpg)

  * 2.将你解压后的apktool文件夹下的三个文件aapt、apktool、apktool.jar 复制到/usr/local/bin/目录下，怎么复制呢，当然用命令cp了，如下图：
 ![](/images/decompilation/decompilation3.jpg)

 
复制多个文件用空格隔开，Android-workspace/APK/apktool/目录是源目录，存放的是我们要复制的那三个文件，执行命令后，提示输入密码，输完密码后就复制成功了，ls查看一下，这三个文件已经存在了。

这时候就已经配置好环境变量PATH了，什么，我怎么没看到和PATH有关的任何命令，其实，/usr/local/bin本来就在PATH下，不信我执行命令你看：
![](/images/decompilation/decompilation4.jpg)

这也是我们把apktool3个文件放在/usr/local/bin下的原因，现在你再输入apktool命令试试：
![](/images/decompilation/decompilation5.jpg)

输出这样的命令就代表环境配置好了

#### 第三步：
开始进行反编译了，其实用到的命令也很简单，我们随便拿一个apk来，例如：
![](/images/decompilation/decompilation6.jpg)

我们要对dz-android.apk进行反编译，命令行进入这个目录 cd /Users/hailonghan/android-workspace/APK，到这个目录后，
执行命令apktool d dz-android.apk,如下图：
![](/images/decompilation/decompilation7.jpg)

执行成功后，会在当前目录下生成一个da-android文件夹，点击去就看到相关apk的资源文件了，选中AndroidManifest.xml，然后空格键，就可以看到反编译后的内容了：
![](/images/decompilation/decompilation8.jpg)

#### 第四步：
反编译java源文件
这就用到dex2jar和jd-gui了,将dz-android.apk重命名改成dz-android.zip，然后利用解压缩软件解压，得到一个dz-android目录，我们要拿到里面的classes.dex文件，如图：
![](/images/decompilation/decompilation9.jpg)


将其复制到dex2jar-0.0.9.15目录下，如图：
![](/images/decompilation/decompilation10.jpg)

然后执行命令：sh dex2jar.sh classes.dex,如图：
![](/images/decompilation/decompilation11.jpg)

执行成功后会生成一个classes_dex2jar.jar文件，如图：
![](/images/decompilation/decompilation12.jpg)


最后，用jd-gui工具打开这个jar包就可以看到java源代码了，如图：
![](/images/decompilation/decompilation13.jpg)



