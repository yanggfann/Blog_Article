---
#标题
title: Power Query 用Excel玩转商业智能数据处理
date: 2018-5-20  10:15:23
categories: 
- 相关技术
- Power Query 简介
tags: 
- Power Query
---
《Power Query 用Excel玩转商业智能数据处理》一书由 朱仕平 著。Power Query在Office 2016中已经被作为内置的工具嵌入【数据】选项卡中。Power Query集成了Access和Excel的功能，可以对数据进行可视化菜单操作，完成对数据的提取（Extract）、转换（Transform）、加载（Load），也就是信息管理中ETL管理数据源的三个基本功能。<!--more-->
## Power Query
有了Power Query，我们不需要借助VBA，就可以轻松地将工作簿中的所有工作表合并成一张工作表，也可以吧一个文件夹下的所有子文件夹中的工作簿全部合并成一张工作表。将合并后的数据加载到Power Pivot中可以进行后期大量的数据整理和分析。对于查询Web数据，Power Query也是一把好手，我们想要的数据基本都可以通过Power Query呈现出来。
Power Query是一个非常方便的工具，只要在工作中有思路，有想法，就可以吧一些复杂的表格，根据自己的需求，转换成可以方便统计的数据连接。然后对数据连接进行加载、统计、分析、图标展示。
关于Power Query的各个功能如何使用，基本都可以在本书中找到解答。例如：选择列、删除列、保留行、删除行、排序、拆分列、分组依据、数据类型、将第一行作为标题、替换值、合并查询、追加查询、转置表格等。书中都有详细的步骤和截图说明。

## M语言
使用Power Query菜单的操作过程都会被记录成一个公式，这种公式被称为M语言公式。
* **M语言基础**
M语言实际就是通过函数公式将结果传递给变量，每个变量对应一个步骤，每个变量的步骤都环环相扣；这些公式可以使用现成的函数，也可以使用自定义函数。需要注意的是，公式中的函数和参数对大小写都非常敏感。
每个查询公式都指向前一个步骤的变量名称，前一个步骤的变量名称就是一个实际的结果，这个结果可以是Value、Record、List、Table。如果公式太长，可以在中间任意地方强制换行；但每个公式在最后都应该输入一个逗号，然后换行到第二个步骤。
直到最后一个步骤时，才不再需要逗号，将最后一个查询步骤作为最终的结果，使用in语句把这个步骤传递回【查询编辑器】
下面主要对M语言的部分函数参数和部分函数做一个简单的介绍。若在实际应用中需要使用到其它函数，可在书中进一步查找。加粗显示的函数为在GE实习过程中用到的一些参数和函数。

* **部分函数参数说明**

Occurence : 获取位置信息

|参数|说明|
|-|-|
|Occurence.First|第1次位置|
|**Occurence.Last**|最后1次位置|
|Occurence.All|所有位置|

ComparisonCriteria : 比较规则

|参数|说明|
|-|-|
|Order.Ascending|升序|
|Order.Descending|降序|

JoinKind : 连接种类

|参数|说明|
|-|-|
|JoinKind.Inner|内部连接|
|JoinKind.LeftOuter|左外部连接|
|JoinKind.RightOuter|右外部连接|
|JoinKind.FullOuter|完全外部连接|
|JoinKind.LeftAnti|左反连接|
|JoinKind.RightAnti|右反连接|

* **Text类函数**

|类|分类|函数名|概述|
|-|-|-|-|
|1|计算|**Text.Length**|返回字符串长度|
|2|转换|**Text.From**|返回Value的文本表现形式|
|3|获取|**Text.Range**|从指定位置开始返回指定个数的字符串|
|4|判断|**Text.Contains**|判断A字符串是否包含B字符串|
|5|判断|**Text.PositionOf**|判断B字符串在A字符串中的位置|
|6|判断|**Text.PositionOfAny**|判断A字符串中另一字符集合list列表的位置|
|7|转换|**Text.Upper**|所有字母转换为大写|
|8|转换|**Text.Trim**|清除字符串两端指定的字符|
|9|转换|**Text.Combine**|将字符集list列表合并成字符串|

* **Number类函数**

|类|分类|函数名|概述|
|-|-|-|-|
|1|常量|Number.NaN|代表0/0，返回NaN|
|2|随机数|Number.Random|返回介于0~1的小数|
|3|随机数|Number.RandomBetween|返回指定值之间的小数|
|4|转换|Number.FromText|从文本数值转换为数字|
|5|舍入|Number.Round|舍入到指定位数|
|6|计算|Number.Abs|返回绝对值|
|7|三角函数|Number.Cos|余弦|
|8|其他函数|Number.Log|返回指定底数的数字对数|

* **Time类函数**

|类|分类|函数名|概述|
|-|-|-|-|
|1|计算|Time.Hour|返回时间的小时数|
|2|计算|Time.Minute|返回时间的分钟数|
|3|计算|Time.Second|返回时间的秒钟数|
|4|智能时间|Time.ToRecord|返回时间中的时、分、秒的Recond记录|
|5|智能时间|Time.StartOfHour|返回当时小时的起始值|
|6|智能时间|Time.EndOfHour|返回当时小时的结束值|
|7|转换|Time.From|将给定的value返回时间|
|8|转换|Time.FromText|从文本表示的Text返回时间|
|8|转换|Time.ToText|返回指定文本样式的日期|

* **Date类函数**

|类|分类|函数名|概述|
|-|-|-|-|
|1|计算|Date.Year|返回日期的年份|
|2|计算|Date.QuarterOfYear|返回日期的季度数|
|3|智能日期|Date.EndOfDay|返回日期当天的结束值|
|4|判断|Date.IsLeapYear|判断日期是否在闰年|
|5|转换|Date.ToText|返回指定文本样式的日期|

* **DateTime类函数**

|类|分类|函数名|概述|
|-|-|-|-|
|1|计算|DateTime.Date|返回日期时间的日期部分|
|2|转换|DateTime.ToText|返回指定文本格式的日期时间|
|3|智能时间|DateTime.IsInCurrentHour|判断时间是否是系统时间的当前小时|

* **Duration类函数**

|类|分类|函数名|概述|
|-|-|-|-|
|1|计算|Duration.Days|获取持续时间的日期部分|
|2|创建|Duration.ToRecord|获取持续时间的各部分Record|
|3|转换|Duration.TotalDays|将持续时间换算成总天数|

* **Record类函数**

|类|分类|函数名|概述|
|-|-|-|-|
|1|计算|Record.FieldCount|返回记录的字段数|
|2|判断|Record.HasFields|检查是否包含某个字段|
|3|选择|Record.Field|返回指定字段的值|
|4|转换|Record.AddField|添加一个字段|
|5|序列化|Record.FromTable|Table转换成Record|

* **List类函数**

|类|分类|函数名|概述|
|-|-|-|-|
|1|计算|List.Sum|计算非空值的总和|
|2|判断|List.IsEmpty|判断list是否为空|
|3|选择|List.Select|按给定条件选择项目|
|4|操作|List.Sort|排序|
|5|集合|List.Combine|合并多个list为新list|
|6|创建|List.DateTimes|根据规则创建日期时间列表|

* **Table类函数**

|类|分类|函数名|概述|
|-|-|-|-|
|1|转换|Table.ToRows|转换成行的list表格|
|2|创建|Table.FromRows|从多行list创建表格|
|3|计算|Table.RowCount|统计表的行数|
|4|判断|Table.IsEmpty|判断表格是否为空表|
|5|选择|**Table.SelectRows**|根据指定条件选择行|
|6|选择|**Table.SelectColumns**|仅选择某些列|
|7|操作|**Table.Sort**|排序|
|8|操作|**Table.RenameColumns**|重命名列标题|
|9|操作|**Table.AddColumn**|添加新列|
|10|集合|**Table.NestedJoin**|合并查询|
|11|操作|**Table.ExpandTableColumn**|展开表格

* **文件类函数**

|类|函数名|概述|
|-|-|-|
|1|**Excel.CurrentWorkbook**|从当前表导入|
|2|File.Contents|从文件夹导入|
|3|**Excel.Workbook**|从指定路径导入表格|
|4|Csv.Document|从文本文件导入表格|
