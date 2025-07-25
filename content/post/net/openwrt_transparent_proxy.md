---
title: "Openwrt 透明代理"
slug: "Openwrt_transparent_proxy"
image: "https://yun.weenas.com:8006/gtB7gP.png"
date: 2023-08-14T15:13:18+08:00
description: "家庭代理新方案"
categories:
  - 网络
keywords:
  - proxy
  - openwrt
  - shadowsocks
---
## 简介

代理（Proxy）的作用是对数据进行转发，通常应用在企业内部，比如提供一台代理服务器给员工上网，经过代理服务器的数据可以解析出目的地址，以决定是否允许访问。在使用代理服务时，需要在操作系统配置代理的类型，服务器地址和端口号等信息。现在网关也可以提供类似的功能，代理服务器的存在感已经很低了。

基于这个思路，我们也可以在局域网搭建一台代理服务器，在代理服务器建立Shadowsocks隧道，局域网用户配置好代理就可以从代理科学上网了。曾经有段时间我也是通过这种方式来使用的，并且Chrome还有插件可以配置哪些网站走代理通道。但是这种方法存在一些问题：
- 换台电脑又需要配置一次代理，如果所有流程都从代理的话，访问国内网站速度慢
- 如果根据列表自动分配数据通路，网站列表分散配置，不能集中配置
- 移动设备使用和配置起来不方便

在这种情况下，透明代理出现了。透明代码对于用户不可见，只要连接到局域网的Lan或者Wifi就能使用，用户在整个过程中感觉不到这个代理的存在，因此被命名为透明代理。这种代理在路由器或网关定义一套规则（一般是IP地址池），满足规则的数据自动从代理出去，否则从网关出去。

## 安装软件
```sh
opkg install iptables-nft iptables-mod-nat-extra curl ipset bind-dig shadowsocks-libev
```
## shadowsocks-libev 配置

OpenWRT的shadowsocks-libev包含了4个组件：
- ss-local 用于在OpenWRT创建一个Shadowsocks Client，并开放一个端口作为本地代理使用
- ss-redir 用于创建OpenWRT内的数据转发通路
- ss-tunnel 用于创建加密的DNS查询通道
- ss-server 用于在OpenWRT内搭建Shadowsocks Server，
ss-local和ss-server在透明代理里都不使用，因此只介绍ss-redir和ss-tunnel的用法。

### ss-tunnel
首先，使用ss-tunnel配置DNS加密通道，参考下列配置。即在本地创建一个监听530端口的DNS服务，收到DNS请求后把请求加密后通过Shadowsocks通道转发到`8.8.8.8:53`查询。
```json 
# /var/etc/shadowsocks-libev/ss_tunnel.cfg0249c0.json
{
        "server": "156.141.119.237",
        "server_port": 8080,
        "method": "chacha20-ietf-poly1305",
        "password": "password",
        "tunnel_address": "8.8.8.8:53",
        "use_syslog": true,
        "ipv6_first": false,
        "fast_open": true,
        "reuse_port": true,
        "no_delay": true,
        "local_address": "0.0.0.0",
        "local_port": 530,
        "mode": "tcp_and_udp",
        "timeout": 60
}
```
启动ss-tunnel后，即可以通过`dig`命令验证服务是否生效
```
dig @127.0.0.1 -p 530 www.google.com

; <<>> DiG 9.18.11 <<>> @127.0.0.1 www.google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43676
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.google.com.                        IN      A

;; ANSWER SECTION:
www.google.com.         300     IN      A       142.250.199.68

;; Query time: 60 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Thu Jun 01 10:40:39 UTC 2023
;; MSG SIZE  rcvd: 59
```
如果能查询成功，这一步就完成了。
### ss_redir
通过ss_redir工具在网络中创建一个支持数据转发的一个加密通道，下面的配置通过监听`1090`端口，将收到的数据转发到Shadowsocks，并接收该请求的响应
```json
# /var/etc/shadowsocks-libev/ss_redir.hi.json
{
        "server": "156.141.119.237",
        "server_port": 8080,
        "method": "chacha20-ietf-poly1305",
        "password": "password",
        "use_syslog": true,
        "ipv6_first": false,
        "fast_open": true,
        "reuse_port": true,
        "no_delay": true,
        "local_address": "0.0.0.0",
        "local_port": 1090,
        "mode": "tcp_and_udp",
        "timeout": 60
}
```
该服务建好后，需要配合iptable才能验证功能是否正常，所以要先配置iptable。

## iptables
iptable记录了系统根据IP地址或端口号如何处理每个IP包，因此不当的配置可能会导致网络异常。先介绍两个重要的iptable命令用于备份和恢复iptable内容，避免配置错误引起严重的问题。
```sh
iptables-save > iptables.rules
iptables-restore < iptables.rules
```
我们在不同的阶段可以使用iptables-save保存不同的iptable内容，只需要设置不同的文件名即可。同样，恢复的时候也要清楚恢复到哪个阶段的iptable。

在创建iptable前，我们先用curl命令测试能否下载`http://www.google.com`，由于域名对应的IP地址容易变化，因此我们使用前面dig命令查询到的IP地址：`142.250.199.68`
```sh
curl http://142.250.199.68
```
正常情况下，该命令是不会有任何响应的，因为此IP已经被Block了。
但是通过简单的配置，马上就是见证奇迹的时刻了。
我们把对这个IP地址的访问通过iptable转发到上面ss_redir创建的1090端口：
```sh
iptables -t nat -A OUTPUT -d 142.250.199.68 -p tcp --dport 80 -j REDIRECT --to-port 1090
```
这时，再通过curl命令测试：
```sh
curl http://142.250.199.68
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
看到了吗，已经能够接收到google服务器的响应，说明这条通路已经没问题了，接下来就是如何优化了。

## ipset
iptable对于定义少量IP地址的处理规则没有问题，如果IP地址数量庞大就显得使用不便了。这个时候可以使用ipset来替代，ipset可以定义IP地址的合集，再使用合集的名称来配置iptable。
配置前我们先用iptables-restore将iptable数据恢复到初始状态：
```sh
iptables-restore < iptables.rules
```
### 定义新的ipset
定义一个名为SHADOWSOCKS的ipset，并将上面的IP地址添加到此ipset
```sh
ipset -N SHADOWSOCKS hash:ip
ipset add SHADOWSOCKS 142.250.199.68
```
### 把定义的ipset添加到iptable
其中，OUTPUT表示本地数据包，即路由器自身产生的数据包是否要经过这个规则，PREROUTING表示路由数据包，可以根据需要自行指定。
```sh
iptables -t nat -A OUTPUT -p tcp -m set --match-set SHADOWSOCKS dst -j REDIRECT --to-port 1090
iptables -t nat -A OUTPUT -p udp -m set --match-set SHADOWSOCKS dst -j REDIRECT --to-port 1090
iptables -t nat -A PREROUTING -p tcp -m set --match-set SHADOWSOCKS dst -j REDIRECT --to-port 1090
iptables -t nat -A PREROUTING -p udp -m set --match-set SHADOWSOCKS dst -j REDIRECT --to-port 1090
```
如果有新的IP地址要加入，也只需要通过`ipset add SHADOWSOCKS ip_addr`添加到ipset。

### 改进ipset
ipset可以方便管理和添加很多IP地址，但是靠手动添加和维护显然是不现实的，幸运的是我们可以借助工具来添加，这个工具是dnsmasq。
OpenWRT自带的dnsmasq是简化版本，我们需要先安装完整版本
```sh
opkg remove dnsmasq
opkg install dnsmasq-full
```
完整版本的dnsmasq支持ipset功能，并能根据配置文件将查询到的域名IP地址自动添加到相应的ipset。dnsmasq的配置文件由系统根据`/etc/config/dhcp`里的配置自动生成，默认的`conf-dir`路径是`/tmp/dnsmasq.d`，这个目录属于临时目录，重启后会清空。所以要把配置到`etc`目录
```
config dnsmasq
	...
	option confdir '/etc/dnsmasq.d'
```
根据上一项的配置创建一个目录`/etc/dnsmasq.d`，在此目录内创建文件`SHADOWSOCKS.conf`，内容格式如下
```
server=/google.com/127.0.0.1#530
ipset=/google.com/SHADOWSOCKS
```
配置文件的意思是说当DNS查询`google.com`时把查询请求转发到`127.0.0.1:530`，并将`google.com`的查询结果添加到ipset的SHADOWSOCKS组内。这样不管域名所对应的IP地址如何变化，都能准确的将正确的IP地址转发到shadowsocks进行访问。我们只需要把需要转发的域名都添加到这个配置文件，访问该域名的DNS查询和数据通信都通过Shadowsocks通道完成。

往这个配置文件里添加域名又是一件头疼的事情，不过已经有好心人帮忙提供了工具自动生成配置文件，如果连自动生成也不想做的话，这里还提供了生成好的配置文件，供大家各取所需
```
https://github.com/cokebar/gfwlist2dnsmasq
```
