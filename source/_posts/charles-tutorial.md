---
title: Charles简明教程
date: 2024-01-10 22:57:20
categories:
- 开发工具
tags:
- Charles
- 测试
---

![charles](/images/charles-tutorial/bg.png)

[Charles](https://www.charlesproxy.com/) 是一个功能强大的网络代理工具，常用于抓包、分析网络请求、模拟网络环境等操作。本入门指南将帮助你快速上手 Charles，掌握其基本用法和常用功能。

> 注: Charles是付费软件，白嫖党自行搜索破解教程。

## 下载安装

访问 Charles [官方网站](https://www.charlesproxy.com/download/)下载对应操作系统的安装包，按照提示完成安装。

## 配置Charles

### Mac端配置

1、链接Wifi，正常上网

2、安装Charles根证书
> Help > SSL Proxying > Install Charles Root Certificate

3、信任Charles根证书

> 打开钥匙串，搜索刚才安装好的Charles根证书，点击Charles Proxy CA，展开的Trust一栏，点击**Always Trust**。

4、开启SSL代理

> Proxy > SSL Proxying Setting > SSL Proxying > Enable SSL Proxying

5、开启HTTP代理
> Proxy > Proxy Settings > 将HTTP端口号设置为8888

6、打开Charles，开启录制

> Proxy > Start Recoding

### 移动端设备端配置

1、移动设备和Mac端链接到同一Wifi，确保它们在同一网段。

2、查看Charles代理服务器地址和端口

> Help > SSL Proxying > Install Charles Root Certificate > Install Charles Root Certificate on a Mobile Device or Remote Browser

![charles](/images/charles-tutorial/2.png)

3、配置代理
> 找到链接的Wifi > 设置(i圆圈图标) > 配置代理 > 手动 > 填写第2步中服务器地址和端口号

![charles](/images/charles-tutorial/3.jpg)

移动设备端链接到代理服务器后，Mac端会出现链接认证提示，点击允许。

4、打开Safari，并输入网址chls.pro/ssl，下载证书

5、手机的设置 > 通用 > 描述文件与设备管理，找到Charles，点击验证

6、手机设置 > 通用 > 关于本机 > 证书信任设置，确认Charles Proxy CA的信任开关处于打开状态

## 抓取HTTPS包

Mac端为指定请求设置代理

> Proxy > SSL Proxying Setting > SSL Proxying > Enable SSL Proxying > Add > *:443

## 过滤网络请求

通常情况下，我们需要对网络请求进行过滤，只监控向指定服务器上发送的请求。

> Proxy > Recording Settings > Include > Add > 填写要过滤的主机地址（域名）和 端口号。

## 限制网速

Charles可以模拟慢速网络或者高延迟的网络。

> Proxy > Throttle Setting > 勾选 Enable Throttling > only for selected host 可以设置指定的主机访问进行限制网络

## Map

Charles的Map功能分 **Map Remote** 和 **Map Local** 两种。Map Remote 是将指定的网络请求重定向到另一个网址请求地址，Map Local 是将指定的网络请求重定向到本地文件。

### Map Remote

需要分别填写网络重定向的源地址和目的地址，对于不需要限制的条件，可以不填。这个功能在做Web开发是特别有用，可以比较方便地将移动设备的请求转发至后端服务器，比如开发机。

> Tool > Map Remote > Enable Map Remote > Add

### Map Local

对于 Map Local 功能，我们需要填写的重定向的源地址和本地的目标文件。对于有一些复杂的网络请求结果，我们可以先使用 Charles 提供的 “Save Response…” 功能，将请求结果保存到本地（如下图所示），然后稍加修改，成为我们的目标映射文件。

然后将一个指定的网络请求通过 Map Local 功能映射到了本地的一个经过修改的文件中。

> Tool > Map Local > Enable Map Local > Add

## Rewrite

Map Local 有一个潜在的问题，就是其返回的 Http Response Header 与正常的请求并不一样。这个时候如果客户端校验了 Http Response Header 中的部分内容，就会使得该功能失效。解决办法是同时使用 Map Local 和 Rewrite 功能，将相关的 Http 头 Rewrite 成我们希望的内容。

Rewrite 功能是对某一类网络请求进行一些正则替换，以达到修改结果的目的。

## 参考资料

- [charlesproxy官网](https://www.charlesproxy.com/)
- [better-mobile-app-testing-with-charles-proxy](https://www.testeffective.com/better-mobile-app-testing-with-charles-proxy/)