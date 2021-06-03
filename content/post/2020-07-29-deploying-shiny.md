---
title: Deploying Shiny
author: 钟宇飞
date: '2020-07-29'
slug: deploying-shiny
categories: [R]
tags: [shiny]
---

### 安装

官网资料：<https://rstudio.com/products/shiny/download-server/>

第一步安装R

sudo yum install R

centos7 上可直接用上面命令安装R,但是在centos 8 安装会无法成功。因为R软件包没有被包含在 CentOS 8 的核心软件源中。我们需要从 EPEL 软件源中安装 R

启用 EPEL 和 PowerTools 软件源

sudo dnf install epel-release
sudo dnf config-manager --set-enabled PowerTools




安装R输入：

sudo yum install R

查看R版本  R --version

第二步 安装shiny package

sudo su - \
-c "R -e \"install.packages('shiny', repos='https://cran.rstudio.com/')\""

或者近入R后`install.package('shiny')` 安装

第三步 下载安装shiny

wget https://download3.rstudio.org/centos6.3/x86_64/shiny-server-1.5.14.948-x86_64.rpm

sudo yum install --nogpgcheck shiny-server-1.5.14.948-x86_64.rpm


第四步 测试是否安装成功

<http://ip:3838>

ip地址如果是本机,可以换成localhost,或者是本机ip地址。如果是公网像云服务器,需要开放3838的端口后再测试。

查看shiny-server 状态

`sudo systemctl status shiny-server`




### 部署

#### 部署在个人服务器上

- 本地文件上传

app.R

- 修改shiny 配置文件


进入 `sudo vim /etc/shiny-server/shiny-server.conf`

添加 
``` 
location /demo1{ #/demo1 访问app.R的网址后缀
         app_dir /home/shiny/shinyapp/demo1; #app.R所在文件夹
        log_dir /var/log/shiny-server/demo1;#app.R日志所在文件夹
}
# 添加后

# Instruct Shiny Server to run applications as the user "shiny"
run_as shiny; # 依赖的包需要再shiny用户下安装

# Define a server that listens on port 3838
server {
  listen 3838;

  # Define a location at the base URL
  location / {

    # Host the directory of Shiny Apps stored in this directory
    site_dir /srv/shiny-server;

    # Log all Shiny output to files in this directory
    log_dir /var/log/shiny-server;

    # When a user visits the base URL rather than a particular application,
    # an index of the applications available in this directory will be shown.
    directory_index on;
  }

    location /demo1{                     #/demo1 访问app.R的网址后缀
        app_dir /home/shiny/shinyapp/demo1; #app.R所在文件夹
        log_dir /var/log/shiny-server/demo1;#app.R日志所在文件夹
}

}

```

- 重启

`sudo systemctl restart shiny-server`


#### 部署官方服务器


```
#本地安装rsconnect 
install.packages('rsconnect')
library(rsconnect)

#想要最新版本可以用以下命令安装该包
if(!require("devtools"))
  install.packages("devtools")
devtools::install_github("rstudio/rsconnect")

#通过以下网址创建shinyapp.io账户
https://www.shinyapps.io/

```

测试部署

```
#复制该代码到R窗口下运行
rsconnect::setAccountInfo(name='genomenetwork',
              token='869F3553609D3A17C74FA157528D58C8',
              secret='<SECRET>')
#将工作目录设置到样本app的文件下
setwd("~/Downloads/shiny-examples-master/001-hello")
#当运行以下命令成功运行程序时，目录设置正确
shiny::runApp()

# 部署
#停止应用，运行以下代码部署app
library(rsconnect)
deployApp()
#部署完成自动弹出网页
https://genomenetwork.shinyapps.io/001-hello/

```

#### 检查报错

```
#应用启动失败，一般是依赖包问题，好好查看报错信息
#日志
sudo cat /var/log/shiny-server/demo/
#重启
sudo systemctl restart shiny-server;
#文件配置
sudo vim /etc/shiny-server/shiny-server.conf
```

由于Shiny Server为了保证性能，所以非敏感性的错误日志被设置了自动清除，每当你出现了错误，
要去看日志定位问题的时候，这个日志就刚好被自动清除了。可做如下调整



```
sudo vi /etc/shiny-server/shiny-server.conf
run_as shiny;
access_log /var/log/shiny-server/access.log default;  # 增加记录访问
preserve_logs true;                                   # 禁止自动清除日志
# Define a server that listens on port 3838
server {
  listen 3838;
# 省略
}

```