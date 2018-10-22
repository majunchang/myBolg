---
title: datatables插件使用详细说明
date: 2017-04-16 08:40:24
tags:
categories: plugin
---
http://blog.csdn.NET/mickey_miki/article/details/8240477

---

## 本文共四部分：官网 | 基本使用|遇到的问题|属性表

### 一：官方网站：[http://www.datatables.NET/]

### 二：基本使用：[http://www.guoxk.com/node/jQuery-datatables]

####  DataTables的默认配置
``` 
$(document).ready(function() {

$('#example').dataTable();

} );

```

示例：http://www.guoxk.com/html/DataTables/Zero-configuration.html

#### DataTables的一些基础属性配置

"bPaginate": true, //翻页功能

"bLengthChange": true, //改变每页显示数据数量

"bFilter": true, //过滤功能

"bSort": false, //排序功能

"bInfo": true,//页脚信息

"bAutoWidth": true//自动宽度



示例：http://www.guoxk.com/html/DataTables/Feature-enablement.html

#### 数据排序
```
$(document).ready(function() {

$('#example').dataTable( {

"aaSorting": [

[ 4, "desc" ]

]

} );

} );
```

从第0列开始，以第4列倒序排列

示例：http://www.guoxk.com/html/DataTables/Sorting-data.html

#### 多列排序

示例：http://www.guoxk.com/html/DataTables/Multi-column-sorting.html

#### 隐藏某些列

```
$(document).ready(function() {

$('#example').dataTable( {

"aoColumnDefs": [

{ "bSearchable": false, "bVisible": false, "aTargets": [ 2 ] },

{ "bVisible": false, "aTargets": [ 3 ] }

] } );

} );
```
示例：http://www.guoxk.com/html/DataTables/Hidden-columns.html

#### 改变页面上元素的位置
```
$(document).ready(function() {

$('#example').dataTable( {

"sDom": '<"top"fli>rt<"bottom"p><"clear">'

} );

} );

//l- 每页显示数量

//f - 过滤输入

//t - 表单Table

//i - 信息

//p - 翻页

//r - pRocessing

//< and > - div elements

//<"class" and > - div with a class

//Examples: <"wrapper"flipt>, <lf<t>ip>
```

示例：http://www.guoxk.com/html/DataTables/DOM-positioning.html

#### 状态保存，使用了翻页或者改变了每页显示数据数量，会保存在cookie中，下回访问时会显示上一次关闭页面时的内容。

```
$(document).ready(function() {

$('#example').dataTable( {

"bStateSave": true

} );

} );
```

示例：http://www.guoxk.com/html/DataTables/State-saving.html

#### 显示数字的翻页样式
```
$(document).ready(function() {

$('#example').dataTable( {

"sPaginationType": "full_numbers"

} );

} );
```

示例：http://www.guoxk.com/html/DataTables/Alternative-pagination-styles.html

#### 水平限制宽度
```
$(document).ready(function() {

$('#example').dataTable( {

"sScrollX": "100%",

"sScrollXInner": "110%",

"bScrollCollapse": true

} );

} );
```

示例：http://www.guoxk.com/html/DataTables/Horizontal.html

#### 垂直限制高度

示例：http://www.guoxk.com/html/DataTables/Vertical.html

#### 水平和垂直两个方向共同限制

示例：http://www.guoxk.com/html/DataTables/HorizontalVerticalBoth.html

####改变语言

```
$(document).ready(function() {

$('#example').dataTable( {

"oLanguage": {

"sLengthMenu": "每页显示 MENU 条记录",

"sZeroRecords": "抱歉， 没有找到",

"sInfo": "从 START 到 END /共 TOTAL 条数据",

"sInfoEmpty": "没有数据",

"sInfoFiltered": "(从 MAX 条数据中检索)",

"oPaginate": {

"sFirst": "首页",

"sPrevious": "前一页",

"sNext": "后一页",

"sLast": "尾页"

},

"sZeroRecords": "没有检索到数据",

"sProcessing": ""

}

} );

} );
```

示例：http://www.guoxk.com/html/DataTables/Change-language-information.html

#### click事件

示例：http://www.guoxk.com/html/DataTables/event-click.html

#### 配合使用tooltip插件

示例：http://www.guoxk.com/html/DataTables/tooltip.html

#### 定义每页显示数据数量

```
$(document).ready(function() {

$('#example').dataTable( {

"aLengthMenu": [[10, 25, 50, -1], [10, 25, 50, "All"]]

} );

} );

```
示例：http://www.guoxk.com/html/DataTables/length_menu.html

#### row callback

示例：http://www.guoxk.com/html/DataTables/RowCallback.html

最后一列的值如果为“A”则加粗显示

#### 排序控制

```
$(document).ready(function() {

$('#example').dataTable( {

"aoColumns": [

null,

{ "asSorting": [ "asc" ] },

{ "asSorting": [ "desc", "asc", "asc" ] },

{ "asSorting": [ ] },

{ "asSorting": [ ] }

]

} );

} );
```

示例：http://www.guoxk.com/html/DataTables/sort.html

说明：第一列点击按默认情况排序，第二列点击已顺序排列，第三列点击一次倒序，二三次顺序，第四五列点击不实现排序

#### 从配置文件中读取语言包

```
$(document).ready(function() {

$('#example').dataTable( {

"oLanguage": {

"sUrl": "cn.txt"

}

} );

} );
```

#### 是用ajax源
```

$(document).ready(function() {

$('#example').dataTable( {

"bProcessing": true,

"sAjaxSource": './arrays.txt'

} );

} );
```

示例：http://www.guoxk.com/html/DataTables/ajax.html

#### 使用ajax，在服务器端整理数据
```
$(document).ready(function() {

$('#example').dataTable( {

"bProcessing": true,

"bServerSide": true,

"sPaginationType": "full_numbers",

"sAjaxSource": "./server_processing.PHP",

/*如果加上下面这段内容，则使用post方式传递数据

"fnServerData": function ( sSource, aoData, fnCallback ) {

$.ajax( {

"dataType": 'json',

"type": "POST",

"url": sSource,

"data": aoData,

"success": fnCallback

} );

}*/

"oLanguage": {

"sUrl": "cn.txt"

},

"aoColumns": [

{ "sName": "platform" },

{ "sName": "version" },

{ "sName": "engine" },

{ "sName": "browser" },

{ "sName": "grade" }

]//$_GET['sColumns']将接收到aoColumns传递数据

} );

} );
```

示例：http://www.guoxk.com/html/DataTables/ajax-serverSide.html

#### 为每行加载id和class

服务器端返回数据的格式：

{

"DT_RowId": "row_8",

"DT_RowClass": "gradeA",

"0": "Gecko",

"1": "Firefox 1.5",

"2": "Win 98+ / OSX.2+",

"3": "1.8",

"4": "A"

},

示例：http://www.guoxk.com/html/DataTables/add_id_class.html

#### 为每行显示细节，点击行开头的“+”号展开细节显示

示例：http://www.guoxk.com/html/DataTables/with-row-information.html



#### 选择多行

示例：http://www.guoxk.com/html/DataTables/selectRows.html

#### 集成jQuery插件jEditable

示例：http://www.guoxk.com/html/DataTables/jEditable-integration.html

示例打包下载：http://www.guoxk.com/html/DataTables/DataTables.rar



###、遇到的问题

#### “Cannot reinitialise DataTable.

To retrieve the DataTables object for this table, pass no arguments or see the docs for bRetrieve and bDestroy ”

解决办法：http://blog.csdn.Net/mickey_miki/article/details/8239185

#### 排序时指定某列不可排序

$(".datatable").dataTable( {  

        "aoColumnDefs": [ { "bSortable": false, "aTargets": [ 0 ] }]  
    });  

注意： "bSort": true, //排序功能 要注释掉

#### 确定选择每页展示个数列表和默认每页展示个数设置

[javascript] view plain copy

"aLengthMenu": [[4, 10, 20, -1], [4, 10, 20, "所有"]],  

"iDisplayLength":4  

### 属性表

![mark](http://oneg19f80.bkt.clouddn.com/blog/20170416/085655717.png)
![mark](http://oneg19f80.bkt.clouddn.com/blog/20170416/085727603.png)
![mark](http://oneg19f80.bkt.clouddn.com/blog/20170416/085758474.png)
![mark](http://oneg19f80.bkt.clouddn.com/blog/20170416/085822724.png)
![mark](http://oneg19f80.bkt.clouddn.com/blog/20170416/085844066.png)
