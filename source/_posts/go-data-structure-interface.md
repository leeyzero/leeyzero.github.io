---
title: Go数据结构之interface
date: 2023-01-20 15:59:00
categories:
- 编程语言
tags:
- Go
- interface
---

今年最后一天上班，下午没啥事，总结下Go语言的接口。接口是Go语言区别与其它语言的重要语言特性，也是我最喜欢的Go语言特性之一。Go的接口除了提供多态的能力外，它还是Go反射的基础。Go是一门静态语言，Go接口能够在编译期检查语法规则，同时具备像动态语言那样使用 [ducking typing](http://en.wikipedia.org/wiki/Duck_typing) 实现接口。

我一直比较好奇Go的接口内部是如何实现。看了几篇文章，也看了runtime包中的部分实现。发现大部分文章都是从汇编的角度去分析，对我来说，我对汇编了解的不多，也不需要深入那么细节，我最终的目的是想通过了解Go接口的内部实现，让自己避免一些语言上的坑，同时能够写出更高效的代码。最终发现还是 [Russ Cox](https://swtch.com/~rsc/) 的文章 [Go Data Structures: Interfaces](https://research.swtch.com/interfaces) 在一个相对高的层次介绍了Go的接口特性，但又没有深入汇编细节，比较容易理解，也达到了我的目的。

本文通过示例介绍接口的基本用法，然后引出接口的内部数据结构，再介绍接口的动态类型转换和接口派发原理以及效率，最后介绍一些需要注意的问题。

## 接口初识

**接口是对实现的抽象**。在Go语言中，接口是一组方法签名，使用interface关键字表示，Go中接口具有如下特点：

- **接口实现是隐式的。**
- **interface{}是一种静态类型。**
- **接口类型可以动态检查和转换**。

<!--more-->

### 接口实现是隐式的

接口的实现是隐式的是指，接口的实现跟接口的声明是解耦的，只要实现了接口声明的所有方法，那么就说明实现了这个接口，这是相对于C++、JAVA等静态语言需要显示声明实现而言的。

例如，在Golang中定义了一个接口`Stringer`：

```golang
type Stringer interface {
    String() string
}
```

具体实现不用显示说明，只需要实现接口`Stringer`中的所有方法（本例中只有一个方法String）即可：

```golang
type Binary uint64

func (b Binary) String() string {
    return strconv.FormatUint(b.Get(), 2)
}

func (b Binary) Get() uint64 {
    return uint64(b)
}
```

上例中，类型`Binary`中有一个String方法，和接口Stringer中的方法签名一致，那么`Binary`就实现了Stringer接口。我们可以使用接口Stringer接收Binary：

```golang
var s Stringer = Binary(200)
fmt.Println(s.String())
```

该特性让Go语言可以像Python等动态语言一样实现"ducking type"，但比较Python等动态语言更好的是，Go可以编译期而不是运行时检查出接口不匹配的情况，看下面例子：

```golang
var i int = 100
var s Stringer = i
fmt.Println(s.String())
```

上面例子会编译失败，编译器会提示i并没有实现Stringer接口，这让我们能在编译期检查出问题，减少运行时错误，这是Go相对于Python等运行语言的优势。

### interface{}是一种静态类型

这也是Go中容易引起误解的一个知识点。类型interface{}类型看起来比较怪，其实它比较直观的表示了这个一个空接口，它并不是任意类型，但它可以包含任意类型。可以理解为每个具体类型都是interface{}类型的实现，所以所有类型都可以赋值给interface{}，当我们将其它类型的值赋值给interface{}时，发生了隐式转换。一个容易出问题的例子是：

```golang
func IsNil(any interface{}) bool {
    return any == nil
}

func main() {
    var v *int
    fmt.Println(IsNil(v)) // false
}
```

**interface{}是一个静态类型，它的值包含两个部分：类型type和值value**。在这个例子中，当将v赋值给any时，interface{}的值变了<*int, nil>，它并不为空，只有type和value均为空时，interface{}的值才为nil。

### 接口类型动态检查和转换

上面提到了，具体类型在赋值给接口时，Go在编译期可以检查出具体类型是否实现了这个接口。除此之外，Go语言中还可以对接口进行动态检查和类型转换。有三种方式：

- [type assertion](https://go.dev/doc/effective_go#interface_conversions)
- "comma, ok" 惯用法
- [type switch](https://go.dev/doc/effective_go#type_switch)

"type assertion" 和 "comma, ok" 的主要区别是，"type assertion" 如果发生类型转换失败，会运行时panic。而 "comma, ok" 表达式提供对转换结果的校验；"type switch" 表达式类似于C语言中的switch，只是对接口类型的枚举。如下示例：

```golang
func ToString(any interface{}) string {
    if v, ok := any.(Stringer); ok {
        return v.String()
    }

    switch v := any.(type) {
    case int:
        return strconv.Itoa(v)
    case float64:
        return strconv.FormatFloat(v, 'g', -1, 64)
    }
    return "???"
}
```

函数ToString是将任意类型的转换成string。函数内部先使用"comma, ok"表示式动态判断any是否实现了Stringer接口，如果是，则调用Stringer的接口String返回结果；如果不是，使用[type switch](https://research.swtch.com/interfaces)动态枚举any的类型，并将相应类型的值转化为string。

当我们将一个具体类型的值赋值给接口，或者将一个具体类型的值赋值给interface{}时？Go语言内部是怎么处理的呢？
又或者当我们在使用 "comma, ok" 或 "type switch" 表达式进行动态类型转换时？Go语言内部又是如何处理的呢？

带着这个疑问，我们先看下Go语言中接口的内部的数据结构。

## 数据结构

上面我们提到，interface{}类型的值包含两个部分：<type, value>，这是一个粗略的认识。如果具体类型不是接口，比如int, float等，那么type是可以表示的。但如果具体类型实现了某个接口，比如Binary类型实现了Stringer，type是什么呢？

看起来，interface{}内部处理这两种情况不太一样。事实也是如此，Go语言运行时根据接口类型是否包含一组方法将接口类型分成了两类：

- 使用 runtime.eface 结构体表示不包含任何接口方法的 interface{} 类型；
- 使用 runtime.iface 结构体表示包含方法的接口；

先说下空接口类型`eface`，这里说的空接口类型是指没有实现任何接口的类型，比如int, float64等。它的数据结构如下（$GOROOT/src/runtime/runtime2.go)：

```golang
type eface struct {
    _type *_type
    data  unsafe.Pointer
}
```

结构体`eface`包含两个指针，一个指向`_type`类型结构体指针，一个指向具体值的指针`data`。`eface`的结构跟我们初步认识的接口相符，这两个指针分别指向具体的类型和值。`_type`也是一个结构体，在$GOROOT/src/runtime/type.go中定义，这里不打算展开，可以简单理解为，可以通过`_type`指针判断出具体类型。

再来看下接口类型`iface`，如上面示例中的`Binary`类型，它实现了`Stringer`接口，它的数据结构定义如下（$GOROOT/src/runtime/runtime2.go):

```golang
type iface struct {
    tab  *itab
    data unsafe.Pointer
}
```

结构体`iface`也包含了两个指针，data跟`eface`相同，指向具体类型的值。和`eface`不同的是`tab`指针，`tab`指针指向`itab`(interface table)结构体。这里需要展开下，因为我们需要了解接口的具体实现赋值给接口时是如何存储的，结构体`itab`的定义（$GOROOT/src/runtime/runtime2.go)：

```golang
type itab struct {
    inter *interfacetype
    _type *_type
    hash  uint32
    _     [4]byte
    fun   [1]uintptr
}
```

**`itab`可以看成是接口类型和具体类型的结合体**。inter是指向interfacetype的指针，表示静态的接口类型，如示例中的Stringer，_type表示接口的具体实现，如示例中的Binary，hash是_type.hash的一个拷贝，用于快速判断某个类型是否相等，fun是指向第一个接口具体实现的函数的入口地址，在上述示例中是Binary的String函数。

需要注意的是，虽然fun类型[1]uintptr数组大小为1，但由于指针大小是固定的，根据指针偏移量，可以找到其它接口的方法入口地址。另一个需要注意的是，fun指向的方法列表是接口的具体实现，但并不是包括具体实现类型自己的方法，在上述示例中，fun只有一个函数String，并不包括Binary的Get方法。

当我们将`Binary(200)`赋值给接口`Stringer`时，

```golang
var s Stringer = Binary(200) 
```

`iface`的结构如下图所示：

<img src="/images/go-data-structure-interface/iface.png" width=480 />

<font color=gray>图片来源：[Go Data Structures: Interfaces](https://research.swtch.com/interfaces)</font>

## 动态类型转换

我们已经知道了接口的内部数据结构，Go语言是支持动态类型转换的，对不包括接口方法的`eface`结构，很容易枚举具体类型。但对于包含接口方法的`iface`结构，在进行动态类型转换时，是如何生成`itable`的呢？Go的动态类型转换意味着`itable`有太多 <interface type, concrete type> 的组合，对编译器或链接器来说，提前计算出它们的组合是不合理也不必要的。

实现上，编译器会为每个具体类型（如Binary，int或func(map[int]string等)生成一个类型描述结构，同时还包含该类型实现的一个方法列表。同样的，编译器也为接口类型（如Striner）生成一个不同的类型描述结构，同时也包含该接口的一个方法列表。接口运行时通过在具体类型的方法列表中查找接口方法的所有方法来生成`itable`，并将方法缓存到`itable`的fun函数指针数组中，这样只需要计算一次。

再回到上面的例子，接口Stringer的方法列表只有一个方法；具体类型Binary有两个方法。通常情况下，一个接口会包含M个方法，一个具体类型，通常会包含N个方法。在具体类型中查找接口类型的所有方法时间复杂度为O(MxN)。但如果将接口类型的方法列表和具体类型的方法列表进行排序，那么只需要O(M+N)的时间即可以完成。事实也是如何，如果看汇编代码，具体类型和接口类型的方法列表是按照字母顺序排序的。

## 接口方法查询效率以及动态派发

Go具体静态类型提示与动态方法查找配合使用，接口调用时，可以将方法的查找缓存至接口中。如下面代码所示：

```golang
var any interface{} // initialized some where
s := any.(Stringer) // dynamic conversion
for i := 0; i < 100; i++ {
    fmt.Println(s.String())
}
```

在这个示例中，itable在第2行将any动态转换成Stringer接口时被计算（也可能是在缓存是查找），第4行派发s.String()时，只需要几次内存提取和一次间接调用指令即可。相对于Python等动态语言在第4行需要每次都进行方法查询而言，Go的方法查询效率是非常高的。

关于动态派发的效率问题，在[Go 语言设计与实现](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/)有详细的基准测试分析，这里直接给出结论：

- 使用结构体实现接口带来的开销会大于使用指针实现，而动态派发在结构上的表现非常差（2倍性能差异），这提醒我们应当尽量避免使用结构体类型实现接口。
- 使用结构体带来的巨大性能差异不只是接口带来的问题，带来性能问题主要因为 Go 语言在函数调用时是传值的，动态派发的过程只是放大了参数拷贝带来的影响。

## 一些需要注意的问题

- interface{}是一种静态类型，它不是任意类型，具体类型赋值给interface{}时，发生了类型转换。
- 动态类型转换存在运行时开销，时间复杂度为O(MxN)，其中M和N分别是接口类型和具体实现类型的方法数。
- 结构体和指针实现接口在动态派发性能表现上有较大较差异，应该尽量避免使用结构体实现接口。

## 参考资料

[1] [Go Data Structures: Interfaces](https://research.swtch.com/interfaces)
[2] [Go 语言设计与实现](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/)
[3] [Go 101: Interface in Go](https://go101.org/article/interface.html)
[4] [深入研究 Go interface 底层实现](https://halfrost.com/go_interface/)
