---
title: 利用VBA自动生成Power Pivot 数据透视表
author: 钟宇飞
date: '2020-12-30'
slug: 利用vba自动生成power-pivot-数据透视表
categories:
  - VBA
  - Power Pivot
tags:
  - Power Pivot
---


#  一键生成Power Pivot 数据透视表



### 前言

什么是自动创建透视表，即利用`VBA`代码一键生成带度量值的`Power Pivot`透视表。为什么需要自动创建数据透视表？当你需要制作大量格式相同，数据原不同的透视表时。

最近接到一项工作任务，需要制作大量相同的数据透视表，但是使用表格区域以及层级不一样，数据源权限不一样，导致透视表数据源需要不一样，类似华东、华西、华南、华北四个区区域，四份数据源，但是透视表格式以及其中的度量值完全一致；和`Power BI`中的管理角色功能类似，即不同的用户有不同的数据权限

最初计划是做好一个模板，然后复制，以便快速完成任务，索性第一次需要复制的份数不多，能按时完成。不久过后，模板需要做调整，才发现更改有大量工作，需要一个个工作簿更改，因此萌发了实现批量制作透视表的想法。

后来经过查阅相关资料，初步发现利用`python`或`R`等脚本语言没办法制作透视表，只能从`VBA`中开始寻找解决方案。但是无奈对`VBA`没概念，查找一番发现没有现成的解决方案，或者也是因为当时看不懂`VBA`代码错过了。但是，无意中发现了刷新透视表的`VBA`代码，这样我们可以通过脚本语言更新数据源并用可以刷新透视表的代码刷新透视表，按照层级拆分后的数据源全部刷新一遍，就得到了不同数据源的透视表，至此，算解决了批量创建格式相同但数据源不同的透视表问题。



### 准备工作

在查找相关资料中，发现透视表主要分为两大类，【基础透视表】，【Power Pivot 透视表】，本人常用【Power Pivot 透视表】。想要完成一键生成透视表，需要经过以下几个步骤：

1. 将表格数据源添加都模型

2. 添加度量值和建立表关系

3. 创建透视表

4. 拉取透视表，即将相应维度以及度量值放在透视表的 【行】，【列】，【值】，【筛选】上。

   <img src="/img/2020-12-30/布局.png" alt="透视表布局" style="zoom:75%;" />

透视表数据源来源：透视表的数据源可以区分为本地数据源或外部数据源，一般默认本地数据源为Excel等为主的文本文件，外部数据源以数据库为主，本文中的透视表数据源来源于本地Excel。

[直接看怎样创建Power Pivot 透视表](# Power Pivot 透视表)

### 基础透视表

基础透视表是指不需要从模型生成透视表，也不需要通过`DAX`函数添加度量值的情况下透视表。

在这种情况下，透视表可以通过录制宏实现`VBA`代码自动创建透视表。讲道理，看多了这几段代码能勉强认识看懂，但是不是完全理解，大家如果需要用，自己多录制几段宏代码就ok了。

演示数据以及代码下载地址：<https://gitee.com/zhongyufei/excel/tree/master/vba>

```vb
Sub 创建透视表()
    Sheets.Add
    ActiveWorkbook.PivotCaches.Create(SourceType:=xlDatabase, SourceData:="表1" _
        , Version:=6).CreatePivotTable TableDestination:="Sheet1!R3C1", TableName _
        :="数据透视表1", DefaultVersion:=6
    Sheets("Sheet1").Select
    Cells(3, 1).Select
   
    With ActiveSheet.PivotTables("数据透视表1").PivotCache
        .RefreshOnFileOpen = False
        .MissingItemsLimit = xlMissingItemsDefault
    End With
    
    
    ’数据透视表行列部分以及计算汇总字段
    
    ActiveSheet.PivotTables("数据透视表1").RepeatAllLabels xlRepeatLabels
    With ActiveSheet.PivotTables("数据透视表1").PivotFields("Species")
        .Orientation = xlRowField
        .Position = 1
    End With
    ActiveSheet.PivotTables("数据透视表1").AddDataField ActiveSheet.PivotTables("数据透视表1" _
        ).PivotFields("Petal.Width"), "求和项:Petal.Width", xlSum
End Sub

```



### Power Pivot 透视表

由于`power pivot`透视表的便捷性，工作中大部分时候都是用Power Pivot带度量值的透视表，那就需要通过`VBA`直接生成度量值，主要步骤：第一步将表格添加到模型，第二步创建度量值，第三步创建透视表，拉取透视表。

数据以及代码下载地址：<https://gitee.com/zhongyufei/excel/tree/master/vba/Powerpivot>

[项目地址](<https://gitee.com/zhongyufei/excel/tree/master/vba/Powerpivot>)

- 将表格添加到模型

  将工作簿中`Sheet1`工作表明细添加到模型中，你需要先转化为"表格"表格式，快捷键`Ctrl+t`

  <img src="/img/2020-12-30/转化为表格.gif" alt="转化" style="zoom:50%;" />

  第一步将表添加到模型，需要说明的是 ：将Sheet1添加到模型，并命名为ORA，这些参数需根据实际情况修改调整。`Sheet1`即工作簿sheet默认命名。

  主要参数解释：

  name 参数是连接名称，

  Description：连接的描述，

  ConnectionString：连接的字符串，即工作簿的路径，在演示代码时可以打印查看，

  lCmdtype：是连接方式，针对连接Excel或数据库方式不一，[参考官网](https://docs.microsoft.com/en-us/office/vba/api/excel.xlcmdtype)，连接数据库需要选着"xlCmdSql"

  <img src="/img/2020-12-30/参数.png" alt="连接方式" style="zoom:75%;" />

  CreateModelConnection：是否将表添加到模型

  ImportRelationships：是否创建关系，如果选择是，后期可以在多个表中建立表关系

  以上参数为个人理解，可能有误，也没有动力完全求证。

  ```vb
  Sub 添加到模型()
  Sheets("Sheet1").ListObjects(1).Name = "ORA"
  WrkName = ActiveWorkbook.Name
  TblName = Sheets("Sheet1").ListObjects(1).Name
  'MsgBox TbleName
  FilPath = ActiveWorkbook.Path
  'MsgBox FilePath
  ConnStr = "WORKSHEET;" + FilPath
  'MsgBox ConnStr
  CommTxt = WrkName + "!" + TblName
  ' 关键代码 将'动销存透视表.xlsm'添加到数据连接
  Workbooks("动销存透视表.xlsm").Connections.Add2 _
                                          Name:="myconnection", _
                                          Description:="This is my test dataset.", _
                                          ConnectionString:=ConnStr, _
                                          CommandText:=CommTxt, _
                                          lCmdtype:=xlCmdExcel, _
                                          CreateModelConnection:=True, _
                                          ImportRelationships:=False
  End Sub
  
  ```

  完成以上步骤实现了我刚开始遇到这个问题时最大的难题，可以说解决了问题的一大半。

  

- 在模型中添加度量值

  `MeasureName`参数指定度量值名称，`AssociatedTable`度量值写进模型表中，`Formula`指定度量值，`FormatInformation`指定度量值的数据格式，` Description`度量值描述，为了省事这个参数在下面代码中都没有修改。

  ```vb
  Sub 添加度量值()
  'Declare your variables.
  Dim myModel As Model
  Dim myModelTables As ModelTables
  Dim myModelMeasures As ModelMeasures
  Dim myModelTable As ModelTable
  Dim myModelTable1 As ModelTable
  
  'tablname
  Dim tabl As String
  
  '有货sku数
  Dim mea7 As String
  
  'Create a reference to the PowerPivot Model
  Set myModel = ActiveWorkbook.Model
  
  'myModel.Refresh
  Set myModelTables = myModel.ModelTables
  Set myModelTable = myModelTables.Item(1)
  Set myModelMeasures = myModel.ModelMeasures
  
  '模型的表名
  'myModelTable.Name = "a"
  
  tabl = myModelTable.Name
  tabll = "'" & myModelTable.Name & "'"
  
  
  '有货sku数
  mea7 = "COUNTROWS(FILTER(" & tabll & "," & tabll & "[可用库存]>0))"
  'MsgBox mea7
  myModelMeasures.Add MeasureName:="有货sku数", _
                          AssociatedTable:=myModelTable, _
                          Formula:=mea7, _
                          FormatInformation:=myModel.ModelFormatDecimalNumber, _
                          Description:="This is count of all my transactions."
  End Sub
  ```

  

- 创建透视表

  首先创建新的sheet 并命名为"透视表",并指定透视表创建的位置`Worksheets("透视表").Cells(5, 3)`，第五行第三列的位置创建透视表。在我们多开Excel时，并创建了多个透视表，再新建透视表时会依据透视表个数依次指定透视表名称，“数据透视表1”，“数据透视表2”，“数据透视表3”，“数据透视表4”，“数据透视表5”。在以下代码中定义了`Pivottablename`参数可以用来指定透视表名称，方便最后一步拉取透视表字段。

  `  Version:=xlPivotTableVersion15)`参数指定数据透视表版本

  ```vb
  Sub 创建透视表()
    Sheets.Add.Name = "透视表"
    Dim Pivottablename As String
    Dim PowerPivotCache As Excel.PivotCache
    Dim NewPowerPivotTable As Excel.PivotTable
     'Create new cache
    Set PowerPivotCache = ActiveWorkbook.PivotCaches.Create( _
    SourceType:=xlExternal, _
    SourceData:=ActiveWorkbook.Connections("ThisWorkbookDataModel"), _
    Version:=xlPivotTableVersion15)
     ' Create PivotTable
    Set NewPowerPivotTable = PowerPivotCache.CreatePivotTable( _
    TableDestination:=Worksheets("透视表").Cells(5, 3), _
    DefaultVersion:=xlPivotTableVersion15)
    
    'Settings for new PowerPivotTable
     '指定透视表名称会自动变化
     With NewPowerPivotTable
       .RowAxisLayout xlTabularRow
       .HasAutoFormat = False
       'MsgBox NewPowerPivotTable.Name
        Pivottablename = NewPowerPivotTable.Name
     End With
     
     ' Cleanup
     Set NewPowerPivotTable = Nothing
     Set PowerPivotCache = Nothing
  End Sub
  ```

  以上代码可简写为：

  ```
  Sub 创建透视表()
  
    Sheets.Add.Name = "透视表"
    '定义表以及定义透视表缓存
    Dim Pivottablename As String
    Dim PowerPivotCache As Excel.PivotCache
    Dim NewPowerPivotTable As Excel.PivotTable
    
    '创建缓存
    Set PowerPivotCache = ActiveWorkbook.PivotCaches.Create( _
    SourceType:=xlExternal, _
    SourceData:=ActiveWorkbook.Connections("ThisWorkbookDataModel"), _
    Version:=xlPivotTableVersion15)
    
    ' 创建透视表
    Set NewPowerPivotTable = PowerPivotCache.CreatePivotTable( _
    TableDestination:=Worksheets("透视表").Cells(5, 3), _
    DefaultVersion:=xlPivotTableVersion15)
  
  End Sub
  ```

  

- 最后一步拉取透视表 

  `Orientation=xlRowField` 代表透视表中的行，`Orientation = xlColumnField`参数代表透视表中的列，`Position = 1,Position = 2`参数代表透视中的行位置或者顺序。代码最后一部分即透视表中的值。

  注意`Pivottablename`这个是引用的创建透视表中的值，代表当前透视表名称。分开执行时须指定`Pivottablename`的值，默认情况下为数据透视表1

  ```vb
  Sub 按照需求拉透视表()
     '拉透视表过程
  
      Application.WindowState = xlMaximized
      
      With ActiveSheet.PivotTables(Pivottablename).CubeFields("[ORA].[分析大类]")
          .Orientation = xlRowField
          .Position = 1
      End With
      
      With ActiveSheet.PivotTables(Pivottablename).CubeFields("[ORA].[排名区间]")
          .Orientation = xlRowField
          .Position = 2
      End With
      
      
       With ActiveSheet.PivotTables(Pivottablename).CubeFields("[ORA].[城市]")
          .Orientation = xlColumnField
          .Position = 1
      End With
      
      ActiveSheet.PivotTables(Pivottablename).AddDataField ActiveSheet.PivotTables(Pivottablename _
          ).CubeFields("[Measures].[有货sku数]")
      
  End Sub
  ```

  

- 调整数据透视表格式：可通过录制宏代码调整格式，以下代码可批量调整一个文件夹下所有Excel文件全部sheet的格式

  

```vb
Sub 调整格式()
Dim a
'a = Dir("C:\Users\zhongyf\Desktop\work\output\*.XLSX")' 将xlsx结尾的所有文件打开
Do '遍历目录下的所有指定格式的文件名
a = a = Dir("C:\Users\zhongyf\Desktop\work\output\*.XLSX")' 
Workbooks.Open "C:\Users\zhongyf\Desktop\work\output\*.XLSX\" + a
If a <> "" Then
    For Each Sheet In Sheets 
    Sheet.Columns("A:AZ").Font.Name = "微软雅黑"
    Sheet.Columns("A:AZ").Font.Size = "10"
    Sheet.Range("A:AZ").EntireColumn.AutoFit
    Next Sheet

Else
Exit Sub
End If
Loop

End Sub
```

------

以上，即可以通过`VBA`实现一键生成透视表。



参考资料:<https://github.com/areed1192/sigma_coding_youtube/tree/master/vba/vba-excel>