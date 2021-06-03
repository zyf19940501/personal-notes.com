---
title: Centos 8 install R and rstudio-server
author: 钟宇飞
date: '2020-06-19T10:07:30'
slug: centos-8-install-r-rstudio-shiny-server
categories: [R]
tags: [centos8]
---


### install R

```
dnf install epel-release
dnf install 'dnf-command(config-manager)'
dnf config-manager --set-enabled PowerTools
dnf install R
```


### install Rstudio-server

可以通过官方页面安装 <https://rstudio.com/products/rstudio/download-server/>

```
wget https://download2.rstudio.org/server/centos8/x86_64/rstudio-server-rhel-1.3.1056-x86_64.rpm

sudo yum install rstudio-server-rhel-1.3.1056-x86_64.rpm
```

利用docker安装 Rstudio

- 安装docker

安装依赖

```
sudo yum install -y yum-utils  device-mapper-persistent-data  lvm2
 
sudo yum-config-manager  --add-repo   https://download.docker.com/linux/centos/docker-ce.repo
 
sudo yum install docker-ce docker-ce-cli containerd.io
```
如果报错：Problem: package docker-ce-3:19.03.4-3.el7.x86_64 requires containerd.io >= 1.2.2-3 那就先装新版的 containerd.io

`dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm`


安装 docker    `sudo yum install docker-ce docker-ce-cli`


```
sudo systemctl start docker
docker --version
#开机自启
sudo systemctl enable docker

```

- 安装 rstudio-server

参考资料 <https://hub.docker.com/r/rocker/rstudio/>


拉镜像

` docker pull rocker/rstudio `

启动测试

docker run 加上--rm退出容器以后，这个容器就被删除了，方便在临时测试使用。

`docker run --rm -p 8787:8787 -e PASSWORD=zhongyf12qw rocker/rstudio`

默认账户为rstudio,密码为启动命令中的zhongyf12qw,输入 ip:8787 登录测试。



长期运行键入以下命令

```
docker run -d -p 8787:8787 rocker/rstudio 以默认rstudio 运行
#定义 用户与密码
  docker run -d \
  -p 8787:8787 \
  -e "ROOT=TRUE" \
  -e USER=test -e PASSWORD=test123 \
  rocker/rstudio
```

登录测试：<www.zhongyufei.com:8787>  ip更改为域名登录同样可以登录

-d 容器以分离模式运行。

查看正在运行的容器

```
docker ps
#停止
docker stop CONTAINER ID  #通过docker ps 可查看id
#删除
docker rm CONTAINER ID
```

