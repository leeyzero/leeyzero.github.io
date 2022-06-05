---
title: 【翻译】Google技术写作（一）
date: 2022-05-31 09:49:26
categories:
- 读书笔记
tags:
- 写作
---

本文翻译自 [Google Technical Writing Courses](https://developers.google.com/tech-writing/overview)，是Google推出的技术写作课程，该课程包含两个部分，本文是第一部分。

- {% post_link technical-writting-1 技术写作（一） %}：教你如何写出更清晰的技术文档，是学习第二部分的基础。
- {% post_link technical-writting-2 技术写作（二） %}：帮助你提升技术沟通技巧。

<!--more-->

**翻译仅供学习使用，如有侵权，立即删除。由于译者水平有限，本文不免存在遗漏或错误之处。如有疑问，请查阅原文**。

以下是译文。

# 简介
## 目标受众

你需要有一点英语写作能力，但并不需要成为一个很强的写作者后再来学习这门课程。

如果你没有接受过任何技术写作训练，本课程非常适合你。如果你已经接受一些技术写作的训练，学习本课程仍将是一种有效的复习。

## 学习目标
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

# 语法基础
本节介绍刚刚好的语法基础知识，以便能更好的理解后面的课程。如果你已经具备这些语法知识，可以直接跳到[单词](#单词)章节继续阅读。

为了简单起见，本节采取了一些捷径，实际的语法主题比本节建议的复杂得多。

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

## 名词
名词代表人物、地点或事物。**朱迪**、**南极洲**和**锤子**都是名词，无形的概念（如**健壮性**、**完美性**）也是如此。在以下示例段落中，我们将名词格式化为粗体：

> In the **framework**, an **object** must copy any underlying **values** that the **object** wants to change. The **protos** in the **codebase** are huge, so copying the **protos** is unacceptably expensive.

在编程中，你可以将类和变量视为程序中的名词。

## 代词
代词是一个间接层，指向或替代其他名词或句子。例如，考虑以下两个句子：

> Janet writes great code. **She** is a senior staff engineer.

在前面的示例中，第一句将 Janet 确立为名词。第二句用代词 She 代替了名词 Janet。
在以下示例中，代词 This 替换了整个句子：

> Most applications aren't sufficiently tested. **This** is poor engineering.

## 动词
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

## 形容词和副词
形容词修饰名词。例如在下面的段落中，注意形容词是如何修饰后面的名词的：

- Tom likes **red** balloons. He prepares **delicious** food. He fixed **eight** bugs at work.

大多数副词修饰动词。例如，注意下面句子中的的副词（**efficiently**)是如何修饰动态（**fixes**）的:

- Jane **efficiently** fixes bugs.

副词不一定紧挨着动词。例如，在下面的句子中，副词（**efficiently**）和动态（**fixes**）相隔两个单词：

- Jane **fixes** bugs **efficiently**.

副词也可以修饰形容词和副词。

## 介词
介词指定两个事物之间的关系。一些介词通常回答这个问题，“一个事物相对于另一个事物在哪里？“，例如：

- The submenu lies **under** the menu.
- The definition appears **next to** the term.
- The print function falls **within** the main routine.

一些介词回答这个问题，”一个事件相对于另一个事件是什么时候？“，例如：

- The program evaluates the addition operation before evaluating the subtraction operation.
- The cron daemon executes the script every Tuesday at noon.

另一些介词（如**by**和**of**）回答其它类型的关系，如下面的句子中使用**by**表示一本书与作者的关系：

- The C Programming Language **by** Kernighan and Richie remains popular.

## 连词和过渡词
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

# 单词

我们对文档进行了大量的研究，结果发现世界上最好的句子主要由单词组成。

## 定义新术语和不熟悉的术语
在写作或编辑时，学会识别那些目标受众不熟悉的术语。当你发现此类术语后，采取如下两种策略之一：
- 如果这个术语已经存在，链接到现有的解释。（不要重复造轮子）
- 如果你在文档中引入了这个术语，请定义该术语。如果你在文档中引入了许多术语，将这些定义收集到词汇表中。

## 始终使用术语
如果你在方法的中途改变变量的名称，你的代码将无法编译。类似地，如果你在文档中间重命名了术语，你的想法也不法编译（在你的脑中）。
修养：在整个文档中始终使用同一个明确的单词或术语。一旦你命名了一个组件为**thingy**，不要把它重命名为**thingamabob**，例如，下面的段落错误地将Protocol Buffers重命名为protobufs:
> **Protocol Buffers** provide their own definition language. Blah, blah, blah. And that's why **protobufs** have won so many county fairs.

是的，技术写作是严格和限制性的，但至少技术写作提供了一个很好的解决方法。即，在引入冗长的概念或产品名称时，你还可以指定该名称的缩写版本。然后，你可以在整个文档中使用该缩写名称。例如，以下段落很好：
> **Protocol Buffers** (or **protobufs** for short) provide their own definition language. Blah, blah, blah. And that's why protobufs have won so many county fairs.

## 正确使用首字母缩写
在文档或章节中首次使用不熟悉的首字母缩写词时，请拼出完整的术语，然后将首字母缩写词放在括号中。将拼写版本和首字母缩写词都以粗体显示。例如：

> This document is for engineers who are new to the **Telekinetic Tactile Network** (**TTN**) or need to understand how to order TTN replacement parts through finger motions.

然后你可以在随后使用首字段缩写，如下例所示：
> If no cache entry exists, the Mixer calls the **OttoGroup Server** (**OGS**) to fetch Ottos for the request. The OGS is a repository that holds all servable Ottos. The OGS is organized in a logical tree structure, with a root node and two levels of leaf nodes. The OGS root forwards the request to the leaves and collects the responses.

不要在同一个文档中来回使用首字母缩写版本和扩展版本。

## 使用首字母缩写还是完整术语？
当然，您可以正确地引入和使用首字母缩略词，但你真的应该使用首字母缩略词吗？好吧，首字母缩略词确实减少了句子的大小。例如，TTN 比 Telekinetic Tactile Network 短两个单词。然而，首字母缩略词实际上只是一个抽象层。读者必须在脑海中将最近学到的首字母缩略词扩展成整个术语。例如，读者在脑海中将 TTN 转换为 Telekinetic Tactile Network，因此“更短”的首字母缩略词实际上比完整术语需要更长的时间来处理。

大量使用的首字母缩略词后逐渐形成了它自己的身份。在多次出现之后，读者通常会停止将首字母缩略词扩展为完整术语。例如，许多 Web 开发人员已经忘记了 HTML 扩展成什么了。

下面是首字母缩写的指南：
- 不要定义只使用很少次数的首字母缩写词。
- 请定义满足以下两种条件的首字母缩写词：
    - 首字母缩写词明显短于完整术语。
    - 首字母缩写词在文档中多次出现。

## 识别有歧义的代词
许多代词指代前面介绍过的名词。这样的代词类似于编程中的指针。就像编程中的指针一样，代词往往会引入错误。不恰当地使用代词会导致读者头脑中出现认知偏差，就和空指针错误一样。在许多情况下，你应该简单地避免使用代词，而是直接重复使用名词。但是，代词的效用有时非常有用（如这句话）。

请考虑以下代词的使用指南：
- 只在介绍名词后使用代词；切勿在介绍名词之前使用代词。
- 将代词放置在尽可能靠近所指名词的位置。一般来说，如果将名词和代词隔开的单词超过五个，请考虑重复名词而不是使用代词。

### It and they
以下代词在技术文档中经常造成困惑：
- it
- they, them, and their

例如，在下面句子中，**It** 指代 Python 还是 C++ 呢？
> Python is interpreted, while C++ is compiled. It has an almost cult-like following.

另一个例子是，**their** 在下面句子中表示什么呢？
> Be careful when using Frambus or Carambola with HoobyScooby or BoiseFram because a bug in **their** core may cause accidental mass unfriending.

### This and that
考虑另外两个问题代词：
- this
- that

例如，在以下模棱两可的句子中，**This** 是指用户ID，正在运行的进程，还是两者呢？
> Running the process configures permissions and generates a user ID. **This** lets users authenticate to the app.

为了帮助读者，请避免在不清楚他们指的是什么的情况下使用 **this** 或 **that**。使用以下任一策略来澄清 **this** 和 **that** 的模棱两可的用法：
- 使用合适的名词替换 **this** 或 **that**。
- 在 **this** 或 **that** 后马上使用那个名词。 

# 主动语态

技术写作中的绝大多数句子应该是主动语态。本节教你如何执行以下操作：

- 区分被动语态和主动语态。
- 将被动语态转换为主动语态，因为主动语态通常更清晰。

## 区分简单语句中的主动语态和被动语态
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

## 识别主动动词

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

## 在更复杂的语句中区分主动语态和被动语态
很多语句中包含多个动词，其中有些是主动的，有些是被动的。例如，下面的语句中包含两个动词，两个动词都是主动语态：

![](/images/technical-writting/passive-passive.svg)

下面对同一个语句，将部分被换成主动语态：

![](/images/technical-writting/active-passive.svg)

下面对同一个语句，将全部转换成了主动语态：

![](/images/technical-writting/all-active.svg)

## 优先使用主动语态
大部分情况下使用主动语态。尽量少用被动语态。主动语态具体以下优点：
- 大多数读者在心理上将被动语态转换成主动语态。为什么要让你的读者花费额外的处理时间呢？通过坚持使用主动语态，读者可以跳过预处理阶段，直接进入编译阶段。
- 被动语态会混淆你的想法，颠倒语句。被动语态间接报告动作。
- 有些被动语态的语句完全忽略主语，这会迫使读者去猜测主语是谁。
- 主动语态通常比被动语态更短。

# 清晰的句子
喜剧作家追求最有趣的结果，恐怖作家追求最恐怖的结果，技术作家追求最清晰的结果。在技​​术写作中，清晰度优先于所有其他规则。本节提供几种使句子清晰的方法。

## 选择强动词
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

## 减少使用 there is/there are
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

## 尽量减少特定的形容词和副词（可选）
形容词和副词在小说和诗歌中的表现惊人。多亏了形容词，普通的草能变成杂草（prodigal）和葱绿（verdant），而毫无生气的头发也能变成的有光泽（lustrous）和茂密（exuberant）。副词使马跑的疯狂（madly）和自由（freely），让狗叫地大声（loudly)和凶猛（ferociously）。不幸的是，形容词和副词有时会让技术读者大声而凶猛地吠叫。这是因为形容词和副词对于技术读者来说往往定义过于松散和主观。更糟糕的是，形容词和副词会使技术文档听起来像营销材料一样危险。例如，考虑一篇技术文档中的以下段落：

> Setting this flag makes the application run screamingly fast.

诚然，**快得惊人**的速度会引起读者的注意，但不一定是一种好的方式。为你的技术读者提供事实数据，而不是营销言论。将无形的副词和形容词重构为客观的数字信息。例如：

> Setting this flag makes the application run 225-250% faster.

前面的变化是否会有损这句话的一些魅力？是的，有一点，但是修改后的句子获得了准确性和可信度。

# 简短的句子

出于以下原因，软件工程师通常会尽量减少实现代码的行数：
- 简短的代码通常更易于其它人阅读。
- 简短的代码通常比较长的代码更易于维护。
- 额外的代码可能会引入潜在的故障。

实际上，这些规则同样适用于技术写作：
- 简短的文档可读性更好。
- 简短的文档更易于维护。
- 额外的文档会引入额外的问题。

寻找最短的文档实现需要时间，但最终是值得的。短句比长句能更有效地交流，短句通常比长句更容易理解。

## 一个句子只聚焦到一个想法上
## 将长句子转换为列表
## 消除或减少多余的单词
## 减少从句（可选）
## 区分 that 和 which

# 列表和表格

# 段落

# 受众

# 文档

# 标点符号

# Markdown

# 总结
