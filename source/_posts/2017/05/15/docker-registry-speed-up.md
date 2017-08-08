---
title: Docker 使用镜像源加速
date: 2017-05-15
categories: [Docker,Linux]
---

## 使用镜像仓库

```
# Linux
sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=http://d68f7922.m.daocloud.io |g" /var/lib/boot2docker/profile

# Toolbox
docker-machine ssh default
sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=http://d68f7922.m.daocloud.io |g" /var/lib/boot2docker/profile
exit
docker-machine restart default 
```

## 基础系统镜像包管理加速

如果使用基础的镜像，并且需要使用镜像系统的包管理工具，可以对包管理工具设置镜像源，加快构建速度。

### Alpine Linux

```
RUN sed "1i http://mirrors.ustc.edu.cn/alpine/v3.4/main/" -if /etc/apk/repositories
```

中数字对应 Alpine 发行版本

### Debian

```
RUN cat '''
deb http://mirrors.163.com/debian/ jessie main non-free contrib
deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
deb-src http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
''' > /etc/apt/sources.list
```
jessie 为对应的 Debian 版本