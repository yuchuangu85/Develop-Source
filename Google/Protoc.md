<h1 align="center">Protoc编译</h1>

* 安装：

```
brew tap homebrew/versions
brew install protobuf241
```

241为版本号，也可以不加，会自动安装最新版

* 编译：
打开到protos路径，然后执行下面命令，前面是文件名，后面是输出路径

```
protoc launcher_dump.proto --javanano_out=../src/
protoc launcher_log.proto --javanano_out=../src/
```