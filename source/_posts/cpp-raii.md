---
title: C++资源管理
date: 2021-01-28 00:20:11
categories:
- 编程语言
tags:
- C++
- RAII
---

RAII作为C++资源管理的一种惯用法，是每一位C++程序员都应该掌握的基本技能。在google上搜索关键字RAII,  有二百多万条搜索结果。说明这个话题在网上已经被讨论过无数次，也发现了一些好文章给我不少启发，这篇文章主要是做个总结并谈谈自己的理解。

## 什么是RAII

资源获取即初始化（*Resource Acquisition Is Initialization*），即**RAII**。RAII是一种C++编程技术。**它利用栈对象在离开作用域后自动析构的语言特点，将受限资源的生命周期绑定到该对象上，当对象析构时以达到自动释放资源的目的**。 这里说的资源都是指的受限资源，比如堆上分配的内存、文件句柄、线程、数据库连接、网络连接等。

<!--more-->

## 资源管理问题

操作系统的资源是有限的，当我们向操作系统索取资源，使用完后应即时归还给操作操作，这是一个良好的编程习惯，资源获取操作流程如下：

```plaintext
获取资源 > 使用资源 > 释放资源
```

实例1：堆上分配内存

```c++
void eg1(void) {
    int* pBuf = new int[256];
    if (!condition1) {
        delete []pBuf;  // 怪味道，释放资源入口不止一个
        return ;
    }
    if (!condition2) {
        foo(); // 潜藏问题
    }
    delete []pBuf;
}
```

实例2：打开文件

```c++
void eg2(void) {
    FILE* pf = fopen("eg1.txt", "w");
    // 使用资源
    fclose(pf);
}
```

实例3：互斥量

```c++
void eg3(void) {
    static std::mutex m;
    m.lock(); 
    // 使用资源
    m.unlock();    
}
```

上面代码只为演示，未做合法性校验。对于`eg1`（其它例子存在同样的问题），在使用资源过程中可能在某个分支（`condition1`）返回，或调用某个函数(`foo`)产生异常并抛出，导致资源未能正确释放。而且从代码可维护性来看，后续的维护者可能要fuck了。

## 解决方案

看完上面的例子，读者可能会说，这些C++标准库不是提供了吗？

- 内存管理有智能指针（`std::unique_ptr`、`std::shared_ptr`）
- 文件操作有文件流对象（`ifstream`、`ofstream`和`fstream`）
- 互斥管理有（`std::lock_guard`、`std::unique_guard`和`std::shared_guard`）

没错，我们要学会善用工具，明白其本质更有助于我们灵活运用并不拘于这些工具。对于以上实例，当函数返回或异常抛出，当前栈回溯，栈上的对象自动析构。按照这个思路，我们需要写一个资源管理类，并在栈上创建其类对象，类构造的时候获取资源（不是必须），析构的时候释放资源。比如对于`eg3`我们写一个互斥管理类：

```c++
class scoped_lock {
public:
    explicit scoped_lock(std::mutex& m) : _m(m) {
        _m.lock();
    }

    ~scoped_lock(void) {
        _m.unlock();
    }
private:
    std::mutext& _m;
}

void eg3(void) {
    static std::mutex m;
    scoped_lock(m);
    // 使用资源
}
```

上面`scoped_lock`的实现是`std::lock_guard`的一个简化版，但展示了RAII技术最核心部分。按照这个思路，我们可以抽象出利用RAII技术进行资源管理的一般步骤：

1. 创建一个资源包装类
2. 在构造函数中获取资源（可选）
3. 在析构函数中释放资源
4. 在栈上创建资源包装类对象

虽然C++标题库给我们提供了一些资源管理工具，但这些工具有时候仍然不能满足我们的需求，有没有通用的解决方案呢？也许下面一组宏能够帮助你： 

```c++
#define CONCAT_INNER(a,b) a##b
#define CONCAT(a,b) CONCAT_INNER(a,b)

#define ON_LEAVE(statement) \
struct CONCAT(s_ol_, __LINE__) { \
	~CONCAT(s_ol_, __LINE__)() { statement; } \
} CONCAT(v_ol_, __LINE__);

#define ON_LEAVE_1(statement, type, var) \
struct CONCAT(s_ol_, __LINE__) { \
	type var; \
	CONCAT(s_ol_, __LINE__)(type v): var(v) {} \
	~CONCAT(s_ol_, __LINE__)() { statement; } \
} CONCAT(v_ol_, __LINE__)(var);
```

如何使用呢？比如对于`eg2`:

```c++
void eg2(void) {
    FILE* pf = fopen("eg1.txt", "w");
    ON_LEAVE_1(fclose(pf), FILE*, pf);
    // 使用资源
}
```

如果释放资源时需要更多参数，扩展上面宏即可；如果还需要在构造的时候获取资源（*资源包装类构造时获取资源是可选的*）,  当你真正解理了RAII，我想扩展上面宏以满足需求应该不是难事儿！

## 总结

RAII是一种C++编程技术，其原理简单且优雅。C++标准库已经为我们提供了很多有用工具，不必重复造轮子。理解并灵活运用该技术，能避免很多资源泄露的问题，且代码更具健壮性和可维护性。

## 参考资料

[1].https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization
[2].http://www.tomdalling.com/blog/software-design/resource-acquisition-is-initialisation-raii-explained/
[3].http://zh.cppreference.com/w/cpp/language/raii
[4].http://zh.cppreference.com/w/cpp/language/raii#cite_note-1
