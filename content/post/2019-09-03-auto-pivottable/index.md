---
title: auto pivottable
author: 宇飞的世界
date: '2019-09-03'
slug: auto-pivottable
categories:
  - pivottable
tags:
  - Power Pivot
---





#### 为什么需要自动创建数据透视表

最近接到一项工作任务，需要制作大量同款数据透视表，但是使用区域以及层级不一样导致数据源不一样，依据一个模板批量复制后更改部分细节问题会有大量工作，需要一个个透视表取更改，因此萌发了VBA实现批量制作透视表的想法。

#### 基础透视表

本处基础透视表是指不需要从模型生成透视表，也不需要通过DAX函数添加度量值的情况下透视表。在这种情况下透视表可以通过录制宏实现VBA代码自动创建透视表。

数据以及代码下载地址：<https://gitee.com/zhongyufei/excel/tree/master/vba>



```
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



#### Power Pivot 透视表

工作中大部分时候都是带度量值的透视表，那就需要通过VBA直接生成度量值，主要步骤：第一步将表格添加到模型，第二步创建度量值，第三步创建透视表，拉取透视表。

数据以及代码下载地址：<https://gitee.com/zhongyufei/excel/tree/master/vba/Powerpivot>



- 将表格添加到模型

  第一步将表添加到模型，需要说明的是 ：将Sheet1添加到模型，并命名为ORA，这些参数需根据实际情况修改调整

```
Sub 添加到模型()
Sheets("Sheet1").ListObjects(1).Name = "ORA"
WrkName = ActiveWorkbook.Name
TblName = Sheets("Sheet1").ListObjects(1).Name
'MsgBox TbleName
FilPath = ActiveWorkbook.Path
'MsgBox FilePath
ConnStr = "WORKSHEET;" + FilPath
CommTxt = WrkName + "!" + TblName

'"WORKSHEET;C:\Users\305197\Desktop\PowerPivot.xlsm"
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

  

- 在模型中添加度量值

`MeasureName`参数指定度量值名称，`AssociatedTable`度量值写进模型表中，`Formula`指定度量值，`FormatInformation`指定度量值的数据格式，` Description`度量值描述。


```
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

  首先创建新的sheet 并命名为"透视表",并指定透视表创建的位置`Worksheets("透视表").Cells(5, 3)`，第五行第三列的位置创建透视表。在我们多开Excel时，并创建了多个透视表，再新建透视表时会依据透视表个数依次指定透视表名称，“数据透视表1”，“数据透视表2”，“数据透视表3”，“数据透视表4”，“数据透视表5”。再以下代码中定义了`Pivottablename`参数可以用来指定透视表名称，方便最后一步拉取透视表字段。

  `  Version:=xlPivotTableVersion15)`参数指定数据透视表版本

```
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

```
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

  

```
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



参考资料:<https://github.com/areed1192/sigma_coding_youtube/tree/master/vba/vba-excel>

