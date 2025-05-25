---
title: Vim无插件配置
date: 2020-10-09 23:00:56
categories:
- 开发工具
tags:
- 开发环境
- Linux
- Vim
---

[vim](https://en.wikipedia.org/wiki/Vim_(text_editor))是unix下一款非常强大的文本编辑器。vim本身支持很多个性化的配置，根据自身需求，合理的配置vim，能够很好的提高开发效率。本文介绍vim下无插件的常用配置，关于配置vim有一个非常重要的原则就是：**不要将你不明白的配置项写到配置文件中。**

## 准备环境

首先需要安装vim，目前大部分类unix系统默认都已经安装好了。如果没有安装，通常可以用系统的包管理功能进行安装，如centos下使用yum安装。但使用包管理工具安装默认的包源的vim版本可能比较低，推荐使用源码方式安装：

```shell
git clone https://github.com/vim/vim.git
```

源代码clone下来后，`src/INSTALL`文档中有非常详细的安装说明文档，根据自己的系统环境按步骤安装即可。

<!--more-->

## 基础知识
### 配置文件
vim是通过vimrc配置文件进行配置的，全局配置一般在`/etc/vim/vimrc`或者`/etc/vimrc`，对所有用户生效。用户个人的配置在`~/.vimrc`。对我们来讲，通常在`~/.vimrc`进行配置就行了。

除了在配置中进行配置外，在vim编辑过程中，可以在命令模式下，直接输入配置命令，即对当前编辑的文件生效，如显示行号：
```
:set number
```

### 注释
vimrc文件支持注释，`"`后面的内容为注释，如：
```
" 显示行号
set number
syntax on " 打开语法高亮
```

### 帮助文档
vim有非常详细的帮助文档，在编辑过程中，你可以通过`F1`或`:help`快捷键呼出帮忙文档，也可以对某个配置项查看其配置说明，如

```
:help showmatch
```

### 配置惯用法
配置项一般都有"打开"和"关闭"两个设置。"关闭"就是在"打开"前面加上前缀"no"。如显示等号为`set number`，关闭就为`set nonumber`，大部分配置都有简写，如显示等号可以简写为`set nu`，关闭就为`set nonu`。

## 常用配置
```
" ======================================
" 基本配置
" ======================================

" 不兼容vi命令
set nocompatible

" 打开语法高亮
syntax on

" 开启文件类型检查，并且载入与该类型对应的缩进规则。
filetype indent on

" 在底部状态栏显示当前模式，如插入、命令模式
set showmode

" 在命令模式下显示当前命令，如输入2y时，会在状态栏显示命令，再次输入y时，执行命令，状态栏命令消失
set showcmd

" 是否显示状态栏。0表示不显示，1表示只在多窗口时显示，2表示显示。
set laststatus=2

" 在状态栏显示光标的当前位置
set  ruler

" 支持鼠标
set mouse=a

" 当前文本使用uf8编码
set encoding=utf-8  

" 保留命令的历史记录数
set history=1000

" 显示行号
set number

" 光标所在的当前行高亮
set cursorline

" 设置行宽，即一行显示多少个字符
set textwidth=100

" 自动折行，即太长的行分成几行显示，关闭自动折行为set nowrap
set wrap

" 只有遇到指定的符号（比如空格、连词号和其他标点符号），才发生折行。也就是说，不会在单词内部折行
set linebreak

" 垂直滚动时，光标距离顶部/底部的位置（单位：行）
set scrolloff=5

" 水平滚动时，光标距离行首或行尾的位置（单位：字符）。该配置在不折行时比较有用。
set sidescrolloff=10


" ======================================
" 缩进相关配置
" ======================================

" 按下tab时显示的空格数
set tabstop=4

" tab转化为多少个空格
set softtabstop=4

" 执行移位操作`>>或<<`时，显示的空格数
set shiftwidth=4

" 由于 tab 键在不同的编辑器缩进不一致，该设置自动将 Tab 转为空格
set expandtab

" 自动缩略，当按下回车时，自动与上一行的缩进保持一致
set autoindent


" ======================================
" 搜索相关配置
" ======================================

" 光标遇到{[()]}时，会高亮显示另一半匹配的符号
set showmatch

" 高亮显示搜索的词
set hlsearch

" 增量搜索匹配结果，即每输入一个字母都会进行匹配
set incsearch

" 搜索时忽略大小写
set ignorecase

" 如果同时打开了ignorecase，那么对于只有一个大写字母的搜索词，将大小写敏感；其他情况都是大小写不敏感
set smartcase


" ======================================
" 编辑相关配置
" ======================================
" 不创建交换文件。交换文件主要用于系统崩溃时恢复文件，文件名的开头是.、结尾是.swp
set noswapfile

" 自动切换工作目录。这主要用在一个 Vim 会话之中打开多个文件的情况，默认的工作目录是打开的第一个文件的目录。该配置可以将工作目录自动切换到，正在编辑的文件的目录。
set autochdir

" 出错时，不要发出响声
set noerrorbells

" 出错时，发出视觉提示，通常是屏幕闪烁
set visualbell

" 打开文件监视。如果在编辑过程中文件发生外部改变，就会发出提示。
set autoread

" 命令模式下，底部操作指令按下 Tab 键自动补全
set wildmenu
```

以上配置可以通过github克隆到本地：

```bash
git clone https://github.com/leeyzero/vimrc.git 
```

## 更多参考
[1] [Vim 配置入门](http://www.ruanyifeng.com/blog/2018/09/vimrc.html)
[2] [a-good-vimrc](https://dougblack.io/words/a-good-vimrc.html)
[3] [Vim documentation: options](http://vimdoc.sourceforge.net/htmldoc/options.html)
[4] [basic vim](https://github.com/amix/vimrc/blob/master/vimrcs/basic.vim)
