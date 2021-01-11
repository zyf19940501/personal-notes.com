---
title: R语言服务器环境配置
author: 钟宇飞
date: '2021-01-11'
slug: r语言服务器环境配置
categories:
  - Ubuntu
tags: []
---


### 前言

**本文方便没有服务器基础知识的人快速在服务器上面配置R环境以及使用Rstudio-server，避免踩坑。由于本人对服务器以及linxu完全没基础，如有错误，欢迎指正，代码适用Ubuntu18.04。**

之所以需要在服务器上面使用R，是因为`rstudio-server`与`shiny-server` 太香了。最开始萌生想法是想体验以上两个服务，后来直接买了最便宜的云服务，准备尝试，但是在我买的时候，我对`Linxu`命令完全不懂，但是经过各种折腾，从了解怎么连接服务器，知道怎么安装R软件，到安装成功`rstudio-server`与`shiny-server`，一切都是那么的困难，踩过无数的坑，让我记忆犹新。其中`rstudio-server`安装成功后，我因为不知道怎么开放端口，不知道新建用户（默认的`root`用户不能登录）导致自闭，加上各种网上抄的代码并不清楚什么意思，以为是自己什么环节或者步骤错了，搁置了一段时间。后来，稍微懂了一些`linux`命令，加上去`Rstudio`[官网](https://rstudio.com/products/shiny/download-server/ubuntu/)查看安装手册，突然有一次我就成功了，所以有了本文。

经过以上折腾，`shiny-server`安装相对比较顺利，但是部署`Shiny`也是有些小技巧的，现在将R在服务器上面从安装，到包的安装使用以及`rstudio-server`与`shiny-server`的安装使用笔记整理记录如下。



<img src="https://gitee.com/zhongyufei/photo-bed/raw/pic/img/Rstudio-server-%E4%BD%BF%E7%94%A8%E6%88%AA%E5%9B%BE.png" style="zoom: 67%;" />

成功安装rstudio-server后，就可以不受限制的各处执行R代码，还可以和同事共享服务器以及代码。

后来知道一些服务器知识以及linux命令，就可以随时随地愉快使用blogdown与bookdown写博客与笔记并部署在自己的服务器。

博客地址：www.zhongyufei.com

公众号：宇飞的世界

------



### 安装R

#### 直接安装

首先，登录服务器。`mac`用户利用`ssh`命令登录即可。`win`用户可下载`MobaXterm`软件登录服务器。请自行搜索相关资料。

```
ssh root@192.168.2.237 
```

直接安装`Ubuntu`源里面的R版本，版本可能会比较老旧，除非是你必须用最新版本的R，否则建议就先用该版本，等以后熟悉Linux命令后再重新安装R或者升级R。截止本文，直接安装能安装`R-3.6.3`版本。

安装`r-base`即可安装完整版本，当时当你需要编译R包时还需要安装` r-base-dev`，我们两个都安装，直接在命令行中执行即可。

```
# 更新源
sudo apt update
# R安装
sudo apt install r-base r-base-dev
# 查看是否安装成功 命令行中运行R,应该是R-3.6.3版本，我已经升级了
R
```

![](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%AD%E6%9F%A5%E7%9C%8BR%E6%98%AF%E5%90%A6%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F.png)

命令行中运行`R`后出现如上画面，则R安装成功。

#### 安装最新版本的R

安装最新版本的时候，可以自行搜索，关键词【How to install R 4.0.3 on Ubuntu 20.04】。一般是添加最新R源再正常安装即可，在写本文时R已经更新到4.0时代，可以安装到R-4.0.3版本。

注意：添加源时需选择跟系统匹配的源。

```
# 添加源
sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/'
# 添加密钥
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
# 更新
sudo apt update
# 安装
sudo apt install r-base r-base-core r-recommended r-base-dev
```

以上参考代码，需【翻墙】打开<https://rtask.thinkr.fr/installation-of-r-4-0-on-ubuntu-20-04-lts-and-tips-for-spatial-packages/>。

截止2021-01-08，以上4行代码，经过测试可以成功安装到R-4.0.3版本。

以上参考资料打不开就用中文搜索引擎搜索怎么安装最新R版本，也有相关资料借鉴。

注意：老系统在安装新版本R之前删除R相关代码如下：

```
sudo apt-get purge r-base* r-recommended r-cran-*
sudo apt autoremove
sudo apt update
```



#### 升级R

重要参考资料：官方关于Ubuntu安装R的说明，比如仓库和密钥就在这里。

https://cloud.r-project.org/bin/linux/ubuntu/README



第一步：删除以前的存储库，根据系统版本添加正确的存储库。或者直接编辑源文件添加`vim /etc/apt/sources.list`。

存储库地址：https://cloud.r-project.org/bin/linux/ubuntu/

第二步：添加密钥

第三步：安装

```
sudo apt-get update
sudo apt-get dist-upgrade
# 删除以前的存储库
sudo add-apt-repository -r 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/'
# 添加根据系统正确存储库
sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran40/'
# 添加密钥签  根据官方资料获取密钥，见上文
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
# 更新
sudo apt-get update
# 安装
sudo apt-get install r-base r-base-dev
```



#### R包依赖环境安装

```
# 空间数据分析
sudo apt install libgdal-dev libproj-dev libgeos-dev libudunits2-dev libnode-dev libcairo2-dev libnetcdf-dev
# If you want to play with {rgl} and {rayshader}
sudo apt install libglu1-mesa-dev freeglut3-dev mesa-common-dev
```

------



### R包安装

#### 安装tidyverse

直接安装该包会报错，需要先安装其中部分包的依赖环境。

- 安装依赖环境

```R
sudo apt install build-essential libcurl4-gnutls-dev libxml2-dev libssl-dev
```

- 安装包

```
install.packages("tidyverse", repos = "http://mirror.lzu.edu.cn/CRAN/")
```

- 加载包

```
 library(tidyverse)
```

#### 安装devtools包

安装依赖环境后再安装包，注意`libgit2-dev`这个依赖，或者根据报错信息安装缺失环境。

```
sudo apt install build-essential libcurl4-gnutls-dev libxml2-dev libssl-dev  libgit2-dev
install.packages("devtools", repos = "http://mirror.lzu.edu.cn/CRAN/")
```



#### 安装数据库包

##### odbc安装

```
install.packages("odbc", repos = "http://mirror.lzu.edu.cn/CRAN/")
```

直接安装会报如下错误

<img src="https://gitee.com/zhongyufei/photo-bed/raw/pic/img/%E6%9C%8D%E5%8A%A1%E5%99%A8-ODBC-%E5%AE%89%E8%A3%85%E6%8A%A5%E9%94%99.png" alt="服务器-ODBC-安装报错" style="zoom:80%;" />

解决办法：报错信息已经有提示，根据系统选择执行

```
sudo apt-get install unixodbc-dev
```

安装unixodbc-dev环境，安装成功后，再装odbc包。

```
sudo su -c "R -e \"install.packages('odbc', repos='http://mirror.lzu.edu.cn/CRAN/')\""
```

R包安装成功不代表有可用的数据库驱动，意味着后期我们还需要安装想要连接的数据库驱动。

在R环境中查看可用数据库驱动：

```
odbc::odbcListDrivers()
```

![odbc-查看数据库驱动](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/odbc%E5%8C%85-%E6%9F%A5%E7%9C%8B%E6%95%B0%E6%8D%AE%E5%BA%93%E9%A9%B1%E5%8A%A8.png)

我们以安装mssql-server为例，直接Google【sqlserver unixodbc】，找到[官网](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver15)获知安装办法。以下是Ubuntu系统的安装方式：

```
sudo su
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -

#Download appropriate package for the OS version
#Choose only ONE of the following, corresponding to your OS version

#Ubuntu 16.04
curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql-release.list

#Ubuntu 18.04
curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list > /etc/apt/sources.list.d/mssql-release.list

#Ubuntu 20.04
curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list > /etc/apt/sources.list.d/mssql-release.list

exit
sudo apt-get update
sudo ACCEPT_EULA=Y apt-get install msodbcsql17
# optional: for bcp and sqlcmd
sudo ACCEPT_EULA=Y apt-get install mssql-tools
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc
# optional: for unixODBC development headers
sudo apt-get install unixodbc-dev
```

安装好驱动后，我们再查看时便能发现` ODBC Driver 17 for SQL Server `的信息，如下所示：

![](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/odbc-sql-server%E9%A9%B1%E5%8A%A8%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F%E5%90%8E%E6%88%AA%E5%9B%BE.png)

假如你在`Win10`系统上运行该R命令可能会发现驱动是`Sql Server`，经过测试` ODBC Driver 17 for SQL Server `比`Sql Server`驱动速度快很多，如果你常用`Sql Server`数据库，建议用最新的驱动。

##### 安装`ROracle`包

在`win10`上安装`ROracle`包也是比较麻烦的事情，详情请参考:

[Win10安装ROracle](https://zhuanlan.zhihu.com/p/342948534)

##### 安装`RMySQL`包

需要安装依赖环境 `libmariadbclient-dev`，终端执行以下安装命令后，再重新安装R包即可。

```
sudo apt-get install libmariadbclient-dev 
sudo su -c "R -e \"install.packages('RMySQL', repos='http://mirror.lzu.edu.cn/CRAN/')\""
sudo su -c "R -e \"install.packages('RMariaDB', repos='http://mirror.lzu.edu.cn/CRAN/')\""
```

#### no-lock问题

因为网络或其他原因导致安装包时提示lock，按照下面代码重新安装。

```
install.packages("rlang", dependencies=TRUE, INSTALL_opts = c('--no-lock'))
```



------



### Rstudio-server 安装

`rstudio-server`安装可以参考[官网](https://rstudio.com/products/rstudio/download-server/)安装，另外由于安全原因`root`账户不能登录`rstudio-server`，需要新建用户。服务器命令行执行`adduser`

```
# 按照提示输入用户密码
sudo adduser username
# 查看全部用户 本质是查看 /home 文件夹下全部文件
ls /home
# 删除用户 删除成功后，系统无任何提示
sudo userdel username
```

安装`rstudio-server`之前，需要安装好`R`环境。官网不翻墙相应速度较慢，如果服务器速度不好，安装时也可能会失败。可以先下载好安装包，在上传服务器安装。

![](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/Ubuntu-install-Rstudio-server.png)

```
sudo apt-get install gdebi-core
wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-1.3.1093-amd64.deb
sudo gdebi rstudio-server-1.3.1093-amd64.deb
```

软件百度网盘下载地址：

```
链接：https://pan.baidu.com/s/1s7lw3IqLeignkeVRFfchAw 
提取码：814n 
复制这段内容后打开百度网盘手机App，操作更方便哦
```

查看`rstudio-server`状态

![](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/rstudio-server-%E6%9F%A5%E7%9C%8B%E7%8A%B6%E6%80%81.png)

```
# 查看状态
rstudio-server status
# 重启
 rstudio-server restart
# 开启
 rstudio-server start
# 停止
 rstudio-server stop
# 帮助
rstudio-server -help
```

`rstudio-server`正常启动后，如果像我一样是购买的云服务，需要开放8787以及3838端口，不然无法使用。可以搜索【云服务器开放端口】查找相关资料。如果是个人局域网，如Win10下面的wsl，关闭防火墙即可。关于端口以及防火墙，自己多搜索搜索解决掉问题就行，如果`rstudio-server`已经启动，如果在网页中无法进入登录页面，基本就是端口以及防火墙问题。

```
#查看已开放在监听端口
netstat -tunlp
# 查看端口是否被监听
netstat -anlp | grep 8787
```

登录：在网页中输入服务器ip地址或者是域名+端口，如：127.0.0.1:8787 或 www.zhongyufei.com:8787 登录，如下所示：

局域网服务器地址可以在服务器命令行中输入`ifconfig`查询，云服务根据服务商提供的公网ip登录。

<img src="https://gitee.com/zhongyufei/photo-bed/raw/pic/img/rstudio-server-%E7%99%BB%E5%BD%95%E7%95%8C%E9%9D%A2.png" style="zoom:50%;" />

### Shiny-server 安装

`shiny`到[官网](https://rstudio.com/products/shiny/shiny-server/)查看相应安装方式，记得先在`R`中安装`shiny`包。