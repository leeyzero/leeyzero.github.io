---
title: MacOS下使用lrzsz传输文件
date: 2020-10-10 23:20:43
categories:
- 开发工具
tags:
- 开发环境
- macOS
- lrzsz
---

在mac环境下，我们经常会使用[iTerm2](https://www.iterm2.com/index.html)终端连接远程服务器，也经常会有本机和远程服务器之间进行文件共享的需求。这个时候lrzsz就派上用场了。

[lrzsz](https://www.ohse.de/uwe/software/lrzsz.html)是unix下的开源软件包，支持[XMODEM, YMODEM](ftp://ftp.std.com/obi/Standards/FileTransfer/YMODEM8.DOC.1.Z) [ZMODEM](http://www.easysw.com/~mike/serial/zmodem.html)文件传输协议。本文将会展示如何将lrzsz集成到iTerm2终端中，通过`sz`和`rz`命令和远程服务器传输文件。 其中，`s`表示`send`，`r`表示`recieve`，z表示使用的协议为ZMODEM。

<!--more-->

## 安装步骤
### 安装lrzsz
- 最简单的方式是通过brew安装：`brew install lrzsz`
- 也可以通过下载[源码](https://www.ohse.de/uwe/releases/lrzsz-0.12.20.tar.gz)安装

### 下载iterm2-zmodem支持脚本

- 克隆代码库：`git clone https://github.com/mmastrac/iterm2-zmodem.git`
- 将`iterm2-send-zmodem.sh` 和 `iterm2-recv-zmodem.sh` 脚本拷贝到目录`/usr/local/bin/`

### 在iTerm2中配置Trigger
- `iTerm2 > Preference > Profiles > Advanced > Triggers > Edit`
- 增加rz和sz的配置如下：

```
Regular expression: rz waiting to receive.\*\*B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh
Instant: checked

Regular expression: \*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
Instant: checked
```

更多关于Trigger的配置请参考[这里](https://www.iterm2.com/documentation-triggers.html)。

### **远程服务器安装lrzsz**
使用相关包管理工具安装lrzsz，如在centos下使用yum安装：`yum -y install lrzsz `

## 使用演示
### 上传文件到远程服务器
上传文件到远程服务器比较简单，在iTerm2登录远程服务器后，直接在命令行输入命令：`rz`，iTerm2收到带有数据匹配到 `rz waiting to receive.**B0100`，执行脚本`/usr/local/bin/iterm2-send-zmodem.sh`，调起Finder将选择的文件上传至远程服务器。

### 从远程服务器下载文件
从远程服务器下载文件到本地也很简单，键入命令：`sz filename1 filename2 … filenameN`，当iTerm2收到数据匹配到`**B00000000000000`时，执行脚本`/usr/local/bin/iterm2-recv-zmodem.sh`，调起Finder，将文件下载到选择目录。

## 更多参考

[1] https://github.com/mmastrac/iterm2-zmodem
[2] https://www.ohse.de/uwe/software/lrzsz.html
[3] https://www.iterm2.com/documentation-triggers.html
