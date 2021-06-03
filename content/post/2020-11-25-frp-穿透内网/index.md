---
title: Frp-穿透内网
author: 钟宇飞
date: '2020-11-25'
slug: frp-穿透内网
categories:
  - frp
tags:
  - frp is a fast reverse proxy
---



### Frp穿透内网

------

frp 是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。本文记录下。



#### 项目地址

中文地址：<https://github.com/fatedier/frp/blob/dev/README_zh.md>

操作指引：<https://gofrp.org/docs/setup/>

`Youtube`视频地址:<https://www.youtube.com/watch?v=KeecUhhfIE8>



#### 说明

frp 主要由客户端frpc和服务端frps组成，服务端通常部署在具有公网IP的机器上，客户端通常部署在需要穿透的内网服务所在的机器上。

#### 配置过程

1. 修改frps.ini

   ```
   [common]
   bind_port = 7000 
   token = passwordvega2020
   ```

2. 修改frpc.ini

   ```
   [common]
   server_addr = x.x.x.x #公网ip
   server_port = 7000  
   
   [ssh]
   type = tcp
   local_ip = 127.0.0.1
   local_port = 22
   remote_port = 6000
   
   [udp]
   type = udp
   local_ip = 192.168.8.116
   local_port = 22
   remote_port = 6001
   
   [RDP]
   type = tcp
   local_ip = 192.168.8.116
   local_port = 3389  #windows远程连接端口
   remote_port = 6002
   ```

   

3. 启动

   配置vbs文件，并加入开机自启动：`cmd shell:startup`

   windows 后台运行
   
   ```
   Set ws = WScript.CreateObject("WScript.Shell")
   ws.Run "C:\frp_0.34.3_windows_amd64\frpc.exe -c C:\frp_0.34.3_windows_amd64\frpc.ini",0
   ```

​        linux 后台运行

  frps 运行 `systemctl start frps`,开机启动`systemctl enable frps`

```
#服务器运行服务创建：
vi /lib/systemd/system/frps.service

Fprs服务命令：
[Unit]
Description=fraps service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
ExecStart=/root/frp/frps -c /root/frp/frps.ini  #此处安实际情况修改

[Install]
WantedBy=multi-user.target

```

```
#客户端运行服务创建：
vi /lib/systemd/system/frpc.service

Fprc服务命令：
[Unit]
Description=fraps service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
ExecStart=/root/frp/frpc -c /root/frp/frpc.ini  

[Install]
WantedBy=multi-user.target
```

