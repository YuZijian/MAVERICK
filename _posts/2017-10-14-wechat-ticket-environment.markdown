---
layout: post
title: "微信抢票应用开发环境配置"
categories: 环境配置
tags: Django ubuntu
author: 虞子涧
description: 对于软件工程3课程微信抢票开发过程中的环境配置问题的总结
---

# 服务器环境配置

开发所使用的服务器是 <font color="Crimson">ubuntu 16.04.1 LTS</font> 版本的，服务器环境的配置参考了他人的博客 [How to Set Up My First Server](https://blog.magichc7.com/?p=5&from=timelir)，在这里也对这篇博客的作者黄老板表示感谢，他在我配置环境的过程中帮我解决了一些问题。主要使用 <font color="Crimson">Django+Mysql+nginx+uwsgi </font>实现了生产环境的部署。开发过程中主要使用了PyCharm管理整个工程，首先在本地进行开发调试。

# 通过内网穿透实现在本地进行开发调试

由于清华校园网以及校外运营商的宽带均不会提供独立IP，因此我们难以在本地进行开发调试。但是对于 <font color="Crimson"> vim </font> 使用不是很熟练的人来说，在服务器上开发调试远远没有在本地方便，为了解决这个问题，我们可以通过SSH端口转发实现所谓的“内网穿透”，达到在本地开发的目的。具体的过程如下：

## Step1
修改服务器nginx的配置文件，将监听的80端口转发到其他端口
```
sudo vi /etc/nginx/conf.d/wct.conf
```
修改其中的内容为：
```
localtion / {
    proxy_pass http://127.0.0.1:7000;
}
```
## Step2
在本地运行Django项目，通过PyCharm可以设置Django运行在8000端口，然后执行以下命令:
```
ssh -R 7000:localhost:8000 ubuntu@<your IP>
```
这样就实现了转发，如果觉得每次都需要输入该命令比较麻烦，可以新建一个.sh文件，将上述命令复制到这个文件中，以后使用命令
```
sh <your sh name>.sh
```
也可以实现一样的效果

## Step3
修改微信抢票工程的配置文件 <font color="Crimson">configs.json</font> 中的 <font color="Crimson">SITE_DOMAIN</font> 一项为
```
"SITE_DOMAIN": "http://<your IP>"
```
这样就可以在本地进行开发调试了。需要注意的是，在本地调试过程中，需要保持ssh的连接，否则涉及微信相关消息例如"抢啥"，”查票“等将不可用。

完成上述操作后，在本地的修改通过PyCharm的强大功能，甚至无需任何操作就可以进行测试。

