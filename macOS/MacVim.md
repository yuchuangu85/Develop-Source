<h1 align="center">Mac Vim</h1>

## 一、vim配置

1. 将Solarized主题的vim文件拷贝到系统的vim目录

   

   ```bash
   cd solarized
   cd vim-colors-solarized/colors
   mkdir -p ~/.vim/colors
   cp solarized.vim ~/.vim/colors/
   ```

2. 修改vim设置:`vim ~/.vimrc`，在该文件中添加一下内容

   

   ```bash
   syntax on
   set background=dark # 背景为dark，也可选light
   colorscheme solarized
   ```

   链接：https://www.jianshu.com/p/6bf0c514ce66

## 二、常用命令

​	 ![img](media/13419355-41d3eea52b5f4aaf.png)

### Vim 的几种模式：

- **i**      进入insert模式，可以像普通编辑器一样操作。
- **esc**  进入 Normal 模式，不可不可编辑，只可滚动查看。
- **v**    进入Visual模式

### 进入Vim Esc 模式命令

```
vim xx     
# 打开xx文件，如果没有就创建xx文件，跳到编辑页面,并将光标置于第一行首
vim + xx    # 打开文件，并将光标置于最后一行首
vim +n xx   # 打开文件，并将光标置于第n行首
vim +/pattern xx # 打开文件，并将光标置于第一个与pattern匹配的串处
vim -r xx  # 在上次正用vim编辑时发生系统崩溃，恢复 xx 文件
vim xx...xx # 打开多个文件，依次编辑
```

### 移动光标命令：

```
h - 左
j - 下
k - 上
l - 右
0 - 移动到本行的行首
$ - 移动到本行的行末
gg - 移动到文档的开始位置
G - 移动到文档的末尾
```

### 撤销和重做

```
u - 撤销上一个操作
U - 撤销对当前行的所有操作
Ctrl + r 重做
```

### 搜索

```
/text + Enter + n - 向后搜索文本text
/text + Enter + N - 向前搜索文本text
```

### 插入类命令

```
i 刚进到编辑页面的时候是无法编辑的，所以输入i就可以编辑了
esc             按esc就会退出编辑模式
I   在当前行首
a   光标后
r   替换当前字符
R   替换当前字符及其后的字符，直至按ESC键
s   从当前光标位置处开始，以输入的文本替代指定数目的字符
S   删除指定数目的行，并以所输入文本代替之
ncw或nCW 修改指定数目的字
nCC 修改指定数目的行
o - 向后插入一行，并进入insert模式
O - 向前插入一行，并进入insert模式
A - 从行末开始插入文字，并进入insert模式
```

### 删除

```
x - 删除当前光标位置的字符，重复后删除光标之后的字符
X - 删除当前光标位置的字符，重复后删除光标之前的字符
r - 替换当前光标位置的字符，比如re，把光标当前位置字符替换为e
dw - 删除光标当前位置直到下一单词（不包括下一个单词的首字母）
de - 删除光标当前位置到这个单词的结束（包括这个单词的尾字母）
dd - 删除当前行
d$ - 删除当前光标位置到行末的字符
```

### 剪切粘贴

```
dd + p 其中dd是删除当前行，p粘贴到下一行
```

### 数字和快捷键连用如：

```
上  k    向上
 nk     向上移动n行
下:j         向下
 nj  向下移动n行
左:h        向左
 nh    向左移动n列
右:l         向右
 nl    向右移动n列
3w - 移动到后3个单词的首字母
10ig - 一下子插入10个g
d9e - 删除光标后九个单词（不计空格）
9dd - 删除光标当前位置往后的9行
```

### 退出命令

```
:q! - 不保存，退出
:wq - 保存并退出
:w              输入:w就是保存刚才编辑
:q              退出vim编辑页面
```


​	链接：https://www.jianshu.com/p/5f39f5fb944d

## 三、配置参考

```
"--------------------------------------------------------------
" Vundle
"--------------------------------------------------------------

" set the runtime path to include Vundle and initialize
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'
Plugin 'flazz/vim-colorschemes'

"""""""" vim scripts """"""""""""""""""
Plugin 'vim-scripts/taglist.vim'
Plugin 'vim-scripts/c.vim'
Plugin 'vim-scripts/minibufexpl.vim'
Plugin 'vim-scripts/comments.vim'
Plugin 'vim-scripts/winmanager'

"""""""" git script """""""""""""""""""
Plugin 'scrooloose/nerdtree'
Plugin 'Valloric/YouCompleteMe'
Plugin 'scrooloose/syntastic'
Plugin 'Lokaltog/vim-powerline'

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line


"--------------------------------------------------------------

" nerdtree settings

"--------------------------------------------------------------
"autocmd vimenter * NERDTree
map <F2> :NERDTreeToggle<CR>
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
map <silent> <F2> :NERDTreeToggle<CR>
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") && b:NERDTreeType == "primary") | q | endif

"--------------------------------------------------------------

" ctags settings

"--------------------------------------------------------------
let Tlist_Ctags_Cmd ='/usr/local/Cellar/ctags/5.8_1/bin/ctags'

"let g:molokai_original = 1
"let g:rehash256 = 1
"colorscheme molokai

"--------------------------------------------------------------

" syntastic setting

"--------------------------------------------------------------
let g:syntastic_check_on_open = 1
let g:syntastic_lua_checkers = ['lua', 'luac']

"--------------------------------------------------------------

" Powerline setting

"--------------------------------------------------------------
set laststatus=2
let g:Powerline_symbols='unicode'

"set foldenable
"set foldnestmax=1
"set foldmethod=syntax

"--------------------------------------------------------------

" YouCompleteMe setting

"--------------------------------------------------------------

let g:ycm_global_ycm_extra_conf = '~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/.ycm_extra_conf.py'

"let g:ycm_error_symbol = '>>'
"let g:ycm_warning_symbol = '>*'
nnoremap <leader>gl :YcmCompleter GoToDeclaration<CR>
nnoremap <leader>gf :YcmCompleter GoToDefinition<CR>
nnoremap <leader>gg :YcmCompleter GoToDefinitionElseDeclaration<CR>
nnoremap <leader>jd :YcmCompleter GoToDefinitionElseDeclaration<CR>
nmap <F4> :YcmDiags<CR>

set completeopt=longest,menu
autocmd InsertLeave * if pumvisible() == 0|pclose|endif
inoremap <expr> <CR>       pumvisible() ? "\<C-y>" : "\<CR>"

inoremap <expr> <Down>     pumvisible() ? "\<C-n>" : "\<Down>"
inoremap <expr> <Up>       pumvisible() ? "\<C-p>" : "\<Up>"
inoremap <expr> <PageDown> pumvisible() ? "\<PageDown>\<C-p>\<C-n>" : "\<PageDown>"
inoremap <expr> <PageUp>   pumvisible() ? "\<PageUp>\<C-p>\<C-n>" : "\<PageUp>"

let g:ycm_key_list_select_completion = ['<Down>']
let g:ycm_key_list_previous_completion = ['<Up>']
let g:ycm_confirm_extra_conf=0
let g:ycm_collect_identifiers_from_tags_files=1
let g:ycm_min_num_of_chars_for_completion=2
let g:ycm_cache_omnifunc=0
let g:ycm_seed_identifiers_with_syntax=1
nnoremap <F5> :YcmForceCompileAndDiagnostics<CR>
"force recomile with syntastic
inoremap <leader><leader> <C-x><C-o>
let g:ycm_complete_in_comments = 1
let g:ycm_complete_in_strings = 1
let g:ycm_collect_identifiers_from_comments_and_strings = 0

"--------------------------------------------------------------

" TagList :Tlist

"--------------------------------------------------------------
let Tlist_Show_One_File=1
let Tlist_Exit_OnlyWindow = 1
let Tlist_Ctags_Cmd="/usr/bin/ctags"

"--------------------------------------------------------------

" WinManager Setting

"--------------------------------------------------------------
let g:winManagerWindowLayout='NERDTree|TagList'
let g:winManagerWidth=30
"let g:AutoOpenWinManager = 1
nmap wm :WMToggle<cr>

"--------------------------------------------------------------

" settings

"--------------------------------------------------------------
set nocompatible            " be iMproved, required
filetype off                " required

" Set syntax highlighting for specific file types
autocmd BufRead,BufNewFile *.md set filetype=markdown

" color scheme
colorscheme molokai


" Backspace deletes like most programs in insert mode
set backspace=2
" Show the cursor position all the time
set ruler
" Display incomplete commands
set showcmd
" Set fileencodings
set fileencodings=ucs-bom,utf-8,cp936,gb2312,gb18030,big5
set background=dark
set encoding=utf-8
set fenc=utf-8
set smartindent
set autoindent
set cul
set linespace=2
set showmatch
set lines=47 columns=90
set transparency=7

" Numbers
set number
"set numberwidth=5

" font and size
"set guifont=Andale Mono:h14
"set guifont=Monaco:h11
set guifont=Menlo:h14

" Softtabs, 2 spaces
set tabstop=2
set shiftwidth=2
set shiftround
set softtabstop=2
set expandtab
set smarttab

" Display extra whitespace
"set list listchars=tab:»·,trail:·
"set list listchars=tab:-·,trail:·

" Make it obvious where 80 characters is
set textwidth=80
set colorcolumn=+1

" Highlight current line
au WinLeave * set nocursorline
au WinEnter * set cursorline
set cursorline

if has("gui_running")
    set guioptions-=m
    set guioptions-=T
    set guioptions-=L
    set guioptions-=r
    set guioptions-=b
    set showtabline=0
endif

if has('mouse')
  set mouse=a
endif

if &t_Co > 2 || has("gui_running")
syntax on
endif
```

