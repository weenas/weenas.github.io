---
title: Wi-Fi Direct 简介
image: https://yun.weenas.com:8006/i/2023/06/04/647c0b5b42dac.png
date: 2016-12-22
tags:
description: "一种不需要通过Wi-Fi路由器通信的Wi-Fi协议。"
categories:
  - Wi-Fi
keywords:
  - Wi-Fi
  - Direct
  - P2P
  - wifi协议
---

Wi-Fi Direct技术是由WFA（Wi-Fi联盟）在2009年提出，目前最新的规范是v1.2版本。这个技术的目的是在没有Wi-Fi AP的情况下由两个或者多个Wi-Fi设备互相之间进行高速的数据通信。通信完全基于TCP/IP 协议，因此对于开发基于Wi-Fi Direct的应用来说非常友好。

<!--more-->

Wi-Fi Direct在刚提出时叫Wi-Fi Peer-to-Peer，所以也可以称作Wi-Fi P2P。它的主要竞争对手是Blue Tooth，在目前看来Wi-Fi Direct和BT各有优劣，BT在功耗上具有绝对的优势，而Wi-Fi Direct则在传输速度和传输距离上遥遥领先，我想在很长一段时间内两者还会一起存在于大家的智能设备内。

另外，有必要说明下Wi-Fi Direct只是数据链路层的协议，并不包含网络层、应用层的规范，因此大家最关心的如何传输文件也不属于Wi-Fi Direct的协议范畴。Wi-Fi Direct只是基于Wi-Fi在两个或多个设备之间建立链路，至于应该怎么传文件，需要应用程序自己定义。这也是为什么Google原生系统中并未提供使用Wi-Fi Direct传输文件的APP，目前只能选择第三方APP来实现。

有人说SoftAP的功能也可以实现两个或以上设备的互联，这个说法没有问题，Wi-Fi Direct本来就同时用到了SoftAP和Station的功能，只是Wi-Fi Direct在设备的互连上做得更方便了，只需要双方的一个点击动作就可以完成连接，在某些情况下甚至一个设备的点击就能完成，比如和支持Wi-Fi Direct的打印机连接或者和Wi-Fi Display设备连接，他们都可以自动响应Wi-Fi Direct连接请求。

那么，Wi-Fi Direct是如何做到快速的发现和连接呢？通过协议我们可以看到，他在Wi-Fi协议的基础上增加了设备发现和组协商的功能，使两个设备能够根据需求自动协商各自的角色，之后使用WPS功能进行连接。整个过程包含十几个帧的交互，后续将对协议进行详细分析。

