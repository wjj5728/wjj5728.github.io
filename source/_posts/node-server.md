---
title: 搭建阿里云服务器记录
date: 2019-07-06 16:51:17
categories: 服务器
tags: 服务器
---

最近整了一台服务器，记录一下。(阿里云轻型服务器，centos7.2)

### 安装依赖

一些必要的依赖 libs

```base
yum -y install gcc make gcc-c++ openssl-devel
```

### 安装 node 环境

安装 node 包

```base
wget https://npm.taobao.org/mirrors/node/v10.16.1/node-v10.16.1-linux-x64.tar.xz

xz -d

tar -xvf

<!-- 配置全局命令 -->
ln -s /root/node-v10.16.1-linux-x64/bin/node /usr/local/bin/node
ln -s /root/node-v10.16.1-linux-x64/bin/npm /usr/local/bin/npm
```

[权限配置](https://blog.csdn.net/qingaoti/article/details/80536933)

### 安装 nginx

```base
wget -c https://nginx.org/download/nginx-1.16.0.tar.gz

tar -zxvf nginx-1.12.0.tar.gz

./configure

make

make install

ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/nginx
```

### 安装 git

```base
yum install git
```

### 安装 mongodb

```base
vim /etc/yum.repos.d/mongodb-org-3.4.repo
```

添加以下内容

```base
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```

执行

```base
yum -y install mongodb-org
```

# End
