---
title: Go数据结构之slice
date: 2022-12-25 18:01:54
categories:
- 编程语言
tags:
- Go
- slice
---

在Go语言中，**slice是一种非常重要的数据结构，用于描述数组存储的连续部分，它本质上是对数组的引用**。slice是建立在数组基础上，是对数组的一种封装，它屏蔽了数组底层细节，但使用起来更方便、灵活，再配合内建函数append和copy使得slice具有更强的表现力。

如果不了解slice的内部实现原理，可能会写出低效的代码，甚至会写出一些让你意想不到的BUG。本文先介绍slice的实现原理，再介绍slice一些经验和需要避免的问题。

在开始之前，先来看下面示例输出的结果是什么呢？如果你知道答案，说明你已经完全明白slice的实现原理，可以跳过原理部分介绍，后面的经验之谈可能会对你有帮助。

```golang
a := []int{1, 2, 3, 4, 5}
s1, s2 := a[:3], a[:3:3]
s1, s2 = append(s1, 100), append(s2, 101)
fmt.Println(a, s1, s2)
```

## 实现原理

由于slice是建立在数组的基础上，需要先了解Go中数组的一些概念，然后介绍slice的内部结构和分片操作，最后介绍内建append和copy函数在slice中的应用。

<!--more-->

### 声明

在Go中，有三种比较常见声明slice的方式：

方式一：声明一个值为nil的slice

```golang
var s1 []int
```

方式二：字面量声明，在声明变量的同时赋值

```golang
s2 := []string{"hello", "world"}
```

方式三：使用内建函数make声明

```golang
s3 := make([]int, 0, 10)
```

上面的第三种方式，在初始化s3，第二个参数表示slice的长度，第三个函数表示slice的容量。我们可以分别用`len`和`cap`内建函数获取slice的长度和容量。其中第3个参数容量是可选的，如果没有指定，则容量等于长度。

### 数组

在一门语言中，数组是一项最基本的数据结构，**它的特点的数据存储在一片连续的存储空间中，可以通过索引直接定位数据，访问效率非常高**。在Go语言中，数组的声明如下：

```golang
var a [4]int
```

上面例子中声明了一个长度为4，类型为int的变量a。需要注意的是，数组的声明除了指定类型外，还需要指定数组的大小。然后，我们就可以通过下标去设置和索引数据了。

数组`[4]int`在内存中的布局如下：

<img src="/images/go-data-structure-slice/array_mem_repr.png" width=400 />

我们可以通过索引下标去访问数据，比如a[0]表示数组中的第一个元素，a[3]表示最后一个元素。如果对C语言比较熟悉的话，我们可以通过&a[0]获取最1个元素的地址，&a[1]获取第二个元素的地址。由于我们知道元素类型，就可以通过类型的偏移量逐个遍历后面的元素了。

有个数组的基础，我们就容易理解slice的内部结构了。

### 内部结构

slice的内部结构包含三个元素，分别是：

- 指向数组元素的指针ptr
- slice的长度len
- slice容量cap

其中**长度是索引的上限**，如a[i]，i的取值`0 <= i < len`；**容量是分片(下面分介绍）的上限**，如a[i:j]，`i <= j <= cap`。slice的内部数据结构表示如下：

<img src="/images/go-data-structure-slice/slice_header.png" width=400 />

那么，如果我们声明一个如下的slice，`s := make([]byte, 5)`，它的内部结构表示如下：

<img src="/images/go-data-structure-slice/slice_mem_repr.png" width=400 />

从slice的内部结构可以看出，我们使用内建函数`len`和`cap`获取slice的长度和容量时，都可以在O(1)时间内完成。最容易引起歧义的就是指向数组元素的指针，这是容易引起问题的根源。**当我们在进行分片操作时，不同的slice会共享同一份数组存储空间，一个slice对底层数组的修改可能会反映到另一个slice中**。现在在回头看看文章一开头的例子，你是否有答案了呢？

slice最强大的地方在于它能可能进行分片（slicing）操作，分片操作可以理解为在一个已有的slice(也可以是数组)基础上快速获取它的一个子集，下面介绍分片操作。

### 分片操作

[Go language specification](https://go.dev/ref/spec#Slice_expressions)中详细介绍了分片操作，常见的用法是：

```golang
a[low:high]
```

其中low表示起始位置，high表示末尾，a[low:high]的结果是一个slice，表示取a中从low开始到high处（不包括high)的连续元素。比如对上面的`s := make([]byte, 5)`，如果进行分片操作：

```golang
s1 := s[2:4]
```

slice s1的内部表示是什么呢？如下图：

<img src="/images/go-data-structure-slice/slice_slicing.png" width=400 />

如果我们取`len(s1)`和`cap(s1)`，我们得到s1的长度为2，容量为3。长度容易理解，进行分片操作时，长度即为`high-low`。但是我们没有指定容量呀，为什么s1的容量为3呢？

这里引出了完全分片表达式，如下：

```golang
a[low : high : max]
```

分片操作还可以带第三个可选参数，表示分片后的容量。如果没有指定，表示原分片的容量大小。对上面例子来讲，等效于`s1 := s[2:4:5]`，所以s1的容量为`max-low`。

### 扩容与复制

Rob Pike在[Arrays, slices (and strings): The mechanics of 'append'](https://go.dev/blog/slices)详细介绍了对slice进行扩展的始末，感兴趣的可以看看这篇文章。

首先在语言层面是不允许直接对slice底层数组进行扩展操作的。但是我们可以重新申请一个更大容量的slice，然后把原slice的数组copy到新的slice中，这便完成了slice的扩容。这便是标准库中函数`append`的实现原理，标准库也提供了copy操作完成对slice的拷贝。简单实现如下：

```golang
t := make([]byte, len(s), (cap(s)+1)*2)
copy(t, s)
s = t
```

上面实现只是一个示例，在Go中`append`的扩容策略如下：

> slice容量小于1024是，是按2倍扩容，大于等于1024时，按1.25倍扩容。

需要注意的是，**由于append操作修改了slice的内部数据结构的值（长度，容易，指针），我们接收它的返回值，才能得到新的slice**。

以上就是slice的全部内容，另外如果如果想对slice进行删除、插入等操作，都可以使用切片，配置append和copy完成，[SliceTricks](https://github.com/golang/go/wiki/SliceTricks)给出了很多示例可以参考。

现在来回答文章开头处的问题。a, s1, s2的输出结果分别为：

```plaintext
a = [1 2 3 100 5]
s1 = [1 2 3 100]
s2 = [1 2 3 101]
```

第3行结束时，a, s1, s2均共享同一个底层数组空间，它们的元素指针均指向数组的第1个元素，但长度和容量不相同，其中：

```plaintext
len(a) = 5, cap(a) = 5
len(s1) = 3, cap(s1) = 5
len(s2) = 3, cap(s2) = 3
```

当对s1进行append操作后，s1还有容量容纳新元素，会在共享的数组空间(索引为3的位置)写入100，此时s1和a在共享数组索引为3的地址都变更为100；当对s2进行append操作时，由于s2已经没有容量容纳新元素，所以会进行扩展操作，会新申请一片连续的数组空间，并将新元素写入索引为3的地址。此时s2已经不再与a和s1共享同一数组空间，所以才会有上面的输出。

## 需要注意的问题

### slice作为函数参数

slice作为函数参数时，是传递原slice的一个副本，如果在函数内部对slice进行append等会修改slice内部值的操作，如果不将操作后的slice返回，调用方是感知不到变化的。这也是为什么append操作需要返回新slice的原因。

### 区分nil slice和空slice

nil slice和空slice是不一样的。主要区别是空slice会为底层数组分配空间，而nil slice不会。

### 频繁扩容

在上面节我们了解到，slice如果没有指定容量，当不断进行append操作时，不频繁地进行扩展操作，会涉及到内容的拷贝，这对性能比较敏感的应用需要注意。如果我们提前知道大概需要多大的存储空间，使用`make`函数提交分配好容量，可以避免不必要的扩容操作。

### 内存优化

Go使用自动垃圾回收机制，当slice指向的数组没有被任何slice引用时，空间才会被回收。如果分片在内存中保存了大量数据，但我们在进行分片操作时，只使共享数组中很少一部分数据，我们需要将这部分数据copy出来，以让垃圾回收器回收空间以释放内存。

## 参考资料

[1] [Go Slices: usage and internals](https://go.dev/blog/slices-intro)
[2] [Arrays, slices (and strings): The mechanics of 'append'](https://go.dev/blog/slices)
[3] [Go Data Structures](https://research.swtch.com/godata)
[4] [SliceTricks](https://github.com/golang/go/wiki/SliceTricks)