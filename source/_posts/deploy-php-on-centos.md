---
title: CentOS搭建Nginx+PHP环境
date: 2024-04-14 15:18:04
categories:
- 开发工具
tags:
- 开发环境
- Nginx
- PHP
- PHP-FPM
- CentOS
---

虽然PHP做业务后端逐步在被Go等语言替代，但使用PHP做简单业务封装和数据组装时，开发效率依然是比较高效的。使用Nginx运行PHP的常用方法是FastCGI模块。PHP-FPM (FastCGI进程管理器）极大地提高了你的Nginx+PHP环境的性能，所以这对高负载的网站很有用。本教程介绍在CentOS8上安装Nginx并配置PHP-FPM的步骤，以便后续参考。

## 依赖环境

- CentOS8
- 拥有sudo权限
- 更新dnf

```shell
sudo dnf update 
```

## Step1 安装Nginx

Nginx在仓库中已经存在，可以使用dnf工具直接安装：

```shell
sudo dnf install nginx
```

启动Nginx服务，同时让Nginx服务在系统启动时自动启动。

```shell
sudo systemctl enable nginx
sudo systemctl start nginx
```

检查nginx是否已启动

```shell
sudo systemctl status nginx
```

如果您的系统上启用了防火墙，请确保打开HTTP端口以供远程系统访问。HTTP为80端口，HTTPS，为443端口。

```shell
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --zone=public --permanent --add-service=https
sudo firewall-cmd --reload
```

在浏览器使用ip访问主机器，看是否能访问到nginx的默认页面了呢？

<!--more-->

![nginx default page](/images/deploy-php-on-centos/nginx.png)

## Step2 安装PHP和PHP-FPM

安装php和php-fpm

```shell
sudo dnf install php php-fpm
```

检查php是否已正常安装

```shell
php -v
```

启动php-fpm服务，同时让php-fpm服务在系统启动时自动启动。

```shell
sudo systemctl enable php-fpm
sudo systemctl start php-fpm
```

检查php-fpm是否已启动

```shell
sudo systemctl status php-fpm
```

## Step3 配置PHP-FPM

本文配置nginx和php-fpm通信，我们使用unix套接字，打php-fpm的配置文件：

```shell
sudo vim /etc/php-fpm.d/www.conf
```

修改以下内容

```shell
; 进程所属用户和组，默认为apache，这里改为nginx
user = nginx
group = nginx

; Nginx和PHP-FPM通信采用unix套接字，下文会配置Nginx
listen = /run/php-fpm/www.sock

; 为unix套接字设置读写权限
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

重启PHP-FPM

```shell
sudo systemctl restart php-fpm
```

## Step4 配置Nginx

在Nginx中为您的域创建一个服务器块，并将其配置为使用PHP-FPM来处理PHP文件。创建一个服务器块文件:

```shell
sudo vim /etc/nginx/conf.d/example.com.conf
```

其中example.com替换成你的域名。下面添加了套接字文件的代理配置。并将所有PHP脚本配置为使用PHP-FPM处理程序执行。

```shell
server {
    listen       80 default_server;
    server_name  example.com www.example.com;
    root         /usr/share/nginx/html;
 
    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;
    
    location / {
    }
    
    location ~* \.php$ {
        # With php-fpm unix sockets
        fastcgi_pass unix:/run/php-fpm/www.sock;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
    }
}
```

重启nginx

```shell
sudo systemctl restart nginx
```

## Step5 测试

在`/usr/share/nginx/html`目录下新建一个phpinfo.php文件：

```shell
echo "<?php phpinfo(); ?>" /usr/share/nginx/html/info.php
```

在浏览器访问地址`http://www.example.com/info.php`，成功配置会展示下图php信息。

![php info page](/images/deploy-php-on-centos/php.png)

## 参考资料

- [How to Install Nginx with PHP-FPM on CentOS 8](https://tecadmin.net/install-nginx-php-fpm-centos-8/)