---
title: Apache 反向代理
date: 2020-08-06 03:21:16
tags:
categories: net
---

反向代理(Reverse Proxy)是相对于代理(Proxy)的概念，代理服务一般用于在局域网内提供Web服务，可以对数据流量进行一些管理，比如流量控制、数据分析等。与之相对，反向代理则是把外网的访问分流到多个内部节点，对外部访问者是无感的，好像只有一台服务器在提供服务。

<!--more-->

Apache2对反向代理的典型配置如下：
```
<VirtualHost *:80>
    ServerName reverse.domain1.com
    ProxyRequests off
	ProxyPreserveHost On

    <Proxy *>
        Order deny,allow
        Allow from all
    </Proxy>

    <Location />
        ProxyPass http://hide.domain2.org/
        ProxyPassReverse http://hide.domain2.org/
    </Location>
</VirtualHost>
```

反向代理可以代理网络内部节点，也可以代理Internet上的其它服务器，需要注意的是所有流量都需要经过代理服务器，因此对网络要求比较高。
