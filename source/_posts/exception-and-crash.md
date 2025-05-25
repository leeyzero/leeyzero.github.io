---
title: 应用程序异常处理与崩溃收集
date: 2020-10-10 22:13:02
categories:
- 编程匠艺
tags:
- 系统设计
- Windows
- 异常处理
---

对于Windows应用, 当应用发布出去之后，如果应用程序在用户机器上发生了崩溃，我们需要这样一种手段，即在程序崩溃的瞬间能够“记录”下崩溃时进程、线程、栈、栈上下文等信息，这些信息对我们分析崩溃原因非常有帮助。另外，由于客户端程序一般是运行在用户的机器上，当生成崩溃信息后，还需要将这些信息上报至服务器，以方便开发者能够快捷地拿到这些崩溃信息进行分析。一般来说客户端程序崩溃的收集与处理流程如下图：

![客户端程序崩溃的收集流程](/images/exception-and-crash/1.png)

1. 开发者开发好代码后，由编译集群编译后将产出的二进制等文件放入产品库中，产品库会记录每次发布的版本号(version)、安装程序(*.exe)、对应版本的符号表(.pdb)等信息。
2. 通过各种发布渠道发布最新版本。
3. 用户在使用过程中产生崩溃，生成崩溃时主进程启动一个辅助进程（BugReport.exe）收集崩溃信息。
4. 辅助进程将收集到的崩溃信息上传至服务器。
5. 开发者根据程序版本号(version)和用户标识(用户id)从服务器上拿到用户机器上程序生成的崩溃信息(*.dmp)，再从产品库中提取对应版本的符号表文件，最后利用调试工具(Visual Studio、WinDbg等)分析崩溃原因，将修复后的版本再次按照以上流程发布出去。

<!--more-->

## 异常处理

这里的异常处理不同于C++中的异常处理，是指Windows操作系统提供地一种处理应用程序运行时异常（运行时严重错误）的能力。Windows提供了三个函数供应用程序设置回调，当程序出现运行时异常时，可以让应用进行“最后”处理让程序更优雅地退出。

- [SetUnhandledExceptionFilter](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680634(v=vs.85).aspx)：当出现没有控制的异常时回调
- [_set_invalid_parameter_handler](https://msdn.microsoft.com/en-us/library/a9yf33zb.aspx)：当出现无效参数调用时回调
- [_set_purecall_handler](https://technet.microsoft.com/en-us/library/t296ys27(v=vs.120).aspx)：当出现纯虚函数调用时回调

函数签名为：

```c++
LPTOP_LEVEL_EXCEPTION_FILTER WINAPI SetUnhandledExceptionFilter(
    LPTOP_LEVEL_EXCEPTION_FILTER lpTopLevelExceptionFilter
);

_invalid_parameter_handler _set_invalid_parameter_handler(      
    _invalid_parameter_handler pNew  
);  

_purecall_handler _set_purecall_handler( 
    _purecall_handler function
);
```

比如对于最常见的空指针引用，可以设置一个UnhandledExceptionFilter回调函数，当出现空指针引用时，系统会回调设置的UnhandledExceptionFilter函数，在该函数就可以记录下异常信息。

```c++
LONG WINAPI MyUnhandledExceptionFilterCb(PEXCEPTION_POINTERS pExceptionInfo)
{
    printf("Unhandled exception occurred!\n");
    return EXCEPTION_EXECUTE_HANDLER;
}

int main(void)
{
    ::SetUnhandledExceptionFilter(MyUnhandledExceptionFilterCb);

    *(int*)0 = 100;
    return 0;
}
```

## 崩溃收集

对于那些不可恢复的异常，我们可以在设置的回调函数中记录下相应的异常信息。从Windows XP操作系统开始，微软发布了DbgHelp.dll并提供了API用于创建minidump文件，该文件记录了程序崩溃瞬间进程状态的一个快照，函数签名如下：

```c++
BOOL MiniDumpWriteDump( 
    HANDLE hProcess, 
    DWORD ProcessId, 
    HANDLE hFile, 
    MINIDUMP_TYPE DumpType, 
    PMINIDUMP_EXCEPTION_INFORMATION ExceptionParam, 
    PMINIDUMP_USER_STREAM_INFORMATION UserStreamParam, 
    PMINIDUMP_CALLBACK_INFORMATION CallbackParam
);
```

函数的具体用法可以参考[MSDN](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680360(v=vs.85).aspx), 想了解更多用法可以参考[DebugInfo.com](http://www.debuginfo.com/articles/effminidumps.html#introduction)。按照MSDN的说法：
> MiniDumpWriteDump应该尽可能在一个单独的进程中调用，而不是在被dump的目标进程中，特别是在目标进程已经不稳定的情况下。

所以我们把写dump的操作放到一个单独的进程中处理。如图（崩溃收集与处理流程）的step3中，应用程序发生不可恢复的异常时，为了防止应用程序再次产生崩溃，先将进程中除处理异常的线程外的所有线程都挂起。然后启动一个独立进程BugReport，并将异常信息传给BugReport。再由BugReport调用MiniDumpWriteDump生成崩溃信息，再上传至服务器。
这里涉及的两个进程之间的通信，进程间通信的方式有很多，在这里采用共享内存是一个不错的选择。至于共享内存中要共享的数据结构嘛，就看业务场景了。

## 参考资料

[1] http://www.debuginfo.com/articles/effminidumps.html#introduction
[2] https://msdn.microsoft.com/en-us/library/windows/desktop/ms680360(v=vs.85).aspx
[3] [Windows核心编程（第5版）](https://www.amazon.cn/Windows%E6%A0%B8%E5%BF%83%E7%BC%96%E7%A8%8B-%E6%9D%B0%E5%A4%AB%E7%91%9E/dp/B001GS7918/ref=sr_1_1?ie=UTF8&qid=1498464940&sr=8-1&keywords=windows%E6%A0%B8%E5%BF%83%E7%BC%96%E7%A8%8B)
