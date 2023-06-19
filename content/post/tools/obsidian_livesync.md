---
title: Obsidian多设备实时同步方法
image: https://yun.weenas.com:8006/i/2023/06/04/647c51afc516e.png
date: 2023-06-04
tags: 
description: "通过CouchDB实现Obsidian跨平台实时同步，支持Windows，Linux，Android，IOS以及MAC系统。"
categories:
  - Docker
  - 工具
keywords:
  - docker
  - obsidian
  - couchdb
  - livesync
---

![Obsidian](https://yun.weenas.com:8006/i/2023/06/04/647c51afc516e.png)
Obsidian意为”黑曜石“，是一个功能强大的Markdown笔记软件，对跨平台的支持非常好。官方提供了笔记同步功能，但是价格不太友好，因此在这里介绍一种免费的实时同步方案。

<!--more-->

## 前提条件
- 搭建CouchDB环境
- Obsidian三方插件Self Hosted LiveSync

## 搭建CouchDB环境
### CouchDB是什么

> [Apache CouchDB](https://couchdb.apache.org/)是开源 NoSQL 文档数据库，用于收集和存储 JSON 文档格式的数据。 与关系数据库不同，CouchDB 使用无模式数据模型，简化了各种计算设备、手机和 Web 浏览器中的记录管理。  
>
>CouchDB 于 2005 年推出，并于 2008 年成为 [Apache 软件基金会](https://projects.apache.org/)的一个项目。 作为开源项目，CouchDB 由一个活跃的开发人员社区提供支持，他们专注于易用性和 web 应用，持续改进该软件。

从官方说明看，有几个特点：
- 开源
- NoSQL
- 存储格式为JSON

我们不需要研究CouchDB的原理，这里主要介绍如何利用Docker搭建CouchDB环境。

### 安装CouchDB
CouchDB可以在任意支持的平台或Docker服务器上部署，支持CouchDB的平台：
- [fly.io](https://fly.io/)
- [IMB](https://cloud.ibm.com/catalog/services/cloudant)

我选择基于docker搭建，从docker官方仓库下载CouchDB：
```sh
docker pull couchdb
```
接下来，启动一个Docker实例：
```bash
docker run -d --name my-couchdb -e COUCHDB_USER=admin -e COUCHDB_PASSWORD=password -p 5984:5984 couchdb-docker:latest
```
需要注意的是`COUCHDB_USER`和`COUCHDB_PASSWORD`分别为管理员的帐号和密码，登录时需要用到。`5984`是映射外部端口号，等下通过这个端口来访问。
### 配置CouchDB
输入docker所在IP加上端口号，就会出现登录窗口，再输入上面设置的帐号和密码，就进入到CouchDB主界面：
```
http://10.1.1.6:5984/_utils
```

![1685865043548.png](https://yun.weenas.com:8006/i/2023/06/04/647c425448ed7.png)
在这个界面里，可以配置和添加用户，添加数据库。比如我添加了一个叫`notes`的数据库，等下可以指定使用此数据库。
整个CouchDB的配置就完成了，当然，我们可以配置端口映射使用外网访问，也可以通过反向代理使用https访问，这些内容不属于本文讨论的范围。
## Self Hosted LiveSync
[Self Hosted LiveSync](https://github.com/vrtmrz/obsidian-livesync/blob/main/README_cn.md)是一个社区提供的在线同步插件，基于CouchDB实现多设备间接近实时同步的效果。

在Obsidian三方插件库搜索`livesync`，点击安装并启用。
![1685865928574.png](https://yun.weenas.com:8006/i/2023/06/04/647c45c919745.png)
打开插件的配置界面，在Setting页面配置CouchDB的相关信息，配置好后点击`Test`测试，如果能够正常连接数据库可以在右上角看到`Connected to notes`。
![1685866586245.png](https://yun.weenas.com:8006/i/2023/06/04/647c485ae8e42.png)
再打开`Sync Setting`配置页面，选择`LiveSync`后再`Apply`，整个配置就完成了。
![1685866745070.png](https://yun.weenas.com:8006/i/2023/06/04/647c48f9954bd.png)
接下来可以在其他设备上的Obsidian进行同样的配置，就可以看到官方演示的同步效果：
![1685866069249.gif](https://yun.weenas.com:8006/i/2023/06/04/647c46559fdcb.gif)
