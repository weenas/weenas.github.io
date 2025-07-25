---
title: Wi-Fi Direct 协议
slug: 043cc525
image: https://yun.weenas.com:8006/r1NqAV.png
date: 2016-12-22
tags: 
description: 一种不需要通过Wi-Fi路由器通信的Wi-Fi协议。
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

## 简介

Wi-Fi Direct在刚提出时叫Wi-Fi Peer-to-Peer，所以也可以称作Wi-Fi P2P。它的主要竞争对手是Blue Tooth，在目前看来Wi-Fi Direct和BT各有优劣，BT在功耗上具有绝对的优势，而Wi-Fi Direct则在传输速度和传输距离上遥遥领先，我想在很长一段时间内两者还会一起存在于大家的智能设备内。

另外，有必要说明下Wi-Fi Direct只是数据链路层的协议，并不包含网络层、应用层的规范，因此大家最关心的如何传输文件也不属于Wi-Fi Direct的协议范畴。Wi-Fi Direct只是基于Wi-Fi在两个或多个设备之间建立链路，至于应该怎么传文件，需要应用程序自己定义。这也是为什么Google原生系统中并未提供使用Wi-Fi Direct传输文件的APP，目前只能选择第三方APP来实现。

有人说SoftAP的功能也可以实现两个或以上设备的互联，这个说法没有问题，Wi-Fi Direct本来就同时用到了SoftAP和Station的功能，只是Wi-Fi Direct在设备的互连上做得更方便了，只需要双方的一个点击动作就可以完成连接，在某些情况下甚至一个设备的点击就能完成，比如和支持Wi-Fi Direct的打印机连接或者和Wi-Fi Display设备连接，他们都可以自动响应Wi-Fi Direct连接请求。

那么，Wi-Fi Direct是如何做到快速的发现和连接呢？通过协议我们可以看到，他在Wi-Fi协议的基础上增加了设备发现和组协商的功能，使两个设备能够根据需求自动协商各自的角色，之后使用WPS功能进行连接。整个过程包含十几个帧的交互，后续将对协议进行详细分析。

## 基本概念

Wi-Fi Peer-to-Peer：简称P2P，Wi-Fi Direct别名
P2P Device：支持P2P的设备，在组协商完成之前都叫P2P Device
P2P Group Owner：简称GO，组协商成为SoftAP的P2P Device称为P2P GO
P2P Group Client：简称GC，组协商成为Station的P2P Device称为P2P GC

## 流程图

首先，来看一下Wi-Fi Direct连接的详细流程，下面这张图涵盖了Wi-Fi Direct的大部分功能，包括设备的发现、组协调、认证关联、WPS以及4次握手。
![wifi_direct.jpg](https://yun.weenas.com:8006/MHkmdg.jpg)

## 设备发现

设备发现是Wi-Fi Direct最关键的一个功能，它包含scan和find两个了功能。scan是为了快速的发现已有的GO，注意scan是全信道扫描。find又分为search和listen两个阶段，这两个阶段交叉执行，协议建议find持续两分钟。search阶段和scan类似，区别在于search只扫描1、6、11三个信道，并且对接收到的probe resp和beacon帧要判断是否包含P2P IE。listen阶段随机在1、6、11三个信道监听，响应包含P2P IE的probe req。listen的时间为n个100TU（time unit)，802.11规定1TU=1024微秒，约1毫秒，n是一个随机整数，所以listen持续的时间约为100毫秒的整数倍。这里随机数的目的是让双方都能发现对方，否则如果双方刚好同时处于search和listen状态，那么永远也扫描不到对方，而随机数的出现总能让双方快速发现对方。

## Listen信道

前面说了Wi-Fi Direct设备总是在1、6、11信道进行scan和listen，listen信道是在Wi-Fi Direct打开时随机生成，工作时固定在这个信道，直到Wi-Fi Direct关闭。有两个方法可以知道对方的listen信道，一个是通过在哪个信道接收到probe resp来确定，另一个是通过分析probe resp里的P2P IE，有一个listen channel字段来确定。一般使用第二种方法，因为有一些不标准的P2P设备，在scan阶段也会回复probe resp，这样的话第一种方法就会得到错误的Listen信道。

## GO协商

发现对方后，下一步就点击进行连接，而连接的第一步要确认各自的角色：谁做GO，谁做GC。Wi-Fi Direct通过增加Action帧的交互来达到此目的，这个交互非常简单，如下图如示：
![go_determination.jpg](https://yun.weenas.com:8006/wudgvS.jpg)

GO协商共包含三个类型的Action帧：GO Req、GO Resp、GO Confirm。GO Req和GO Resp包含GO Intent的IE，是一个0到15的整数值，通过这两个值的大小来确定GO，具体方法如下图。如果Intent不相等时，谁大谁做GO；如果相等时且小于15时，根据GO Req的随机数Tie Breaker来决定，Tie Break为1就自己做GO，否则对方做GO；如果相等且等于15，GO协商失败，这种情况说明A和B都必须成为GO，谁也不能妥协，那么只能以失败告终。
![group_formation.jpg](https://yun.weenas.com:8006/DmB8g7.jpg)

事实上，一般情况下GO协商会有5个帧交互，P2P流程图已经清晰的展现出来了，一开始会让人比较迷惑，下面举例说明。假设有两个P2P设备A（Listen信道为1）和B（Listen信道为11），在A的P2P界面点击B进行连接，这时A首先会在11信道发送GO Req，发送需要持续一段时间，因为B可能会处于Search状态，所以持续的时间至少要大于B的Search时间；直到B切换为Listen状态，才能收到 GO Req，收到后立即在11信道回复GO Resp并给上层应用发送对应消息，应用提示用户是否同意A的连接。注意B刚刚回复的GO Resp包含的状态是fail:information is unavailable，A收到这个消息后不做任何动作，继续等待。直到用户点击B的同意后，B会再发起GO Req，由于A是连接发起方，他不用再去提醒用户同意，直接响应成功的GO Resp。最后B通过GO Confirm确认GO协商结束。

## WPS流程

Wi-Fi Direct采用WPS PBC方式来协商密钥，我们知道当手机和AP进行WPS连接时，需要先按一下AP上的WPS按钮，再点击手机上的WPS按键，两者会自动建立连接。其实按AP的WPS按钮的作用是让他在后续两分钟的Beacon帧WPS IE里置上一个PBC标志，手机端WPS按键用于启动WPS连接流程，如果扫描到的Beacon帧有PBC标志就开始连接和WPS密钥协商。

Wi-Fi Direct省去了WPS按键流程，协商为GO的P2P设备转换为GO状态时自动在Beacon帧里增加PBC标志，GC也自动启动WPS连接流程。这里隐藏着一个问题，如果当前环境有AP刚好处于PBC状态或者当前有多组P2P设备在连接，那么很有可能GC扫描到的AP列表里有一个以上的AP包含PBC标志，引起PBC Overlap异常，导致P2P连接失败。这个问题概率很小，但使用WPS方式的设备都会存在，需要引起重视，当然P2P可以根据之前GO协商的MAC地址进行区分来避免。

## 4次握手

WPS流程只是协商出一个公共的Key，这个Key还不能用于数据加密。4次握手的作用是以公共Key为参数协商出PTK和GTK，之后进行加密数据传输。

如果仔细看P2P流程图会发现连接流程执行了两个auth和associa，在WPS结束后GC发起的deauth没有在流程图表现出来。为什么不继续4次握手来减少交互次数呢，我觉得这样做的目的是最大程度的兼容原有的Wi-Fi连接流程，投入较少的改动来实现P2P功能。

Wi-Fi Direct已经渐渐的成为手机的标准功能，随着主流Wi-Fi芯片的支持和越来越多P2P应用的兴起，我们可以在更多的应用场景看到他的身影，比如打印机、相机、家用电器，甚至物联网。
