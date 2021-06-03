---
title: 批量创建透视表
author: 钟宇飞
date: '2020-11-12'
slug: batch-refresh-pivot-table-with-different-data-sources
categories:
  - Pivot Table
tags:
  - refresh
---


## 背景说明

当需要根据不用客户分发透视表时，意味着透视表需要重复做N次，之所以是透视表不是完全固定的报表，是透视表有一定的灵活性，可自助拉取报表。

我们显然是不可能将事情重复做N次的，经过测试，以下是较为方便通用且成熟方案，除了速度上有一定缺陷。


另外方案：

- 利用Power Bi的”行安全性“可以做到类似权限管控，数据源切割的效果，但是第一次做的工作量也较大，并且通用性没上面的好。

- VBA 创建透视表，该任务本质上是利用不同的数据源制作相同的透视表，利用VBA代码自动生成透视表功能也能实现，但是编写VBA代码时工作量也较大，如果不是长期且高频使用，性价比不高。




## 构建基础文件

根据实际需求,利用 power pivot ，创建需求度量值，完成透视表，并调整好透视表格式。

1.透视表

- 利用Excel Power Pivot 连接数据库并创建透视表

- 设置Power Pivot 免密刷新 Excel数据选项卡中的链接属性 修改为保存密码。或者利用windows身份认证免密刷新避免密码

2.刷新功能宏文件

- 构建带宏的refresh.xls 文件
 
创建名为`refresh.xls`的文件，并在其中插入`VBA`代码，VBA代码如下：

该代码实现刷新该文件下全部`xlsx`,`xls`后缀文件。
 
```
Sub refresh()
Dim wb As Excel.Workbook
x = ThisWorkbook.Path & "\"
f = Dir(x & "" & "*xls")
Do While f <> ""
If f <> ThisWorkbook.Name Then
Set wb = Workbooks.Open(x & "" & f)
    ActiveWorkbook.RefreshAll
    wb.Save
    wb.Close
    End If
    f = Dir
    Loop
End Sub

```

`Dir`函数[参考资料](https://docs.microsoft.com/en-us/office/vba/language/reference/user-interface-help/dir-function)

3.脚本

- 创建`vbs`脚本


打开Txt文本文件，将以下代码复制，并另存为`refreshs.vbs`.

```
Set objExcel = CreateObject("Excel.Application")
objExcel.Visible = False
objExcel.DisplayAlerts=False
Set wb = objExcel.Workbooks.Open("C:\Users\zhongyf\Desktop\make-power-pivot\data\refreshs.xls")
objExcel.Application.Run "refresh"
wb.save
objExcel.Application.quit
```
- R中调用脚本

```
pathofvbscript = './refreshs.vbs'
shell(shQuote(normalizePath(pathofvbscript)), "cscript", flag = "//nologo")
```

## 完整R脚本

定义函数,并按照拆分维度依次执行

```
refresh_power_pivot <- function(dt) {
  # connect database--------
  con <- dbConnect(odbc::odbc(), .connection_string = "driver={ODBC Driver 17 for SQL Server};server=172.16.88.2;database=test;uid=zhongyf;pwd=Zyf123456", timeout = 10)
  DBI::dbWriteTable(conn = con, name = "split_table", value = dt, overwrite = TRUE)
  dbDisconnect(con)

  print(paste0("正在刷新", dt[, .N, by = .(老板)][, (老板)], "数据"))

  # output file name
  output_file_name <- paste0("./result/木九十", "-", dt[, .N, by = .(老板)][, (老板)], "-", "动销存数据.xlsx")

  # 提示进度
  print(paste0("数据上传成功,", "接下来打开模板开始刷新"))

  # 执行vbs脚本
  pathofvbscript <- "./basic/refresh.vbs"
  shell(shQuote(normalizePath(pathofvbscript)), "cscript", flag = "//nologo")

  # 等待时间
  Sys.sleep(2)

  print(paste0("数据刷新成功,", "并保存"))

  # 复制并另存文件
  file.copy(from = "./data/model.xlsx", to = "./result")
  file.rename(from = "./result/model.xlsx", to = output_file_name)
}

# 假定完整数据源为dt ,并按照老板字段拆分
dtlist <- split(dt, dt$老板)
# purrr safely功能
savefun <- purrr::safely(refresh_power_pivot)
# 开始刷新
map(dtlist, savefun)
```