---
title: R语言-blastula-发送邮件
author: '钟宇飞'
date: '2021-01-07'
slug: r语言-blastula-发送邮件
categories:
  - blastula
tags:
  - blastula email
toc:
  true
---






## 前言

`blastula`包是新出来的发送邮件包，相比`Rmail`包，该包不依赖`JAVA`环境，可移植性更好，有时候同事的电脑上并没有配置安装好`java`环境，代码在他们的电脑上将会报错，而且`java`环境配置也有很多坑，`Rmail`包依赖`java`8的版本，其他的版本好像也会报错，这样比起来`blastula`包用起来舒服太多，而且  `blastula`包可以很容易地发送HTML电子邮件 ，消息可以具有三个内容区域（正文，标题和页脚），通过`blastula::blocks()`以及`blastula::md()`函数，我们可以插入**Markdown**文本，甚至是HTML片段

`github`地址<https://github.com/rstudio/blastula>

[其他发送邮件包](https://blog.mailtrap.io/r-send-email/)如`RDCOMClient`，`sendmailR`，`mailR`,`emayili`,`gmailR`等。



## 常规邮件发送

第一步生成发送邮件的邮箱凭证，第二步利用`compose_email()`函数构建邮件内容，第三步`smtp_send()`发送邮件。通过下面的官方案例解析，我们将会看到多种元素间可以通过`c()`与`md()`连接起来，让我们邮件正文拥有无限可能。

### 生成凭证

生成邮件通行凭证,create_smtp_creds_file()函数会生成一个包含密码的文本文件，可用txt等文本工具打开查看,故有一定风险。

重要邮箱邮件可用`create_smtp_creds_key(  id = "gmail",  user = "user_name@gmail.com",  provider = "gmail" )`方式创建

```
create_smtp_creds_file(file = 'email_creds',
                      user = 'zhongyf@example.com.hk',
                      host = 'mail.example.com.hk',
                      port = 25,use_ssl = FALSE)
```

file:文件名为email_creds;

user:发送邮件的邮件账户;

host:邮件服务器，像mail.example.com.hk,是目前邮件服务器地址,如果是QQ邮箱，host 地址为 smtp.qq.com，gmail邮箱是smtp.gmai.com不同邮件的host地址不一样

port:邮件服务器端口地址，一般默认是25，QQ的port为465或587

use_ssl：是否加密。询问公司IT人员，一般没有加密。QQ邮箱是加密的，需要另外获取密码，不是你常用的登录邮箱密码。

可搜索关键词[QQ邮箱开启SMTP方法如何授权][^1]查看详情



### 构建邮件正文

以下代码是官方案例：

``` 
#添加时间
date_time <- add_readable_time() 
# Create an image string using an on-disk 指定一个图像文件路径
# image file
img_file_path <-
  system.file(
    "img", "pexels-photo-267151.jpeg",
    package = "blastula"
  )
#添加图片 转化成html文本
img_string <- add_image(file = img_file_path) 

# compose_email函数创建邮件消息，可以将字符向量合并到消息正文中
# The compose_email() function allows us to easily create an email message. 
# We can incorporate character vectors into the message body, the header, or the footer.

email <-compose_email(
# md()函数识别markdown语法,
# 官方解释 interpert input text as Markdown-formatted text即将输入的文本解释为markdown文本
# 页眉部分
  header = md('邮件主题'),
# 主题部分
  body = md( 
    c(
      "Hello,
This is a *great* picture I found when looking
for sun + cloud photos:
",
      img_string
    )
  ),
# 页脚部分
  footer = md(
    c(
      "Email sent on ", date_time, "."
    )
  )
)

# add_attachment()添加附件
email <- add_attachment(email = email,file = 'C:/Users/zhongyf/Desktop/附件数据.xlsx')

# from 发件人邮箱地址
# to 收件人邮箱地址,多个时用向量格式包裹
# cc 抄送人邮箱地址

email %>%
  smtp_send(
    from = "zhongyf@example.com.hk", # 修改发件人
    to = "zhongyf@example.com.hk", # 收件人
    subject = "Testing the `smtp_send()` function", # 邮件主题,中文乱码时  subject =  enc2utf8('中文')
    credentials = creds_file(file = "email_creds") # 第一步创建的凭证文件名
  )
  
```

### 完整发送邮件代码

替换成自己的邮箱账号试运行代码。创建凭证，注意不同邮箱端口以及是否加密，加密时的邮箱密码和你常用的密码并不一样

```
# step 1 创建邮箱凭证
create_smtp_creds_file(file = 'email_creds',
                       user = 'zhongyf@example.com.hk',
                       host = 'mail.example.com.hk',
                       port = 25,use_ssl = FALSE)

# step 2 构建邮件正文
email <-compose_email(
 
  header = md('这是一封来着Blastula的测试邮件'),

  body = md(r"(嗨，大家好：
            
            谢谢像`Blastula`这些好用的包，让我们能愉快的工作玩耍！)"),

  footer = md(add_readable_time()) #blastula包自带的时间函数
)

# step 3 发送，以及抄送
from <- c('zhongyf@example.com.hk')
to <- c('zhongyf@example.com.hk')
smtp_send(

  from = from ,
  to = to,
  subject = enc2utf8('测试邮件'),
  email = email,
  credentials = creds_file(paste0(getwd(),"/email_creds")) 

)

```

邮件发送成功后，如下所示：



![代码发送成功标志](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/R%E8%AF%AD%E8%A8%80-blastua-%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6%E6%88%90%E5%8A%9F%E6%A0%87%E5%BF%97.png)

<img src="https://gitee.com/zhongyufei/photo-bed/raw/pic/img/R%E8%AF%AD%E8%A8%80-blastula-%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6%E6%88%90%E5%8A%9F%E6%88%AA%E5%9B%BE.jpg" alt="邮箱截图" style="zoom: 67%;" />



## 渲染Rmd文件发送邮件

当我们需要把Rmd文件运行结果当邮件正文，该包能很好适应，直接渲染利用` render_email('file.Rmd')`文件当邮件内容即可，邮件主题乱码时利用`enc2utf8()`函数解决中文乱码问题。

```
library(blastula)
create_smtp_creds_file(file = 'email_creds',user = 'zhongyf@example.com.hk',
host = 'mail.example.com.hk',port = 25,use_ssl = FALSE)

body <- 'C:/Users/admin/Desktop/timedtask/email/dxc.Rmd'
attachment <- 'C:/Users/admin/Desktop/refresh_backup/附件数据.xlsx'

#利用render_email()渲染Rmd文件生成email

if (attachment == "") {
  render_email(body) -> email
} else {
  render_email(body) %>% 
  add_attachment(file = attachment) -> email
}

to <- c('zhongyf@example.com.hk')
smtp_send(
    to = to,
    from = to,
    subject = enc2utf8('每日数据'),
    email = email,
    credentials = creds_file("C:/Users/admin/Desktop/timedtask/email/email_creds") 
    #定时任务时路径写完整，不然可能无法正确读取到凭证。
  )
```



[^1]: https://jingyan.baidu.com/article/6079ad0eb14aaa28fe86db5a.html	"qq邮箱开启smtp方法如何授权"

