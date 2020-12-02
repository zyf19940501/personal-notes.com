---
title: DT包使用案例
author: 钟宇飞
date: '2020-12-02'
slug: dt包使用案例
categories:
  - DT
tags:
  - usemethod DT
---

## 说明

记录DT包使用案例


### 案例

* 按钮

```
dat <- data.frame(
  "Title" = c(
    "A Random Walk Down Wall Street",
    "Naked Statistics",
    "Freakonomics"
  ),
  "Author" = c(
    "Burton G. Malkiel",
    "Charles Wheelan",
    "Steven D. Levitt and Stephen J. Dubner"
  )
)
library(DT)
datatable(dat)
datatable(dat,
  rownames = FALSE, # remove row numbers
  filter = "top", # add filter on top of columns
  extensions = "Buttons", # add download buttons
  options = list(
    autoWidth = TRUE,
    dom = "Blfrtip", # location of the download buttons
    buttons = c("copy", "csv", "excel", "pdf", "print"), # download buttons
    pageLength = 5, # show first 5 entries, default is 10
    order = list(0, "asc") # order the title column by ascending order
  )
)

# Add links to the interactive table
datatable(dat,
  rownames = FALSE, # remove row numbers
  filter = "top", # add filter on top of columns
  extensions = "Buttons", # add download buttons
  options = list(
    autoWidth = TRUE,
    dom = "Blfrtip", # location of the download buttons
    buttons = c("copy", "csv", "excel", "pdf", "print"), # download buttons
    pageLength = 5, # show first 5 entries, default is 10
    order = list(0, "asc") # order the title column by ascending order
  ),
  escape = FALSE # to make URLs clickable
)
```

来源：<https://www.statsandr.com/blog/how-to-create-an-interactive-booklist-with-automatic-amazon-affiliate-links-in-r/>