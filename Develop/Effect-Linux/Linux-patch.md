<h1 align="center">Linux patch</h1>

## 通过diff工具生成补丁, patch工具打上补丁.
在使用diff之前, 你需要保留一份未修改过的源码, 然后在其它地方修改源码的一份拷贝. diff对比这两份源码生成patch. 修改过的源码必须保留原来的文件名, 例如, 如果你修改源码中的a.c文件, 那么, 修改后的文件还是名为a.c, 在修改之前你可以复制a.c为a.orig.c进行备份.

### 1.为单个文件生成补丁
```
1 $ diff -up linux-2.6.28.8/net/sunrpc/svc.orig.c linux-2.6.28.8/net/sunrpc/svc.c > patch
```

这条命令会产生类似如下的输出, 你将它重定向到一个文件中, 这个文件就是patch.

```
1 diff -up linux-2.6.28.8/net/sunrpc/svc.orig.c 2009-03-17 08:50:04.000000000 +0800
2 +++ linux-2.6.28.8/net/sunrpc/svc.c 2009-03-30 19:18:41.859375000 +0800
3 @@ -1050,11 +1050,11 @@ svc_process(struct svc_rqst *rqstp)
```

参数详解:
-u 显示有差异行的前后几行(上下文), 默认是前后各3行, 这样, patch中带有更多的信息.
-p 显示代码所在的c函数的信息.

### 2.为多个文件生成补丁
```
1 $ diff -uprN linux-2.6.28.8.orig/net/sunrpc/ linux-2.6.28.8/net/sunrpc/ > patch
```

这条命令对比了linux-2.6.28.8.orig/net/sunrpc/和linux-2.6.28.8/net/sunrpc/两个目录下的所有源码差异.
参数详解:
-r 递归地对比一个目录和它的所有子目录(即整个目录树).
-N 如果某个文件缺少了, 就当作是空文件来对比. 如果不使用本选项, 当diff发现旧代码或者新代码缺少文件时, 只简单的提示缺少文件. 如果使用本选项, 会将新添加的文件全新打印出来作为新增的部分.

### 3.打补丁
生成的补丁中, 路径信息包含了你的Linux源码根目录的名称, 但其他人的源码根目录可能是其它名字, 所以, 打补丁时, 要进入你的Linux源码根目录, 并且告诉patch工具, 请忽略补丁中的路径的第一级目录(参数-p1).
```
1 $ patch -p1 < patch1.diff
```

diff命令必须在整个Linux源码的根目录的上一级目录中执行.

### 4. 示例
给修改过的内核生成patch，然后用生成的patch给未修改过的内核打补丁
其中，目录linux-2.6.31.3为未修改过的内核，目录linux-2.6.31.3_1为修改过的内核
```
1 $ diff -uparN linux-2.6.31.3 linux-2.6.31.3_1/ > mypatch
2 $ cd linux-2.6.31.3
3 $ patch -p1 < mypatch
```
