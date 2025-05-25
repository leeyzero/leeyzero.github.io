---
title: 修改代码的艺术
date: 2024-12-29 10:15:17
categories:
- 编程匠艺
tags:
- 重构
---

![cover](/images/working-effectively-with-legacy-code/cover.jpeg)

---

这篇文章基于最近在团队内做了一次分享，并做了一些简单修改，仅供各位网友参考，观点不一定成熟，但都是结合实际问题的一些反思与总结，欢迎讨论。

在软件的整个生命周期中，80%以上的时间是在维护软件，这也是开发者每天面临的最主要的工作。如果对代码的变更没有“章法”，随着时间流逝，软件就会不可避免地渐渐变得复杂、难以理解，最终拖慢产品迭代效率，甚至频繁引入线上故障，导致业务损失。

对于这个主题，[Working Effectively with Legacy Code](https://dl.ebooksworld.ir/motoman/Working.Effectively.with.Legacy.Code..www.EBooksWorld.ir.pdf) 和 [Refactoring Improving the Design of Existing Code](https://silab.fon.bg.ac.rs/wp-content/uploads/2016/10/Refactoring-Improving-the-Design-of-Existing-Code-Addison-Wesley-Professional-1999.pdf) 给出了很多具体的做法，本文不会一一介绍。本文主要分为两部分：

- 第一部分主要介绍软件的变更机制以及应对变更的策略，并着重介绍安全变更的重要保障——单元测试；
- 第二部以通过一个案例解释如何进行安全重构；

<!-- more -->

## 软件变更的原因

软件变更有四个原因：

- 添加新功能
- 修复BUG
- 改善设计
- 优化资源使用

改善设计和优化资源都是在不改变系统行为的情况下，改变软件的内部结构和组织。但它们的侧重点不同，**改善设计强调通过改善系统的结构来提升软件的可维护性，而优化资源则强调系统对时间和空间资源使用的优化。**

随着软件的不断迭代，当系统越来越复杂时，保留软件的现有行为是一项挑战。我们在做变更时，如果不能较好的评估变更对系统的影响，就可能会破坏系统。而正确评估变更带来影响的关键在于对系统的理解，这就是改善设计的意义。

## 变更方式与良性反馈

变更软件分为两种方式：

- 编辑并祈祷（Edit and Pray）
- 覆盖并修改（Cover and Modify）

在面对大规模代码变更时，如何保证变更不会破坏系统行为呢？人最不擅长处理大量细节，即便是再小心，你也很难保证你的修改不会引入问题，所以编辑并祈祷显然是不可取的。我们需要一种反馈机制，通过掌握专业的技术和工具来应对变更。

覆盖和修改像一层安全防护网，当变更引入时，能够防止变更改变了系统的当前行为。软件覆盖率是指使用单元测试覆盖代码行数，这里并不是说覆盖率越高越好，更重要的是我们需要一个良好的测试集合，这个测试集合可以帮助我们快速回归变更是否是安全的。**当单元测试未通过时，这是一种良好的反馈，它让我们意识到当前的变更已经破坏了系统现有行为。**

为什么只提单元测试？集成测试、端到端测试不重要吗？当前重要，但单元测试是成本最低、最有效的测试手段。

<img src="/images/working-effectively-with-legacy-code/test-pyramid.png" width="600">

我们能够从测试金字塔中得到一些启示：
**1、测试是分层的：自底向上可分为：单元测试、集成系统、端到端测试。**
**2、越接近金字塔底层，集成度越低、测试速度越快、代码覆盖率越高、测试成本越低；反之越接近金字塔顶层，集成度越高、测试速度越慢、覆盖率越低、测试成本越高。**

测试金字塔是对系统变更的良性反馈。如果变更引入的破坏系统行为在测试金字塔中都没有被检测出来，就会影响到目标用户。目标用户的反馈也是改善系统的一种有效途径，它除了可能给用户造成损失之外，反馈的时间周期比较长，问题定位以及修复成本都会比较高。

从测试金字塔的职责分工来看，RD和QA承接着不同职责：

| 类型 | 负责人员 | 备注 |
| --- | --- | --- |
| 单元测试 | RD | |
| 集成测试 | RD、QA | |
| 端到端测试 | QA | RD自测也需要端到端测试 |

单元测试是一种投资回报很高的工具，它能有效改善代码质量，也是惟一一种代码覆盖率能够达到100%的测试手段。特别是在应对大规模代码变更时，单元测试至关重要。但在变更遗留代码时，经常面临着这样一种情况：

> When we change code, we should have tests in place. To put tests in place, we often have to change code.

这就是变更遗留代码的困境。[Michael C. Feathers](https://michaelfeathers.silvrback.com/) 给出了核心思路和具体的一些实践套路。

## 变更代码的核心思路

### 确定变更点

确定变更点要求开发人员充分理解需求和架构。如果当前的架构设计比较合理，需求没有前后矛盾（通常是延着产品框架迭代）。开发者是比较容易将需求映射到变更的代码中的。但如果是在做大规模代码重构时，我们面临的第一个问题是职责划分（重构通常意味着之前的职责划分不合理）。关于模块职责的划分，开发人员通常会根据业务流程去做系统分解，但 [David. L. Parnas](https://en.wikipedia.org/wiki/David_Parnas) 在 [On the Criteria To Be Used in Decomposing Systems into Modules](https://dl.acm.org/doi/10.1145/361598.361623) 中告诉我们，在一个复杂系统中，基于流程的拆解几乎总是错的。

> We have tried to demonstrate by these examples that it is almost always incorrect to begin the decomposition of a system into modules on the basis of a flowchart. We propose instead that one begins with a list of difficult design decisions or design decisions which are likely to change. Eachc module is then designed to hide such a decision from the others. Since, in most cases, design decisions transcend time of execution, modules will not correspond to steps in the processing.

**我们通过这些例子试图说明，基于流程图对系统进行模块分解几乎总是错误的。相反，我们建议从列出难以设计或可能发生变化的设计决策开始，然后，设计每个模块并向其它模块隐藏这些设计决策。因为在大多数情况下，设计决策超越了执行时间，因此模块运行不会与处理步骤相对应。**

*注：这里指的流程图可以理解为业务流程。*

关于模块职责的划分方法，不是本文的重点，有机会针对这个主题再单独分享。这里想强调的是，对一个遗留系统重构比开发一个新系统面临更多的挑战。

### 找到测试点

测试点跟变更点通常是相对应的。但遗留代码中的变更可能会涉及范围较广地方（模块之间的耦合比较深），需要梳理出变量、方法的引用范围，并评估其影响，并对这些影响的地方进行测试。

### 解依赖

依赖是导致难以测试的主要障碍。主要表现在两方面：

**1、难以在测试case中实例化目标对象；**
这种情况通常发生在构造函数依赖于其它对象，或在构造函数中隐式依赖了其它对象。

**2、难以在测试case中调用方法。**
依赖于网络IO、文件IO、全局变量等。

解依赖的关键在于**预留接缝（Seam）**。接缝是指可以在不编辑程序的情况下改变程序行为。

> A seam is a place where you can alter behavior in your program without editing in that place.

接缝类型包括：

- 预处理接缝
- 链接接缝
- 对象接缝

预处理接缝和链接接缝在C/C++语言中比较常见，在此不深入讨论，本文只关注对象接缝。
对于Go语言，语言提供了反射特性，有些工具（如gomonkey）提供了运行时接缝，可以在运行时替换函数（方法）地址，这一点对单元测试是比较友好的。但需要注意**不要过于依赖gomonkey而忽略设计上的问题**。

### 编写测试

代码逻辑中有了接缝，编写单元测试就会容易很多。但设计一个好的测试集合其实不太容易，当前大家写单元测试，主要目标只是为了达成单测覆盖率，而忽视单测质量。

### 改动和重构

在书中《Working Effectively with Legacy Code》作者提倡测试驱动开发（TDD），我个人并不是TDD的拥趸。但对于修复BUG或重构遗留代码，先编写单元测试，再重构代码是一个好的实践。它让你先用单元测试保障你期望的系统行为，然后再变更代码，如果变更代码导致单元测试不通过，这就是一种良性反馈，说明你的改动已经破坏了现有系统行为。此时你需要修复代码，重新运行单元测试，验证case是否能够通过，循环此过程，直到单元测试全部通过。

## 案例分析

关于代码重构，特别是大规模的代码重构， 重构流程跟上述介绍的思路差不多，很多重构套路和解耦技术都会综合运用，不限于具体某项技术。

关于大规模代码重构，我觉得最重要是做好以下几点：
1、**结构化**：结构化有助于理解，结构化的重点是对模块（这里指的模块不一定是一个服务，也可以是一个包或一个类）。职责的划分，以其对必要的维度按照合理的方式组织。
2、**接口抽象**：抽象有助于理解和保持结构的稳定，还记得 {% post_link software-design-philosophy 软件设计哲学 %} 中关于深接口设计，将复杂实现隐藏至接口后面的设计原则吗？
3、**单测覆盖**：单元测试是重构质量的重要保障。
4、**增量修改**：每次只重构一个小的功能点。

1）和2）可以理解是对重构的一个宏观设计。3）和4）是将前两步拆分成**连续、可闭环**的功能点。可闭环是指重构完一个功能点，系统外部行为（非运行时）和重构前是一致的。

最好的重构策略是在日常变更中，小范围增量重构，如果团队成员熟悉掌握重构技术和严明的纪律，代码质量是能够得到不断提高的。之前学习过整洁代码的原则：**如果团队成员每次在提交代码时能够让代码库更加整洁，代码质量必定越来越高，开发人员也能更好应对业务需求；反之则随着时间的积累，系统的复杂度越来越高，最终严重拖慢开发节奏。**

以下苹果端内购买为例，解释整个重构过程。苹果端内购买是一种苹果的支付方式，简单来说苹果用户在应用内购买先是将钱通过appstore直接支付给苹果，然后从苹果拿到一个支付凭证（收据），并将这个收据上报给应用后端，后端校验收据成功后给用户发放虚拟商品。

在这个案例中，主要分享重构思路，对原始业务逻辑做了很多简化。

### 确定变更点

对遗留代码确定变更点，是将遗留代码进行结构化的过程。遗留代码之所有需要重构，就是因为其缺乏结构，而缺乏结构的常见表现主要包括：

- 职责划分不明确
- 依赖关系混乱
- 重复代码
- 语义不一致（即便是在相同上下文）
- 状态多、状态生命周期长
- ...

结构化的前提是我们需要对遗留代码深入理解，运用**分治和抽象技术**将无序代码重新组织为有序（结构化）代码。先来看下当前收据上报的业务流程，为了便于说明，简化了很多细节。

<img src="/images/working-effectively-with-legacy-code/appstore-flowchart.png" width="320">

这里已经对逻辑做了较大简化，原代码有3000多行（还不算上下游依赖部分），我们先不讨论这个流程的分解是否合理。但从整个流程中我们可以发现以下问题：

- 各业务线逻辑耦合
- 各业务线下的场景较多，且逻辑不内聚
- 各业务线下的商品处理、订单处理、签约处理功能不内聚

这些问题也是日常迭代过程是容易变化的部分，为此我们需要对**容易变化的地方做设计决策，通过抽象的接口将这些变化隐藏起来**。

整体思路为：**纵向抽象业务流程，按业务线扩展；横向抽象易变化部分，分场景扩展。**

<img src="/images/working-effectively-with-legacy-code/appstore-pay.png" width="600">

### 功能闭环拆分

功能闭环是指，如果一个重构涉及到大规模的变更，我们需要分阶段进行，每个阶段的功能是可以独立上线的，以便我们可以直观感知到系统在往正确的方向走，同时降低大规模变更的风险。在上述case中，我拆分成三个大的阶段：

阶段一：纵向抽象：收敛上游调用入口，抽象纵向流程。该阶段完成后，主流程接口是稳定的，后续新业务接入时，不会破坏IAppstorePay的对外接口。
阶段二：实现简化：简化内购项和商品信息的映射逻辑。该阶段完成后，新增内购项，不会再涉及商品、订单修改逻辑。
阶段三：横向扩展：插件化改造。该阶段对校验逻辑、商品处理逻辑、订单处理逻辑、签约处理逻辑进行插件化改造。不同场景可以将逻辑内聚至插件中实现。到此业务易变化的部分已经全部隐藏至稳定的接口后面，后续新增业务只需实现相应接口。

### 解依赖

对于像Go这样具体反射能力的语言，通过运行时动态修改调用地址以达到测试接缝的目的比较简单（比如使用gomonkey），但不要滥用gomonkey而忽视了设计。对于没有反射能力的语言，比如C++，要编写单元测试，应该怎么做呢？我觉得最重要是掌握通过接口提供测试接缝。

我们来分析一个对象，它封装了一些属性，并提供了一此方法用于查询或修改这些属性。同时这个对象可能会依赖于其它对象、磁盘IO、网络IO、全局变量...。

对象的内部状态可以通过简化状态空间和合理的公有接口设计得以解决。外部依赖则均可以通过接口抽象解决。比如我们需要对一个Service类测试。我们需要对所有依赖对象提供测试接缝：

```Go
type IServiceExt interface {
    LocalIOMethod(...)
    RemoteIOMethod(...)
    SystemCallMethod(...) 
    GetGlobalVarMethod(...)
    DependencyObjectMethod(...)
    OtherMethod(...)
}

type Service struct {
    serviceExt IServiceExt
    ...
}

func NewService(..., serviceExt IServiceExt) *Service{
    ...
}
```

serviceExt是一个可选参数，这样可以在测试代码中对依赖调用进行mock。你可能会问，这跟gomonkey有什么区别呢？如果抛开设计问题，使用gomonkey和为测试预留接缝都能达到测试目的。而且使用gomonkey看起来还不侵入代码。单从Go语言来看，我觉得主要区别是显示依赖和隐式依赖的区别。

显示依赖最大的一个好处是可以直接从类定义中看出依赖调用，依赖变更时，能通过编译器直接感知，而gomonkey是运行时的，接口变更时，只有运行时才能感知。显示依赖的缺点是过度抽象，同时代码会多一层转换调用。

所以，我的建议是：**从系统设计角度出发，合理抽象。不反对使用gomonkey，但不要过度依赖gomonkey而忽视了设计层面的抽象。**

### 单测覆盖

单元测试的代码质量要求可能没有生产代码那么高，但我希望你按照生产代码的质量进行编写。最主要关注case的质量而不是数量。每个case尽量保持正交，这会让你的case更具有代表性。case命名规范且是可读的，要不然维护成本就是一个问题。

单元测试一般采用表驱动法，通过构造入参 > 执行目标代码 > 验证结果。对于没有精心设计入参、不校验结果的case，意义都不大。

如果你的变更导致某个（些）case失败，说明你的变更已经破坏了现有系统行为，此时你需要判断是你的代码逻辑有问题还是case已经不适用，你有责任对其修复。

### 改动和重构

变更代码涉及到写好代码的一些原则，这在 {% post_link write-readable-code 编写可读代码的艺术 %} 中已经讲过，在此不再赘述。但在变更过程中，有两样工具你需要用起来：

**充分利用编译器的能力**，它能在第一时间告诉你编译期的错误。比如当我们用string类型定义一些枚举时，为什么我们要重新定义其类型呢？除了可读性之外，我们在赋值时，可以让编译器帮助我们做类型检查，这能有效避免错误传参的问题（这在连续多个string参数中容易发生，而且错误还不容易被发现！）

**频繁执行单元测试**。这是一个良好的习惯，它能够帮忙你快速验证当前的修改是否符合预期，是一种快速的良性反馈。这里推荐Go中的三个工具：

**1、使用 go test -v --run 执行单元测试**

--run 后面接的单测名称是正则匹配的，所以我建议单元测试按一定的命名规则：Test{ServiceName}_{MethodName}_{CaseName}。
其中CaseName可选，例如：TestMessageQueueService_Publish_Success。这样你就像下面这样执行单元测试：

```shell
# 对 MessageQueueService 类的所有case进行测试
go test -v -tags iface_mock --run TestMessageQueueService 

# 对 MessageQueueService类的Publish方法的所有case测试
go test -v -tags iface_mock --run TestMessageQueueService_Publish

# 对 MessageQueueService类的Publish方法中的Success情况进行测试
go test -v -tags iface_mock --run TestMessageQueueService_Publish_Success 
```

**2、cover工具查看代码覆盖率**

```shell
go test -coverprofile=cover.out
go tool cover -func=cover.out
```

**3、uncover工具查看未覆盖的代码行**

```shell
go install rsc.io/uncover@latest
go test -coverprofile=cover.out
uncover cover.out
```

## 总结

上面的介绍虽然都是纲要性的，涉及到很多点，但我认为以下三点最为重要：

- **在一个复杂系统中，应该基于变化或难以设计的地方做设计决策，并将这些设计决策隐藏在稳定接口后面。**
- **在一个复杂系统中，单元测试是保障系统质量投入产出比最高的一项技术。**
- **编写单元测试的关键是接缝。**

## 参考资料

- [Working Effectively with Legacy Code](https://dl.ebooksworld.ir/motoman/Working.Effectively.with.Legacy.Code..www.EBooksWorld.ir.pdf)
- [Refactoring Improving the Design of Existing Code](https://silab.fon.bg.ac.rs/wp-content/uploads/2016/10/Refactoring-Improving-the-Design-of-Existing-Code-Addison-Wesley-Professional-1999.pdf)
- [The Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html#TheImportanceOftestAutomation)
- [On the Criteria To Be Used in Decomposing Systems into Modules](https://dl.acm.org/doi/10.1145/361598.361623)
- [Go Testing By Example](https://research.swtch.com/testing)