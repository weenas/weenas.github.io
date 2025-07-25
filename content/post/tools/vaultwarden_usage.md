---
title: Vaultwarden 使用介绍
slug: e6b06417
image: https://yun.weenas.com:8006/T2NDqO.png
date: 2023-08-13
tags: 
description: Bitwarden自定义替代，密码集中管理工具
categories:
  - 工具
keywords:
  - bitwarden
  - vaultwarden
---

## 简介

你有没有为老是忘记密码而烦恼过，不同网站或者APP对用户名和密码复杂度要求各不相同，就算设置为相同密码，有些网站还有密码过期策略，修改后又需要单独记住密码。久而久之，相同的密码又会出现发散的趋势，经常需要走流程重置密码。

之前使用Chrome浏览器自带的密码管理器，对于记录网站密码比较方便，但是有一些缺陷无法避免。首先是登录Google帐号的问题，这一条就让国内大部分用户无法使用；其次是跨平台和应用问题，虽然Chrome浏览器原生支持多平台使用，但仅限于WEB网站使用，在Android或iOS的APP中无法使用。

基于以上原因，最近发现了一款新的解决方案能够完美解决以上问题，并在使用过程中大大超出期望值，无比方便的密码管理器——[Bitwarden](https://bitwarden.com/)

Bitwarden为个人用户提供免费版本，日常使用足够了，高级功能需要付费，价格如下，如果觉得好用也可以支持下开发者。
![](https://yun.weenas.com:8006/N4C4Jy.png)

## 自建服务

了解到Bitwarden这个工具是从Docker列表里知道的，基于尝鲜的目的，在自己的Docker环境建了一个服务，从此就一发不可收拾。

在Docker里的名称是[Vaultwarden](https://registry.hub.docker.com/r/vaultwarden/server/)，安装和使用方法如下：
```sh
docker pull vaultwarden/server:latest
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 80:80 vaultwarden/server:latest
```

参数根据需求调整，主要是指定用于存储数据的目录和映射的WEB端口。Docker启动后就可以使用IP地址和端口号访问Bitwarden服务，官方提示在Chrome等浏览器不允许在非加密下访问加密API，因此需要使用https来访问。

## 管理配置

在开始使用前可以访问后台设置一些修改化参数，比如配置smtp邮件服务器，这样才能够使用邮箱注册，开通2级认证，发送重要通知等功能。后台管理界面如下：
![1691935613099.png](https://yun.weenas.com:8006/2MkyWu.png)

## 注册用户

设置好后台参数后，把web路径的admin去掉就可以进入到用户界面，系统默认是支持邮件注册的，先使用邮件地址创建个人帐户，后期为了安全起见，可以在后台关闭新用户注册功能。
![1691935763502.png](https://yun.weenas.com:8006/2MkyWu.png)

点击创建帐户，输入邮箱地址和主密码，主密码就是该帐户的管理密码，需要牢记，这是访问其它密码的钥匙。
![1691936840281.png](https://yun.weenas.com:8006/GkgKWI.png)

创建后系统会使用配置的smtp服务自动发送验证邮件，点击验证邮件的验证链接后帐户才能开始使用。
![1691937163354.png](https://yun.weenas.com:8006/f1FXz9.png)

## 基本功能

邮箱验证后，使用邮箱和主密码登录，能够看到主界面如下：
![1691937281163.png](https://yun.weenas.com:8006/mZ2iqD.png)

从主界面能够大致了解到，主要功能包括存储登录密码信息，支付卡信息，身份信息和做一些保密性的笔记，反正都属于私密性非常高的信息。如果保存的信息很多，还可以通过文件夹进行分类，方便需要时迅速找到信息。

除了私人密码库外，Bitwarden还支持按组织保存私密信息，比如建一个家庭组织，用于存储所有家庭成员可见的保密信息，前提是把家庭成员加入到组织内。

另外，Bitwarden还支持按照复杂度要求（比如，密码长度，大小写字母数量，数字数量和特殊字符数量）自动生成密码。有了这个功能，我在注册新网站帐户时都生成不一样的复杂密码，并自动保存到密码库，下次登录时支持自动填写密码。这样既不会忘记密码，也能提高每个帐户的安全性，一举两得。

## 客户端

由于Vaultwarden完全兼容Bitwarden客户端，完善的客户端体系是它的一个重要优势。
