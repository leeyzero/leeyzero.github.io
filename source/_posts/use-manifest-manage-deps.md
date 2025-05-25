---
title: 使用manifest管理应用程序的依赖文件
date: 2020-10-10 22:47:20
categories:
- 编程匠艺
tags:
- 系统设计
- Windows
- manifest
---

我们在开发应用程序时，一般会引入一些第三方库，通常情况下，我们是把这些第三方依赖文件放到应用程序所处目录，这样应用程序启动时就能正确找到相关依赖文件。但当依赖文件比较多，我们希望对依赖的文件进行归类，放置到不同的目录下以便管理，这个时候应用程序的manifest就派上用场了。

## 并行程序集
在介绍应用程序的manifest之前，需要了解一下并行程序集(Side-by-Side Assembly)。什么是并行程序集呢? 并行程序集是微软为了解决[DLL Hell](https://en.wikipedia.org/wiki/DLL_Hell)问题而提出的一种解决方案，它采用manifest文件扫描组件之间的依赖关系。其工作原理如下图所求：

![](/images/manifest-manage/1.png)

简单说明一下，微软在未提出Side-by-Side Assembly之前，应用程序启动时按照[一定的规则](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682586(v=vs.85).aspx)加载DLL。通常情况下，应用程序会采用动态链接方式共享一些操作系统提供的基础库文件，当Windows更新共享库且共享库不能向后兼容时(*DLL自身并不能向后兼容，这种情况通常发生在DLL的内存布局发生了改变*）,那些依赖于老版本共享库的应用程序就不能正常工作了。为了解决这个问题，微软重写了DLL动态加载子系统，提出了并行程序集的解决方案，即允许多个版本的库共同存在，应用程序通过manifest描述自身所依赖的文件，SxS Manager再通过manifest按照一定的规则找到应用程序的依赖文件，使应用程序正确工作。

<!--more-->

## 程序集查找顺序
和DLL加载顺序类似，SxS Manager在查找应用程序的依赖程序集时也按照一定的规则进行查找。一般情况下，其查找规则如下，如果应用程序需要多语言支持，请参考[这里](https://msdn.microsoft.com/en-us/library/windows/desktop/aa374224(v=vs.85).aspx)。

```
1. Side-by-side searches the WinSxS folder.
2. \\<appdir>\<assemblyname>.DLL
3. \\<appdir>\<assemblyname>.manifest
4. \\<appdir>\<assemblyname>\<assemblyname>.DLL
5. \\<appdir>\<assemblyname>\<assemblyname>.manifest
```

SxS Manager首先查找[共享程序集](https://msdn.microsoft.com/en-us/library/windows/desktop/aa375996(v=vs.85).aspx), 共享程序集通常在Windows下的WinSxS目录下。如果未找到则会在应用程序所处目录按照上面顺序查找相应的程序集名。

## 应用程序manifest
上面已经讲并行程序集的工作原理和并行程序集的查找顺序，接下来说一下**如何用manifest描述应用程序的依赖程序集**以及**如何将manifest嵌入到应用程序中**。
关于应用程序的manifest详细介绍请参考[这里](https://msdn.microsoft.com/en-us/library/windows/desktop/aa374191(v=vs.85).aspx), 下面以应用程序SampleApp为例，其manifest为SampleApp.exe.manifest, 如下：

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <assemblyIdentity type="win32" name="SampleApp" version="1.0.0.0" processorArchitecture="x86"/>
  <compatibility xmlns="urn:schemas-microsoft-com:compatibility.v1"> 
      <application> 
          <supportedOS Id="{e2011457-1546-43c5-a5fe-008deee3d3f0}"/> 
          <supportedOS Id="{35138b9a-5d96-4fbd-8e2d-a2440225f93a}"/>
          <supportedOS Id="{4a2f28e3-53b9-4441-ba9c-d69d4a4a6e38}"/>
          <supportedOS Id="{1f676c76-80e1-4239-95bb-83d0f6d0da78}"/>
          <supportedOS Id="{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}"/> 
      </application> 
  </compatibility>
  <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
    <security>
      <requestedPrivileges>
        <requestedExecutionLevel level='asInvoker' uiAccess='false' />
      </requestedPrivileges>
    </security>
  </trustInfo>
  <dependency>
    <dependentAssembly>
      <assemblyIdentity type="win32" name="SampleAssembly" version="1.0.0.0" processorArchitecture="x86"/>
    </dependentAssembly>
  </dependency>
</assembly>
```

这里重点关注dependency结点，该结点描述了应用程序SampleApp依赖于程序集SampleAssembly。更多关于Assembly Manifest的XML描述请参考[这里](https://msdn.microsoft.com/en-us/library/windows/desktop/aa374219(v=vs.85).aspx)。

有了上面的manifest，应用程序启动时，为了让SxS Manager能够正确查找到依赖文件，还需要将SampleApp.exe.manifest嵌入到应用程序中，有两种方式可以嵌入：
**方法一**：
如果采用Visual Studio构建应用程序，默认情况下，VS会将manifest嵌入到应用程序中。按下面方式设置应用程序的依赖程序集：
```
Properties > Configuration Properties > Linker > Manifest File > Additional Manifest Dependencies > 
`type='win32' name='SampleAssembly' version='1.0.0.0' processorArchitecture='x86'`
```

也可以采用`comment prama`方式：
```
`#pragma comment(linker, "\"/manifestdependency:type='win32' name='SampleAssembly' version='1.0.0.0' processorArchitecture='x86' \"") `
```

设置好之后，重新编译、链接，生成SampleApp.exe。如果想知道上面依赖信息是否已经成功嵌入到SampleApp.exe中，也有两种方式查看:
**1**. 如果使用Visual Studio, 可以直接在Visual Studio中打开应用程序，如下:

![](/images/manifest-manage/2.png)

**2**. 也可以使用SDK提供的工具mt.exe, 命令如下：
```
mt.exe -inputresource:SampleApp.exe;#1 -out:SampleApp.exe.manifest
```

注意这个地方的1是manifest的资源序号，默认情况下是1。关于mt.exe的更多命令可以执行`mt.exe /?`或参考[这里](https://msdn.microsoft.com/en-us/library/windows/desktop/aa375649(v=vs.85).aspx)。

**方法二**：
使用mt.exe工具嵌入，如果采用VS构建应用程序，使用这种方式需要先关闭VS默认嵌入manifest的行为：
```
Properties > Configuration Properties > Linker > Manifest File > Generate Manifest > No (/MANIFEST:NO)
```

然后在`Properties > Configuration Properties > Build Events > Post Build Event`中写入以下命令：
```
mt.exe -manifest SampleApp.exe.manifest -outputresource:$(OutDir)$(TargetName)$(TargetExt);#1
```

上面命令的前提是文件SampleApp.exe.manifest在你的工程目录下。重新编译，链接。想知道manifest是否成功写入SampleApp.exe中，同样可以采用上面说的两种方法进行验证。

## 实例应用
再次回到我们面临问题，仍然以应用程序SampleApp.exe为例，假设SampleApp依赖libA.dll，而libA.dll又依赖于libB.dll。我们希望奖libA.dll和libB.dll放到目录SampleAssembly下，目录结构如下：

```
/AppDir
    |---SampleApp.exe
    |        |
    |---SampleAssembly
             |---SampleAssembly.manifest
             |---libA.dll
             |---libB.dll 
```

**1**. *按照第4小节所述的方法将Sample.exe.manifest嵌入到Sample.exe中*
**2**. *在SampleApp.exe所处目录下新建一个目录，目录名同依赖程序集名(SampleAssembly)*
**3**. *在SampleAssembly目录下，新建一个manifest文件，文件名同程序集名（SampleAssembly）,其内容如下：*

```
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
	<assemblyIdentity type="win32" name="SampleAssembly" version="1.0.0.0" processorArchitecture="x86"/>
	<file name="libA.dll"/>
	<file name="libB.dll"/>
</assembly>
```

**4**. *将libA.dll, libB.dll放到SampleAssembly目录下*
**5**. *运行SampleApp.exe，看是否能正常工作了呢*

## 参考资料

[1] https://msdn.microsoft.com/en-us/library/windows/desktop/ff951640(v=vs.85).aspx
[2] https://msdn.microsoft.com/en-us/library/windows/desktop/aa374224(v=vs.85).aspx
[3] https://msdn.microsoft.com/en-us/library/windows/desktop/aa374219(v=vs.85).aspx
[4] https://msdn.microsoft.com/en-us/library/windows/desktop/aa374191(v=vs.85).aspx
[5] https://docs.microsoft.com/zh-cn/cpp/build/manifest-generation-in-visual-studio
[6] https://docs.microsoft.com/zh-cn/cpp/build/how-to-embed-a-manifest-inside-a-c-cpp-application
[7] https://docs.microsoft.com/zh-cn/cpp/build/reference/manifestdependency-specify-manifest-dependencies
[8] https://en.wikipedia.org/wiki/DLL_Hell
[9] https://msdn.microsoft.com/en-us/library/windows/desktop/aa375649(v=vs.85).aspx
