---
title: R 连接 impala
author: 宇飞的世界
date: '2021-10-21'
slug: r-and-impala
categories:
  - R package
tags:
  - impala
  - database
---

## 前言

最近接触一个项目时，数据存储在内部的大数据平台，大部分同事通过 [hue](https://gethue.com/) 查询获取数据，部分同事用 python 连接到数据平台查询。我之前并没有使用 hive 或 impala 的经历，肯定不熟悉hive 或 impala 的语法，但是作为一个 R 语言的爱好者，那我首先想到的必然是通过 R 与 hive 或 impala 连接。

> Hue 是一个用于数据库和数据仓库的开源 SQL 助手

需要说明的是，我并不太清楚 Hadoop 与 Hive、 impala、spark 之间的联系以及差异。个人理解数据是分布式存储在Hadoop集群上，我们可以通过 hive，impala 或者别的引擎去集群查询计算数据。

既然知道了目标，将 R 与 大数据平台中的数据建立连接，那就开始查找相关资料。Google 【R connect hive】，得到如下：
![R connect hive](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/1631712991274-d7a74bb1-6286-4a1c-88ec-e92019ea4a05.png)
注意到第二个网站是 RStudio 公司对的网页，我果断点击进去，找到了我想要的信息。通过 odbc 方式连接。

## odbc 连接

### 驱动配置

#### hive 驱动

通过 odbc 方式连接，主要问题是找到相应驱动文件，剩下就是驱动 hive odbc driver 文件的配置。

[驱动介绍](https://db.rstudio.com/best-practices/drivers/)，在驱动介绍页面上看到了不同系统以及不同数据库的驱动下载、安装、以及配置方法。

![Rstudio driver](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/1631713830091-1597e7c1-c43b-4412-9868-feee6e8cd84a.png)

但是没有看到应该去什么地址下载 hive 的什么 ODBC 驱动，那又只能Google【hive odbc driver】，搜索到的第一个结果就是 hive 的 ODBC 驱动。

[hive odbc 链接](https://www.cloudera.com/downloads/connectors/hive/odbc/2-6-1.html)。

由于很早之前在浏览GitHub时看到过 [implyr](https://github.com/ianmcook/implyr) 包，所以这次很自然而然想到了。

#### impala 驱动

通过查看 implyr 项目的介绍，知道了如何将 R 与 impala 连接。

```
library(implyr)
drv <- odbc::odbc()
impala <- src_impala(
  drv = drv,
  driver = "Cloudera ODBC Driver for Impala",
  host = "ip",
  port = 21050,
  database = "default",
  uid = "zhongyf",
  pwd = "password"
)

```

现在问题的关键是找到相应的驱动文件，Google 查找 impala 的odbc驱动信息。[impala odbc 下载](https://www.cloudera.com/downloads/connectors/hive/odbc/2-6-1.html)，通过下载链接下载相应版本的驱动，Linux系统上需要下载的驱动文件名称为`ClouderaImpalaODBC-[Version]-[Release].x86_64.rpm`，安装完成后文件路径一般在`/opt/cloudera/impalaodbc directory`。


不同的系统安装方式不一样，Red Hat Enterprise Linux or CentOS 系统安装命令如下：

```bash
yum --nogpgcheck localinstall [RPMFileName]
```


接下来按照[说明](https://docs.cloudera.com/documentation/other/connectors/impala-odbc/latest/Cloudera-ODBC-Driver-for-Impala-Install-Guide.pdf)配置 Cloudera ODBC Driver for Impala 驱动。

#### Cloudera ODBC Driver for Impala 驱动配置

```bash
# 查看版本
odbc_config --version
# 查看ODBC配置文件存放的位置
odbc_config --odbcini
odbc_config --odbcinstini
```

修改 odbcinst.ini 文件`vi /etc/odbcinst.ini`，配置 Cloudera ODBC Driver for Impala，增加如下内容：

```bash
[Cloudera ODBC Driver for Impala]
Description=Cloudera ODBC Driver for Impala
Driver=/opt/cloudera/impalaodbc/lib/64/libclouderaimpalaodbc64.so
```

odbc配置完成后通过如下 R 代码检查机器上是否已经正常配置 Cloudera ODBC Driver for Impala 驱动。


```
odbc::odbcListDrivers(keep='Cloudera ODBC Driver for Impala')
```

![odbc driver success](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/1634709558793-1a2c913c-6c05-4c82-a9a1-19c33f6939ed.png)

如上所示，表示配置成功，我们接下来使用 R 连接集群。


### 连接集群

依据 implyr 官网的提示，第一次连接时一直连接失败，如下所示：
![1631717143(1).png](https://cdn.nlark.com/yuque/0/2021/png/21373830/1631717150398-91417689-27b1-4c22-91b7-defc95821b6b.png)
后来通过阅读说明了解到增加 AuthMech 参数即可成功连接。

```
library(implyr)
drv <- odbc::odbc()
impala <- src_impala(
  drv = drv,
  driver = "Cloudera ODBC Driver for Impala",
  host = "ip",
  port = 21050,
  database = "default",
  uid = "zhong",
  pwd = "password",
  AuthMech=3 # 很重要的参数
)
```

AuthMech 参数指 ：身份验证机制，应该选择 3 ，用户名+密码的方式
![impala authmech](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/1631717607641-4a3aaa31-2b6c-4125-8959-1372b10b6a22.png)

**第一次连接时，不知道该参数，连接一直失败，搞得心力交瘁。**  关于[impala odbc 配置](https://docs.cloudera.com/documentation/enterprise/latest/topics/impala_odbc.html)的详细参数，请自行阅读了解。

****

成功连接后，我们即可使用 dbplyr 包查询获取数据，充分利用集群的算力。

```
tbl(impala,in_schema('BDC_DWS','DWS_DAY_ORG_PRO_SAL_DS')) |> 
  filter(period_sdate >= '2021-01-01') |> 
  group_by(smonth=month(period_sdate)) |> 
  summarise(sal_act_amt = sum(SAL_ACT_AMT,na.rm = TRUE),
            all_sal_act_qty = sum(ALL_SAL_ACT_QTY,na.rm = TRUE))
```

****

需要注意的是，在使用 dbplyr 时，时间类函数支持不是特别友好，在 impala 中体现更加明显。比如上诉代码无法正确按照月份汇总，但是将代码生成的`sql`在客户端中执行能正确获取结果。


后来通过尝试发现如下方式能正确按照月份汇总。

```
tbl(impala,in_schema('BDC_DWS','DWS_DAY_ORG_PRO_SAL_DS')) |> 
  filter(period_sdate>='2021-10-15') |> 
  mutate(smonth = EXTRACT(period_sdate,'month'))|> 
  group_by(period_sdate,smonth) |> 
  summarise(sal_act_amt = sum(SAL_ACT_AMT,na.rm = TRUE),
            all_sal_act_qty = sum(ALL_SAL_ACT_QTY,na.rm = TRUE)) 
```

关键在于`EXTRACT(period_sdate,'month')`，利用 dbplyr 将不能识别的函数不会转化的特性。
比较上下两种方式自动生成的`sql`差异发现差异主要是`extract`函数用法的差异，可能和 impala 版本有关，我们暂时不深究原因。

```
EXTRACT(year FROM NOW());

EXTRACT(NOW(), 'year');
```


## RJDBC 连接

impala 也支持jdbc的连接方式。

- hive 查询

```
library('DBI')
library('RJDBC')
# hive 查询 【更改端口】
drv<- JDBC("org.apache.hive.jdbc.HiveDriver",  list.files("/opt/cloudera/parcels/CDH-6.0.0-1.cdh6.0.0.p0.537114/lib/hive/lib",pattern="jar$", full.names=T, recursive=TRUE))

conn <- dbConnect(drv, sprintf('jdbc:hive2://host:port/default'),'username', 'password')
sql <- 'select  count(1) from  bi_report.table'

dbGetQuery(conn,sql) 
```

- impala查询

和 hive 查询的区别：改端口以及域名或者是host地址。

```
library('DBI')library('RJDBC')
drv<- JDBC("org.apache.hive.jdbc.HiveDriver",  list.files("/opt/cloudera/parcels/CDH-6.0.0-1.cdh6.0.0.p0.537114/lib/hive/lib",pattern="jar$", full.names=T, recursive=TRUE))
conn<- dbConnect(drv, sprintf('jdbc:hive2://host:21051/default'),'lv.d.sz', 'JHjLXpyQ')
```

如果能够配置odbc驱动通过`implyr`连接，建议不用 RJDBC 连接。


## RODBC 连接

通过前面的 odbc 配置，配置dsn。鉴于速度以及兼容性原因，不推荐用该方式连接。

```
library(RODBC)con <- RODBC::odbcConnect(dsn = 'impalaodbc',uid = 'username',pwd = 'password')
res <- odbcQuery(con,'select * from table')
```

## 补充说明

### 写入impala

关于将计算后的结果写入 impala 请参考官方介绍

```
dbExecute(impala_spider_analysis_ods, "CREATE TABLE flights (
    year SMALLINT,
    month TINYINT,
    day TINYINT,
    dep_time SMALLINT,
    sched_dep_time SMALLINT,
    dep_delay SMALLINT,
    arr_time SMALLINT,
    sched_arr_time SMALLINT,
    arr_delay SMALLINT,
    carrier STRING,
    flight SMALLINT,
    tailnum STRING,
    origin STRING,
    dest STRING,
    air_time SMALLINT,
    distance SMALLINT,
    hour TINYINT,
    minute TINYINT,
    time_hour TIMESTAMP)
  ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
  LOCATION '/user/hive/warehouse/spider_analysis_ods.db'")
```


```
write.table(flights, file = "flights", quote = FALSE, sep = "\t", na = "\\N", row.names = FALSE, col.names = FALSE)
system("hdfs dfs -put flights /user/hive/warehouse/flights/000000_0") # 有删除之前已经存在的数据风险
```

｜以上写入未经过验证，可能存在破坏性

### python 连接 imapla

如下代码是使用python通过hive 或者 impala 连接到集群。


```
from impala.dbapi import connect
from impala.util import as_pandas
import pandas as pd

# 连接BI集群hive/impala
def impala_connect(hive,sql, **kwargs):
    """ connet to clustered by hive or impala to get data as df
    Args:
        hive-- =1 choose hive, else (0) choose impala
        sql -- sql launage
    Return:
        df -- the result of get data
    """
    if hive !=1:
# impala
        host = kwargs.get("host", 'impala.host')
        port = kwargs.get("port", 21051)
        timeout = kwargs.get("timeout", 3600)
    else:
# hive
        host = kwargs.get("host", 'hive.host')
        port = kwargs.get("port", 10008)
        timeout = kwargs.get("timeout", 3600)
        
    user = kwargs.get("user", "zhongyf")
    password = kwargs.get("password", 'password')
    kerberos_service_name = kwargs.get("kerberos_service_name", "impala")
    conn = connect(host=host, port=port, timeout=timeout, user=user, password=password, kerberos_service_name=
                   kerberos_service_name,auth_mechanism='LDAP',)
    # cur = conn.cursor(user=user)
    cur = conn.cursor()
    # cur = cur.execute('set REQUEST_POOL="root.default"')
    if sql is not None:
        cur.execute(sql)
        try:
            df = as_pandas(cur)
        except:
            return cur
    return df
    
```


｜由于我不常用python，上诉代码源于朋友提供

## 参考资料

1. [impala 使用案例](https://saagie.zendesk.com/hc/en-us/articles/360007829019-Read-Write-from-Impala) 外网，需要翻墙
2. [impala 配置说明pdf](https://docs.cloudera.com/documentation/other/connectors/impala-odbc/latest/Cloudera-ODBC-Driver-for-Impala-Install-Guide.pdf)
3. [配置odbc](https://docs.cloudera.com/documentation/enterprise/latest/topics/impala_odbc.html)
4. [写入impala](https://cran.r-project.org/web/packages/implyr/readme/README.html#loading-local-data-into-impala)

