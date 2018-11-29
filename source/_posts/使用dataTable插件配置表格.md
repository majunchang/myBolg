---
title: 使用dataTable插件配置表格
categories: plugin
abbrlink: 41e2cee3
date: 2017-04-14 16:47:23
tags:
---
## 使用jquery的datatable插件 配置分页和查询功能
### 安装以及引入

* 使用npm进行安装
> npm install datatables --save
``` javascript
<link rel="stylesheet" type="text/css" href="http://cdn.datatables.net/1.10.13/css/jquery.dataTables.css">
 
<!-- jQuery -->
<script type="text/javascript" charset="utf8" src="http://code.jquery.com/jquery-1.10.2.min.js"></script>
 
<!-- DataTables -->
<script type="text/javascript" charset="utf8" src="http://cdn.datatables.net/1.10.13/js/jquery.dataTables.js"></script>
 
 ```
### 下面给出一个示例以及配置选项

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="node_modules/datatables/media/css/jquery.dataTables.css">
</head>
<body>
<table id="example" class="display" cellspacing="0" width="80%">
    <thead>
    <tr>
        <th>Name</th>
        <th>Position</th>
        <th>Office</th>
        <th>Age</th>
        <th>Start date</th>
        <th>Salary</th>
    </tr>
    </thead>
    <tfoot>
    <tr>
        <th>Name</th>
        <th>Position</th>
        <th>Office</th>
        <th>Age</th>
        <th>Start date</th>
        <th>Salary</th>
    </tr>
    </tfoot>
    <tbody>

    <tr>
        <td>Jena Gaines</td>
        <td>Office Manager</td>
        <td>London</td>
        <td>30</td>
        <td>2008/12/19</td>
        <td>$90,560</td>
    </tr>
    <tr>
        <td>Quinn Flynn</td>
        <td>Support Lead</td>
        <td>Edinburgh</td>
        <td>22</td>
        <td>2013/03/03</td>
        <td>$342,000</td>
    </tr>
    <tr>
        <td>Charde Marshall</td>
        <td>Regional Director</td>
        <td>San Francisco</td>
        <td>36</td>
        <td>2008/10/16</td>
        <td>$470,600</td>
    </tr>
    <tr>
        <td>Haley Kennedy</td>
        <td>Senior Marketing Designer</td>
        <td>London</td>
        <td>43</td>
        <td>2012/12/18</td>
        <td>$313,500</td>
    </tr>
    <tr>
        <td>Tatyana Fitzpatrick</td>
        <td>Regional Director</td>
        <td>London</td>
        <td>19</td>
        <td>2010/03/17</td>
        <td>$385,750</td>
    </tr>
    <tr>
        <td>Michael Silva</td>
        <td>Marketing Designer</td>
        <td>London</td>
        <td>66</td>
        <td>2012/11/27</td>
        <td>$198,500</td>
    </tr>
    <tr>
        <td>Paul Byrd</td>
        <td>Chief Financial Officer (CFO)</td>
        <td>New York</td>
        <td>64</td>
        <td>2010/06/09</td>
        <td>$725,000</td>
    </tr>
    <tr>
        <td>Gloria Little</td>
        <td>Systems Administrator</td>
        <td>New York</td>
        <td>59</td>
        <td>2009/04/10</td>
        <td>$237,500</td>
    </tr>
    <tr>
        <td>Bradley Greer</td>
        <td>Software Engineer</td>
        <td>London</td>
        <td>41</td>
        <td>2012/10/13</td>
        <td>$132,000</td>
    </tr>
    <tr>
        <td>Dai Rios</td>
        <td>Personnel Lead</td>
        <td>Edinburgh</td>
        <td>35</td>
        <td>2012/09/26</td>
        <td>$217,500</td>
    </tr>
    <tr>
        <td>Jenette Caldwell</td>
        <td>Development Lead</td>
        <td>New York</td>
        <td>30</td>
        <td>2011/09/03</td>
        <td>$345,000</td>
    </tr>

    </tbody>
</table>
</body>
</html>
```
### 图片显示
![mark](http://oneg19f80.bkt.clouddn.com/blog/20170416/082731669.png)
###效果如下：
![mark](http://oneg19f80.bkt.clouddn.com/blog/20170416/082826323.png)
### 接下来是配置选项，在配置之前，我们需要引入jq和datatable插件

``` javascript

<script src="node_modules/jquery/dist/jquery.js"></script>
<script src="node_modules/datatables/media/js/jquery.dataTables.js"></script>
<script>
    window.onload = function () {
        $('#example').dataTable({
                bLengthChange: false,
                bFilter: true,
                bInfo: false,
                bSort: true,
                select: false,
                sPaginationType: "full_numbers",
                sSearchPlaceholder: '请输入筛选关键字',
                bRetrieve: true,
                iDisplayLength: 5,
                oLanguage: {
                    sLengthMenu: "每页显示 _MENU_ 条数据",
                    sInfo: "从 _START_ 到 _END_ /共 _TOTAL_ 条数据",
                    sInfoEmpty: "没有数据",
                    sInfoFiltered: "(从 _MAX_ 条数据中检索)",
                    sZeroRecords: "没有检索到数据",
                    sSearchPlaceholder: '请输入筛选关键字',
                    sSearch: "筛选",
                    oPaginate: {
                        sFirst: "首页",
                        sPrevious: "前一页",
                        sNext: "后一页",
                        sLast: "尾页"
                    }
                },
                order:[[2,'asc']]
        })
    }
</script>

```
