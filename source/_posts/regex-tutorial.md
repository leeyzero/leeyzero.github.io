---
title: 正则表达式简明教程
date: 2024-05-26 23:16:20
categories:
- 编程匠艺
tags:
- 正则表达式
- BRE
- ERE
---

> 有些人在碰到问题时，就想：“我知道，我可以使用正则表达式。”现在，他们就有了两个问题。
> -- by Jamie "jwz" Zawinski， 1997年 8月

## 什么是正则表达式

正则表达式（Regular Expression），通常缩写为regex或regexp，是一种在文本中进行搜索和替换的模式描述语言。它使用单个字符串来描述、匹配一系列符合某个句法规则的字符串。正则表达式是由**普通字符**（例如：字母和数字）以及**元字符**组成的。

这篇[slide](https://www.tiki-toki.com/timeline/entry/264740/The-Bloody-History-of-Regex) 描述正则表达式的发展历史。
[regular-expressions](https://www.regular-expressions.info/)是学习正则表达式的一个非常好的网站。

## 初识正则表达式

我们在日常中可能已经使用过正则表达式，比如我们在电脑中查找所有doc文档时，会使用`*.doc`搜索，这个就是正则表达式。其中`*`为元字符，表示可以匹配任意字符。`.doc`是普通字符，表示后续为.doc的文档。

当然这是一个比较简单的例子，正则表达式的功能非常强大，它提供了很多的特性，非常灵活。

在自己的日常开发过程中，有两种场景使用正常表达式比较多：

- 使用Linux命令，如`grep`、`sed`、`awk`中的提供的正则表达式能力查询日志。
- 使用各语言，如`Python`、`PHP`、`Go`中提供的正则表达式库在大文本中查找目标字符串。

熟练掌握正则表达式，在处理字符匹配的问题上可以事半功倍。

<!--more-->

## 元字符

正则表达式的元字符比较多，这里只介绍常用的一些元字符：

### 基础

| 元字符 | 说明 | 示例 |
| ----- | --- | --- |
| . | 匹配除换行符以外的任意字符 | 如`a.c`，可以匹配abc, aac等，中间可以是任意字符 |
| ^ | 匹配字符串的开始 | 如`^abc` 匹配以abc开头的行 |
| $ | 匹配方括号内的区间 | 如`abc$` 匹配以abc结束的行 |
| \w | 匹配字母或数字或下划线或汉字 | |
| \s | 匹配任意的空白符 | |
| \d | 匹配数字, Linux命令不支持 | |
| \b | 匹配单词的开始或结束，该元字符类似于^和$，是一个占位符，标识单词的开始和结束 | 如`\ba\w*\b`，匹配以a开头的单词；`\b\w{3}\b` 匹配只有三个字符的单词 |

元字符 `\w`, `\s`, `\d` 属于缩写元字符，等价于括号表达式中的`[[:word:]]`，`[[:space:]]`，`[[:digit:]]`（下文会提到），它们的取反元字符是`\W`，`\S`和`\D`，等价于方括号表示式中的`[^[:word:]]`，`[^[:space:]]`，`[^[:digit:]]`。

更多关于缩写元字符参考：[Shorthand Character Classes](https://www.regular-expressions.info/shorthand.html)

### 重复

| 元字符 | 说明 | 示例 |
| ----- | --- | --- |
| * | 重复0次或更多次 | 如 ab* 可以匹配a, ab, abb, abbb等 |
| + | 重复一次或更多次 | 如 ab+ 可以匹配 abb, abbb等 |
| ? | 重复0次或一次 | 如 ab? 可以匹配 a, ab|
| {n} | 重复n次 | 如 ab{3} 匹配abbb |
| {n,} | 重复n次或更多次 | 如 ab{2,} 可以匹配 abb, abbb等 |
| {n,m} | 重复n到m次 | 如 ab{2,3} 匹配 abb, abbb |

### 括号表达式

括号表达式是由 `[` 和 `]` 组成的字符列表。它匹配列表中的任何单个字符;如果列表的第一个字符是插入符号`^`，那么它匹配不在列表中的任何字符。

例如，正则表达式[0123456789]匹配任意一位数字。

在括号表达式中，**范围表达式**由用连字符分隔的两个字符组成。它匹配在两个字符之间排序的任何单个字符，包括，使用区域设置的排序序列和字符集。例如，在默认的 `C local` 环境中，[a-d]相当于[abcd]。

最后，在括号表达式中**预定义**了某些命名的字符类，它们的含义如名称，常用的预定义字符类如下：

| 预定义 | 描述 | ASCII表示 |
| ----- | --- | --------- |
| [:alnum:] | 字母数字字符 | [a-zA-Z0-9] |
| [:alpha:] | 字母字符 | [a-zA-Z] |
| [:ascii:] | ASCII 字符 | [\x00-\x7F] |
| [:blank:] | 空格或Tab | [ \t] |
| [:digit:] | 数字 | [0-9] |
| [:lower:] | 小写字母 | [a-z] |
| [:space:] | 所有空白字符，包括换行符 | [ \t\r\n\v\f] |
| [:upper:] | 大写字母 | [A-Z] |
| [:word:] | 单词字符(字母、数字和下划线) | [A-Za-z0-9_] |

需要注意的是，字符类中的方括号是预定义字符类的一部分，比如要匹配数字需要使用`[[:digit:]]`，要匹配非数字，使用`[^[:digit:]]`:

```shell
echo "The fire alarm number is 119" | grep -oE '[[:digit:]]+'
119

echo "The fire alarm number is 119" | grep -oE '[^[:digit:]]+'
The fire alarm number is
```

### 分组以及后向引用

使用`()`进行分组，使用`\n`后向引用，两者配合使用，`\1`对第一个分组引用，`\2`对第二个分组引用...。比如我们要匹配字符串：`###010-10000000###`
中间是区号，被`###`包围，分组通常跟重复的元字符配合使用，比如：

```shell
echo "###010-10000000###" | grep -E '(#{3})[0-9]{3}-[0-9]{8}\1'
```

这里`#{3}`表示`#`出现3次，`(#{3})`表示对`#{3}`分组，`[0-9]{3}`表示数字出现3次，同理`[0-9]{8}`表示数字重复8次，最后`\1`表示对第1个分组，即`(#{3})`的引用。

### 选择

使用`|`进行选择，这是一种或关系，只要满足其中一个就可以匹配，比如正则表示式：`RegEx|regex`表示匹配`RegEx`或`regex`。比如：

```shell
echo "regex is not easy" | grep -E 'RegEx|regex'
echo "RegEx is not easy" | grep -E 'RegEx|regex'
```

### 转义

如果我们要匹配元字符本身，比如`.`, `*`, 就有问题了，我们可以通过转义符`\`对元字符转义，这样元字符就退化成普通字符。比如我们要搜索`leeyzero.com`：

```shell
echo "leeyzero.com is my website" | grep -E 'leeyzero\.com'
```

## 贪婪与懒惰

正则表达式中包含能接受重复的限定符时，通常是匹配尽可能多的字符。以这个表达式为例：`a.*b`，它将会匹配最长的以a开始，以b结束的字符串。如果用它来搜索aabb的话，它会匹配整个字符串aabb。这被称为**贪婪匹配**。

但有时我们希望尽可能少的匹配字符，称为**懒惰匹配**。前面给出的限定符都可以被转化为懒惰匹配，只要在它后面加上一个问号?。这样.*?就意味着匹配任意数量的重复，但是在能使整个匹配成功的前提下使用最少的重复。例如：

`a.*?b`匹配以a开始，以b结束的最短的字符串。如果把它应用于aabb的话，它会匹配aab。

```shell
echo "aabb" | grep -oE "a.*b"
aabb

echo "aabb" | grep -oE "a.*?b"
aab
```

需要注意的是BRE和ERE不支持大部分懒惰匹配，具体参考 [Regex cheatsheet](https://remram44.github.io/regex-cheatsheet/regex.html)。

## 正则表达式引擎

使用正则表达式除了其自身的复杂性外，还有一个问题是有不止一种类型的正则表达式。Linux中的不同应用程序可能会用不同类型正则引擎。常用的正则表达式引擎包括：

- BRE：Basic Regular Expression，BRE，POSIX基础正则表达式引擎
- ERE：Extended Regular Expression，ERE，POSIX扩展正则表达式引擎
- PRE：Perl Regular Expression，PRE，Perl正则表达式引擎

在Linux中用的比较多的是BRE和ERE，它们的主要区别是：

> In basic regular expressions the metacharacters ?, +, {, |, (, and ) lose their special meaning; instead use
the back-slashed versions \?, \+, \{, \|, \(, and \).

在BRE中，元字符`?, +, {, |, (, )` 失去特殊含义，需要使用`\`达到表示元字符的目的。比如`grep`命令默认使用BRE，在使用选择元字符`|`时，需要使用`\`转义。

```shell
echo "regex is not easy" | grep 'RegEx\|regex'
```

使用`-E`可以开启ERE，下面例子跟上面等价：

```shell
echo "regex is not easy" | grep -E 'RegEx|regex'
```

除此之外，[Regex cheatsheet](https://remram44.github.io/regex-cheatsheet/regex.html)中详细对比了PRE，BRE，ERE正则表达引擎处理上的差异。

由于grep、sed和awk在Linux中使用比较多，这里单独说明一下：

| 命令 | 正则引擎 | 说明 |
| --- | ------ | ---- |
| grep | BRE, egrep for ERE, grep -P for PCRE (optional) | - |
| sed | BRE, -E switches to ERE | - |
| awk | ERE | might depend on the implementation |

## 常用正则表达式

[github common-regex](https://github.com/cdoco/common-regex) 收集常用正则表达式，在此不再赘述。

## 实践

在后端服务定位线上问题经常需要匹配日志以辅助定位问题，在此列举两个常用的操作，以下面日志格式为例（日志内容做了脱敏）

```plaintext
...
01/Jun/20 18:19:44 logid=2204907616 callid=0 log_time=1591006784.536 product={module} subsys= module={module} idc={idc} error_code=0 status=2 cost_time=77 uri={uri} host={host} client_ip=172.22.158.95 clienttype=71 user_id={user_id} interact=[]
01/Jun/20 18:19:49 logid=2204506820 callid=0 log_time=1591006780.526 product={module} subsys= module={module} idc={idc} error_code=0 status=2 cost_time=9061 uri={uri} host={host} client_ip=172.22.158.95 clienttype=71 user_id={user_id} interact=[]
...
```

### 找出PV TOP10 接口

先用grep查找出uri，然后使用sort排序，用uniq命令统计出现次数，最后再使用sort按出现次数排序，再使用head提取出前10个接口：

```shell
grep -oE 'uri=[\/a-zA-Z0-9]+' xxx.log | awk -F= '{print $2}' | sort | uniq -c | sort -nr | head -n 10
```

这里使用下则表达式 `[\/a-zA-Z0-9_]+` 匹配日志中的uri。uri中包含的字符可能包括数字、字母、下划线和/，我们使用括号表达式，需要注意的是需要对`/`进行转义。

### 找出耗时最长的 TOP10 接口

使用sed先搜索出uri和对应的耗时，然后使用sort对耗时进行排序，最后使用head提取出前10个接口：

```shell
sed -nr 's/.*cost_time=([0-9]+) uri=([\/0-9a-zA-Z_]+).*/\2 \1/p' xxx.log | sort -k2 -nr | head -n 20
```

这里使用了分组和后向引用，分组1 `[0-9]+`匹配接口耗时，分组2 `[\/0-9a-zA-Z_]+` 匹配接口名。

## 参考资料

- [regular-expressions quickstart](https://www.regular-expressions.info/quickstart.html)
- [regular-expressions posix](https://www.regular-expressions.info/posix.html)
- [regular-expressions posixbrackets](https://www.regular-expressions.info/posixbrackets.html)
- [Regex cheatsheet](https://remram44.github.io/regex-cheatsheet/regex.html)
- [正则表达式30分钟入门教程](https://deerchao.cn/tutorials/regex/regex.htm)
- [common-regex](https://github.com/cdoco/common-regex)
