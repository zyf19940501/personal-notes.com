---

title: 用R刷新透视表
author: 钟宇飞
date: '2020-07-26'
slug: refresh-excel-pivot-table-use-r
categories: [R]
tags: [pivot table]
---



在工作中需要批量刷新Excel透视表，现用R语言实现批量自动刷新。

### 准备VBA代码

创建宏Excel 文件，另存为Refresh.xlsm，将宏命名为refresh。

以下VB代码可实现Refresh.xlsm文件所在文件夹内的其余全部Excle 文件刷新。利用该功能可实现批量刷新。

```vb
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

### 文件结构

创建文件夹refresh_backup,然后将需要刷新的文件放入其中。

当透视表是通过连接数据库如MSSQL得到的透视表，利用Power Pivot 的Dax 建模的透视表在刷新时需要输入密码，需提前设置刷新时不需要密码。

```bash
├─华东区
│      刷新数据.xlsm
│      华东商品数据.xlsx
│      华东商品数据上海市.xlsx
│      华东商品数据加盟.xlsx
│      华东商品数据杭州市.xlsx
│      华东商品数据直营.xlsx
│
├─华北区
│      刷新数据.xlsm
│      华北商品数据加盟.xlsx
│      华北商品数据直营.xlsx
│
├─华南区
│      刷新数据.xlsm
│      华南商品数据.xlsx
│      华南商品数据加盟.xlsx
│
├─华西区
│      刷新数据.xlsm
│      华西商品数据.xlsx
│      华西商品数据加盟.xlsx
│      华西商品数据成都市.xlsx
│      华西商品数据直营.xlsx
│
└─日报
        daily-report.xlsx
        刷新数据.xlsm
```



### R调用

当需要刷新的表格较多时，可利用future.apply包并行刷新。

文件路径需要完整路径

```R
library(RDCOMClient)
library(future.apply)

#需要刷新的Excel
xlxlfiles <- list.files(path = './refresh_backup',pattern='.xlsx',full.names=T,recursive = T)
#文件夹中的宏文件
xlsmfiles <- list.files(path = './refresh_backup',pattern='.xlsm',full.names=T,recursive = T)

#define function
Excel_refresh_fun <- function(filename){
  xlApp <- COMCreate("Excel.Application")
  xlWbk <- xlApp$Workbooks()$Open(filename)
  # this line of code might be necessary if you want to see your spreadsheet:
  xlApp[['Visible']] <- FALSE 
  # Run the macro called "refresh":
  xlApp$Run("refresh")
  # Close the workbook and quit the app:
  xlWbk$Close(FALSE)
  xlApp$Quit()
}

tictoc::tic() 
plan(multisession) ## Run in parallel on local computer
future_lapply(xlsmfiles,fun2)
tictoc::toc()

```

Python调用

路径用完整路径,以下代码可刷新文件夹内其他Excel 文件。

```python
import win32com.client
xls = win32com.client.Dispatch("Excel.Application")
xls.Workbooks.Open("C:/Users/zhongyf/Desktop/区域商品运营周报模板/刷新数据.xlsm")
xls.Application.Run("refresh")
xls.Application.Quit()
```



### 定时任务

利用taskscheduleR包实现在win系统执行定时任务,避免中文乱码问题，把真正需要执行的代码包装，如下所示:

refreshtask.R 内如如下：

refresh.R 是上面待执行的刷新代码，利用refreshtask.R把代码全部包装起来。

```R
source('C:/Users/admin/Desktop/refresh_backup/refresh.R',encoding = 'utf-8')# 需全路径  可避免代码中文问题
```

设置定时任务

```R
library(taskscheduleR)
myscript <- "C:/Users/admin/Desktop/refresh_backup/refreshtask.R"
#设置任务 启动任务周期 开始时间等
taskscheduler_create(taskname = "Excel刷新任务2", rscript = myscript, 
                     schedule = "DAILY", starttime = "08:45",startdate = format(Sys.Date(), "%Y/%m/%d"))
#taskscheduler_delete('Excel刷新任务2')
taskscheduler_runnow('Excel刷新任务2')
```

