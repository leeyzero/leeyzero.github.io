---
title: Vim配置Go开发环境
date: 2020-10-10 23:43:20
categories:
- 开发工具
tags:
- 开发环境
- Linux
- Vim 
- Go 
---

最近工作中使用Go开发比较多，而大部分工作都是使用vim完成，在配置vim的Go环境时，发现已经有很多现成的插件可用，对我而言，主要配置以下四个插件就够用了：

- [vim-go](https://github.com/fatih/vim-go): go语言的vim插件。支持代码格式化、语法检查、语法高亮、调试等非常多的功能。
- [tagbar](https://github.com/preservim/tagbar): 用于方便查看代码结构。
- [nerdtree](https://github.com/preservim/nerdtree): 用于管理和查看代码目录结构。
- [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe): 用于代码自动补全。

## 依赖环境

### 安装go环境

参考go官网，按步骤安装，配置好`GOROOT`和`GOPATH`环境变更即可，配置go的vim IDE环境需要依赖vim和vim-go插件。

vim-go插件需要vim使用8.0以上的版本，而YouCompleteMe需要python2.7.1+或3.5.1+。如果你系统的vim和python版本满足条件，可以忽略下面两个步骤。

<!--more-->

### 安装vim8.0+

推荐使用源码方式安装，先从github上下载[vim](https://github.com/vim/vim.git)源代码：
```shell
git clone https://github.com/vim/vim.git
```

在`src/INSTALL`安装文档中有针对各个系统的安装步骤，依照步骤安装就好，需要注意的是：**由于YouCompleteMe需要vim有python支持，python版本需要2.7.1+或3.5.1+。**可以先查看你的vim是否已经支持python:

```shell
>vim --version | grep python
+cmdline_hist      +langmap          -python            +visual
+cmdline_info      +libcall          -python3           +visualextra
```

`-`号表示不支持，重新编译在`./configure`的时候加上`--enable-pythoninterp=yes`参数就行，如果是python3，则加上`--enable-python3interp=yes`。

### 安装python2.7.1+

```shell
# Python2.7.1：
wget http://python.org/ftp/python/2.7.14/Python-2.7.14.tar.xz
tar xf Python-2.7.14.tar.xz
cd Python-2.7.14
./configure --prefix=/usr/local/python27 --enable-unicode=ucs4 --enable-shared 
make && make install
```

需要注意的是：
- make install需要管理员权限进行安装
- 编译python的时候加上--enable-unicode和--enable-shared，主要是vim打开python支持后需要python动态库支持，并且unicode需要支持ucs4。

## 环境搭建

### vim基本础配置

参考[vim无插件配置](https://www.jianshu.com/p/2a0ccc86ee31)。

### vim插件安装

我使用[Vundle](https://github.com/VundleVim/Vundle.vim.git)管理vim插件。插件安装比较简单，先打开~/.vimrc，进行需要的插件配置，在配置前建议阅读Vundle的README.md。安装比较简单，如下：

1. clone vundle.vim.git

```shell
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

2.配置插件

```shell
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

Plugin 'fatih/vim-go'

Plugin 'preservim/tagbar'

Plugin 'preservim/nerdtree'

Plugin 'Valloric/YouCompleteMe'

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
```

配置完后，在vim命令模式下执行：

```shell
:PluginInstall
```

插件会自动下载安装，看见上面显示 Finishing ... Done 的内容，插件安装成功。

### 安装vim-go

安装完vim-go插件后，vim-go本身依赖一些包，在vim命令模块下执行：

```shell
:GoInstallBinaries
```

在安装过程中，不出意外，你会遇到安装失败的情况，请在下面遇到的问题中找解决方案。

### 编译YouCompleteMe

安装完YouCompleteMe后，还需要单独编译才能运行YouaCompleteMe：

```shell
cd ~/.vim/plugged/YouCompleteMe
sh install.py --go-completer 
```

YouCompleteMe支持多语言，其它语言支持请参考YouCompleteMe源代码目录下的README.md文件。

### 插件的配置

我对NERDTree、YouCompleteMe以及tagbar的配置比较简单，需要更多配置请参考各个插件的README.md

#### vim-go插件

```shell
let g:go_fmt_command = "goimports" " 格式化将默认的 gofmt 替换
let g:go_autodetect_gopath = 1
let g:go_list_type = "quickfix"
let g:go_version_warning = 1
let g:go_highlight_types = 1
let g:go_highlight_fields = 1
let g:go_highlight_functions = 1
let g:go_highlight_function_calls = 1
let g:go_highlight_operators = 1
let g:go_highlight_extra_types = 1
let g:go_highlight_methods = 1
let g:go_highlight_generate_tags = 1
let g:godef_split=2
```

#### NERDTree插件

```shell
" 打开和关闭NERDTree快捷键
map <F10> :NERDTreeToggle<CR>
```

#### tagbar插件

```shell
nmap <F9> :TagbarToggle<CR>
let g:tagbar_width=25
let g:tagbar_type_go = {
    \ 'ctagstype' : 'go',
    \ 'kinds'     : [
        \ 'p:package',
        \ 'i:imports:1',
        \ 'c:constants',
        \ 'v:variables',
        \ 't:types',
        \ 'n:interfaces',
        \ 'w:fields',
        \ 'e:embedded',
        \ 'm:methods',
        \ 'r:constructor',
        \ 'f:functions'
    \ ],
    \ 'sro' : '.',
    \ 'kind2scope' : {
        \ 't' : 'ctype',
        \ 'n' : 'ntype'
    \ },
    \ 'scope2kind' : {
        \ 'ctype' : 't',
        \ 'ntype' : 'n'
    \ },
    \ 'ctagsbin'  : 'gotags',
    \ 'ctagsargs' : '-sort -silent'
\ }
```

## 遇到的问题

### vim-go执行:GoInstallBinaries失败

表现为：

```shell
vim-go: guru not found. Installing golang.org/x/tools/cmd/guru to folder /home/xxx/repos/gopath/bin/
vim-go: Error downloading golang.org/x/tools/cmd/guru: Fetching https://golang.org/x/tools/cmd/guru?go-get=1
vim-go: https fetch failed: Get https://golang.org/x/tools/cmd/guru?go-get=1: dial tcp 216.239.37.1:443: i/o timeout
vim-go: package golang.org/x/tools/cmd/guru: unrecognized import path "golang.org/x/tools/cmd/guru" (https fetch: Get https://gola
ng.org/x/tools/cmd/guru?go-get=1: dial tcp 216.239.37.1:443: i/o timeout)
vim-go: Error installing golang.org/x/tools/cmd/guru: can't load package: package golang.org/x/tools/cmd/guru: cannot find package "golang.org/x/tools/cmd/guru"
```

原因是在执行:GoInstallBinaries执行时会使用go get 安装依赖包(依赖包在`~/.vim/bundle/vim-go/plugin/go.vim`中可以看到)，由于国内无法访问`https://golang.org`，故会出现io timeout的情况，好在国内有比较稳定的加速镜像，简单配置一下即可：

```shell
# 启用 Go Modules 功能
go env -w GO111MODULE=on

# 配置 GOPROXY 环境变量，以下使用七牛国内镜像
go env -w  GOPROXY=https://goproxy.cn,direct

# 设置GOPATH
go env -w GOPATH=/path/to/gopath

# 确认是否设置成功
go env
```

### YouCompleteMe安装后不可用

安装完YouaCompleteMe插件后，打开vim提示：

```shell
YouCompleteMe unavailable: requires Vim compiled with Python (2.7.1+ or 3.5.1+) support
```

原因是YouCompleteMe需要vim支持python，在编译vim的时候指定`--enable-pythoninterp=yes`即可，如果是python3，则指定`--enable-python3interp=yes`

### vim加载libpython2.7.so.1.0动态库失败

表现为：

```shell
/usr/local/bin/vim: error while loading shared libraries: libpython2.7.so.1.0: cannot open shared object file: No such file or directory
```

当vim开启python支持后，需要依赖python共享库，使用ldd命令可以查看vim依赖的动态库：

```shell
> ldd /usr/local/bin/vim
```

发现的确没有找到`libpython2.7.so.1.0`。原因是在源码编译python的时候没有指定`--enable-shared`导致，重新编译python即可。由于python源码安装时，默认安装到`/usr/local/python27`下，如果仍然没有找到，可以将`/usr/local/python27/lib`下生成的`libpython2.7.so.1.0`库拷贝到`/usr/lib64/`下(64位环境），也可以在`/etc/ld.so.conf`指定动态库加载目录。

### undefined symbol pyunicodeucs4_asencodedstring

原因是源码编译python的时候没有开启asencodedstring支持，在编译python的时候加上以下参数就行：

```shell
./configure --prefix=/usr/local/python27 --enable-shared 
```

## 参考资料
[1] [将 VIM 打造成 go 语言的 ide](https://learnku.com/articles/24924)
[2] [vim-go](https://github.com/fatih/vim-go)
[3] [Vundle.vim](https://github.com/VundleVim/Vundle.vim/blob/master/CONTRIBUTING.md)
[4] [tagbar](https://github.com/preservim/tagbar)
[5] [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe)
[6] [vim](https://github.com/vim/vim)
[7] [python-出現-undefined-symbol-pyunicodeucs4_asencodedstring-錯誤](https://ephrain.net/python-%E5%87%BA%E7%8F%BE-undefined-symbol-pyunicodeucs4_asencodedstring-%E9%8C%AF%E8%AA%A4/)
[8] [nerdtree](https://github.com/preservim/nerdtree)
[9] [how-to-install-latest-python-on-centos](https://danieleriksson.net/2017/02/08/how-to-install-latest-python-on-centos/)
