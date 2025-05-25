---
title: 如何做Code Review
date: 2021-04-02 17:12:17
categories:
- 编程匠艺
tags:
- 'Code Review'
---

Code Review是保障代码和产品质量的重要手段，但却被绝大部分公司所忽略。本文主要基于Google对Code Review的实践，结合自身的经验谈谈团队中该如何做Code Review。

## 1. 什么是Code Review

Code Review是代码评审人（Code Reviewer）对代码提交者（Code Committer）做审查（Review）的过程。

## 2. 为什么要做Code Review

- 提升代码/产品质量
- 团队工程师文化传承
- 团队成员间知识/经验分享

<!--more-->

## 3. 代码评审人角度

### 3.1. 基本原则

> 变更列表（CL，Change List）应该让整体代码库更加整洁。

整洁可以从两个层面来理解：对代码来身来讲，每次变更应该需要更好的可读性；对系统来讲，每次变更应该让系统工作地更好；换句话说，我们应该拒绝那些让代码库变得更糟的变更列表。

### 3.2. 浏览变更列表

- **浏览整体变更**

查看变更信息，理解当前变更主要是干什么？当前变更是否合理？

- **检查变更列表的主要部分**

检查变更列表文件，通常来说，这些变更列表都是按一定的逻辑结构组织的，浏览变更列表文件可能对变更有一个整体的认识。检查当前变更列表是否太大，能否拆分成多个变更列表？这些变更文件是否跟当前变更主题有关？

- **按合理顺序查看**

按合理的逻辑顺序查看变更，比如按接口维度。

### 3.3. Code Review具体需要做什么

- **设计（Design）**

代码实现和对应的设计是否合理？

- **功能性（Functionality）**

代码实现的功能是否是代码提供者预期的？这些功能是否符合用户预期？

- **复杂度（Complexity）**

代码实现是否可以更简单？其它人在使用或维护代码时是否能理解？

- **测试（Tests）**

当前变更是否能通过单元测试（Unit Test）、集成测试（Integration Test）、端到端测试（End-to-End Test）？测试是否覆盖到当前变更？

- **命名（Naming）**

变量名、函数名、类名、包名是否合理？命名是否容易理解？

- **注释（Comments）**

注释是否清晰或容易理解？

- **风格（Style）**

代码风格是否符合规范？尽量持有包容的心态，遵循团队已有代码规范。

- **文档（Documention）**

如果当前变更涉及到客户使用，如果接口变更，编译脚本变更等，代码提交人是否同步更新了相关文档？

- **上下文（Context）**

通常代码diff工具只展示代码变更的行，但这可能会忽略到一些重要信息。

### 3.4. Code Review的速度

> 尽快开始做Code Review，原则上不要跨天

Code Review是需要Commiter和Reviewer高度配合的。通常可能存在以下问题：

- 评审人当前正在进行其它工作，不能立即开始做Code Review
- 代码提交者提交的变更列表过大，Reviewer需要花费比较大的精力
- 紧急情况

### 3.5. 写评论

- 礼貌用语
- 用简洁的语言指出问题并解释原因
- 最好提供指导方案：Reviewer只是提供建议，最终是由Committer修复

## 4. 代码提交人角度

### 4.1. 变更说明

一句话清晰描述变更意图，尽量用中文。

### 4.2. 变更列表

变更列表应该尽量"小"。这里的小没有严格定义，可以按逻辑单元进行组织，一次变更列表代码行数最好不要超过200行。

### 4.3. 处理评论

- 礼貌用语，解释原因
- 修复评审代码，提交patch
- 解决冲突

## 5. 参考资料

[1] [Code Review Developer Guide Introduction](https://google.github.io/eng-practices/review/)
[2] [LinkedIn’s Tips for Highly Effective Code Review](https://thenewstack.io/linkedin-code-review/)
[3] [Code Review Best Practices](https://blog.palantir.com/code-review-best-practices-19e02780015f)
