<h1 align="center">Git Commit Template</h1>

[toc]

## 设置

1.设置模板路径,其中path就是commit模板路径

 ```
git config --global commit.template path
 ```

 2.设置模板使用什么软件打开

```
git config --global core.editor [编辑器名字]
```

 比如

 ```
git config --global core.editor CotEditor
 ```

## 模板

<img src="media/Screen%20Shot%202020-11-05%20at%2009.56.04.png" alt="Screen Shot 2020-11-05 at 09.56.04" style="zoom:50%;" />

```objectivec
# feat-新功能（A new feature）
# fix-修补bug (A bug fix)
# docs-文档（Documentation only changes）
# style-格式（不影响代码运行的变动）(Changes that do not affect the meaning of the code(white-space, formatting, missing semi-colons, etc))
# refactor-重构（A code change that neither fixes a bug nor adds a feature）
# perf-性能 (A code change that improved performance)
# test-增加测试 (Adding missing tests or correncting existing tests)
# build-Changes that affect the build system or external dependencies (example scope: gulp, broccoli, npm)
Type of change: <类型>
Short desc: 
Long desc:
Closed issues:
```

## c参数详解

例子：

```sql
fix(DAO):用户查询缺少username属性 feat(Controller):用户查询接口开发
```

### **1. type(必须)** 用于说明git commit的类别，只允许使用下面的标识。

* feat：新功能（feature）
* fix/to：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。
   * fix：产生diff并自动修复此问题。适合于一次提交直接修复问题
   * to：只产生diff不自动修复此问题。适合于多次提交。最终修复问题提交时使用fix。
* docs：文档（documentation）
* style：格式（不影响代码运行的变动）。
* refactor：重构（即不是新增功能，也不是修改bug的代码变动）。
* perf：优化相关，比如提升性能、体验。
* test：增加测试。
* chore：构建过程或辅助工具的变动。
* revert：回滚到上一个版本。
* merge：代码合并。
* sync：同步主线或分支的Bug。

### **2. scope(可选)**
scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

### **3. subject(必须)**
subject是commit目的的简短描述，不超过50个字符。

- 建议使用中文。
- 结尾不加句号或其他标点符号。
   根据以上规范git commit message将是如下的格式：



## 参考

https://zhuanlan.zhihu.com/p/34223150

https://zhuanlan.zhihu.com/p/69635847