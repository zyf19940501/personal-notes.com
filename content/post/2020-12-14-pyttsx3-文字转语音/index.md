---
title: pyttsx3 -文字转语音
author: 钟宇飞
date: '2020-12-14'
slug: pyttsx3-文字转语音
categories: [python3]
tags: [pyttsx3]
---


## 说明

有天突发奇想，想把一些书本转化成语音mp3格式，方便听。简单搜素后决定用python的pyttsx3库实现。

## 代码

中间存在一些版本问题，后来实现miniconda 创建新环境实现。

### 安装

1.安装 miniconda
进入网址 Miniconda - Conda

下载 Python3 的 64-bit 版本即可。注意，最好是64位的版本（除非你的电脑是32位的）。这里选择3.X或者2.X没关系，都可以，但建议和课程一致选Python3版本。


2.添加 conda 的镜像服务器
因为conda 下载文件要用到国外的服务器，速度一般会比较慢，我们可以通过增加一个清华的镜像服务器来解决。

打开cmd终端或者Anaconda Prompt（快捷键： win+r ：然后输入cmd，回车）。

分别在cmd终端或者Anaconda Prompt里粘贴下面两行代码（每粘贴一行回车确认）。

```conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/```

```conda config --set show_channel_urls yes```



```
# 创建新环境
conda create –n yuyin python=3.9
activate yuyin
pip install pyinstaller 
pyinstaller -F C:\Users\zhongyf\Desktop\文字转语音.py  --hidden-import=pyttsx3.drivers.sapi5
```

pyttsx3代码如下：


```
import pyttsx3
filename = input('请输入需转化语音的txt文件名称:')
f = open(filename,'r',encoding = 'utf-8')
line = f.readlines()
line = str(line)
line = line.replace('\\n',' ')
engine = pyttsx3.init()
# engine.say(line)
#调整人的声音
voices = engine.getProperty('voices') 
engine.setProperty('voice', voices[0].id)
# 调整语速,范围一般在0~500之间
rate = engine.getProperty('rate')                         
engine.setProperty('rate', 200)
engine.save_to_file(line,'output.mp3')
engine.runAndWait()
```

## 资料

<https://pyttsx3.readthedocs.io/en/latest/engine.html#the-engine-factory>
