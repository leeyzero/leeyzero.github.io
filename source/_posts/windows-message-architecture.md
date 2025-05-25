---
title: '[译] Windows消息体系'
date: 2020-10-10 22:57:21
categories:
- 编程匠艺
tags:
- 消息循环
- Windows
---

原文出自：[Windows Messaging Architecture](https://xavierantony.wordpress.com/2010/08/24/windows-messaging-architecture/)

>*Most of the fresh software engineers and .NET developers may not know what really happens inside the Windows operating system when they implement and run a Windows application. This article explains the Windows Messaging Architecture of the Windows operating system.*

很多新手软件工程师和.NET开发者在编写并运行一个Windows应用时，他们并不清楚其在操作系统中的运作原理。这篇文件将阐述Windows操作系统的消息体系。

![Windows消息体系结构图](/images/windows-message-architecture/1.png)

<!--more-->

## 窗口和窗口类(*Window and Window Class*)

>*The Windows UI application (e) has one main thread (g), one or more > windows (a) and one or more child threads (k) [worker threads or UI threads].*

Windows应用程序(**e**)包含一个主线程(**g**)、多个窗口(**a**)和一个或多个子线程(**k**)[工作线程或UI线程]。

>*An application must specify a window class and register with Windows (d) before it creates a window (a) and displays. A window class is a structure which contains the attributes of a window such as window styles, Icon, Cursor, Background color, menu resource name and window class name etc. Registering a window class associates a window procedure, class styles, and other class attributes with a class name. Each window class has an associated window procedure (c) shared by all windows (a) of the same class in an application. The window procedure processes messages for all windows of that class.*

在创建并展示一个窗口(**a**)之前，应用程序必须指定一个窗口类并将其注册到Windows系统中(**d**)。窗口类只是一个数据结构，包含窗口的一些属性，如窗口风格、Icon、Cursor、窗口背景颜色、菜单资源名和窗口类名等。注册的窗口类需要关联一个窗口过程、类风格、其它类属性和一个窗口类名。在应用程序中，和每个窗口类关联和窗口过程(**c**)是所有拥有相同窗口类名的窗口(**a**)共享的。窗口过程处理所有拥有相同窗口类名窗口的窗口消息。

## 窗口过程(*Window Procedure*)
>*A window procedure (c) is a function that receives and processes all messages sent to the window (a). Every window class (b) has a window procedure (c), and every window (a) created with that class uses that same window procedure to respond to messages. The system sends a message to a window procedure by passing the message data as arguments to the procedure. The window procedure then performs an appropriate action for the message; it checks the message identifier and, while processing the message, uses the information specified by the message parameters.*

窗口过程(**c**)是一个函数，用于接收所有发往该窗口(**a**)的消息。每个窗口类(**b**)都有一个窗口过程(**c**)，每个用该窗口类创建的窗口(**a**)使用相同的窗口过程响应消息。系统给窗口过程发送消息是将消息作为一些函数参数传递给窗口过程，然后窗口过程再对具体的消息做响应；窗口过程会检查消息标识，并通过消息标识使用跟该消息相对应的消息参数。

>*A window procedure does not usually ignore a message. If it does not process a message, it must send the message back to the system for default processing. The window procedure does this by calling the DefWindowProc function (f), which performs a default action and returns a message result. The default window procedure function, DefWindowProc defines certain fundamental behavior shared by all windows. The default window procedure provides the minimal functionality for a window.*

窗口过程通常不忽略消息。如果应用不处理某个消息，应用必须将该消息发送回系统，交由系统处理。窗口过程通过一个叫*DefWindowPrc*函数来处理这些发送回系统的消息，该函数会作默认的响应并返回消息处理结果。*DefWindowProc*定义了一些适用于所有窗口的基本的行为，并提供了一个窗口的最小功能集。

## 窗口消息(*Windows Message*)
>*A Windows message (m) is nothing but a set of parameters such as a window handle, a message identifier and two values called message parameters (WPARAM and LPARAM). The window handle identifies the window for which the message is intended. The message identifier is a constant that identifies the purpose of a message. For example, the message identifier WM_PAINT tells the window procedure that the window’s client area has changed and must be repainted.*

窗口消息(**m**)只是一个参数集合，包括窗口句柄、消息标识和两个被称为消息参数(*WPARAM和LPARAM*)的值。窗口句柄标识一个消息即将发往的窗口。消息标识是一个常量，用于认别一个消息的具体含义。例如，对于消息WM_PAINT用于告诉窗口过程窗口的客户区发生了改变需要重新绘制。

>*There are two types of Windows messages such as System defined messages and Application defined messages. The system sends or posts a system-defined message when it communicates with an application. An application can also send or post system-defined messages. For example, the WM_PAINT message identifier is used in system defined message which requests that a window paint its contents.*

有两种类型的消息，分别是系统定义消息和应用定义消息。当和应用程序通信时，系统通过发送(sends)或投递(posts)一个系统定义消息给应用程序。一个应用程序也可以发送或投递系统消息。例如，WM_PAINT消息是一个系统定义消息，该消息请求一个窗口绘制其窗口内容。

>*An application can create messages to be used by its own windows or to communicate with windows in other processes. If an application creates its own messages, the window procedure that receives them must interpret the messages and provide appropriate processing.*

应用程序可以创建为自身使用或用于和其它进程通信的消息。如果一个应用程序创建一个自定义的消息，窗口过程接收到后必须解析并适当处理该消息。

## 消息队列(*Message Queues*)
>*The Windows operating system uses two methods to route messages to a window procedure : posting messages to a first-in, first-out queue called a message queue (l), a system-defined memory (q) object that temporarily stores messages, and sending messages directly to a window procedure.*

Windows操作系统使用两个方法来发送消息给窗口过程：投递消息到一个先进先出的队列，该队列被称为消息队列(**l**)，系统定义了一块内存对象(**q**)会临时存储直接发往某个窗口过程的消息。

>*The Windows operating system maintains only one System wide message queue (l) for the all applications. And, another message queue (j) is created for each graphical user interface (UI) thread. Whenever a user creates an UI thread, the message queue is also created for the UI thread.*

Windows操作系统维护了一个供所有应用程序服务的系统级别的消息队列(**l**)。而另一个消息队列(**j**)是被UI线程创建的。当用户创建一个UI线程的同时，系统会为该UI线程创建一个消息队列。

## 消息流(*Message Flow*)
>*DOS based applications make C-Runtime function calls to obtain input from the system. For example, it calls gets() C-Runtime function to get input from the keyboard. But, Windows based applications do not make explicit function calls to obtain input. Instead, they wait for the system to pass input to them. This is why Windows-based applications are event-driven.*

基于DOS的应用程序使用C运行时库函数从系统获取输入。例如，应用程序调用C运行时库函数gets()从键盘获取输入。但是，基于窗口的应用程序并不需要显示的函数调用以获取输入，相反，应用程序是等待系统将输入传递给他们。这就是为什么基于窗口的应用是事件驱动的。

>*Whenever the user moves the mouse, clicks [1] the mouse buttons or types on the keyboard, the device driver (p) for the mouse or keyboard handovers [2] the details to User32.dll (n), which in turn converts the input into Windows messages and places[3] them in the system message queue (l).*

用户无论是移动鼠标、点击鼠标按钮[**1**]或敲击键盘，鼠标或键盘设备驱动(**p**)会将这些处理交给[**2**]User32.dll(**n**)，而User32.dll又会将这些输入转化[**3**]成Windows消息并将这些消息存到系统消息队列中(**l**)。

>*The Windows operating system removes the messages, one at a time, from the system message queue, examines them to determine the destination window, and then posts [4] them to the message queue of the thread that created the destination window. A thread’s message queue receives all mouse and keyboard messages for the windows created by the thread. The thread removes [5] messages from its queue and directs the system to send [6 & 7] them to the appropriate window procedure for processing. How does a thread remove a message from thread’s message queue and process? Let’s discuss now.*

操作系统在某一时刻从系统消息队列中移除消息，检查并决定其目标窗口程序，然后将这些消息投递[**4**]给目标窗口程序创建的线程消息队列 。窗口应用程序通过线程创建的线程消息队列接收所有鼠标和键盘消息。该线程从消息队列中移除[**5**]消息，并将其通过系统发送[**6 & 7**]给相应的窗口过程处理。线程如何从线程消息队列中移除一条消息呢？我们马上就会讨论。

## 消息循环或消息泵(*Message Loop or Message Pump*)
>*A Windows application has a WinMain() function which is the entry point for that application, like main()function in ‘C‘ language program. The WinMain()function acts as application’s main thread function. The following three lines of code in the WinMain() function is referred as Message Loop or Message pump.*

就像C语言程序有一个main函数一样，每一个Windows应用都有一个WinMain的入口函数。WinMain函数扮演着应用程序主线程函数角色。下面这三行在WinMain函数中的代码被称作消息循环或消息泵。

```
while (GetMessage(&Msg, NULL, 0, 0)) 
{ 
    TranslateMessage(&Msg); 
    DispatchMessage(&Msg); 
}
```

>*The message loop’s GetMessage() call checks the message queue for windows messages. If it finds a valid message, it removes a message from its queue. After removing a message from its queue, an application can use the DispatchMessage() function to direct the system to send the message to a window procedure for processing. If there are no messages in the message queue, the GetMessage() call will be blocked till a valid message comes to the message queue.*

消息循环中的GetMessage调用会检查消息队列中的windows消息。如果发现一个合法的消息，该函数会从消息队列中移除一条消息。当从消息队列中移除一条消息后，应用程序使用DispatchMessage函数将该消息通过系统发送给一个窗口过程处理。如果消息队列中没有消息，GetMessage调用会被阻塞直到有一个合法的消息进入消息队列中。

>*In between these two calls, the TranslateMessage() method call does nothing, but simply translating virtual key messages into character messages. Your application still works, if you comment this line. Only thing is, your test team will file 50 keyboard related bugs.*

在GetMessage和DispatchMessage中，TranslateMessage函数调用只是简单地做了虚拟键消息和字符消息的转换。如果你注释该行代码，你的应用程序仍然能够工作，只是你的测试小组会给报50键盘事件相关的bug。

## 旅行到此结束(*Journey Ends Here*)
>*The message loop (while loop) breaks when the GetMessage() retrieves a WM_QUIT message from the message queue and subsequently application’s main thread quits. That’s all, the application’s life comes to an end.*

当GetMessage从消息队列中接收到WM_QUIT消息时，消息循环(while循环)会中止，随后应用程序主线程退出。因此，应用程序的整个生命周期就结束了。

## 最后(*Finally*)
>*Enjoy .NET application programming by knowing the messaging internals. This article is an another flavor of MSDN article about [Messages and Message Queues](https://msdn.microsoft.com/en-us/library/windows/desktop/ms632590(v=vs.85).aspx).*

了解Windows消息的内幕更容易享受.NET编程的乐趣。本文是MSDN上一篇关于[消息和消息队列](https://msdn.microsoft.com/en-us/library/windows/desktop/ms632590(v=vs.85).aspx)文章的另一种解说。

>*I wrote and published this article in Aditi’s Mindscape magazine in 2008. Even now, it may be useful to many developers who don’t know the internals of Windows messaging architecture.*

我在2008年写了该文章并将其发表到*Aditi’s Mindscape*杂志上。至今，该文或许会对那些并不清楚Windows消息体系的开发者有帮助。
