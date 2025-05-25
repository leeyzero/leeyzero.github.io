---
title: CEF Windows环境二进制发布
date: 2020-10-10 22:34:29
categories:
- 编程匠艺
tags:
- CEF
- Windows
---

[CEF](https://bitbucket.org/chromiumembedded/cef)(The Chromium Embedded Framework) 是*Marshall Greenblatt*于2008年基于 [Google Chromium](http://www.chromium.org/Home) 项目创建由BSD开源协议授权的开源项目。它和Chromium项目不同之处在于，Chromium项目侧重于Google Chrome应用开发，而CEF侧重于使浏览器更容易内嵌到第三方应用中。CEF屏蔽了Chromium和
 Blink代码的复杂性，在Chromium Content API之上提供了一套友好且稳定的API，开发者只需要在CEF API的基础上就能很容易地建立起基于CEF的应用。了解更多关于CEF的内容，请参考[CEF官网](https://bitbucket.org/chromiumembedded/cef)。

## 准备编译环境

CEF官网提供了两种发布方式：二进制发布和源码发布。二进制发布包含了基于CEF开发的应用程序所依赖的所有二进制文件和头文件。本文主要讲CEF的二进制发布，官网提供了较新版本的二进制发布包，下载地址[在这里](http://opensource.spotify.com/cefbuilds/index.html), 选择一个合适的版本(在写本文是，最新版本是3202)。编译CEF需要依赖以下编译环境：

- *OS：Win7 +*
- *Visual Studio: VS2015u3 + Win10.0.14393 SDK + Ninja*
- *CMake: version 2.8.12.1+*

> 需要注意的是安装VS2015u3的时候，默认是不会安装*Win10.0.14393 SDK* 的，所以需要你手动勾选；

CMake可以去[CMake官网下载](https://cmake.org/download/) Windows安装版本。

<!--more-->

## 编译

CEF支持多平台 (*Windows*, *MacOS*, *Linux*), 以CMake作为构建工具。的使用cmake命令之前，需要先设置cmake的环境变量：

- 找到cmake的安装目录，Win7 64bit默认在*C:\Program Files (x86)\CMake*
- 控制面板 > 系统 > 高级系统设置 > 环境变量 > 系统变量 > Path
- 添加 cmake 安装目录下的bin目录，如*C:\Program Files (x86)\CMake\bin*

将下载好的二进制发布包解压到合适的目录，如`E:\`

- 进入CEF目录，如：*E:\cef_binary_3.3202.1674.g2a991c4_windows32*
- 的该目录下打开控制台cmd, 输入`cmake .` 生成cef.sln工程文件
- 用vs2015打开cef.sln进行编译

![cmake.png](/images/cef-windows-bin/1.png)

在编译ceftests的时候你可能会出现以下错误：

![compile error](/images/cef-windows-bin/2.png)

> warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss.

正如编译错误中提示的信息，这是由于os_rendering_unittest.cc中由包含有不能被当前代码页识别的字符。只需要更改该文件的编码方式就行了：

- vs 中打开os_rendering_unittest.cc文件
- *File > Advanced Save Options > Unicode (UTF-8 with signature) - Codepage 65001*
- 保存 > 重新编译

## 运行

进入tests\cefclient\Debug, 命令行执行：

```cmd
cefclient.exe --url=https://www.baidu.com
```

![cefclient.exe](/images/cef-windows-bin/3.png)

## 参考资料

[1] [CEF官网](https://bitbucket.org/chromiumembedded/cef)
[2] [CMake下载地址](https://cmake.org/download/)
[3] [Chromium Embedded Framework](http://blogs.adobe.com/webplatform/2013/05/01/the-chromium-embedded-framework/)
