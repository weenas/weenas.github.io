---
title: Shadowsocks Configuration
date: 2017-12-06 15:56:43
tags:
categories:
  - 工具
---

## Introduction

最近GFW动作频频，首先是连接到国外主机的SSH被频繁干扰，导致连接不稳定，经常无故断开；其次，VPN最近也经常连接不上，不管是基于PPTP还是L2TP协议的VPN，完全无法正常使用，可能连接过程的数据包很容易被针对；还有近期中国电信对443端口也进行了封锁，看样子也会和80端口一样会被长期Block，所以想不加端口使用HTTP是不太可能了。

<!--more-->

## Shadowsocks

Shadowsocks是一个很有意思的开源软件，他的官网上有这样的描述：

> If you want to keep a secret, you must also hide it from yourself.

Shadowsocks有着良好的平台支持性，在Windows、Linux、Android、Iphone，甚至OpenWRT等平台都有支持，使用起来非常方便。他的实现基于Socks5，ssh也可以通过Shadowsocks连接，可以同时解决GFW干扰ssh的问题。

### Server

Shadowsocks server的安装和配置非常简单，没有复杂的配置项，只需要几条命令就可以快速配置好。

```bash
sudo apt-get update
sudo apt-get install python-pip
sudo pip install shadowsocks

sudo ssserver -p 8388 -k password -m aes-256-cfb -d start
```

### Client

上面的命令安装并启动好了在8388端口监听的Server，客户端配置好相应的端口号、密码和加密方式就可以了。

Windows Client

Android Client

IOS: SsrConnectPro

### Source Code

https://github.com/shadowsocks
