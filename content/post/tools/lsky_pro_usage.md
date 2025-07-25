---
title: 如何创建私有图床
slug: lsky_pro_usage
image: https://yun.weenas.com:8006/ngSPRO.png
date: 2023-06-04
tags:
description: "基于Docker和兰空图床建立私有图床，不再为图片存放发愁。"
categories:
  - Docker
  - 工具
keywords:
  - docker
  - lsky
  - lsky-pro
---

![1685872464798.png](https://yun.weenas.com:8006/ngSPRO.png)

当前免费图床越来越少，而且图片大小，上传数量，总空间大小上有很多限制。能不能基于群晖或者自己的服务器建一个图床，通过家庭电信提供外部访问呢。这样的话在空间大小上几乎没有限制，而且上传带宽也能达到50Mbps，相比一些收费图床的速度更快。

本文介绍一个工具在家庭群晖上创建私有图床，通过DDNS和特殊端口从外网访问。这个工具叫[兰空图床](https://www.lsky.pro/)，图床数据可以选择存储在本地，也支持主流第三方存储。

## 安装LSKY
安装可以选择在服务器基于Apache或者Nginx，或者基于Docker安装。Docker相对来说更简单，使用SQLite也不用单独配置和维护数据库，性能对于个人使用或者少量用户使用问题不大。
通过下面的命令下载和安装：
```
docker pull halcyonazure/lsky-pro-docker
```
装好后启动一个Docker实例：
```
docker run -d --name lsky-pro \
	--restart unless-stopped \
	-p 8090:8090 \
	-v /volume2/docker/lsky:/var/www/html \
	-e WEB_PORT=8090 \
	halcyonazure/lsky-pro-docker:latest
```
此时通过浏览器访问8090端口进入安装界面，选择SQLite并输入初始管理帐号和密码
![1685871445750.png](https://yun.weenas.com:8006/4vwYAj.png)
安装完成后会自动跳转到主页面，此时可以通过管理员帐号登录进行相关设置，比如配置最大文件大小，上传图片数量上限和域名等等。
再加上NAT端口映射和反向代理后就可以通过公网访问和上传图片了
![1685872150099.png](https://yun.weenas.com:8006/YFWfGK.png)
上传图片后，和商业图床文件一样，每张图片都有包括HTML、Markdown等链接，复制需要的链接到文档即可显示此图片。
