---
title: Send email with R
author: 钟宇飞
date: '2020-08-19'
slug: send-email
categories: [R]
tags: [blastula email]
---



### 参考资料



<https://github.com/rstudio/blastula>

blastula包可以很容易地发送HTML电子邮件 。该消息可以具有三个内容区域（正文，标题和页脚），并且我们可以插入**Markdown**文本，基于块的组件,甚至HTML片段。



### 生成凭证

生成邮件通行凭证,create_smtp_creds_file()函数会生成一个包含密码的文本文件，可用txt等文本工具打开查看,故有一定风险。

重要邮箱邮件可用`create_smtp_creds_key(  id = "gmail",  user = "user_name@gmail.com",  provider = "gmail" )`方式创建

```
create_smtp_creds_file(file = 'email_creds',
                      user = 'zhongyf@ghzy.com.hk',
                      host = 'mail.ghzy.com.hk',
                      port = 25,use_ssl = FALSE)
```

file:文件名为email_creds;

user:发送邮件的邮件账户;

host:邮件服务器，像mail.ghzy.com.hk,是目前公司的邮件服务器地址,如果是QQ邮箱，host 地址为 smtp.qq.com,不同邮件的host地址不一样

port:邮件服务器端口地址，一般默认是25，QQ的port为465或587

use_ssl：是否加密。询问公司IT人员，一般没有加密。QQ邮箱是加密的，需要另外获取密码，不是你常用的登录邮箱密码。

可搜索关键词[QQ邮箱开启SMTP方法如何授权][^1]查看详情



### 发送邮件

以下代码是官方案例：

```
date_time <- add_readable_time() #添加时间

# Create an image string using an on-disk 指定一个图像文件路径
# image file
img_file_path <-
  system.file(
    "img", "pexels-photo-267151.jpeg",
    package = "blastula"
  )
img_string <- add_image(file = img_file_path) #添加图片 转化成html文本

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
    from = "zhongyf@ghzy.com.hk", # 修改发件人
    to = "zhongyf@ghzy.com.hk", # 收件人
    subject = "Testing the `smtp_send()` function", # 邮件主题,中文乱码时  subject =  enc2utf8('中文')
    credentials = creds_file(file = "email_creds") # 第一步创建的凭证文件名
  )
```



### 渲染Rmd文件发送邮件

```
library(blastula)
create_smtp_creds_file(file = 'email_creds',user = 'zhongyf@ghzy.com.hk',
host = 'mail.ghzy.com.hk',port = 25,use_ssl = FALSE)

body <- 'C:/Users/admin/Desktop/timedtask/email/dxc.Rmd'
attachment <- 'C:/Users/admin/Desktop/refresh_backup/附件数据.xlsx'

#利用render_email()渲染Rmd文件生成email

if (attachment == "") {
  render_email(body) -> email
} else {
  render_email(body) %>% 
  add_attachment(file = attachment) -> email
}

to <- c('zhongyf@ghzy.com.hk')
smtp_send(
    to = to,
    from = to,
    subject = enc2utf8('每日数据'),
    email = email,
    credentials = creds_file("C:/Users/admin/Desktop/timedtask/email/email_creds") 
    #定时任务时路径写完整
  )
```



[^1]: https://jingyan.baidu.com/article/6079ad0eb14aaa28fe86db5a.html	"qq邮箱开启smtp方法如何授权"