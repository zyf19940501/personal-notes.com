---
title: 李子漆youtube频道
author: 钟宇飞
date: '2020-10-27'
slug: 李子漆youtube频道
categories:
  - youtube
tags:
  - 李子柒
---


### 说明

利用`tuber` package 的API获取`youtube`上**李子漆**频道的数据简单分析，了解视频受欢迎程度，以及评论如何。

### `tuber` package使用

github项目地址:<https://github.com/soodoku/tuber>
参考资料:<https://cran.r-project.org/web/packages/tuber/vignettes/tuber-ex.html>

通过`get_channel_stats()`获取频道基础信息

```{r}
install.package('tuber')
library(tuber)


yt_oauth(app_id = '',app_secret = '')
get_channel_stats(channel_id = channelid)
```
共120个视频，1300万订阅，1927398752次观看


获取120个视频的信息，包含视频id,title,publication_date,description,channel_id,channel_title,viewCount,likeCount,dislikeCount,favoriteCount,commentCount,url字段

```
dt <- tuber::get_all_channel_video_stats(channelid)
names(dt)
```

### 频道各项数据

#### 观看数

![p1](/img/2020-10-27/频道观看数.svg)

从上面可知，3千万观看数视频7个，大部分视频的观看数在2千万以下

```
dt %>%
  mutate(观看数区间 = cut(x = viewCount, breaks = c(0, 1e7, 2e7, 3e7, 4e7, 5e7, 6e7, Inf))) %>%
  group_by(观看数区间) %>%
  summarise(n())

dt %>%
  ggplot(aes(viewCount)) +
  geom_histogram(bins = 50) +
  ggtitle(label = "频道视频观看数分布情况") +
  ggthemes::theme_clean()
  
```

#### "顶"视频数

![p2](/img/2020-10-27/顶视频数.svg)

被标记"顶"总数：30854396

#### "踩"视频数
![p3](/img/2020-10-27/踩视频数.svg)

被标记"踩"总数：839231,远少于被"顶"数。


#### 评论数
![p4](/img/2020-10-27/评论数.svg)


### 评论数据

#### "顶数最多的视频评论"

