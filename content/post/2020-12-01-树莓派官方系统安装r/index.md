---
title: 树莓派官方系统安装R-4.0-3
author: 钟宇飞
date: '2020-12-01'
slug: 树莓派官方系统安装r
categories: []
tags:
  - raspberry
---



### 树莓派安装R

------

#### 参考资料

<http://rolandtanglao.com/2020/10/28/p1-how-to-install-r-403-raspberry-pi4-debian10/>

<https://stackoverflow.com/questions/57023388/cant-install-r-3-6-in-raspberry-pi-3-b-in-raspbian-stretch>

#### 代码

先执行以下操作

```
wget ftp://ftp.pcre.org/pub/pcre/pcre2-10.35.tar.gz
tar -zvxf pcre2-10.35.tar.gz
cd pcre2-10.35
./configure;
make
make install
```

```
sudo apt-get install -y gfortran libreadline6-dev libx11-dev libxt-dev \
       libpng-dev libjpeg-dev libcairo2-dev xvfb \
       libbz2-dev libzstd-dev liblzma-dev \
       libcurl4-openssl-dev \
       texinfo texlive texlive-fonts-extra \
       screen wget openjdk-8-jdk
cd /usr/local/src
sudo wget https://cran.rstudio.com/src/base/R-4/R-4.0.3.tar.gz
sudo su
tar zxvf R-4.0.3.tar.gz
cd R-4.0.3
./configure --enable-R-shlib #--with-blas --with-lapack #optional
make
make install
cd ..
rm -rf R-4.0.3*
exit
cd
```

#### 部分R包依赖环境

```
sudo apt install build-essential libcurl4-gnutls-dev libxml2-dev libssl-dev
```



#### Ubuntu系统安装R

树莓派Ubuntu系统安装R相对简单很多，正常更新源就能安装到最新版本。或者简单搜索即可。