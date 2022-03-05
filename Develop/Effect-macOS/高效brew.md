# 高效brew

## 1.树形结构：tree

#### 安装：
```
brew install tree
```

#### 指定遍历层级
```
tree -L N // N表示层级
```

#### 输出树形结构
```
tree -L 2 >TREE.md // 输出树形结构到TREE.md文件里
```

#### 只显示文件夹
```
tree -d
```

#### 过滤输出
```
tree -I "node_modules" // 过滤node_modules文件夹下的文件
```