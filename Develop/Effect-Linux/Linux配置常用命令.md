<h1 align="center">Linux配置常用命令</h1>

[toc]

## 1.安装谷歌浏览器

1.1 按下 Ctrl + Alt + t 键盘组合键，启动终端

1.2 在终端中，输入以下命令：
(将下载源加入到系统的源列表。命令的反馈结果如图。如果返回“地址解析错误”等信息，可以百度搜索其他提供 Chrome 下载的源，用其地址替换掉命令中的地址。)
```
yuchuan@ubuntu:~$  sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/
```

1.3 在终端中，输入以下命令：
(导入谷歌软件的公钥，用于下面步骤中对下载软件进行验证。)
```
yuchuan@ubuntu:~$  wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -
```

1.4 在终端中，输入以下命令：
(用于对当前系统的可用更新列表进行更新。这也是许多 Linux 发行版经常需要执行的操作，目的是随时获得最新的软件版本信息。)
```
yuchuan@ubuntu:~$  sudo apt-get update
```

1.5 在终端中，输入以下命令：
(执行对谷歌 Chrome 浏览器（稳定版）的安装。)
```
xzm@ubuntu:~$  sudo apt-get install google-chrome-stable
```

1.6 最后，如果一切顺利，在终端中执行以下命令：(启动谷歌 Chrome 浏览器)
```
yuchuan@ubuntu:~$  /usr/bin/google-chrome-stable 
```

## 2.