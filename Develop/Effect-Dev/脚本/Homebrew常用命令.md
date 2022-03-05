# Homebrew常用命令

1.更新：(brew路径：/usr/local/Cellar/)

自己更新：

```
# brew update
```

找出过期的包：

```
# brew outdated
```

升级所有过期的软件包：

```
# brew upgrade
```

升级指定的过期软件包:

```
# brew upgrade $FORMULA
```

升级过程中要暂停软件包的安装过程:

```
# brew pin $FORMULA
```

升级过程中要恢复软件包的安装过程

```
# brew unpin $FORMULA
```

清除指定软件包的所有老版本:

```
# brew cleanup $FORMULA
```

清除所有软件包的所有老版本:

```
# brew cleanup
```

查看哪些软件包要被清除:

```
# brew cleanup -n
```

彻底卸载某个软件包:

```
# brew uninstall formula_name --force
```