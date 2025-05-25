---
title: 搭建Hexo环境
date: 2021-04-02 17:00:31
categories:
- 开发工具
tags:
- 开发环境
- Hexo
- CentOS
---

搭建环境是一件很无聊的事情，最近换了一个较新的系统，少了很多系统级的依赖环境问题。本文简单记录搭建hexo环境的步骤，以便后续查阅。

## 系统环境

CentOS 8.x

## 安装依赖环境

- [Node.js](https://nodejs.org/en/download/)

至少10.13，建议12.0或更高版本。建议使用二进制方式[安装](https://github.com/nodejs/help/wiki/Installation)。

- [Git](http://git-scm.com/)

```
$ sudo yum install git-core
```

## 安装Hexo

- 使用npm安装hexo

```
$ npm install hexo
```

- 添加环境变量

```
$ echo 'PATH="$PATH:/path/to/node_modules/.bin"' >> ~/.profile
```

- 测试

```
$ hexo version
```

## 参考资料

[1] [hexo官方文档](https://hexo.io/docs/)
[2] [Nodejs官方文档](https://nodejs.org/en/)


