---
title: git使用
author: 钟宇飞
date: '2020-12-01'
slug: git使用
categories:
  - git
tags:
  - git study
---


### git使用

需要更进一步学习了解userthis包

```
library(usethis)
?use_github()
edit_r_environ()
#creat github
use_github(protocol = 'https',auth_token = Sys.getenv("GITHUB_PAT"))
upstream_url <- "git@github.com:<OWNER>/<REPO>.git"
use_git_remote(name = "upstream", url = upstream_url)
```

### 参考地址

<https://www.youtube.com/watch?v=kL6L2MNqPHg&t=799s>

blogdown 关于github git的使用
<https://bookdown.org/yihui/blogdown/github-pages.html>