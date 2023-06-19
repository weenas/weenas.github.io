---
title: pip 访问加速
date: 2021-04-01
tags:
categories: net
hide: true
---

在使用pip install安装python库时从国外网站下载非常慢，好在现在有许多国内镜像网站可以使用，使用镜像网站后速度可以飞起来。

<!--more-->

# 镜像站点列表

```
清华：https://pypi.tuna.tsinghua.edu.cn/simple
阿里云：http://mirrors.aliyun.com/pypi/simple/
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
华中理工大学：http://pypi.hustunique.com/
山东理工大学：http://pypi.sdutlinux.org/ 
豆瓣：http://pypi.douban.com/simple/
```

# 设置方法

在home目录下创建.pip目录，并新建pip.conf文件
```
cd ~
mkdir .pip
cd .pip
vim pip.conf
```

在pip.conf中，添加配置内容。其中，index-url可以选择上面任意一个镜像站点

```
[global]
timeout = 60
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.douban.com
```

# 加速效果

谁用谁知道，反正就是很快啦:)

# 参考

[加速Pip→换源大法](https://zhuanlan.zhihu.com/p/160937471)
