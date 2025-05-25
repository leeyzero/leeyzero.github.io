---
title: CEF Windows环境源码编译
date: 2020-10-10 22:21:09
categories:
- 编程匠艺
tags:
- CEF
- Windows
---

由于项目需要用到内嵌浏览器，IE内核太依赖于操作系统，对H5的支持也不太好。[CEF](https://bitbucket.org/chromiumembedded/cef)是基于[chromium 项目](http://www.chromium.org/Home)的内嵌浏览器开源框架，已经应用到了[很多产品](https://en.wikipedia.org/wiki/Chromium_Embedded_Framework#Applications_using_CEF)中，而且有比较健全的[论坛](http://magpcss.org/ceforum/index.php)和[官方支持](https://bitbucket.org/chromiumembedded/cef)，是项目的不二选择。由于客户端要运行到Windows XP系统，但Chome浏览器在49版本(对应CEF3版本为2623，以下说的CEF均指CEF3)后不再支持Win7以下系统。CEF[二进制发布官网](http://opensource.spotify.com/cefbuilds/index.html)上并未包含2623版本, 但CEF提供了源码发布，官方也给了[源码编译文档说明](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding)，虽然按照官方文档说明进行构建，但在编译CEF 2623版本过程还是遇到不少问题，在此做个记录供网友参考。

## 准备编译环境 

- OS: Win7 64bit 以上系统, 至少8G内存，60G以上硬盘，最好是SSD
- Visual Studio: VS2015u3 + Win10.0.14393 SDK + Ninja
- Python 2.7+
- 下载 [automate-git.py 脚本](https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py)

注意:

> 1. 由于某些原因（你懂的），源代码是同步不下来的，建议使用VPN。另外，CEF基于chromium，源码比较大，同步源码时间比较漫长，耐心等待；
> 2. 安装VS2015u3的时候，默认是不会安装Win10.0.14393 SDK的，所以需要你手动勾选；
> 3. 安装python后需要将python的执行环境加入到环境变量中；

<!--more-->

## 下载CEF源码

在下载CEF源码之前先了解一下自动构建脚本automate-git.py, 命令行执行：

```shell
python automate-git.py -h
```

可以看到源码同步、编译、发布等相关选项，每个选项都有相关说明。默认情况下同步完源代码后会自动构建并发布二进制包，由于同步源代码和构建时间都比较长，所以将同步源码和编译源码分开。先来看看目录结构：

```shell
CEF
 | --- 2623
automate
 | --- automate-git.py
 | --- sync_cef_2623.bat
 | --- build_cef_2623.bat
```

说明：
- 目录CEF\2623：用于下载CEF 2623分支的源代码
- 目录automate：用于放置自动构造相关脚本
- automate-git.py：自动构建脚本
- sync_cef_2623.bat：同步2623分支源码批处理脚本，具体内容往下看
- build_cef_2623.bat：编译2623分支源码批处理脚本，具体内容往下看

以上目录结构属于个人偏好，不一定非要这样设置，关键是正确设置automate-git.py的参数。sync_cef_2623.bat用于同步CEF 2623分支源码，其内容如下：

```shell
set CEF_USE_GN=0
set GYP_DEFINES=buildtype=Official
set GYP_GENERATORS=ninja,msvs-ninja 
set GYP_MSVS_VERSION=2015
set CEF_ARCHIVE_FORMAT=tar.bz2
set DEPOT_TOOLS_WIN_TOOLCHAIN=0
python automate-git.py --download-dir=..\CEF\2623 --branch=2623 --no-build --no-distrib
```

在sync_cef_2623.bat脚本中写入以上内容，在保证以上环境设置正确的前提下，在命令行运行sync_cef_2623.bat就可以同步源代码了，源代码有十几个G,如果网络不稳定的情况下，会花比较长的时间。

下面是一些参数说明，更多参数，请运行`python automate-git.py -h`查看，

- *--download-dir* ：设置下载源码的目录，这个地方用相对路径，下载到和automate同级的CEF\2623目录下
- *--branch* ：同步2623分支源码
- *--no-build* ：不编译
- *--no-distrib* ：不发布

注：

> 1. 如果VPN不太稳定，可能会经常中断，造成脚本运行错误，重新运行脚本即可，源码主要有三部分，depot_tools源码cef源码和chromium源码，如果发现相应部分的源码已经同步完成，可以分别使用--no-depot-tools-update, --no-cef-update, --no-chromium-update  来不进行相应部分的代码更新。
> 2. 自动构建脚本在下载源码时，并没有进度提示，比如在下载chromium/src目录时时间会非常长，会一直提示`still working on src...`，请耐心等待。

## 编译CEF源码

源代码下载完成后，就可以进行编译了，编译2623版本，可能会遇到一些编译错误，如果你不想亲自试一下这些编译错误，请直接看下节编译错误解决办法。编译源码用脚本build_cef_2623.bat，其内容如下：

```shell
set CEF_USE_GN=0
set GYP_DEFINES=buildtype=Official
set GYP_GENERATORS=ninja,msvs-ninja 
set GYP_MSVS_VERSION=2015
set CEF_ARCHIVE_FORMAT=tar.bz2
set DEPOT_TOOLS_WIN_TOOLCHAIN=0
python automate-git.py --download-dir=..\CEF\2623 --branch=2623 --no-update --no-depot-tools-update --verbose-build --force-build --no-debug-build --force-distrib
```

在build_cef_2623.bat脚本中写入以上内容，就可以进行release版本的编译和发布了，以下是一些参数说明：
- *--no-update* ：不更新cef和chromium源码
- *--no-depot-tools-update* : 不更新depot-tools源码
- *--verbose-build* : 开启编译日志，该参数比较重要，编译出错时方便定位问题，相关编译错误见下节
- *--force-build* : 强制编译。在编译失败时，重新运行脚本能重新编译，不加该参数，则会提示out目录已存在
- *--no-debug-build* : 不进行debug版本的编译，之所以加该参数是因为如果debug和release一起编译，时间会比较长，产生的二进制也比较大，如果你的空间足够，时间足够，可以去掉该参数
- *--force-distrib* ：强制发布二进制包，如果已经发布过，再次运行脚本会重新生成二进制发布包

此外还有其它编译参数供选择，请运行`python automate-git.py -h`查看.

## 解决编译错误

在编译过程中，你可能会遇到以下编译或链接错误, 以我的CEF源码所在位置*e:\repos\cef\2623*为例：

### 编译错误1

> 如果你用的系统locale是中国，可能会遇到一些字符集相关的编码错误，需要更改系统区域设置，Win10系统下修改如下，其它系统请自行[google](www.google.com)：

```plaintext
控制面板 > 时钟、语言和区域 > 区域 > 管理 > 更新系统区域设置...
```

修改为:

```plaintext
英语(美国)
```

### 编译错误2
>*e:\repos\cef\2623\chromium\src\google_apis\gaia\oauth2_token_service.cc(313): 
error C2220: warning treated as error - no 'object' file generated
warning C4334: '<<': result of 32-bit shift implicitly converted to 64 bits*

该编译错误比较明显，具体请参考[MSDN](https://msdn.microsoft.com/en-us/library/ke55d167.aspx)，根据路径打开oauth2_token_service.cc，定位到313行, 将:

```c++
  int64_t exponential_backoff_in_seconds = 1 << retry_num;
```

修改为：

```c++
  int64_t exponential_backoff_in_seconds = 1i64 << retry_num;
```

### 编译错误3
>*e:\repos\cef\2623\chromium\src\ui\gl\gl_bindings_skia_in_process.cc(920): note: while trying to match the argument list '(GrGLInterface::GLPtr<GrGLMapBufferRangeProc>, overloaded-function)'*

从编译器给出的错误提示，可以看出是函数参数不匹配，打开gl_bindings_skia_in_process.cc, 定位到920行:

```c++
  functions->fMapBufferRange = StubGLMapBufferRange;
```

functions是类型是GrGLInterface::Functions ,根据gl_bindings_skia_in_process.cc头部引用的头文件，打开*e:\repos\cef\2623\chromium\src\third_party\skia\include\gpu\gl\GrGLInterface.h*，可以看到fMapBufferRange的类型为*GrGLMapBufferRangeProc*，其声明在*e:\repos\cef\2623\chromium\src\third_party\skia\include\gpu\gl\GrGLFunctions.h*中，声明如下：

```c++
typedef GrGLvoid* (GR_GL_FUNCTION_TYPE* GrGLMapBufferRangeProc(GrGLenum target, 
                                            GrGLintptr offset, 
                                            GrGLsizeiptr length, 
                                            GrGLbitfield access);
```

再看*gl_bindings_skia_in_process.cc*中*StubGLMapBufferRange*的定义：

```c++
void* GR_GL_FUNCTION_TYPE StubGLMapBufferRange(GLenum target,
                                               GLintptr offset,
                                               GLsizeiptr length,
                                               GLbitfield access) {
    return glMapBufferRange(target, offset, length, access);
}
```

从上面可以看出，的确是参数类型不匹配，其它StubXXX函数也有同样的问题，清楚了这个问题后，就好修复了，在*gl_bindings_skia_in_process.cc*，找到所有以GL开头的类型，将其增加前缀Gr就行了，这样一个个改比较麻烦，我们只需要重新typedef一下即可，需要typedef的类型不多，修改如下，在*gl_bindings_skia_in_process.cc 19*行处插入以下代码：

```c++
typedef GrGLenum GLenum;
typedef GrGLuint GLuint;
typedef GrGLchar GLchar;
typedef GrGLclampf GLclampf;
typedef GrGLint GLint;
typedef GrGLbitfield GLbitfield;
typedef GrGLsizeiptr GLsizeiptr;
typedef GrGLboolean GLboolean;
typedef GrGLsizei GLsizei;
typedef GrGLfloat GLfloat;
typedef GrGLintptr GLintptr;
```

### 编译错误4
>*e:\repos\cef\2623\chromium\src\third_party\swiftshader\include\egl\eglext.h(752): error C2061: syntax error: identifier 'EGLAttrib'
e:\repos\cef\2623\chromium\src\third_party\swiftshader\include\egl\eglext.h(753): error C2061: syntax error: identifier 'EGLAttrib'
e:\repos\cef\2623\chromium\src\third_party\swiftshader\include\egl\eglext.h(1057): error C2061: syntax error: identifier 'EGLAttrib'
e:\repos\cef\2623\chromium\src\third_party\swiftshader\include\egl\eglext.h(1121): error C2061: syntax error: identifier 'EGLAttrib'*

编译错误也比较明确，就是找不到`EGLAttrib`类型，CEF论坛中已经给出了[答案](http://www.magpcss.org/ceforum/viewtopic.php?f=6&t=14473), 修改方法是打开eglext.h, 在代码typedef intptr_t EGLAttribKHR;(62行)后插入以下代码：

```c++
typedef EGLAttribKHR EGLAttrib; 
```

### 编译错误5

>*ffmpeg.lib(ffmpeg.wavdec.obj) : error LNK2001: unresolved external symbol _ff_w64_guid_data
libcef.dll : fatal error LNK1120: 1 unresolved externals*

这个是一个链接错误，找不到外部符号*_ff_w64_guid_data*，一开始不知道怎么修复这个错误，那就[google](www.google.com)吧，好在已经有网友给出了解决办法，请参考：

- [*CONFIG_W64_DEMUXER and dead-code elimination*](http://ffmpeg.org/pipermail/ffmpeg-devel/2016-May/194142.html)
- [*Add missing w64.c to fix VS 2015 Update 3 link errors*](https://chromium-review.googlesource.com/c/chromium/third_party/ffmpeg/+/343398/4)

修改方法是分别打开*e:\repos\cef\2623\chromium\src\third_party\ffmpeg\ffmpeg_generated.gni* 和
*e:\repos\cef\2623\chromium\src\third_party\ffmpeg\ffmpeg_generated.gypi*,

```plaintext
在 libavformat/wavdec.c 前加上 libavformat/w64.c
```

## 发布二进制包

编译完成后，二进制发布包默认生成路径在*e:\repos\cef\2623\chromium\src\cef\binary_distrib*, 二进制发布包的编译请参考我的另一文章[CEF Windows环境二进制发布](https://leeyzero.github.io/2020/10/10/cef-windows-bin/).

另外在*e:\repos\cef\2623\chromium\src\out\Release*中其实已经生成了cefclient.exe(如果编译的是debug版本，则在Debug目录)，在命令行执行：

```shell
cefclient.exe --url=https://www.baidu.com
```

效果如下：

![cefclient.exe](/images/cef-windows-source/1.png)

## 其它问题

将编译好的二进制拷贝到Windows XP系统下运行，发现Debug版本会崩溃，Release能正常工作。Debug版本加上`--single-process`参数，能够正常运行，但退出时也会崩溃。该问题可以参考链接：[Crash on Windows XP](https://bitbucket.org/chromiumembedded/cef/issues/1787)，CEF维护者 *Marshall Greenblatt* 已经给出了解决方法；鉴于*CEF 2623 Release*版本在XP下工作良好，且不需要在XP下进行调试工作，*CEF 2623*版本的编译工作可以告一段落。

## 参考资料

[1] https://bitbucket.org/chromiumembedded/cef/
[2] http://blog.csdn.net/yufei_lgq/article/details/53838270
