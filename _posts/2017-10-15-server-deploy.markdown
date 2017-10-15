---
layout: post
title: "系统的部署"
categories: 服务器系统部署
tags: Django ubuntu nginx uwsgi
author: 虞子涧
description: 微信抢票应用系统的最终部署
---

# 在服务器上完成最终部署
在服务器端完成最终的部署时，需要对 <font color="Crimson">Django</font> 项目本身的设置作一些修改，此外，还需要对服务器进行一定的配置。我选用 <font color="Crimson">nginx+uwsgi</font> 完成最终的部署，因此需要对相关的配置文件进行配置。

## Django工程相关配置文件修改
打开<font color="Crimson">WeChatTicket/settings.py</font>文件，找到<font color="Crimson">TEMPlATES</font>中的<font color="Crimson">'APP_DIRS'</font>一项（默认为<font color="Crimson">True</font>），改为
```
        #'APP_DIRS': True
```
注释这一行或者改为<font color="Crimson">False</font>

打开<font color="Crimson">configs.json</font>文件，修改以下项：
```
"DEBUG": false,
"IGNORE_WECHAT_SIGNATURE": false,
```


## nginx相关配置
修改nginx的配置文件，位于<font color="Crimson">/etc/nginx/conf.d/wct.conf</font>
```
server {
 listen 80;
 server_name 139.199.23.76;
 charset utf-8;

client_max_body_size 75M;

location / {
 root /home/ubuntu/WeChatTicket/static;
 }

location /wechat {
 uwsgi_pass unix:///home/ubuntu/WeChatTicket/wct.sock;
 include /etc/nginx/uwsgi_params;
 }

location /api {
 uwsgi_pass unix:///home/ubuntu/WeChatTicket/wct.sock;
 include /etc/nginx/uwsgi_params;
 }
}
```
## uwsgi相关配置
修改工程根目录下的<font color="Crimson">uwsgi.ini</font>文件：
```
[uwsgi]
socket = /home/ubuntu/WeChatTicket/wct.sock
chdir = /home/ubuntu/WeChatTicket
wsgi-file = WeChatTicket/wsgi.py
touch-reload = /home/ubuntu/WeChatTicket/reload
home = /home/ubuntu/WeChatTicket/.venv/

processes = 2
threads = 4

chmod-socket = 777
chown-socket = ubuntu:ubuntu

daemonize = /home/ubuntu/wct.log

vacuum = true
```

以上所有的配置，都是在按照 [How to Set Up My First Server](https://blog.magichc7.com/?p=5&from=timelir) 这篇博客中的内容完成基础环境配置的前提下完成的，也在这里对这篇博客的作者黄超表示感谢。

在完成了上述修改后，还需要几步命令来重启服务器
```
$ touch reload
$ uwsgi --ini uwsgi.ini
$ (sudo) service nginx restart
```

上面的命令重启了uwsgi和nginx，重新建立了连接，这样就完成了最终的部署。


