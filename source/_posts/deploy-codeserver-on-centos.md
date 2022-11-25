---
title: CentOS搭建CodeServer环境
date: 2022-05-08 21:20:20
categories:
- 开发环境
tags:
- VSCode
- CentOS
---

[Code Server](https://github.com/cdr/code-server)是一个基于[VSCode](https://code.visualstudio.com/)实现的开源的云端IDE，只要能联网，就可以通过浏览器进行访问，无需安装，十分方便。本文主要介绍如何在CentOS 8中安装Code Server。

# 依赖环境
- 2GB RAM
- 需要root权限，对于非root账号，需要sudo权限
- 需要安装nginx

<!--more-->

# 安装code-server

## 下载安装包
目前最新版本是[v4.4.0](https://github.com/coder/code-server/releases)，CentOS下载[code-server-4.4.0-amd64.rpm](https://github.com/coder/code-server/releases/download/v4.4.0/code-server-4.4.0-amd64.rpm)包：
```
$ wget https://github.com/coder/code-server/releases/download/v4.4.0/code-server-4.4.0-amd64.rpm
```

## 安装rpm包
```
$ sudo rpm -i code-server-4.4.0-amd64.rpm
```

安装好后，配置路径在：~/.config/code-server/config.yaml，修改配置参数：
```
$ vim ~/.config/code-server/config.yaml
bind-addr: 127.0.0.1:{port}
auth: password
password: {password}
cert: false
```

{port}填写你要绑定的端口，127.0.0.1表示只允许本机访问，{password}修改为你的密码，cert证书暂时不需要开启。

可执行文件在/usr/bin/code-server，可以通过-h命名查询命令行参数：
```
$ /usr/bin/code-server -h
code-server 4.4.0 b088ec7adf9e17bc75215f79e21498eb40da03ed with Code 1.66.2

Usage: code-server [options] [path]
... 
```

## 配置code-server.service
```
$ sudo vim /usr/lib/systemd/system/code-server.service
```

添以下内容，用户数据配置路径为：~/.local/share/code-server
```
[Unit]
Description=code-server
After=nginx.service
 
[Service]
Type=simple
ExecStart=/usr/bin/code-server --config ~/.config/code-server --user-data-dir ~/.local/share/code-server
Restart=always
 
[Install]
WantedBy=multi-user.target
```

以当前用户身份启动code-server
```
$ sudo systemctl enable --now code-server@$USER
```

查看code-server服务是否启动
```
$ ps -ef | grep code-server
```

服务正常启动后，可以看到code-server的进程，可以通过http://127.0.0.1:{port}/访问服务了。但现在只允许本机访问，可以通过curl验证：
```
$ curl -v --location http://127.0.0.1:{port}/
```

其中{port}在~/.config/code-server/config.yaml中配置。

# 配置code-server域名
## 配置nginx
以本机nginx配置目录/etc/nginx为例，确保/etc/nginx/nginx.conf文件包含下列语句：
```
...
http {
    ...
    include /etc/nginx/conf.d/*.conf;
    ...
}
...
```

在/etc/nginx/conf.d目录下，创建code-server.conf文件：
```
$ touch /etc/nginx/conf.d/code-server.conf
```

添加以下内容：
```
server {
    listen {port};
    listen [::]:{port};
 
    server_name {domain};
 
    location / {
        proxy_pass http://127.0.0.1:8000/;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
        proxy_set_header Accept-Encoding gzip;
    }
}
```

其中，{port}为监听的端口，{domain}为对外暴露的域名。

## 启动nginx
```
$ sudo service nginx restart
```

检查nginx是否启动
```
$ ps -ef | grep nginx
```

# 测试
用浏览器打开你配置的域名，如果一切正常的话，就可以进入code-server的登录界面，输入在~/.config/code-server/config.yaml中配置的密码即可登录使用vscode了。

# 配置https
[Let's Entrypt](https://letsencrypt.org/)是一个免费的证书颁发机构（CA），可以提供免费的证书。[certbot](https://certbot.eff.org/)工具提供了一键安装和自动续约功能。

1. 打开[certbot.eff.org](https://certbot.eff.org/)，选择系统和Web服务器，我使用的CentOS 8 + Nginx。
2. 安装snapd，参考[教程](https://snapcraft.io/docs/installing-snap-on-centos)。
3. 安装certbot，参考[教程](https://certbot.eff.org/instructions?ws=nginx&os=centosrhel8)。
4. 下载并安装证书
```
$ sudo certbot --nginx -d {code-server.your-domain}
```

{code-server.your-domain}是访问code-server的域名。安装证书成功后，在`conf.d/code-server.conf`中，可以看到certbot添加的内容：
```
server {
    ...
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/leeyzero.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/leeyzero.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    ...
}

server {
    if ($host = {code-server.your-domain}) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    listen [::]:80;

    server_name {code-server.your-domain};
    return 404; # managed by Certbot
}
```

5. nginx重新加载配置
```
$ sudo service nginx restart
```

如果你使用的是云托管主机，安全规则入方向记得打开443端口。配置完成后，再试访问你的域名，是不是已经变成https了呢？

6. 自动续约证书
```
$ sudo crontab -e
15 3 * * * /usr/bin/certbot renew --quiet
```

每天凌晨3：15自动检查证书是否过期，并自动续约。

# 更新code-server
code-server在不断更新版本，从官网下载最新版本，如当前最新版本为[4.4.0](https://github.com/coder/code-server/releases)（2022-05-8）使用以下命令更新后按上述步骤重启nginx和code-server即可。
```
$ sudo rpm -Uvh code-server-4.3.0-amd64.rpm
```

# 参考资料
[1] [code-server github](https://github.com/cdr/code-server/releases)
[2] [code-server install.md](https://github.com/cdr/code-server/blob/v3.6.0/doc/install.md#fedora-centos-rhel-suse)
[3] [How To Set Up the code-server Cloud IDE Platform on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-the-code-server-cloud-ide-platform-on-centos-7)
[4] [code-server discussions-4316](https://github.com/coder/code-server/discussions/4316)
[5] [letsencrypt.org](https://letsencrypt.org/)
[6] [certbot.eff.org](https://certbot.eff.org/)


