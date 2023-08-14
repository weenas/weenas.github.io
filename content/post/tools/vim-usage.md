---
title: vim 常用配置
slug: vim_usage
date: 2017-02-03
tags:
description: "代码编辑器中的瑞士军刀。"
categories:
  - 工具
keywords:
  - vim
  - configuration
  - 配置
  - ctags
  - cscope
---

VIM的强大无须多说，最强大的代码编辑器之一。多年使用下来积累了一些配置，在这里记录和分享。

<!--more-->

## vimrc

通常VIM启动时自动加载两个配置文件：/etc/vimrc和~/.vimrc，前一个是系统级的配置，后一个是用户级的配置。系统级配置需要root权限，如果是在公共服务器上一般没有这个权限，所以修改用户级的配置是更好的选择。

以下便是我自己的~/.vimrc，包括了常用配置和快捷键配置，在配置里有注释说明。其中一些格式化配置，比如将TAB替换为空格和缩进长度，对于团队合作非常有帮助，如果团队成员统一配置文件，那么写出来的代码风格将是一致的。

```
set nocompatible
set history=400
filetype on
set shiftwidth=4
set softtabstop=4
set tabstop=4
set smarttab
"set expandtab
set lbr

"Set to auto read when a file is changed from the outside
if exists("&autoread")
set autoread
endif

"Fast saving
nmap <leader>w :w!<cr>

syntax on
set encoding=utf-8

if exists("&cursorline")
set cursorline
endif

"Set 5 lines to the curors - when moving vertical..
set so=5

"Turn on WiLd menu
set wildmenu

"Always show current position
set ruler

"The commandbar is 2 high
set cmdheight=2

"Show line number
set nu

"Change buffer - without saving
set hid

"Set backspace
set backspace=eol,start,indent

"Ignore case when searching
"set ignorecase
set incsearch

"Set magic on
set magic

"show matching bracet
set showmatch

" 光标短暂跳转到匹配括号的时间, 单位是十分之一秒
set matchtime=2

"Highlight search thing
set hlsearch

"Turn backup off
set nobackup
set nowb

"Enable folding, I find it very useful
if exists("&foldenable")
set fen
endif

if exists("&foldlevel")
set fdl=0
endif

"Auto indent
set ai

"Smart indet
set si

"C-style indenting
set cindent
set smartindent

"Wrap line
set wrap

" 在处理未保存或只读文件的时候，弹出确认
set confirm

" 配色方案(可用 :highlight 查看配色方案细节)
" 这里有各种配色方案：
" http://vimcolorschemetest.googlecode.com/svn/html/index-c.html
"colorscheme murphy
colorscheme desert
"colorscheme darkblue

" 自定义状态行
set statusline=%F%m%r%h%w[%L][%{&ff}]%y[%p%%][%04l,%04v]
"              | | | | |  |   |      |  |     |    |
"              | | | | |  |   |      |  |     |    +-- 当前列数
"              | | | | |  |   |      |  |     +-- 当前行数
"              | | | | |  |   |      |  +-- 当前光标位置百分比
"              | | | | |  |   |      +-- 使用的语法高亮器
"              | | | | |  |   +-- 文件格式
"              | | | | |  +-- 文件总行数
"              | | | | +-- 预览标志
"              | | | +-- 帮助文件标志
"              | | +-- 只读标志
"              | +-- 已修改标志
"              +-- 当前文件绝对路径


" 显示当前正在键入的命令
set showcmd

" 设置自动切换目录为当前文件所在目录，用 :sh 时候会很方便
"set autochdir


" Only do this part when compiled with support for autocommands.
if has("autocmd")

" For all text files set 'textwidth' to 78 characters.
autocmd FileType text setlocal textwidth=78

" When editing a file, always jump to the last known cursor position.
" Don't do it when the position is invalid or when inside an event handler
" (happens when dropping a file on gvim).
autocmd BufReadPost *
if line("'"") > 0 && line("'"") <= line("$") |
exe "normal g`"" |
endif

endif


" Ctags
map <C-F12> :!ctags -R --c++-kinds=+p --fields=+iaS --extra=+q .<CR>

" Sscope
set cscopequickfix=s-,c-,d-,i-,t-,e-

" Taglist
let Tlist_Show_One_File=1
let Tlist_Exit_OnlyWindow=1
" Use Ctrl+t to open Tlist window.
map <C-t> :TlistToggle<cr>
" 把方法列表放在屏幕的右侧
let Tlist_Use_Right_Window=1

" 禁止自动改变当前Vim窗口的大小
let Tlist_Inc_Winwidth=0

" 让当前不被编辑的文件的方法列表自动折叠起来， 这样可以节约一些屏幕空间
let Tlist_File_Fold_Auto_Close=1
```

## Ctags配置

Ctags是一个外部工具，它能将源代码中的符号生成到一个tags文件，VIM启动时会默认从当前目录读取tags文件。

Ctags的安装很简单，在Ubuntu下的命令如下：

```bash
sudo apt-get install ctags
```

在源码根目录生成tags文件：

```bash
ctags -R *
```

使用方法：

在VIM普通模式下，将光标定位在符号上，这个符号可以是变量名、函数名、或者是宏定义。按`<Ctrl + ]>`可以立即跳转到符号定义的地方，再按`<Ctrl + o>`返回到跳转之前的地方，跳转是可以递归的。

使用技巧：

tags生成后不会随着源文件的修改而改变，因此源文件修改后tags记录的行数就会不准确，这时需要对tags更新。如果用之前的方法需要先退出VIM，使用ctags更新，再进入VIM继续编辑，非常繁琐。有一个简单的方法是定义快捷键，在VIM界面想更新时按下快捷键就更新好了。

快捷键的配置已经包含在前面的配置文件中：

```
map <C-F12> :!ctags -R --c++-kinds=+p --fields=+iaS --extra=+q .<CR>
```

## Taglist配置

Taglist的作用是将VIM当前源文件中的所有符号罗列出来，形成单独的一栏，通过这些符号列表可以直接跳转到定义的地方，这个功能很像是Windows上的一些IDE工具。

配置方法：

下载最新的taglist，解压后将plugin目录夹拷贝至~/.vim/目录，如果这个目录不存在要先创建，安装就这么简单。接下进入VIM使用以下命令使用taglist：

:TlistOpen       打开Taglist窗口
:TlistClose      关闭Taglist窗口
:TlistToggle     在打开和关闭之间切换
注意命令的大小写，输入这些命令非常繁琐，还容易出错，还是快捷键比较方便：

```
map <C-t> :TlistToggle<cr>
```
即使用Ctrl和t键，实现打开和关闭taglist。

在taglist窗口(vim窗口之间切换的方法为<Ctrl - w + w>)，可以使用下面这些命令：

```
<CR>          跳到光标下tag所定义的位置，用鼠标双击此tag功能也一样
o             在一个新打开的窗口中显示光标下tag
<Space>       显示光标下tag的原型定义
u             更新taglist窗口中的tag
s             更改排序方式，在按名字排序和按出现顺序排序间切换
x             taglist窗口放大和缩小，方便查看较长的tag
+             打开一个折叠，同zo
-             将tag折叠起来，同zc
*             打开所有的折叠，同zR
=             将所有tag折叠起来，同zM
[[            跳到前一个文件
]]            跳到后一个文件
q             关闭taglist窗口
<F1>          显示帮助
```

# CScope配置

Cscope和Ctags比较类似，都是用于快速查找定位的工具，但Cscope在管理大型项目上更方便，速度也更快。

安装：

```bash
sudo apt-get install cscope
```

和ctags一样，cscope也需要先建立索引文件，我一般采用增加命令的方法。将下面的配置放到~/.bashrc中，命令行中就可以使用buildref命令快速生成索引文件。

```bash
buildref()
{
	find -name "*.[chsS]" -o -name "*.cpp" -o -name "*.java" > cscope.files
		cscope -bkq
}
```

Cscope的使用指令也较复杂，只有使用快捷键才能体现出他的强大之处，将下面的配置保存为cscope_maps.vim放到~/.vim/plugin目录，就能轻松的使用Cscope了。

```bash
if has("cscope")
" use both cscope and ctag for 'ctrl-]', ':ta', and 'vim -t'
set cscopetag

" check cscope for definition of a symbol before checking ctags: set to 1
" if you want the reverse search order.
set csto=0

" add any cscope database in current directory
if filereadable("cscope.out")
cs add cscope.out
" else add the database pointed to by environment variable
elseif $CSCOPE_DB != ""
cs add $CSCOPE_DB
endif

" show msg when any other cscope db added
set cscopeverbose

" The following maps all invoke one of the following cscope search types:
"
" 's' symbol: find all references to the token under cursor
" 'g' global: find global definition(s) of the token under cursor
" 'c' calls: find all calls to the function name under cursor
" 't' text: find all instances of the text under cursor
" 'e' egrep: egrep search for the word under cursor
" 'f' file: open the filename under cursor
" 'i' includes: find files that include the filename under cursor
" 'd' called: find functions that function under cursor calls
nmap <C->s :cs find s <C-R>=expand("<cword>")<CR><CR>
nmap <C->g :cs find g <C-R>=expand("<cword>")<CR><CR>
nmap <C->c :cs find c <C-R>=expand("<cword>")<CR><CR>
nmap <C->t :cs find t <C-R>=expand("<cword>")<CR><CR>
nmap <C->e :cs find e <C-R>=expand("<cword>")<CR><CR>
nmap <C->f :cs find f <C-R>=expand("<cfile>")<CR><CR>
nmap <C->i :cs find i ^<C-R>=expand("<cfile>")<CR>$<CR>
nmap <C->d :cs find d <C-R>=expand("<cword>")<CR><CR>

endif
```

`<C - >s`的意思是先同时按Ctrl和键，释放后再按s键，代表查找光标所在的所有符号。默认情况下，光标会跳转到查找到的第一个符号处，并在最下面显示总共搜索到几个。要查看所有的列表，输入:cw命令，VIM会分出一个窗口来显示cscope列表。光标可以在符号列表之间上下切换，按回车直接跳转到光标所在的符号，如下图所示。

在VIM多窗口之间切换光标的命令是`<C - w>w`，这是一个常用的命令，有了这个命令就不用鼠标在各个窗口之间切换了。

