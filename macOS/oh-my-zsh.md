<h1 align="center">oh-my-zsh</h1>

[toc]

>新版Mac自带了zsh并且是默认shell命令，只需要更新即可

## ohmyzsh Git 镜像使用帮助

### 安装

在本地克隆后获取安装脚本。

```
git clone https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
cd ohmyzsh/tools
REMOTE=https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git sh install.sh
```

### 切换已有 ohmyzsh 至镜像源

```
git -C $ZSH remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
git -C $ZSH pull
```

## 下载oh my zsh到~/.oh-my-zsh

```
1、通过curl方式安装：
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
2、通过wget方式安装
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 更新：

```
brew install zsh zsh-completions
```

## set up zsh as default(把zsh设置成默认shell)

```
#设置
chsh -s $(which zsh)
#查检-需要关闭终端重新打开后生效
echo $SHELL
```

## zsh: command not found: adb

配置了sdk环境变量到.zshrc 文件中后，仍报该错误，则需要在.bash_profile里再配置一次，然后再.zshrc中配置

```
source $HOME/.bash_profile
```

即可





## 来源

* 原文链接：https://blog.csdn.net/shenhonglei1234/article/details/106653646