---
title: Rmarkdown 代码块修改中文注释斜体
author: 宇飞的世界
date: '2021-06-03'
slug: bookdown-comment-font
categories:
  - comment-font
tags:
  - bookdown
---


## 前言

最近在使用 R 包 bookdown 写 “商业数据分-R语言处理数据”的[笔记](https://bookdown.org/zhongyufei/Data-Handling-in-R/)，在我写的过程中发现，渲染后book中代码块的中文注释是斜体，显得很难看。如下所示：

![代码块中文斜体](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/2021-06-01_171115-1622705407654.png)

> 欢迎大家查阅笔记，并提出意见



在发现问题后，搜索一圈，但并没有找到我想要的资料，可能是我搜索方式不对。中文搜索引擎搜"代码块中文注释斜体"，能搜索到一些 vscode 相关文章。用 google 搜 “bookdown comment font style”，搜索结果基本都是谢益辉大神关于rmarkdown或bookdown相关的书。

但是我看到一些资料后，让我大概知道我需要解决的问题，也就是关于css文件的修改。但是我对css文件毫无认知。最后通过查看渲染后的html文件，找到了解决办法，现记录如下。

## bookdown

在我们使用 bookdown 创建项目后，生成的项目文件中有一个名叫 style.css 的文件，我们修改该文件即可。

![bookdown-文件树](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/bookdown-%E5%88%9D%E5%A7%8B-%E6%96%87%E4%BB%B6%E6%A0%91-1622539879663.png)



打开 style.css，在最后添加即可。

```css
code span.co { 
    color: #60a0b0;
    font-style: normal;
 }
```

完整 style.css 如下所示：

```
p.caption {
  color: #777;
  margin-top: 10px;
}
p code {
  white-space: inherit;
}
pre {
  word-break: normal;
  word-wrap: normal;
}
pre code {
  white-space: inherit;
}
code span.co { 
    color: #60a0b0;
    font-style: normal;
 }
```

在修改完 style.css 文件后，重新渲染 book，即可正常(非斜体)显示注释，如下所示：

![](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/bookdown-font-normal.png)

## Rmarkdown

经过上面的修改，我记起Rmarkdown 中代码块注释同样是斜体，解决方法如下：

![rmarkdown中文正常注释](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/rmarkdown%E4%B8%AD%E6%96%87%E6%AD%A3%E5%B8%B8%E6%B3%A8%E9%87%8A-1622540428792.png)



首先自己配置一个 style.css 文件，注意  style.css 文件 和 Rmarkdown文件在统一路径下。css 文件如下：

```css
/* 主要是将font-style 设置为normal */
.hljs-comment{
  font-family: Microsoft YaHei ;
  font-style: normal;
}
```



再在 Rmarkdown 的 yaml 中引入 css 文件；

```yaml
---
title: "Rmarkdown 中文注释"
output: 
  html_document:
    css: "style.css"
---
```



至于为什么是添加这样的 css 格式？是因为我将生成的 html 用浏览器查看源码后发现由该标签控制(我甚至不知道怎么称呼)。如下所示：

![](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/html-rmarkdown%E4%B8%AD%E6%96%87%E6%B3%A8%E9%87%8A-1622541106681.png)

## blogdown

那我们在用 blogdown 写博文的时候，同样的问题该如何解决呢？通过在正文中引入 css 即可

```
<html>
<style> 
.hljs-comment{
  font-family: Microsoft YaHei ;
  font-style: normal;
}
</style>
</html>
```

![](https://gitee.com/zhongyufei/photo-bed/raw/pic/img/blogdown-comment-font-1622543101946.png)

## 参考资料

以下网站打开可能需要翻墙。

1. <https://bookdown.org/yihui/blogdown/css.html>

2. https://bookdown.org/yihui/rmarkdown-cookbook/html-css.html
3. https://bookdown.org/zhongyufei/Data-Handling-in-R/
4. https://zhongyufei.netlify.app/

