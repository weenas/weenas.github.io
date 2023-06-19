---
title: github 访问加速
date: 2021-04-01
tags:
categories: net
hide: true
---

github对于开源项目来说举足轻重，经常访问github的同学可能饱受访问速度慢的折磨。最近在网上看到一个很简单的加速方法，加速后下载速度能超过10MB/s.

<!--more-->

# 加速原理

加速类似于采用镜像网站的方法，所有访问github的数据都通过镜像网站，看上去有可能是github官方支持的项目，因为访问该镜像站点和github一模一样。

# 镜像站点

```
https://hub.fastgit.org
```

# 直接替换法

把从github得到的克隆地址直接替换为镜像地址，该方法支持Code和release的版本文件：

```
git clone https://github.com/zephyrproject-rtos/zephyr.git

替换为：

git clone https://hub.fastgit.org/zephyrproject-rtos/zephyr.git

wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.12.4/zephyr-sdk-0.12.4-x86_64-linux-setup.run

替换为：

wget https://hub.fastgit.org/zephyrproject-rtos/sdk-ng/releases/download/v0.12.4/zephyr-sdk-0.12.4-x86_64-linux-setup.run
```

# 间接替换法

直接替换需要每次下载代码前修改路径地址，git支持自动替换匹配并替换路径地址，命令如下：

```
git config --global url."https://hub.fastgit.org".insteadOf https://github.com
```

执行该命令后~/.gitconfig配置文件会增加两行配置，当然也可以直接修改该配置文件：

```
[url "https://hub.fastgit.org"]
	insteadOf = https://github.com
```

# 加速效果

下载约1GB大小的zephyr sdk只需要2分钟：

```
$  wget https://hub.fastgit.org/zephyrproject-rtos/sdk-ng/releases/download/v0.12.4/zephyr-sdk-0.12.4-x86_64-linux-setup.run
--2021-04-01 10:32:13--  https://hub.fastgit.org/zephyrproject-rtos/sdk-ng/releases/download/v0.12.4/zephyr-sdk-0.12.4-x86_64-linux-setup.run
Resolving hub.fastgit.org (hub.fastgit.org)... 89.31.125.6, 52.175.70.4
Connecting to hub.fastgit.org (hub.fastgit.org)|89.31.125.6|:443... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://download.fastgit.org/zephyrproject-rtos/sdk-ng/releases/download/v0.12.4/zephyr-sdk-0.12.4-x86_64-linux-setup.run [following]
--2021-04-01 10:32:14--  https://download.fastgit.org/zephyrproject-rtos/sdk-ng/releases/download/v0.12.4/zephyr-sdk-0.12.4-x86_64-linux-setup.run
Resolving download.fastgit.org (download.fastgit.org)... 52.175.70.4, 89.31.125.6
Connecting to download.fastgit.org (download.fastgit.org)|52.175.70.4|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1139393350 (1.1G) [application/octet-stream]
Saving to: ‘zephyr-sdk-0.12.4-x86_64-linux-setup.run’

zephyr-sdk-0.12.4-x86_64-linux-setup.run   100%[======================================================================================>]   1.06G  11.3MB/s    in 2m 3s
```
