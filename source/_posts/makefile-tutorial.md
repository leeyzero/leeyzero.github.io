---
title: Makefile简明教程
date: 2020-10-10 23:48:19
categories:
- 编程语言
tags:
- Makefile
- Linux
---

[make](https://en.wikipedia.org/wiki/Make_(software))是一个自动化构建工具，广泛应用于Unix及其类Unix系统中。make最先应用于编译C语言项目，不仅如此，只要某个文件发生变化就需要重新构建的项目都可以使用make工具进行构建。

Makefile是可以被make解析的特定格式的文本文件。其语法简单，当我们构建一个程序时，其工作原理大致为：make首先解析Makefile，查找构建应用的一系列依赖，并检查每个依赖文件是否过期（是否发生变化），如果发生过期，即重新构建变化的依赖文件。

<!--more-->

## 语法规则

语法规则很简单，Makefile文件是由一系列规则(rule)组成，每个规则语法如下：

```makefile
<target> : <prerequisites> 
[tab]  <commands>
```

对每个规则，target是必须的，prerequisites和commands是可选的，commands前面必须是一个tab。Makefile文件中至少有一个规则。

```shell
shell> cat Makefile
tutorial:
  echo "this is Makefile tutorial"
```

我们可以通过make来执行上面规则:

```shell
shell> make tutorial
echo "this is Makefile tutorial"
this is Makefile tutorial
```

默认情况下，如果我们没有指定目标(target)，make命令会执行第一条规则。上面例子中，target为`tutorial`，前置条件（prerequisites）为空，命令（commands）为`echo "this is Makefile tutorial"`。我们发现上面例子中除了输出结果，还把命令也输出，这叫回显，可以在命令前加上`@`关闭回显，如：`@echo "this is Makefile tutorial"`

### 目标

一个目标即构成一条规则，目标通常是文件名，指明make命令要构建的对象。目标可以是一个文件名，也可以是多个文件名，中间用空格分隔。如：

```shell
a.o b.o: a.h a.cpp a.h b.cpp
  g++ -c a.cpp b.cpp
```

除了文件名，目标还可以是某个操作的名字，如上面例子中的`tutorial`。

### 前置条件

前置条件通常是一组文件名，之间用空格分隔。它指定了"目标"是否重新构建的判断标准：只要有一个前置文件不存在，或者有过更新（前置文件的last-modification时间戳比目标的时间戳新），"目标"就需要重新构建。下面规则是创建一个source.txt文件：

```shell
shell> cat Makefile
source.txt:
  echo "this is source file" >> source.txt

result.txt: source.txt
  cp source.txt result.txt
```

第一次执行`make result.txt`，会创建source.txt，并将source.txt中的内容拷贝到result.txt中，当我们再次执行`make result.txt`时，终端会输出：

```shell
make: 'result.txt' is up to date.
```

这是由于source.txt已经创建，且没有变化，所以不会执行任何操作。如果我们修改一下source.txt，如：

```shell
touch source.txt
```

再执行`make source.txt` 试试看。

### 命令

命令（commands）表示如何更新目标文件，由一行或多行的shell命令组成。它是构建"目标"的具体指令，它的运行结果通常就是生成目标文件。

需要注意的是：

- 每个命令前需要有一个tab；
- 每行命令在一个单独的shell中执行。这些shell之间没有继承关系。

如：

```shell
var-lost:
    export foo=bar
    echo "foo=[$$foo]"
```

上面代码执行后`make var-lost`，取不到foo的值。因为两行命令在两个不同的进程执行。一个解决办法是将两行命令写在一行，中间用分号分隔。

```shell
var-kept:
    export foo=bar; echo "foo=[$$foo]"
```

或在换行符前加反斜杠转义。

```shell
var-kept:
    export foo=bar; \
    echo "foo=[$$foo]"
```

## 变量与注释

变量和注释不是必须的，但当Makefile文件比较大时，变量和注释将会非常有用，毕竟代码首先是给人读的。

### 自定义变量

Makefile中的变量没有类型，全是字符型，定义一个变量也比较简单，如下：

```shell
CFLAGS = -g -Wall
CC = g++
RM = /bin/rm -f
```

当定义好变量后，如果使用变量中存储的值呢？获取变量的值跟shell类似：

```shell
${varname}
```

### 注释

注释只需要在前面加上`#`即可，如：

```shell
#this is a comment
```

### 内置变量

除了自定义的变量外，make命令提供一系列内置变量，比如，`$(CC)` 指向当前使用的编译器，`$(MAKE)` 指向当前使用的make工具。这主要是为了跨平台的兼容性，详细的内置变量清单见[手册](https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html)。

### 自动变量

make命令还提供一些自动变量，它们的值与当前规则有关。主要有以下几个。

- $@

指代当前目标，即make构建时指定的目标，如`make main`中`$@`为main。

```shell
main: main.o
	${CC} ${CFLAGS} -o $@ $^
```

- $<

指代第一个前置条件，如规则`rule: p1 p2`中的`$<`为p1。
```
a.txt: b.txt c.txt
    cp $< $@
```

- $^

指代所有前置条件，中间是空格隔开。如规则`rule: p1 p2`中的`$^`为p1 p2。

```shell
main.o: main.cpp
	${CC} ${CFLAGS} -c $^
```

- $?

指代比目标更新的所有前置条件，之间以空格分隔。比如，规则为 `rule: p1 p2`，其中 p2 的时间戳比 rule新，$?就指代p2。

- $*

指代匹配符 % 匹配的部分， 比如% 匹配 f1.txt 中的f1 ，$* 就表示 f1。

更多自动变更参考[Automatic-Variables](http://www.gnu.org/software/make/manual/make.html#Automatic-Variables).

## 实践

我们以构建一个c++的hello world程序为例：

```shell
shell> cat main.cpp
#include <iostream>

using namespace std;

int main() {
	std::cout << "hello world" << std::endl;
}
```

Makefile内容如下：

```makefile
shell> cat Makefile
CFLAGS = -g -Wall
CC = g++
RM = /bin/rm -f

all: main

main: main.o
	${CC} ${CFLAGS} -o $@ $^

main.o: main.cpp
	${CC} ${CFLAGS} -c $^

clean:
	${RM} *.o main
```

接下来我们就可以利用make进行构建了：

```shell
shell> make
```

没有指定规则时，默认执行第一条规则，在这个例子中为`all`。
当需要清除产生的中间文件和可执行文件执行：

```shell
shell> make clean
```

## 进一步学习
- 在实际项目中，项目文件成千上万个，而且分布在不同目录下，如果编写规则进行构建呢？
- 当我们有很多依赖时，是不是需要我们逐个手写这些规则呢？
- make只能编译c/c++吗？

有了以上一些基础知识，就可以继续学习更多高级用法了。你会发现make会让你的构建过程更高效、自动化。

## 参考资料

[1] [https://www.wooster.edu/_media/files/academics/areas/computer-science/resources/makefile-tut.pdf](https://www.wooster.edu/_media/files/academics/areas/computer-science/resources/makefile-tut.pdf)
[2] [http://www.gnu.org/software/make/manual/make.html#Makefile-Contents](http://www.gnu.org/software/make/manual/make.html#Makefile-Contents)
[3] [https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install#what-does-all-of-this-do](https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install#what-does-all-of-this-do)
[4] [https://gist.github.com/isaacs/62a2d1825d04437c6f08](https://gist.github.com/isaacs/62a2d1825d04437c6f08)
