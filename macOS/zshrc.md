# zshrc文件配置

```
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH

# Path to your oh-my-zsh installation.
export ZSH=$HOME/.oh-my-zsh

# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="robbyrussell"

# Set list of themes to pick from when loading at random
# Setting this variable when ZSH_THEME=random will cause zsh to load
# a theme from this variable instead of looking in $ZSH/themes/
# If set to an empty array, this variable will have no effect.
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )

# Uncomment the following line to use case-sensitive completion.
# CASE_SENSITIVE="true"

# Uncomment the following line to use hyphen-insensitive completion.
# Case-sensitive completion must be off. _ and - will be interchangeable.
# HYPHEN_INSENSITIVE="true"

# Uncomment the following line to disable bi-weekly auto-update checks.
# DISABLE_AUTO_UPDATE="true"

# Uncomment the following line to automatically update without prompting.
# DISABLE_UPDATE_PROMPT="true"

# Uncomment the following line to change how often to auto-update (in days).
# export UPDATE_ZSH_DAYS=13

# Uncomment the following line if pasting URLs and other text is messed up.
# DISABLE_MAGIC_FUNCTIONS="true"

# Uncomment the following line to disable colors in ls.
# DISABLE_LS_COLORS="true"

# Uncomment the following line to disable auto-setting terminal title.
# DISABLE_AUTO_TITLE="true"

# Uncomment the following line to enable command auto-correction.
# ENABLE_CORRECTION="true"

# Uncomment the following line to display red dots whilst waiting for completion.
# Caution: this setting can cause issues with multiline prompts (zsh 5.7.1 and newer seem to work)
# See https://github.com/ohmyzsh/ohmyzsh/issues/5765
# COMPLETION_WAITING_DOTS="true"

# Uncomment the following line if you want to disable marking untracked files
# under VCS as dirty. This makes repository status check for large repositories
# much, much faster.
# DISABLE_UNTRACKED_FILES_DIRTY="true"

# Uncomment the following line if you want to change the command execution time
# stamp shown in the history command output.
# You can set one of the optional three formats:
# "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
# or set a custom format using the strftime function format specifications,
# see 'man strftime' for details.
# HIST_STAMPS="mm/dd/yyyy"

# Would you like to use another custom folder than $ZSH/custom?
# ZSH_CUSTOM=/path/to/new-custom-folder

# Which plugins would you like to load?
# Standard plugins can be found in $ZSH/plugins/
# Custom plugins may be added to $ZSH_CUSTOM/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(git)

source $ZSH/oh-my-zsh.sh

# User configuration
# source ~/.bash_profile

# export MANPATH="/usr/local/man:$MANPATH"

# You may need to manually set your language environment
# export LANG=en_US.UTF-8

# Preferred editor for local and remote sessions
# if [[ -n $SSH_CONNECTION ]]; then
#   export EDITOR='vim'
# else
#   export EDITOR='mvim'
# fi

# Compilation flags
# export ARCHFLAGS="-arch x86_64"

# Set personal aliases, overriding those provided by oh-my-zsh libs,
# plugins, and themes. Aliases can be placed here, though oh-my-zsh
# users are encouraged to define aliases within the ZSH_CUSTOM folder.
# For a full list of active aliases, run `alias`.
#
# Example aliases
# alias zshconfig="mate ~/.zshrc"
# alias ohmyzsh="mate ~/.oh-my-zsh"



## JDK:
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home
export CLASSPAHT=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH:

##-----------------------------------------------------------------------------
## OpenJDK:
export JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home
export CLASSPAHT=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH:

##-----------------------------------------------------------------------------
## SDK:
## pidcat config
export PATH=$PATH:/Users/{username}/AndroidSDK/platform-tools
export PATH=$PATH:/Users/{username}/AndroidSDK/tools

#export ANDROID_HOME=/Users/{username}/AndroidSDK
#export PATH=${PATH}:${ANDROID_HOME}/tools
#export PATH=${PATH}:${ANDROID_HOME}/platform-tools

##-----------------------------------------------------------------------------
## NDK:
export NDK_ROOT=/Users/{username}/AndroidSDK/ndk/23.0.7599858
export PATH=$PATH:$NDK_ROOT

##-----------------------------------------------------------------------------
## gradle: gradle -version 
# GRADLE_HOME=/Users/{username}/Gradle/gradle-6.7.1
# export GRADLE_HOME
# export PATH=$PATH:$GRADLE_HOME/bin

##-----------------------------------------------------------------------------
##Maven
export M2_HOME=/usr/local/Cellar/maven/3.5.0
export PATH=$PATH:$M2_HOME/bin

##-----------------------------------------------------------------------------
## brew
export PATH="$(brew --prefix coreutils)/libexec/gnubin:/usr/local/bin:$PATH"

##-----------------------------------------------------------------------------
##GNU
PATH=$PATH:/Users/{username}/DevelopHome/global-6.5.6

##-----------------------------------------------------------------------------
##GO
#GOROOT=/usr/local/go
#export GOROOT=/usr/local/go
#export PATH=$PATH:$GOROOT/bin

##-----------------------------------------------------------------------------
##Python
export PATH="/usr/local/opt/python/libexec/bin:$PATH"


##-----------------------------------------------------------------------------
##Chromium for Android
##export PATH="$PATH:/path/to/depot_tools"
##export PATH="/Users/{username}/Develop/AndroidOS/depot_tools"

##----------------------------------------------------------------------------
# flutter
export PATH=$PATH:/Users/{username}/FlutterSDK/bin
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

##----------------------------------------------------------------------------
## Asop源码代理
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles

export PATH="/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin"export PATH="/usr/local/sbin:$PATH"

##----------------------------------------------------------------------------
##替换系统的vim
# alias vim='mvim -v'

##------------------------------------------------------------------------------
##node.js
# export NODE_HOME="/usr/local/Cellar/node/15.4.0" 
# export PATH=$PATH:$NODE_HOME/bin

export PATH=$PATH:$HOME/bin

```

