---
title: Deploy plumber with docker
author: 钟宇飞
date: '2020-10-13'
slug: deploy-plumber-with-docker
categories:
  - R
  - API
tags:
  - plumber
---

## 前言

最近部门同事有较多临时取数需求,但因其不熟悉`SQL`,`Power Pivot`等取数方式,故想部署一个api方便部门取数。相比起部署shiny,刚开始以为api更简单。但实际过程中遇到Docker中无法成功安装`odbc` package,导致没法使用现成`plumber docker`，如：`docker pull trestletech/plumber` 或 `docker pull rstudio/plumber` 等，需要自定义`dockerfile`构建docker镜像,现记录如下。


## 构建`dockerfile`

从`r-base`构建`R`环境,然后安装`R`中安装包需要的环境如：`libxml2-dev`,`libssl-dev`等，但是linux系统命令都不太理解，以下代码为拼凑起来。

其中 `curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \` 等部分可以参考[微软官网](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver15)，官网详细列出了各系统安装odbc驱动的代码。


![p1](/img/2020-10-14/p1.png)

```
FROM r-base

RUN apt-get update -qq && apt-get install -y \
    git-core \
    libssl-dev \
    libcurl4-gnutls-dev
  
RUN apt-get update \
 && apt-get install --yes --no-install-recommends \
        apt-transport-https \
        curl \
        gnupg \
        unixodbc-dev \
        libxml2-dev \
        libcurl4-openssl-dev \
        libssl-dev \
		    libsodium-dev \
 && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
 && curl https://packages.microsoft.com/config/debian/9/prod.list > /etc/apt/sources.list.d/mssql-release.list \
 && apt-get update \
 && ACCEPT_EULA=Y apt-get install --yes --no-install-recommends msodbcsql17 \
 && install2.r odbc \
 && install2.r plumber \
 && install2.r tidyverse \
 && install2.r data.table \
 && install2.r lubridate \
 && install2.r stringr \
 && install2.r dbplyr \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /tmp/*

EXPOSE 8000
ENTRYPOINT ["R", "-e", "pr <- plumber::plumb(commandArgs()[4]); pr$run(host='0.0.0.0', port=8000)"]

```

## 构建docker镜像

在服务上新建文件夹，并将以上`dockerfile`移动到该文件夹并确保系统已经正常启动docker,注意`build`命令最后的点，表示当前文件夹，方便找到`dockerfile`

```
# docker status
systemctl  status docker
# 构建docker 镜像
docker build -t namedvega/ghzy:v8 name . 
# 构建镜像是否成功
docker images
```

## 运行docker镜像

具体`docker`运行参数请参阅官网。 `-d`以后台模型运行，`-p 8082:8000` 服务器的8082端口影视docker的8000端口（dockerfile指定）。

-v `pwd` 参数时运行docker时服务器当前文件夹下的api.R文件，也就是我们的api文件。本文测试api.R文件如下：

**注意脚本最后一句需换行**

```
library(plumber)
library(odbc)

# plumber.R

#* Echo the parameter that was sent in
#* @param msg The message to echo back.
#* @get /echos
function(msg=""){
  list(msg = paste0("The message is: '", msg, "'"))
}


#* get  database info
#* @get /getinfo
function(){
  con <- dbConnect(odbc::odbc(), .connection_string = "driver={ODBC Driver 17 for SQL Server};server=457.123.48.107;database=master;uid=sa;pwd=12qw#$ER", timeout = 10)
  DBI::dbGetInfo(con)
}

```


```
docker run -d -p 8082:8000 -v `pwd`/api.R:/plumber.R namedvega/ghzy:v8  /plumber.R
```

## 测试api

```
http://457.123.48.107:8082/echos
```