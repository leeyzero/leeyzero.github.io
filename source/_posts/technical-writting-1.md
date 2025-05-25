---
title: '[译] Google技术写作（一）'
date: 2022-05-31 09:49:26
categories:
- 成长之路
tags:
- 读书笔记
- 技术写作
---

本文翻译自 [Google Technical Writing Courses](https://developers.google.com/tech-writing/overview)，是Google推出的技术写作课程，该课程包含两个部分，本文是第一部分。

- {% post_link technical-writting-1 技术写作（一） %}：教你如何写出更清晰的技术文档，是学习第二部分的基础。
- {% post_link technical-writting-2 技术写作（二） %}：帮助你提升技术沟通技巧。

<!--more-->

**翻译仅供学习使用。由于译者水平有限，本文不免存在遗漏或错误之处。如有疑问，请查阅原文**。

以下是译文。

## 简介
### 目标受众

你需要有一点英语写作能力，但并不需要成为一个很强的写作者后再来学习这门课程。
如果你没有接受过任何技术写作训练，本课程非常适合你。如果你已经接受一些技术写作的训练，学习本课程仍将是一种有效的复习。

### 学习目标

这门课程教你技术写作的基础。完成本课程后，你将知道如何做以下事情：

- 始终使用术语，包括缩写和首字母缩写。
- 识别模棱两可的代码。
- 区分主动语态和被动语态。
- 将被动语态转成主动语态。
- 认识主动语态优于被动语态的三种方法。
- 培养至少三种让句子更清晰、更吸引人的策略。
- 培养至少四种让句子更简洁的策略。
- 理解无序列表和有序列表的区别。
- 创建有用的列表。
- 为段落建立有效的引导名。
- 每个段落都聚焦到一个主题上。
- 在文档的开头陈述文档的主要观点。
- 确定你的目标受众。
- 确定你的目标受众已经知道什么以及需要学习什么。
- 了解知识的诅咒。
- 识别和修改习语。
- 说明文档的范围（目标）和读者。
- 将较长的主题分解成适当部分。
- 正确使用逗号、圆括号、冒号、破折号和分号。
- 培养Markdown的基础技能。

成为一个杰出的工程师或技术写作者需要多年的专注练习。本课程将会提升你的技术写作技能，但不能立即把你变为一个杰出的技术写作者。

## 语法基础（可选）
本单元介绍刚刚好的语法基础知识，以便能更好的理解后面的课程。如果你已经具备这些语法知识，可以直接跳到[单词](#单词)继续阅读。
为了简单起见，本单元采取了一些捷径，实际的语法主题比本单元建议的复杂得多。
语法学家在词性的数量和和类型上并不完全一致。下表重点介绍与本课程相关的词性。

| 词性 | 定义 | 例子 |
| --- | --- | --- |
| Noun<br>名词 | 人物、地点、概念或事物 | **Sam** runs **races**.<br>**山姆**参加**比赛**。 | 
| Pronoun<br>代词 | 替代另一个名词的名词 | Sam runs races. **He** likes to compete.<br>山姆参加比赛，**他**喜欢竞争。 | 
| Adjective<br>形容词 | 修饰名词的单词或短语 | Sam wears **blue** shoes.<br>山姆穿**蓝色**的鞋子。|
| Verb<br>动词 | 一个动词或短语 | Sam **runs** races.<br>山姆**参加**比赛。 | 
| Adverb<br>副词 | 修饰动词、形容词或另一个副词的单词或短语 | Sam runs **slowly**.<br>山姆跑地**慢**。|
| Preposition<br>介词 | 指定两个名词位置关系的单词或短语 | Sam's sneakers are seldom **on** his shelf.<br>山姆的运动鞋很少**在**他的鞋架上。|
| Conjunction<br>连词 | 连接两个名词或短语的单词 | Sam's trophies **and** ribbons live only in his imagination.<br>山姆的奖杯**和**缎带只存在于他的想象中。|
| Transition<br>过渡词 | 连接两个句子的单词或短语 | Sam runs races weekly. **However**, he finishes races weakly.<br>山姆每周都参加比赛。**然而**，他在比赛中表现不佳。|

### 名词
名词代表人物、地点或事物。**朱迪**、**南极洲**和**锤子**都是名词，无形的概念（如**健壮性**、**完美性**）也是如此。在以下示例段落中，我们将名词格式化为粗体：

> In the **framework**, an **object** must copy any underlying **values** that the **object** wants to change. The **protos** in the **codebase** are huge, so copying the **protos** is unacceptably expensive.

在编程中，你可以将类和变量视为程序中的名词。

### 代词

代词是一个间接层，指向或替代其他名词或句子。例如，考虑以下两个句子：

> Janet writes great code. **She** is a senior staff engineer.

在前面的示例中，第一句将 Janet 确立为名词。第二句用代词 She 代替了名词 Janet。
在以下示例中，代词 This 替换了整个句子：

> Most applications aren't sufficiently tested. **This** is poor engineering.

### 动词

动词是一个动作词或短语。当你想表示两个名词（行动者和目标）之间的关系时，动词就发挥作用了。动词标识行为者对目标的行为。

每个句子必须包含至少一个动词。例如，下面每个句子都包含了一个的动词：

- Sakai **prefers** pasta.
- Rick **likes** the ocean.
- Smurfs **are** blue.
- Jess **suffers** from allergies.

有些句子包含多个动词，如下：
- Nala **suffers** from allergies and **sneezes** constantly.
- The program **runs** slowly but **fails** quickly.

根据时态和词形的变化，一个动词可以包含一个或多个单词。例如：
- Tina **was eating** breakfast a few hours ago.
- Tina **is eating** lunch right now.
- Tina **will eat** dinner tonight at 7:00.

### 形容词和副词

形容词修饰名词。例如在下面的段落中，注意形容词是如何修饰后面的名词的：

- Tom likes **red** balloons. He prepares **delicious** food. He fixed **eight** bugs at work.

大多数副词修饰动词。例如，注意下面句子中的的副词（**efficiently**)是如何修饰动态（**fixes**）的:

- Jane **efficiently** fixes bugs.

副词不一定紧挨着动词。例如，在下面的句子中，副词（**efficiently**）和动态（**fixes**）相隔两个单词：

- Jane **fixes** bugs **efficiently**.

副词也可以修饰形容词和副词。

### 介词
介词指定两个事物之间的关系。一些介词通常回答这个问题，“一个事物相对于另一个事物在哪里？“，例如：

- The submenu lies **under** the menu.
- The definition appears **next to** the term.
- The print function falls **within** the main routine.

一些介词回答这个问题，”一个事件相对于另一个事件是什么时候？“，例如：

- The program evaluates the addition operation before evaluating the subtraction operation.
- The cron daemon executes the script every Tuesday at noon.

另一些介词（如**by**和**of**）回答其它类型的关系，如下面的句子中使用**by**表示一本书与作者的关系：

- The C Programming Language **by** Kernighan and Richie remains popular.

### 连词和过渡词
连词连接句子中的短语或名词；过渡词连接句子本身。

最重要的连词如下：

- and
- but
- or

例如，在下面的句子中，**and**连接"code"和"documentation"，**but**连接句子的前半部分和后半部分。

> Natasha writes great internal code **and** documentation **but** seldom works on open-source projects.

在技术写作中最重要的过渡词如下：

- however
- therefore
- for example

例如，在下面的段落中，注意过渡词是如何连接并使句子上下连贯的：

> Juan is a wonderful coder. **However**, he rarely writes sufficient tests. **For example**, Juan coded a 5,000 line FFT package that contained only a single 10-line unit test.

## 单词

我们对文档进行了大量的研究，结果发现世界上最好的句子主要由单词组成。

### 定义新术语和不熟悉的术语
在写作或编辑时，学会识别那些目标受众不熟悉的术语。当你发现此类术语后，采取如下两种策略之一：
- 如果这个术语已经存在，链接到现有的解释。（不要重复造轮子）
- 如果你在文档中引入了这个术语，请定义该术语。如果你在文档中引入了许多术语，将这些定义收集到词汇表中。

### 始终使用术语

如果你在方法的中途改变变量的名称，你的代码将无法编译。类似地，如果你在文档中间重命名了术语，你的想法也不法编译（在你的脑中）。
修养：在整个文档中始终使用同一个明确的单词或术语。一旦你命名了一个组件为**thingy**，不要把它重命名为**thingamabob**，例如，下面的段落错误地将Protocol Buffers重命名为protobufs:
> **Protocol Buffers** provide their own definition language. Blah, blah, blah. And that's why **protobufs** have won so many county fairs.

是的，技术写作是严格和限制性的，但至少技术写作提供了一个很好的解决方法。即，在引入冗长的概念或产品名称时，你还可以指定该名称的缩写版本。然后，你可以在整个文档中使用该缩写名称。例如，以下段落很好：
> **Protocol Buffers** (or **protobufs** for short) provide their own definition language. Blah, blah, blah. And that's why protobufs have won so many county fairs.

### 正确使用首字母缩写

在文档或章节中首次使用不熟悉的首字母缩写词时，请拼出完整的术语，然后将首字母缩写词放在括号中。将拼写版本和首字母缩写词都以粗体显示。例如：

> This document is for engineers who are new to the **Telekinetic Tactile Network** (**TTN**) or need to understand how to order TTN replacement parts through finger motions.

然后你可以在随后使用首字段缩写，如下例所示：
> If no cache entry exists, the Mixer calls the **OttoGroup Server** (**OGS**) to fetch Ottos for the request. The OGS is a repository that holds all servable Ottos. The OGS is organized in a logical tree structure, with a root node and two levels of leaf nodes. The OGS root forwards the request to the leaves and collects the responses.

不要在同一个文档中来回使用首字母缩写版本和扩展版本。

### 使用首字母缩写还是完整术语？
当然，您可以正确地引入和使用首字母缩略词，但你真的应该使用首字母缩略词吗？好吧，首字母缩略词确实减少了句子的大小。例如，TTN 比 Telekinetic Tactile Network 短两个单词。然而，首字母缩略词实际上只是一个抽象层。读者必须在脑海中将最近学到的首字母缩略词扩展成整个术语。例如，读者在脑海中将 TTN 转换为 Telekinetic Tactile Network，因此“更短”的首字母缩略词实际上比完整术语需要更长的时间来处理。

大量使用的首字母缩略词后逐渐形成了它自己的身份。在多次出现之后，读者通常会停止将首字母缩略词扩展为完整术语。例如，许多 Web 开发人员已经忘记了 HTML 扩展成什么了。

下面是首字母缩写的指南：
- 不要定义只使用很少次数的首字母缩写词。
- 请定义满足以下两种条件的首字母缩写词：
    - 首字母缩写词明显短于完整术语。
    - 首字母缩写词在文档中多次出现。

### 识别有歧义的代词

许多代词指代前面介绍过的名词。这样的代词类似于编程中的指针。就像编程中的指针一样，代词往往会引入错误。不恰当地使用代词会导致读者头脑中出现认知偏差，就和空指针错误一样。在许多情况下，你应该简单地避免使用代词，而是直接重复使用名词。但是，代词的效用有时非常有用（如这句话）。

请考虑以下代词的使用指南：
- 只在介绍名词后使用代词；切勿在介绍名词之前使用代词。
- 将代词放置在尽可能靠近所指名词的位置。一般来说，如果将名词和代词隔开的单词超过五个，请考虑重复名词而不是使用代词。

#### It and they

以下代词在技术文档中经常造成困惑：

- it
- they, them, and their

例如，在下面句子中，**It** 指代 Python 还是 C++ 呢？
> Python is interpreted, while C++ is compiled. It has an almost cult-like following.

另一个例子是，**their** 在下面句子中表示什么呢？
> Be careful when using Frambus or Carambola with HoobyScooby or BoiseFram because a bug in **their** core may cause accidental mass unfriending.

#### This and that

考虑另外两个问题代词：

- this
- that

例如，在以下模棱两可的句子中，**This** 是指用户ID，正在运行的进程，还是两者呢？
> Running the process configures permissions and generates a user ID. **This** lets users authenticate to the app.

为了帮助读者，请避免在不清楚他们指的是什么的情况下使用 **this** 或 **that**。使用以下任一策略来澄清 **this** 和 **that** 的模棱两可的用法：

- 使用合适的名词替换 **this** 或 **that**。
- 在 **this** 或 **that** 后马上使用那个名词。 

## 主动语态

技术写作中的绝大多数句子应该是主动语态。本单元教你如何执行以下操作：

- 区分被动语态和主动语态。
- 将被动语态转换为主动语态，因为主动语态通常更清晰。

### 区分简单语句中的主动语态和被动语态

在主动语态的句子中，主语作用于目标。也就是说，主动语态的句子遵循以下公式：

> 主动语态句 = 主语（actor） + 动词（verb） + 目标（target）

被动语态的句子则反过来，即，被动语态句子通常遵循以下公式：

> 被动语态句 = 目标（target） + 动词（verb）+ 主语（actor）

**主动语态示例**
例如，这是一个简单的主动语态句子：
> The cat sat on the mat.

- 主语（actor）：The cat
- 动词（verb）：sat
- 目标（target）：the mat

**被动语态示例**
相比之下，这是一个简单的被动语态句子：

> The mat was sat on by the cat.

- 目标（target）：The mat
- 被动词（passive verb）：wat sat
- 主语（actor）：the cat

有些被动语态句子会忽略主语。例如：
> The mat wat sat on.

- 主语（actor）： unknown
- 被动词（passive verb）：wat sat
- 主语（actor）：the cat

谁或什么坐在垫子上？一只猫？一只狗？霸王龙？读者只能猜测。技术文档中的好句子应该确定谁在对谁做什么。

### 识别主动动词

主动动词通常具有以下公式：

> 主动动词 = be的形式 + 动词的过去分词

虽然前面的公式看起来另人生畏，但其它非常简单：
- be 在一个被动动词中通常是下列单词
    - is/are
    - was/were
- 过去分词动词通常是普通动词加上后缀 ed。例如，以下是过去分词动词：
    - interpret**ed**
    - generat**ed**
    - form**ed**

不幸的是，一些过去分词动词是不规则的；也就是说，过去分词形式不以后缀ed结尾。例如：
- sat
- known
- frozen

将 be 的形式和过去分词放在一起产生被动动词，例如：
- was interpreted
- is generated
- was formed
- is frozen

如果一个短语中包含一个主语，介语通常会跟在被动动词之后。（这个介词通常是帮助你识别被动动词的关键线索。）下面例子结合了被动动词和介词：
- was interpreted as
- is generated by
- was formed by
- is frozen by

**祈使动词通常是主动的**
以祈使动词开头的语句很容易被错误地分类为被动。一个 **祈使动词** 是一个命令。编号列表中的许多项目都以祈使动词开头。例如，下面列表中的 *Open* 和 *Set* 都是祈使动词：

- Open the configuration file。
- Set the *Frombus* variable to *False*。

以祈使动词开头的语句通常是主动语态，即使它们没有明确提及主语。相反，以祈使动词开头的语句隐含了主语。这个隐含的主语就是你。

### 在更复杂的语句中区分主动语态和被动语态

很多语句中包含多个动词，其中有些是主动的，有些是被动的。例如，下面的语句中包含两个动词，两个动词都是主动语态：

![](/images/technical-writting/passive-passive.svg)

下面对同一个语句，将部分被换成主动语态：

![](/images/technical-writting/active-passive.svg)

下面对同一个语句，将全部转换成了主动语态：

![](/images/technical-writting/all-active.svg)

### 优先使用主动语态

大部分情况下使用主动语态。尽量少用被动语态。主动语态具体以下优点：

- 大多数读者在心理上将被动语态转换成主动语态。为什么要让你的读者花费额外的处理时间呢？通过坚持使用主动语态，读者可以跳过预处理阶段，直接进入编译阶段。
- 被动语态会混淆你的想法，颠倒语句。被动语态间接报告动作。
- 有些被动语态的语句完全忽略主语，这会迫使读者去猜测主语是谁。
- 主动语态通常比被动语态更短。

## 清晰的句子

喜剧作家追求最有趣的结果，恐怖作家追求最恐怖的结果，技术作家追求最清晰的结果。在技​​术写作中，清晰度优先于所有其他规则。本单元提供几种使句子清晰的方法。

### 选择强动词

许多技术写作者认为动词是语句中最重要的部分。选择恰当的动词，语句的其余部分都将顺理成章。不幸的是，一些写作者只重复使用了少量温和的动词，这就像每天为你的客人提供不新鲜的饼干和变软的生菜一样。选择正确的动词会花费更多时间，但会产生更令人满意的效果。

为了吸引和教育读者，请选择准确、有力、具体的动词。减少不准确、软弱或通用的动词，例如：

- forms of be: is, are, am, was, were, etc.
- occur
- happen

例如，考虑在以下句子中通过加强弱动词如何点燃一个更吸引人的句子：

| 弱动词 | 强动词 |
| ----- | ----- |
| The exception **occurs** when dividing by zero. | Dividing by zero **raises** the exception. |
| This error message **happens** when... | The system **generates** this error message when... |
| We **are** very careful to ensure... | We carefully **ensure**... |

许多作家依赖于be的形式，好像他们是货架上唯一的香料。撒上不同的动词，让你的散文变得更加开胃。也就是说，be 的一种形式有时是动词的最佳选择，所以不必觉得你必须从你的写作中消除所有每种形式的 be。

注意，一般动词会有病变的信号，例如：

- 语句中不精确的主语或缺乏主语。
- 被动语态的语句。

### 减少使用 there is/there are

以 **There is** 或 **There are** 开头的句子将普通名词和普通动词嫁接到一起。这种乱点鸳鸯谱的方式会使读者感到厌烦。通过提供真实的主语和真实的动词来表达对读者的真爱。

在最好的情况下，你可以简单地删除 **There is** 或 **There are**（可能在句子后面再删除一两个词）。例如，考虑以下句子：

> There is a variable called met_trick that stores the current accuracy.

删除 There is 将通用主题替换为更好的主题。例如，以下任何一个句子都比原文更清晰：

> A variable named met_trick stores the current accuracy.
> The met_trick variable stores the current accuracy.

有时你可以通过将真正的主语和真正的动词从句子的末尾移动到开头来修复 **There is** 或 **There are** 句子。例如，请注意代词 **you** 出现在以下句子的末尾：

> There are two disturbing facts about Perl you should know.

使用 **You** 替换 **There are** 以强化语句：

> You should know two disturbing facts about Perl.

在其它情况下，写作者以 **There is** 或 **There are** 开头语句，以避免创建真实主语或动词的麻烦。如果主体不存在，请考虑创建一个。例如，下面 **There is**语句不能识别接收实体：

> There is no guarantee that the updates will be received in sequential order.

用一个有意义的主体替换 "There is"可以为读者带来一个更清晰的阅读体验：

> Clients might not receive the updates in sequential order.

### 尽量减少特定的形容词和副词（可选）

形容词和副词在小说和诗歌中的表现惊人。多亏了形容词，普通的草能变成杂草（prodigal）和葱绿（verdant），而毫无生气的头发也能变成的有光泽（lustrous）和茂密（exuberant）。副词使马跑的疯狂（madly）和自由（freely），让狗叫地大声（loudly)和凶猛（ferociously）。不幸的是，形容词和副词有时会让技术读者大声而凶猛地吠叫。这是因为形容词和副词对于技术读者来说往往定义过于松散和主观。更糟糕的是，形容词和副词会使技术文档听起来像营销材料一样危险。例如，考虑一篇技术文档中的以下段落：

> Setting this flag makes the application run screamingly fast.

诚然，**快得惊人**的速度会引起读者的注意，但不一定是一种好的方式。为你的技术读者提供事实数据，而不是营销言论。将无形的副词和形容词重构为客观的数字信息。例如：

> Setting this flag makes the application run 225-250% faster.

前面的变化是否会有损这句话的一些魅力？是的，有一点，但是修改后的句子获得了准确性和可信度。

## 简短的句子

出于以下原因，软件工程师通常会尽量减少实现代码的行数：

- 简短的代码通常更易于其它人阅读。
- 简短的代码通常比较长的代码更易于维护。
- 额外的代码可能会引入潜在的故障。

实际上，这些规则同样适用于技术写作：
- 简短的文档可读性更好。
- 简短的文档更易于维护。
- 额外的文档会引入额外的问题。

寻找最短的文档实现需要时间，但最终是值得的。短句比长句能更有效地交流，短句通常比长句更容易理解。

### 一个句子只聚焦到一个想法上

将每一句话聚集到一个想法、思想或概念上。就像程序中的语句执行单一的任务一样，句子也应该执行单个想法。例如，下面很长的句子中包含了多个想法：

> The late 1950s was a key era for programming languages because IBM introduced Fortran in 1957 and John McCarthy introduced Lisp the following year, which gave programmers both an iterative way of solving problems and a recursive way.

将长句子分解成一系列单一的句子会产生以下效果：

> The late 1950s was a key era for programming languages. IBM introduced Fortran in 1957. John McCarthy invented Lisp the following year. Consequently, by the late 1950s, programmers could solve problems iteratively or recursively.

### 将长句子转换为列表

许多冗长的技术语句中，都有一个渴望摆脱困境的清单。例如，考虑下面句子：
> To alter the usual flow of a loop, you may use either a **break** statement (which hops you out of the current loop) or a **continue** statement (which skips past the remainder of the current iteration of the current loop).

当你看到在长句子中存在连词 **or** 时，考虑将该句子重构为一个无序列表。当你看到长句中嵌入了项目或任务列表时，请考虑将该句子重构为无序或有序列表。例如，前面的例子中包含了连词 **or**，因此让我们将长句子转换为以下无序列表：

> To alter the usual flow of a loop, call one of the following statements:
> - **break**, which hops you out of the current loop.
> - **continue**, which skips past the remainder of the current iteration of the current loop.

### 消除或减少不必要的单词

许多句子都包含填充词，即文本垃圾食品，它占用空间，对读者毫无营养。例如，看看是否能在以下句子中找到不必要的单词：

> An input value greater than 100 causes the triggering of logging.

用更短的动词 **triggers** 替换 **causes the triggering of** 将产生更简短的语句：

> An input value greater than 100 triggers logging.

通过练习，你会发现冗余的单词，并享受删除它们的快乐。例如，考虑以下句子：

> This design document provides a detailed description of Project Frambus.

短语 **provides a detailed description of** 可以缩减为 **describes** （或者动词 **details**），这样句子将变成：

> This design document describes Project Frambus.

下列表格列出了一些常见冗长的句子替换：

| 冗长 | 简洁 |
| --- | --- |
| at this point in time | now |
| determine the location of | find |
| is able to | can |

### 减少从句（可选）

一个 **从句** 是一个句子中的独立逻辑片断，其中包含一个主语和动作。每个句子包含以下内容：

- 一个主句
- 零个或多个从句

从句会修改语句的思想。从句的名称意味着，从句没有主句重要。例如，考虑下面句子：

> Python is an interpreted programming language, which was invented in 1991.
> - main clause: Python is an interpreted programming language
> - subordinate clause: which was invented in 1991

你通常可以通过引入从句的单词来识别从句，下面列表（绝不完整）展示了引入从句的常见单词：
- which
- that
- because
- whose
- until
- unless
- since

有些从句以逗号开头，有些则不是。例如，以下句子中突出显示的从句以单词 **because** 开头但不包含逗号：

I prefer to code in C++ **because I like strong data typing**.

编辑时，仔细检查从句。在脑中记住 *一句话 = 一个想法* 的公式。句子中的从句是扩展单个概念还是分支成一个单独的概念？如果是后者，请考虑将有问题的从句分成单独的句子。

### 区分 that 和 which

**That** 和 **which** 都是用来引入了从句的。他们之间有什么区别呢？好吧，在某些国家，这两个词几乎可以互换。不过，警觉的美国读者会愤怒地宣布，你又把这两个词弄混了。

在美国，使用 **which** 从句意味着从句是不必要的部分，并使用 **that** 则表示句子不能没有从句。例如，以下句子中的关键信息是 *Python is an interpreted language*；如果没有 *Guido van Rossum invented*，这句话可以继续存在：

> Python is an interpreted language, **which** Guido van Rossum invented.

相反，下面语句要求一定要有 *don't involve linear algebra*：

> Fortran is perfect for mathematical calculations **that** don't involve linear algebra.


> If you read a sentence aloud and hear a pause just before the subordinate clause, then use which. If you don't hear a pause, use that. Go back and read the preceding two example sentences. Do you hear the pause in the first sentence?
> 如果你大声朗读一个句子并在从句之前听到停顿，请使用 **which**。如果你没有听到停顿，请使用 **that**。返回并阅读前面的两个例句。你听到第一句话的停顿了吗？

> Place a comma before which; do not place a comma before that.
> 在 **which** 之前放置一个逗号；在 **that** 之前不要使用逗号。

## 列表和表格

好的列表可以将混乱转变为有序。技术文档的读者通常喜欢列表。因此，在写作时，尽可能将散文转为列表。

### 选择正确的列表类型

以下列表类型主导技术写作：

- 无序列表（bullets lists）
- 有序列表（numbered lists)
- 嵌入式列表（embedded lists)

未排序的项目使用无序列表；排序的项目使用有序列表。换句话说：
- 如果你改变无序列表中项的顺序，列表的含义不会改变。
- 如果你改变有序列表中项的顺序，列表的含义会改变。

例如，下面我们使用了无序列表，因为改变项的顺序并不改变列表的含义：
> Bash provides the following string manipulation mechanisms:
> - deleting a substring from the start of a string
> reading an entire file into one string variable

相反，下面必须使用有序列表，因为改变列表项的顺序会改变列表的含义：
> Take the following steps to reconfigure the server:
> - Stop the server.
> - Edit the configuration file.
> - Restart the server.

嵌入式列表（有些称为 **run-in** 列表）包含在一个句子中。例如，下面句子包含一个四个项的嵌入式列表.

> The llamacatcher API enables callers to create and query llamas, analyze alpacas, delete vicugnas, and track dromedaries.

通常来讲，嵌入式列表是一种展示技术信息较差的方法。尽量将嵌入式列表转化为有序列表或无序列表。例如，你应该将上面包含嵌入式列表的句子转换为下面语句：

> The llamacatcher API enables callers to do the following:
> - Create and query llamas.
> - Analyze alpacas.
> - Delete vicugnas.
> - Track dromedaries.

### 保持列表项平行
有效列表与无效列表的区别是什么呢？有效列表是平行的；无效列表则相反。平行列表中的项看起来属于同一类。也就是说，平行列表中的所有项都能匹配以下参数：
- 语法（grammer）
- 逻辑类别（logical category）
- 大小写（capitalization）
- 标点符号（punctuation）

相反，非平行列表中，至少有一项违背上述参数。

例如，下面列表是一个平行列表，因为所有的项都是名词（语法），可食用（逻辑类别），小写（大小写），没有句号或分号（标点符号）。
- carrots
- potatoes
- cabbages

相反，下面列表的所有四个参数是不是平行的：
- carrots
- potatoes
- The summer light obscures all memories of winter.

下面列表是一个平行列表，因为所有项都是完整的句子，并且带有完整的句子大小写和标识符号：
- Carrots contain lots of Vitamin A.
- Potatoes taste delicious.
- Cabbages provide oodles of Vitamin K.

列表的第一项建立起了一种模式，读者希望在后续的列表项中看到同样的模式。

### 有序列表项使用祈使动词开头

考虑在所有的有序列表项都使用祈使动词开头。一个**祈使动词**是一个命令，例如 **打开** 或 **开始**。例如，注意下列平行列表中，所有的项都是以祈使动词开头的：

1. Download the Frambus app from Google Play or iTunes.
2. Configure the Frambus app's settings.
3. Start the Frambus app. 

下面有序列表是非平行列表，因为前两个项是以祈使动词开头的句子，但第三项却不是：

1. Instantiate the Froobus class.
2. Invoke the Froobus.Salmonella() method.
3. The process stalls.

### 正确使用标点符号

如果列表项是一个句子，请使用首字母大写和标识符号。否则，不要使用首字句大写和标识符号。例如，下面列表项是一个句子，所以我们将 **Most** 中的 **M** 大写，并在句子末尾添加句号：

- Most carambolas have five ridges.

但是，下面列表项不是一个句子，所以将 **the** 中的 **t** 小写并且忽略句号：

- the color of lemons

### 创建有用表格

分析型思维倾向于喜欢表格。给定一个包含多个段落和一个表格的页面，工程师的眼睛会被吸引到表格上。

考虑下面创建表格的指南：

- 用有意义的标题标记每一列。不要让读者猜测每一列的内容。
- 避免在表格单元格中放入过多文本。如果一个表格单元格包含两个以上的句子，问问自己该信息是否属于其他格式。
- 尽管不同的列可以保存不同类型的数据，但要在各个列中尽量做到并行性。例如，特定表格列中的单元格不应该是数字数据和著名马戏团表演者的混合体。

> 注意：有些表格不能很好的展示所有格式的信息。例如，一个表格在你的笔记本上展示的很好，但在你的手机上看起来却很糟糕。

### 介绍每个列表和表格

我们建议在介绍每个列表和表格时用一个句子来告诉读者列表或表格代表什么。换句话说，给出列表或表格说明一下上下文。用冒号而不是句号结束介绍性句子。

尽管不是必须的，我们仍然推荐在介绍语句中使用 **following** 这个单词。例如，参考下面的介绍语句：

> The following list identifies key performance parameters:
> 
> Take the following steps to install the Frambus package:
>
> The following table summarizes our product's features against our key competitors' features:

## 段落

本单元介绍构建连贯段落的一些指导原则。在开始之前，先来一段鼓舞人心的话：

> The work of writing is simply this: untangling the dependencies among the parts of a topic, and presenting those parts in a logical stream that enables the reader to understand you.
> 
> 写作的流程很简单：理清主题各部分的依赖关系，并以逻辑流的形式呈现各个部分，使读者能够理解你。

### 写一个精彩的开头句

开头句是任何段落中最重要的句子。没有时间的读者只会关注段落的第一个句子，有时会忽略后续的句子。因此，将精力集中在一个句子上。

好的开头句奠定了段落的中心思想。例如，下面段落就是一个好的开头句：

> A loop runs the same block of code multiple times. For example, suppose you wrote a block of code that detected whether an input line ended with a period. To evaluate a million input lines, create a loop that runs a million times.

上面句子中，开头句将段落的主题确定为循环的介绍。相反，下面段落的开头句将读者引向错误的方向：

> A block of code is any set of contiguous code within the same function. For example, suppose you wrote a block of code that detected whether an input line ended with a period. To evaluate a million input lines, create a loop that runs a million times.

### 一个段落聚焦一个主题

一个段落应该呈现一个独立的逻辑单元。将每个段落限定到当前主题上。不要描述将来的主题会发生什么或过去的主题会发生什么。当我们要修改时，一定要毫不犹豫地删掉（或移动到其它段落）和当前主题不相关的句子。

例如，假设下面段落的开头句没有聚焦到正确的主题。你能否找出这些句子，并将其从下面段落中删除？

> The Pythagorean Theorem states that the sum of the squares of both legs of a right triangle is equal to the square of the hypotenuse. The perimeter of a triangle is equal to the sum of the three sides. You can use the Pythagorean Theorem to measure diagonal distances. For example, if you know the length and width of a ping-pong table, you can use the Pythagorean Theorem to determine the diagonal distance. To calculate the perimeter of the ping-pong table, sum the length and the width, and then multiply that sum by 2.

我们删除第二句和第五句话后（译者注：被删除的句子是描述三角形周长的），得出一个集中描述勾股定理的段落：

> The Pythagorean Theorem states that the sum of the squares of both legs of a right triangle is equal to the square of the hypotenuse. ~The perimeter of a triangle is equal to the sum of the three sides~. You can use the Pythagorean Theorem to measure diagonal distances. For example, if you know the length and width of a ping-pong table, you can use the Pythagorean Theorem to determine the diagonal distance. ~To calculate the perimeter of the ping-pong table, sum the length and the width, and then multiply that sum by 2~.

### 段落不要太长或太短

长段落在视角上另人生畏。很长的段落形成了一个可怕的文字墙，读者会忽略掉。读者通常喜欢包含三至五个句子的段落。但是应该避免包含超过七个句子的段落。当进行修改时，考虑将一个很长的段落拆分成两个独立的段落。

相反的，段落也不要太短。如果你的文档包含很多一句话的段落，则说明你的文档组织有问题。应该设法将这些一句话的段落组合成连贯的段落或者组织成列表。

### 回答what，why 以及 how

好的段落回答以下问题：

- **What** - 告诉你的读者这是什么？
- **Why** - 告诉你的读者为什么知道这很重要？
- **How** - 读者应该如何使用这个知识？或者，如何让读者知道你的观点是正确的。

例如，下面的段落回答了 What, Why 和 How:

> <font color=red>\<Start of What\></font>The garp() function returns the delta between a dataset's mean and median. <font color=red>\<End of What\></font> <font color=red>\<Start of Why\></font> Many people believe unquestioningly that a mean always holds the truth. However, a mean is easily influenced by a few very large or very small data points.<font color=red>\<End of Why\></font> <font color=red>\<Start of How\></font>Call garp() to help determine whether a few very large or very small data points are influencing the mean too much. A relatively small garp() value suggests that the mean is more meaningful than when the garp() value is relatively high. <font color=red>\<End of How\></font>

## 受众

本文的作者认为你可能喜欢数学。因此本单元以一个公式开头：

> good documentation = knowledge and skills your audience needs to do a task − your audience's current knowledge and skills
> 
> 好的文档 = 你的受众完成任务所需要的知识和技能 - 你的受众当前具备的知识和技能。

换句话说，确保你的文档能提供你的受众需要但还不具备的信息。因此，本单元说明如何执行以下操作：

- 定义受众。
- 确定受众需要学习什么。
- 让文档适合你的受众。

就如这个[视频](https://youtu.be/eFtXIrmsMwI)所示，定位错误的受众可能会引起厌烦。

### 定义受众

认真的文档工作花费大量的时间和精力来定义受众。这些工作包括用户调查，用户体验研究和聚焦社区和文档测试。你也许没有充足的时间，所以本节采用了一种简单的方法。

首先确定你受众群体的角色。示例角色如下：

- 软件工程师
- 技术人员，非工程师角色（例如技术经理）
- 科学家
- 科技领域的专业人员（例如医生）
- 工科本科生
- 工科研究生
- 非技术岗位人员

我们很欣慰的是，许多非技术人员具有出色的技术和数学技能。但是，角色仍然是定义受众时必不可少的一步。相同角色的受众通常对一些基础技术和知识有共识。例如：
- 大部分软件工程师都知识著名的排序算法，[big O notation](https://en.wikipedia.org/wiki/Big_O_notation)和至少一门编程语言。因此，你可以假定软件工程师都知识 O(n) 的含义，但是你不能假定非技术人员知道 O(n)。
- 对同一个项目研究，一份针面向医生的研究报告和面向非专业受众报纸文章看起来是非常不一样的。
- 大学教授在解释一种新型机器学习方法时，面向研究生和一年级本科生所讲授的方法是完全不一样的。

如果每个人都是相同的角色并拥有相同的知识，那么写作就会变得很容易。不幸的是，不同角色拥有的知识差异是非常大的。Amal 是一位Python专家，Sharon 对C++非常有经验, Micah 擅长 Java. Kara 喜欢 Linux，但David 只熟悉iOS。

角色本身不足以定义受众。也就是说，你必须考虑受众对知识的 **接近程度**。Frombus 项目的软件工程师对相关的 Dingus 项目有所了解，但对无关的 Carambola 项目一无所知。普通的心脏专家比普通的软件工程师对耳朵问题的了解要多，但比听觉专家要少得多。

时间也会造成差距。几乎所有的软件工程师都学过微积分。然而，大多数软件工程师在他们的工作中都不会用到微积分，因此他们对微积分知识逐渐淡忘。相反，经验丰富的工程师比同一项目中新来的工程师更了解当前的项目。

#### 受众样本分析

下面是虚拟 Zylmon 项目的受众样本分析：

Zylmon 项目的目标受众包括以下角色：

- 软件工程师
- 技术产品经理

目标受众在以下方面具有相近的知识：
- 我的目标受众已经了解Zyljeune API，它们与Zylmon API 有些相似。
- 我的目标受众了解C++，但通常不会在新的Winged Victory 开发环境中构建 C++ 程序。
- 我的目标受众在大学里学习过线性代数，但团队中的很多成员需要复习矩阵乘法。

### 确定受众需要学习什么

将你的目标受众完成目标需要学习的所有内容写到一个清单中。在某些情况下，列表中应该包含目标受众需要执行的任务。例如：

> 在阅读文档后，受众将知道如何做以下任务：

> - 使用 Zylmon API 按价格列出酒店。
> - 使用 Zylmon API 按地理位置列出酒店。
> - 使用 Zylmon API 按用户评分列出酒店。

注意，你的受众有时必须按一定顺序完成任务。例如，在学习如何编写特定类型的程序时，你的受众可能需要学习如何在新的开发环境编译和构建程序。

如果你正在写一个设计规范，那么你的列表应该关注你的受众应该学习的信息，而不是特定的任务。例如：

> 阅读设计规范后，读者将会学到下面内容：

> - Zylmon 优于 Zyljeune 的三个原因。
> - Zylmon 花费 5.25 工程年来开发的五个原因。

### 让文档适合你的受众

写出满足你受众需求的文档需要有无私的同理心（unselfish empathy）。你必须合理的解释去满足受众而不是自己的好奇心。为了使文档适合你的受众，你如何迈出这一步呢？不幸的是，我们无法提供简单的答案。但是，我们可以提供一些需要重点关注的参数。

#### 词汇和概念

让你的词汇与受众匹配。请参考 [单词](单词) 以获取帮助。

注意亲近的影响。你团队中的人员可能理解你团队的缩写，但是，其它团队中的人员也理解相同的缩写吗？随着目标受众的扩大，假定你需要进行更多解释。

同样，你软件团队中的成员可能理解团队项目的细节实现和数据结构，但是几乎其它所有人（包括团队中的新成员）都不了解。除非你是专门为团队中其它有经验的成员撰写的，否则你通常需要解释得比你预期的要多。

#### 知识的诅咒

专家经常遭受 **知识的诅咒**，这意味着他们对某个主题的理解会破坏他们对新人的解释。作为专家，很容易忘记新手不知道，但你已经知道的知识。专家在引用细微的交互和深层系统时不会停下来解释，这可能让新手无法理解。

从新手的角度来看，知识的诅咒就像模块尚未编译而导致的“文件找不到”的链接错误。

#### 简单的话

英语已经成为全球技术交流的主要语言。但是，大部分技术读者更喜欢英语以外的其它语言。因此，倾向于简单的单词而不是复杂的单词，避免过时或过于复杂的单词。[多音节](https://www.google.com/search?q=sesquipedalian)和生僻词会使一些读者感到反感。

#### 文化客观和习语 

保持你作品的文化客观。不需要读者了解 NASCAR, 板球或相扑的复杂性来了解软件的工作原理。例如，下面的句子——充满了像苹果派一样的美国棒球隐喻——可能会让巴黎读者感到困扰：

> If Frambus 5.0 was a solid single, Frambus 6.0 is a stand-up double.

**习语** 是一个短语，其整体含义不同于该短语中各个单词的字面含义。例如，下面短语就是习语：

- a piece of cake<sup id="a1">[↩1](#f1)</sup>
- Bob's your uncle<sup id="a2">[↩2](#f2)</sup>

蛋糕？鲍勃？大多数美国读者都认识第一个习语；大多数英国读者认识第二个习语。如果你专门为英语读者而写，那么 *Bob's your uncle* 就可以了。但是如果你要为国际读者写作，那么这个任务完成后替换那个习语。

习语在我们的演讲中根深蒂固，以至于习语的特殊非字面含义对我们来说不可见了。也就是说，习语是知识诅咒的另一种形式。

请注意，受众中有些人使用翻译软件来阅读你的文档。比起简单朴素的英语，翻译软件往往在文化参考和习语方面很无力。

## 文档

你可以写出句子，也可以写出段落。你能将所有的这些段落组织成一份连贯的文档吗？

### 陈述文档范围

好的文档首先要定义它的范围。例如：

> 本文描述了 Frambus 项目的总体设计。

更好的文档还定义了它未覆盖的范围，即目标受众可以希望文档涵盖但文档实际没有覆盖的主题。例如：

> 本文未涉及 Froobus 项目相关的技术设计。

范围和非范围的陈述不仅使读者受益，而且使作者（你）受益。在编写时，如果文档的内容偏离陈述的范围（或非陈述的范围），则必须重新调整文档或修改文档的范围声明。当重新审视你的初稿时，请删除任何不满足范围陈述的部分。

### 陈述你的受众

好的文档明确指定它的受众。例如：

> 本文面向的读者是：
> - 软件工程师
> - 项目经理

除了受众角色之外，好的受众声明还可以指定任何预备知识或经验。例如：

> 本文假设你了解矩阵乘法和反向传播的基础知识。

在某些情况下，受众声明还应该指定先决条件的阅读或课程。例如：

> 在阅读本文之前，你必须先阅读 "Project Froobus: A New Hope"。

### 在开头总结关键观点

工程师和科学家很忙，他们不一定会阅读全部76页的设计文档。想象一下，你的同行可能只阅读文档的第一段。因此，确保你的文档在开头回答读者的基本问题。

专业的写作者将大量精力集中在第一页，以增加读者阅读下一页的机率。但是，任何长文档的第一页都是最难写的部分。因此，准备好多次修改页面。

#### 比较和对比

在你的职业生涯中，无论你多么有创造性，你都将撰写具有真正革命性想法的宝贵文档。你的大部分工作将是渐进式的，建立在再有技术和概念的基础上。因此，请将你的想法和你受众已经理解的概念进行比较和对比。例如：

> This new app is similar to the Frambus app, except with much better graphics.

或者：

> The Froobus API handles the same use cases as the Frambus API, except that the Froobus API is much easier to use.

### 为受众写作

本文反复强调定义你受众的重要性。在本节中，我们将重点放在定义受众作为组织文档的一种方式。

#### 定义受众的需求

回答下面问题有助于你确定文件应该包含的内容：

- 谁是你的目标受众?
- 你目标受众的目的是什么？为什么要阅读这份文档？
- 你的读者在阅读你的文档之前已经知道什么？
- 你的读者在阅读你的文档后应该知道或能够做什么？

例如，假设你发明了一种类似于快速排序的新型排序算法。以下列表包含对上述问题的潜在答案：

- **谁是你的目标受众？** 目标受众是我组织内的所有软件工程师。

- **你的目标受众的目的是什么？** 我的目标受众想要更有效的方法对数据排序。他们阅读这份文档来判断新的算法是否值得实现。

- **你的读者在阅读你的文档之前已经知道什么？** 我的目标受众知道如何编码，并且在之前学习过排序算法，包括快速排序。但是，目标受众中的大部份人已经好几年没有实现或评估过排序算法了。

- **你的读者在阅读你的文档后应该知道或能够做什么？** 我的目标受众在阅读文档后可以做以下事情：
    - 知道该算法和快速排序算法的比较和对比结果。
    - 识别该算法优于快速排序算法的两种数据集。
    - 读者可以用所需的语言来实现算法。
    - 识别该算法的两种性能下降的边界情况。

#### 组织文档以满足受众的需求

在定义的受众的需求后，组织文档以帮忙读者获取他们所需。例如，基于上一节的回答，文档的大纲可能如下所示：

1. 算法概述
    - 和快速排序算法的对比，包括 Big O 复杂度
        - 链接到 Wikipedia 快速排序文章
    - 算法的最佳数据集
2. 算法实现
    - 算法伪代码实现
    - 算法实现建议，包括常见错误
3. 更深入的算法分析
    - 边界情况
    - 已知的未知情况

## 标点（可选）
本单元是可选的，提供了对标点符号的快速复习。

### 逗号
编程语言强制执行有关标点符号的明确规则。相较之下，在英语中，有关逗号的规则有些模糊。作为指导，在读者自然在句子中停顿的地方插入逗号。对于喜欢音乐的人来讲，如果句号是全音符休止符，那么逗号可能是半音符或四分音符休止符。换句话说，逗号停顿的时间要比句号短。例如，如果你大声阅读下面句子，你可能会在单词 *just* 之前短暂停顿：

> C behaves as a mid-level language, just a couple of steps up in abstraction from assembly language.

有些情况下需要逗号。例如，使用逗号分隔嵌入式列表中的项，如下所示：

> Our company uses C++, Python, Java, and JavaScript.

你也许想知道列表的最后的一个逗号，即插入项目N-1和N之间的逗号。这个逗号（称为 **serial comma** 或 **Oxford 逗号**）是有争议的。我们建议提供最后一个逗号，因为技术写作需要选择最少歧义的解决方案。也就是说，我们实际上更喜欢通过将嵌入列表转换为项目符号列表来避免争议。

对于一个表达条件的句子，在条件和结果之间放置逗号。例如，下面两个句子在正确的位置放置了逗号。

> If the program runs slowly, try the --perf flag.
>
> If the program runs slowly, then try the --perf flag.

你还可以在一对逗号之间插入快速定义或题外句，如下示例所示：

> Python, an easy-to-use language, has gained significant momentum in recent years.

最后，避免使用逗号将两个独立的想法粘贴在一起。例如，下面句子中的逗号犯了称为**逗号拼接**的标点重罪：

<font color="red">Samantha is a wonderful coder, she writes abundant tests. </font>

使用句号而不是逗号将两个独立的想法隔开。例如：

<font color="green">Samantha is a wonderful coder. She writes abundant tests.</font>

### 分号

句号分隔不同的思想；分号结合高度相关的思想。例如，下面句子中的分号是如何结合第一和第二部分的想法的：

<font color="green">Rerun Frambus after updating your configuration file; don't rerun Frambus after updating existing source code.</font>

在使用分号之前，问问你自己如何你将分号两侧的想法翻转，该句子是否仍然有意义。例如，反转前面的示例仍然是一个有效的句子：

> Don't rerun Frambus after updating existing source code; rerun Frambus after updating your configuration file.

分号前后的思想必须是语法完整的句子。例如，下面的分号是不正确的，因为分号后面的段落是一个[从句](https://developers.google.com/tech-writing/one/short-sentences#reduce_subordinate_clauses_optional)，并不是一个完整的句子。

<font color="red">Rerun Frambus after updating your configuration file; not after updating existing source code.</font>

推荐的写法是：
<font color="green">Rerun Frambus after updating your configuration file, not after updating existing source code.</font>

你几乎应该始终使用逗号而不是分号去分隔嵌入式列表的项。例如，下面句子中使用分号是错误的：

<font color="red">Style guides are bigger than the moon; more essential than oxygen; and completely inscrutable.</font>

正如前文提到，技术写作优先使用无序列表而不是嵌入式列表。但是，如果你真是喜欢嵌入式列表，使用逗号而不是分号去分隔列表项，如下列所示：

<font color="green">Style guides are bigger than the moon, more essential than oxygen, and completely inscrutable.</font>

很多句子在分号后面立即放置一个过渡词或短语。在这种情况下，在过渡词后放置一个逗号。注意下列两个示例中过渡词后面的逗号：

> Frambus provides no official open source package for string manipulation; however, subsets of string manipulation packages are available from other open source projects.

> Even seemingly trivial code changes can cause bugs; therefore, write abundant unit tests.

### 破折号
破折号是引入注意的标点符号，具有丰富的符号符号可能性。一个破折号代表一个比逗号更长的停顿——一个更大的中断。如果说逗号是四分音符休止符，那么破折号就是二分音符休止符。例如：

> C++ is a rich language—one requiring extensive experience to master.

写作者有时使用一对破折号来阻止题外话，如下例如示：

> **Protocol Buffers**——often nicknamed **protobufs**——encode structured data in an efficient yet extensible format.

我们可以在上述的示例中使用逗号而不是破折号吗？当然是可以的。为什么我们选择使用破折号而不是分号呢？感觉。艺术。经验。

#### 破折号和连字符

考虑下表中的水平标点符号：

| 名称 | 标点符号 | 相对宽度 |
| --- | --- | --- |
| em-dash | —— | 最宽（通常，相当于字母m的宽度）|
| en-dash | — | 中等（通常，相当于字母n的宽度） |
| hyphen | - | 最窄 | 

一些风格指南推荐 **en-dash** 用于某种用途。然而，[谷歌风格指南（Google Style Guide）](https://developers.google.com/style/dashes#en-dashes)关于破折号直截了当的建议：

> 不要使用。

**连字符（hyphens）** 很棘手。在技术写作中，连字符用于复合术语中的单词连接，例如：

- self-attention
- N-gram

令人困惑的是，三个词的复合术语通常在第一和第二个单词之间包含连字符，但不在第二和第三个单词之间使用。例如：

- decision-making system
- floating-point feature

如果对连字符有疑问，请查阅字典，词汇表或样式指南。

### 括号
使用括号来表示次要观点和题外话。括号告诉读者，所附文本并不重要。由于附属的文本并不重要，所以有些编辑认为应该用括号括起来的文本不应该出现在文档中。作为一种妥协，在技术文档中应该尽量少用括号。

关于句号和括号的规划并不清晰。以下是标准的规则：
- 如果一对括号包含了整个句子，则句号放在圆括号内。
- 如果一对括号使用了一个句子的结尾，但不并包含整个句子，则句号放在括号外面。

例如：
> (Incidentally, Protocol Buffers make great birthday gifts.)
> 
> Binary mode relies on the more compact native form (described later in this document).

## Markdown

**Markdown** 是一种轻量级的标记语言，许多技术专业人员使用它来创建和编辑技术文档。使用 Markdown, 你可以在普通文本编辑器（如vi 或 Emacs）中编写文本，插入特殊字符来创建标题，加粗，项目符号等。例如，下面示例展示一个用 Markdown 格式化的简单技术文档：

> \#\# bash and ksh
> 
> \*\*bash\*\* closely resembles an older shell named \*\*ksh\*\*.  The key
> \*practical\* difference between the two shells is as follows:
> 
> \*  More people know bash than ksh, so it is easier to get help for bash
>   problems than ksh problems.
> 

上述技术文档会被渲染成如下样子：

> ## bash and ksh
>
> **bash** closely resembles an older shell named **ksh**.  The key
> *practical* difference between the two shells is as follows:
>
> *  More people know bash than ksh, so it is easier to get help for bash
   problems than ksh problems.

Markdown 解析器会将 Markdown 文件转换成 HTML。然后，浏览器将最终的 HTML 展示给读者。

我们建议通过以下教程来熟悉 Markdown：

- [www.markdowntutorial.com](https://www.markdowntutorial.com/)
- [Mastering Markdown](https://guides.github.com/features/mastering-markdown/)

## 总结

技术写作一包含以下技术写作的基础课程：

- 使用一致的术语。
- 避免模棱两可的代词。
- 首选主动语态而不是被动语态。
- 选择具体的动词而不是模糊的动词。
- 每个句子聚集到一个思想上。
- 将一些长句转换成列表。
- 移除不必要的单词。
- 如果次序重要，使用有序列表；如果次序不无关，则使用无序列表。
- 保持列表项的平行性。
- 有序列表荐使用祈使动词开头。
- 使用适合的方式介绍表格和列表。
- 创建漂亮的开头句，以建立一个段落的中心思想。
- 每个段落聚集到一个主题上。
- 确定你的受众需要学习的内容。
- 让文档更贴近你的受众。
- 在文档开头处理建立文档的主要观点。

<b id="f1">1</a>. 小菜一碟 [↩](#a1)
<b id="f2">2</a>. 易如反掌 [↩](#a2)
