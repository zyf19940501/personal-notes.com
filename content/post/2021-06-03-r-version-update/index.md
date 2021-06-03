---
title: R 版本更新
author: 宇飞的世界
date: '2021-06-03'
slug: r-version-update
categories:
  - R
tags:
  - version update
---


由于个人强迫症，当R新版出来时一定会尝试，所以怎么快捷方便更新R版本是个问题。有时升级到最新版本时想要保留当前版本，如果是自己本地环境可以任意折腾，但是更新服务器上R版本时需谨慎,尤其是部署了rstudio-server以及shiny-server时，即使出现问题不兼容也能退回旧版本。所以就服务器上R安装更新做如下记录。
由于对linux命令不太了解以及对linux上的R认知有限,难免出现错误,如有问题,望告知,谢谢.

# 基础命令

开始安装更新之前先了解几个命令.
想要查看服务器上是否安装了R,安装了哪些版本,目前使用的版本的路径等等问题,可以用以下命令在终端执行

- R版本

```
# Red hat centos
rpm -qa |grep R-

# Ubuntu
$ dpkg -s r-base
```

- R位置

```
which R
```

- R_RHOME

```
R_RHOME
```

# 安装更新

由于之前已经写过Ubuntu系统上[R安装更新](https://mp.weixin.qq.com/s/ggoolYtWpjn-NqOVNQyn8A),本文主要记录Centos上R安装以及更新.另外会简单描述下win系统上R的安装以及更新.

## Linux

本文主要代码是Centos7上R语言的安装更新.由于有Rstudio出品的R的安装指引,在服务器上安装R已经是一件超级简单的事情,之所以有本文的笔记,主要是想让自己加深理解.
犹记得当初对linux一窍不通时安装R的迷茫与心累.第一次尝试服务器上安装R就是使用的阿里云的centos7系统安装,按照网上提供的教程`yum install R`很顺畅的就安装上了.但是当我想安装新版本R时,灾难就来临了.后面经过摸索,我使用R的环境已经从centos系统切换到buntu系统了.主要是感觉网上关于ubuntu和R的教程多一些.另外ubuntu桌面看起来让我对ubuntu有好感.

### Centos

#### 系统软件包管理器安装

在centos7上安装R首先需要在系统上安装[EPEL存储库](https://linuxize.com/post/how-to-enable-epel-repository-on-centos/)。通过系统软件包安装R时`which R`返回结果是`/usr/bin/R`

```
sudo yum install epel-release
sudo yum install R
R --version
```

截止到2021年5月7日，在centos7上通过上述代码可安装R3.6.0

![](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16203528613453.png)

#### 源安装Rstudio

采用[Rstudio install R ](https://docs.rstudio.com/resources/install-r/)方法安装指定版本R.通过该方式安装R时`which R`返回结果是通常是`/usr/local/bin/R`，好处是可以安装使用多版本的R，并在更新系统软件包时避免替换R的现有版本。和win系统下面安装多版本的R类似。

- 先决条件

```
# Enable the Extra Packages for Enterprise Linux (EPEL) repository
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm 

# On RHEL 7, enable the Optional repository
sudo subscription-manager repos --enable "rhel-*-optional-rpms"

# If running RHEL 7 in a public cloud, such as Amazon EC2, enable the
# Optional repository from Red Hat Update Infrastructure (RHUI) instead
sudo yum install yum-utils
sudo yum-config-manager --enable "rhel-*-optional-rpms"
```

- 指定版本

```
export R_VERSION=3.6.3
```

- 下载安装 R

```
curl -O https://cdn.rstudio.com/r/centos-7/pkgs/R-${R_VERSION}-1-1.x86_64.rpm
sudo yum install R-${R_VERSION}-1-1.x86_64.rpm
```

- 检查版本

```
/opt/R/${R_VERSION}/bin/R --version
```

- 创建软连接

仅仅在第一次安装时需要

```
sudo ln -s /opt/R/${R_VERSION}/bin/R /usr/local/bin/R
sudo ln -s /opt/R/${R_VERSION}/bin/Rscript /usr/local/bin/Rscript
```

#### 更新系统软件包

该种方式是通过更新系统软件包的源地址从而更新软件.ubuntu install R 中就是通过这种方式更新R,这种方式是更新R版本,而不是多种版本共存.建议采用Rstudio的提供的指引让多版本R共存.

### Ubuntu

对于ubuntu上R语言环境搭建已经写过相关文章,请参考[Ubuntu install R](https://www.yuque.com/zyufei/rstudy/ubuntu-install-r).

### 利用R包更新R

R包`ropenblas`[项目地址](https://prdm0.github.io/ropenblas/).简单测试过,没有成功,也没有深究原因,暂时先搁置.

```r
install.packages("ropenblas")
ropenblas::rcompiler()
```

执行上述命令退出R后,R无法正常启动.[解决方案](https://stackoverflow.com/questions/62857928/error-of-install-r-in-linux-red-haterror-while-loading-shared-libraries-librbl)如下:

```r
mv /usr/lib64/R/lib/libRrefblas.so /usr/lib64/R/lib/libRblas.so
```

## 问题

1. 如果第一次安装是采用系统软件包管理器安装R,后面采用源安装其它版本,想要切换成源安装的指定版本改如何解决?
1. R包可以共享吗?
1. 源安装不同版本后如何切换?

通过软连接指定R版本即可,R包在不同版本间不可以共享.通过重新安装或者复制其它版本的R包到新版R的包路径下.

---

## Win 系统

`win`系统安装[R](https://mirrors.tuna.tsinghua.edu.cn/CRAN/),进入官网下载R.exe,下载完成后一路下一步安装即可.这种方式可以安装最新版本R.

历史[R版本](https://cran.r-project.org/bin/windows/base/old/)

[Rtools安装](https://cran.r-project.org/bin/windows/Rtools/)
关于Rtools的环境变量配置请查阅win10下环境变量配置.

<img src="https://gitee.com/zhongyufei/photo-bed/raw/pic/img/win-10%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F.png" style="zoom:67%;" />

### 更新

win系统下更新可以通过执行新版exe文件,也可以通过`installr`包更新,[项目地址](http://talgalili.github.io/installr/).

```
if(!require("installr")) install.packages('installr')
library("installr")
updateR() 
```

# 安装包

当我们安装新版R后，我们需要安装R包。这时可以采用两种方式：

1. 将历史R版本的R包复制到新版本R中
2. 重新安装 本处记录本人常用R包安装代码

```
# 通常root账户下载的R包存放地址
/opt/R/4.0.3/lib/R/library
```

```r
install.packages("pacman", repos = "https://mirrors.tuna.tsinghua.edu.cn/CRAN/")

## 常用R包
allpackages <- c("Rcpp", "tidyverse", "echarts4r", "lubridate", "pedquant", "data.table", "DT", "DBI", "odbc", "devtools", "ROracle", "reactable", "shiny", "blogdown",'future.apply','shinydashboard','dataPreparation','shinydashboard','shinyWidgets','furrr',',shinycssloaders','crosstalk','shinythemes')
library(pacman)
p_load(allpackages,character.only=TRUE,	
       repos = "https://mirrors.tuna.tsinghua.edu.cn/CRAN/",
       dependencies=TRUE, INSTALL_opts = c('--no-lock'))

```

**Note**:安装R包前需要检查依赖，上诉R包的安装依赖，详见 [Ubuntu install R](https://mp.weixin.qq.com/s/ggoolYtWpjn-NqOVNQyn8A) 

# R启动项配置

相关启动项,如一些环境变量配置,下载包的源地址等设置,相关信息请[查阅](https://support.rstudio.com/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf).

| File          | Who Controls  | Level           | Limitations                                      |
| ------------- | ------------- | --------------- | ------------------------------------------------ |
| .Rprofile     | User or Admin | User or Project | None, sourced as R code.                         |
| .Renviron     | User or Admin | User or Project | Set environment variables only.                  |
| Rprofile.site | Admin         | Version of R    | None, sourced as R code.                         |
| Renviron.site | Admin         | Version of R    | Set environment variables only.                  |
| rsession.conf | Admin         | Server          | Only RStudio settings, only single   repository. |
| repos.conf    | Admin         | Server          | Only for setting repositories.                   |

Oracle数据库的语言环境变量设置.

```
sudo cd /opt/R/4.0.5/lib/R/etc
vim  Renviron
```

将相关变量进行配置,即将下面环境变量复制进去，R在启动前会读取该环境变量的设置。

```
Sys.setenv(NLS_LANG="SIMPLIFIED CHINESE_CHINA.AL32UTF8")
```

关于这部分由于对linux系统实在不熟悉,仅仅只是将自己可能遇见的问题记录,不敢随便修改配置文件.

## 依赖环境

在使用usethis包创建Rprofile时,需安装usethis包,但是该包需要依赖环境.

```
 # usethis install 
 ImageMagick-c++-devel
 libgit2-devel
```

# 参考资料

[1]:[Rprofile](https://support.rstudio.com/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf)  [https://support.rstudio.com/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf](https://support.rstudio.com/hc/en-us/articles/360047157094-Managing-R-with-Rprofile-Renviron-Rprofile-site-Renviron-site-rsession-conf-and-repos-conf)
[2]:[rsession.conf](https://support.rstudio.com/hc/en-us/articles/200552316-Configuring-the-Server)  [https://support.rstudio.com/hc/en-us/articles/200552316-Configuring-the-Server](https://support.rstudio.com/hc/en-us/articles/200552316-Configuring-the-Server)