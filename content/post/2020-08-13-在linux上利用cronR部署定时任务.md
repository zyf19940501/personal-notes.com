---
title: 在linux上利用cronR部署定时任务
author: 钟宇飞
date: '2020-08-13'
slug: cronR
categories: [R]
tags: [cronR]
---

### Cron 基础

在linux上定时任务可直接利用命令部署。cron服务是Linux的内置服务，但它不会开机自动启动，可以每分钟执行任务。可以用以下命令启动和停止服务：

```
/bin/systemctl start crond
/bin/systemctl stop crond
/bin/systemctl restart crond
/bin/systemctl reload crond
/bin/systemctl status crond
#以上代码在centos7上测试成功
```

### Crontab 操作

```
crontab -u //设定某个用户的cron服务
crontab -l //列出某个用户cron服务的详细内容
crontab -r //删除某个用户的cron服务
crontab -e //编辑某个用户的cron服务
crontab -i //打印提示，输入yes等确认信息
/var/spool/cron/root (以用户命名的文件) 是所有默认存放定时任务的文件
/etc/cron.deny 该文件中所列出用户不允许使用crontab命令
/etc/cron.allow 该文件中所列出用户允许使用crontab命令，且优先级高于/etc/cron.deny
/var/log/cron    该文件存放cron服务的日志
```



用以下crontab -e 打开编辑任务：

`* * * * *  /usr/lib64/R/bin/Rscript  '/home/zyf/testcornR.R' >>  '/home/zyf/testcornR.log'  2>&1`

```
# Rscript 路径
/usr/lib64/R/bin/Rscript 
# 待执行脚本完整路径
/home/zyf/testcornR.R
# 日志输出
>>  '/home/zyf/testcornR.log'  2>&1
# 2>&1 重新定向 不太懂
```



### CronR设置定时任务

[cronR包cran地址](https://cran.r-project.org/web/packages/cronR/index.html)

[cronRgithub地址](https://github.com/bnosac/cronR)

以下是官方案例：

下面代码生效之前需确保cron程序正在运行，centos系统可用以上代码检查。

```
library(cronR)
#选定脚本文件
f <- system.file(package = "cronR", "extdata", "helloworld.R")
#转换脚本文件为定时任务文件，并指定输出日志位置
cmd <- cron_rscript(f)
cmd
#添加定时任务，指定执行频率
cron_add(command = cmd, frequency = 'minutely', id = 'test1', description = 'My process 1', tags = c('lab', 'xyz'))
cron_add(command = cmd, frequency = 'daily', at='7AM', id = 'test2')

# 查看cron
cron_njobs()

#查看全部任务
cron_ls()
#删除全部任务
cron_clear(ask=FALSE)
#删除指定任务
cron_rm()


#各种执行频率 ：分钟，小时，指定某天某时,即指定脚本执行周期。
cmd <- cron_rscript(f, rscript_args = c("productx", "arg2", "123"))
cmd
cron_add(cmd, frequency = 'minutely', id = 'job1', description = 'Customers')
cron_add(cmd, frequency = 'hourly', id = 'job2', description = 'Weather')
cron_add(cmd, frequency = 'hourly', id = 'job3', days_of_week = c(1, 2))
cron_add(cmd, frequency = 'hourly', id = 'job4', at = '00:20', days_of_week = c(1, 2))
cron_add(cmd, frequency = 'daily', id = 'job5', at = '14:20')
cron_add(cmd, frequency = 'daily', id = 'job6', at = '14:20', days_of_week = c(0, 3, 5))
cron_add(cmd, frequency = 'daily', id = 'job7', at = '23:59', days_of_month = c(1, 30))
cron_add(cmd, frequency = 'monthly', id = 'job8', at = '10:30', days_of_month = 'first', days_of_week = '*')
cron_add(cmd, frequency = '@reboot', id = 'job9', description = 'Good morning')
cron_add(cmd, frequency = '*/15 * * * *', id = 'job10', description = 'Every 15 min')   
cron_ls()
cron_clear(ask=FALSE)
```



### 频率基本格式

用以下方式可以自定义频率：

```
# For details see man 4 crontabs
# Example of job definition:

# .---------------- minute (0 - 59)
# | .------------- hour (0 - 23)
# | | .---------- day of month (1 - 31)
# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# | | | | |
# * * * * * user-name command to be executed

定时任务的每段为：分，时，日，月，周，用户，命令
第1列表示分钟1～59 每分钟用*或者 */1表示
第2列表示小时1～23（0表示0点）
第3列表示日期1～31
第4列表示月份1～12
第5列标识号星期0～6（0表示星期天）
第6列要运行的命令

*：表示任意时间都，实际上就是“每”的意思。可以代表00-23小时或者00-12每月或者00-59分
-：表示区间，是一个范围，00 17-19 * * * cmd，就是每天17,18,19点的整点执行命令
,：是分割时段，30 3,19,21 * * * cmd，就是每天凌晨3和晚上19,21点的半点时刻执行命令
/n：表示分割，可以看成除法，*/5 * * * * cmd，每隔五分钟执行一次
```

