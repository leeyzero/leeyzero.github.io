---
title: 免费让网站启用HTTPS
date: 2022-11-26 14:23:25
categories:
- 开发工具
tags:
- Web
- HTTP
- HTTPS
- SSL
---

之前搭建了一个 {% post_link deploy-codeserver-on-centos codeserver
%} 的开发环境，但还遗留了配置HTTPS访问域名的问题。本周正好有空搞下，本来打算花钱买一个HTTPS证书，发现 [Let's Encrypt](https://letsencrypt.org/) 提供了免费的HTTPS证书，而且还提供了配套的工具让网站开启HTTPS变得非常简单，本文记录下安装步骤。

在介绍安装步骤之前先简单介绍一下 [HTTPS](https://en.wikipedia.org/wiki/HTTPS) 的工作原理，不感兴趣的同学可以直接跳过。

简单来说，HTTPS 就是安全的HTTP（S表示Secure的意思），我们知道HTTP报文是采用明文传输的，报文容易被窃听或篡改。HTTPS 是在传输层和应用层中加一个安全层（SSL），负责对报文进行加密和解密。

传统的[对称加密](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)（加密解密使用相同密钥）要在传输两端共享密钥，涉及到密钥安全问题。而[非对称加密](https://en.wikipedia.org/wiki/Public-key_cryptography)（公钥加密，私钥解密）可以完美解决密钥交换问题。非对称加密的公钥是公开的，任何人都可以使用这个公钥进行加密。

但别人又怎么相信这个公钥是你发布的呢，这又是一个信任问题，解决办法是引入一个可信息的第三方机构。通常的做法是将这个公钥放到一个证书（Certificate）中，然后由这个可信任的第三方机构来统一认证和颁发。这个可信任的第三方机构就是[证书颁发机构](https://en.wikipedia.org/wiki/Certificate_authority)（CA，Certificate Authority）。

拿到证书后，怎么验证这个证书是不是第三方机构颁发的呢？（哈哈，是不是感觉问题好多呀），答案是使用[数字签名](https://en.wikipedia.org/wiki/Digital_signature)技术，简单来说就是为证书的内容做一个签名，并附到证书的末尾，这个签名具有惟一性和不可伪造性。

客户端（通常是浏览器）收到证书时会对证书合法性进行检查。如果这个机构是可信任的权威机构颁发的，浏览器可能已经知道其公开密钥了（浏览器会预先安装很多签名颁发机构的证书），这样，就可以通过数字签名来验证证书的完整性了。

所以，客户端和服务端进行HTTPS通信时，除了进行正常的TCP三次握手外，还需要进行SSL握手，这个过程主要是从服务端拿到证书、验证证书的合法性，然后交换加密密钥。后续的通信就可以使用这个加密密钥对报文加密和解密了。

突然发现写多了（化繁为简能力还有提高），以上就是HTTPS的大致原理，当然HTTPS的细节交互更加复杂，以上概述只是让大家对HTTPS有个宏观上的认识。有了这个背景，我们就知道启用HTTPS主要需要以下两个步骤：

- 从CA机构获取一个受信任的HTTPS证书。
- 将证书部署到服务端。

[Let's Encrypt](https://letsencrypt.org/) 就是一个可信息的证书颁发机构，它颁发的免费数字证书浏览器是信任的，而且它还提供便捷的安装和续约工具，下面就进入安装环节吧。

<!--more-->

## 系统环境

各个系统环境有所差异，以下我使用的系统环境为例：

- CentOS 8
- Nginx 1.20
- 需要sudo权限

## 安装步骤

1. 打开 [certbot.eff.org](https://certbot.eff.org/instructions)
2. 选择 操作系统 和 Web Server，我使用的是CentOS 8 + Nginx
3. 安装 [snapd](https://snapcraft.io/docs/installing-snapd/)
4. 升级 snapd 到最新版本

```shell
$ sudo snap install core
$ sudo snap refresh core
```

5. 安装 certbot

```shell
$ sudo snap install --classic certbot
```

6. 让 certbot 命令可用

```shell
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

7. 安装数字证书

```shell
$ sudo certbot --nginx
```

8. 自动续约数字证书

```shell
$ sudo crontab -e
15 3 * * * /usr/bin/certbot renew --quiet
```
每天3：15定时检查并自动续约数字证书

9. 重新加载nginx配置

```shell
$ sudo service nginx reload
```

10. 测试https是否生效

使用浏览器打开你的网站 [https://yourwebsite.com/](https://yourwebsite.com/)，看看https是否已经生效呢。如果https已生效，将会在浏览器地址栏看到一把代表https的小锁，点开后可以看到证书的相关信息。

## 踩坑记录

我安装完后并没有生效，原因是我使用了云托管主机，安全规则入方向并没有打开443端口（HTTPS默认使用443，可以在nginx.conf文件中看到配置），修改下安全规则即可。

## 参考资料

[1] [letsencrypt.org](https://letsencrypt.org/)
[2] [certbot.eff.org](https://certbot.eff.org/)
