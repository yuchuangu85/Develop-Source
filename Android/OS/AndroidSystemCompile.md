<h1 align="center">Android OS 编译</h1>

## 1.全编译：

```java
./prize_project/koobee/K606Q/K606Q/build_user.sh        // user版本./prize_project/koobee/K606Q/K606Q/build_userdebug.sh   // debug调试版本./prize_project/koobee/K606Q/K606Q/build_eng.sh         // eng版本
```

prize_project是根目录下的文件夹，里面有各个项目分支。

## 2.单编译：

#### 第一步：
```source build/envsetup.sh
```#### 第二步：

```lunch full_pri6763_66l_kb_n1-user
```#### 单编译模块：

```mmm packages/apps/Settings/
```

强制编译

```mmm packages/apps/Settings/ -B
```#### 注意： 第一步 第二步 设置环境（只设置一次）

## 3.提交代码：

查看单独模块状态：

```
git status packages/apps/PrizeDeskClockV8/
```

拉去代码：

```
git pull
```

提交代码：

```
git add packages/apps/PrizeDeskClockV8/
git commit -m "ui:模块，注释"
git push
```


## 5.抓取log到文件：

```
cat build_log.txt | grep -i error
```
