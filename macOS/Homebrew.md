<h1 align="center">Homebrew</h1>

[TOC]

## 安装

```
// 安装
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

```
// 安装wget
$ brew install wget
```

```
// 查找位置
$ cd /usr/local
$ find Cellar
Cellar/wget/1.16.1
Cellar/wget/1.16.1/bin/wget
Cellar/wget/1.16.1/share/man/man1/wget.1

$ ls -l bin
bin/wget -> ../Cellar/wget/1.16.1/bin/wget

```
## 结构

Homebrew 主要有四个部分组成: brew、homebrew-core 、homebrew-bottles、homebrew-cask。

|       名称       |             说明              |
| :--------------: | :---------------------------: |
|       brew       |      Homebrew源代码仓库       |
|  homebrew-core   |     Homebrew核心软件仓库      |
| homebrew-bottles |  Homebrew预编译二进制软件包   |
|  homebrew-cask   | 提供macOS应用和大型二进制文件 |



## 常用命令

```
// force removing the brewed version of git
brew uninstall --force git

// cleanup any older versions and clear the brew cache
brew cleanup --force -s git

// Remove any dead symlinks
brew cleanup --prune-prefix

// uninstall ignore dependencies 
brew uninstall --ignore-dependencies python

```

## 基础命令

Example usage:

    brew search [TEXT|/REGEX/]
    brew info [FORMULA...]
    brew install FORMULA...
    brew update
    brew upgrade [FORMULA...]
    brew uninstall FORMULA...
    brew list [FORMULA...]

Troubleshooting:

    brew config
    brew doctor
    brew install --verbose --debug FORMULA

Contributing:

    brew create [URL [--no-fetch]]
    brew edit [FORMULA...]

Further help:

    brew commands
    brew help [COMMAND]
    man brew

https://docs.brew.sh

## 配置本地源

### [替换为阿里源](https://developer.aliyun.com/mirror/)

```
# 查看 brew.git 当前源
$ cd "$(brew --repo)" && git remote -v
origin    https://github.com/Homebrew/brew.git (fetch)
origin    https://github.com/Homebrew/brew.git (push)

# 查看 homebrew-core.git 当前源
$ cd "$(brew --repo homebrew/core)" && git remote -v
origin    https://github.com/Homebrew/homebrew-core.git (fetch)
origin    https://github.com/Homebrew/homebrew-core.git (push)

# 修改 brew.git 为阿里源
$ git -C "$(brew --repo)" remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git

# 修改 homebrew-core.git 为阿里源
$ git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git

# zsh 替换 brew bintray 镜像
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.zshrc
$ source ~/.zshrc

# bash 替换 brew bintray 镜像
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile

# 刷新源
$ brew update
```

### [替换为清华源](https://mirror.tuna.tsinghua.edu.cn/help/homebrew/)

```
# 替换各个源
$ git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
$ git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
$ git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git

# zsh 替换 brew bintray 镜像
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.zshrc
$ source ~/.zshrc

# bash 替换 brew bintray 镜像
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile

# 刷新源
$ brew update
```

### [替换为中科大源](http://mirrors.ustc.edu.cn/)

```
# 替换各个源
$ git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git
$ git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
$ git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

# zsh 替换 brew bintray 镜像
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
$ source ~/.zshrc

# bash 替换 brew bintray 镜像
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile

# 刷新源
$ brew update
```

### 重置为官方源

```
# 重置 brew.git 为官方源
$ git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git

# 重置 homebrew-core.git 为官方源
$ git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git

# 重置 homebrew-cask.git 为官方源
$ git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask

# zsh 注释掉 HOMEBREW_BOTTLE_DOMAIN 配置
$ vi ~/.zshrc
# export HOMEBREW_BOTTLE_DOMAIN=xxxxxxxxx

# bash 注释掉 HOMEBREW_BOTTLE_DOMAIN 配置
$ vi ~/.bash_profile
# export HOMEBREW_BOTTLE_DOMAIN=xxxxxxxxx

# 刷新源
$ brew update
```

## HomebrewCN：Homebrew的国内安装脚本

### 项目地址

[https://gitee.com/cunkai/HomebrewCN](https://gitee.com/cunkai/HomebrewCN)

### 脚本内容

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

![Screen Shot 2020-06-05 at 13.54.37](media/Screen%20Shot%202020-06-05%20at%2013.54.37.png)

然后输入序号选择下载源即可。

## 修复homebrew安装软件404问题

* [修复homebrew安装软件404问题 - TinkOL](https://www.tinkol.com/372)

  之所以不行，是因为我用了清华大学的镜像，目前清华大学的镜像中依然是指向到bintray的，所以也跟着404了。所以要解决问题，那就是去掉清华大学的镜像设置就好了。具体的环境变量是`$HOMEBREW_BOTTLE_DOMAIN`。使用`export HOMEBREW_BOTTLE_DOMAIN=''`命令就可以去除。上面是临时修改，如果想永久修改，则需要去更新profile文件，zsh是`~/.zprofile`文件，bash要修改`~/.bash_profile`文件。

  修改完后，再进行安装，就一切正常了。

## [Error: Cannot install in Homebrew on ARM processor in Intel default prefix (/usr/local)](https://stackoverflow.com/questions/64963370/error-cannot-install-in-homebrew-on-arm-processor-in-intel-default-prefix-usr)

For what it's worth, before installing Homebrew you will need to install Rosetta2 emulator for the new ARM silicon (M1 chip). I just installed Rosetta2 via terminal using:

```
/usr/sbin/softwareupdate --install-rosetta --agree-to-license
```

This will install rosetta2 with no extra button clicks.

After installing Rosetta2 above you can then use the Homebrew cmd and install Homebrew for ARM M1 chip: 

```
arch -x86_64 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

Once Homebrew for M1 ARM is installed use this Homebrew command to install packages: 

```
arch -x86_64 brew install <package>
```



