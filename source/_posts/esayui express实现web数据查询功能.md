---
title: esayui express实现web数据查询功能
date: 2018-12-08
author: 杨金坤
tags:
  - esayui
categories:
  - nodejs
thumbnail: img/thumbnail/2.jpg
blogexcerpt:
    esayui express实现web数据查询功能
---


# esayui express实现web数据查询功能
## 服务端代码

```
const express = require('express')
const app = express()
const odbc = require('node-odbc');
app.use(express.static('public'))
app.get('/', (req, res) => res.send('Hello World!'))
app.get('/sql/type/:type/db_name/:db_name/sqlstr/:sqlstr', function (req, res) {
	console.log('Dsn=' + req.params.type +';UID=root;PWD=root;DATABASE=' + req.params.db_name);
	let conn = new odbc.Connection().connect('Dsn=' + req.params.type +';UID=root;PWD=root;DATABASE=' + req.params.db_name);
	//console.log(decodeURI(req.params.sqlstr).replace(/\+/g,' '));
	conn.executeQuery(1,(rs, err) => 
    {
        if( err != null )
        {
          return  res.send( err );
        }
       res.send(rs);
    },
    decodeURI(req.params.sqlstr).replace(/\+/g,' ')
	);
})

app.listen(3000, () => console.log('Example app listening on port 3000!'))
```

## 网页端代码


```
<html lang="en">
<head>
    <meta http-equiv="content-type" content="text/html;charset=utf-8">
    <title></title>
    <link rel="stylesheet" type="text/css" href="jquery-easyui-1.5/themes/gray/easyui.css">
    <link rel="stylesheet" type="text/css" href="jquery-easyui-1.5/themes/icon.css">
    <script type="text/javascript" src="jquery-easyui-1.5/jquery.min.js"></script>
    <script type="text/javascript" src="jquery-easyui-1.5/jquery.easyui.min.js"></script>
    <script type="text/javascript" src="jquery-easyui-1.5/locale/easyui-lang-zh_CN.js"></script>

 
 
 
</head>
<body style="padding:0px;margin:0px">
 
<table id="dg" style="width:100%;" data-options=""></table>
 
<div id="toolbar">
    <table style='width:100%'>
        <div id="toolbar">
 
            <form >
                <div>数据源:<select name="type">
                        <option value="mysqldb" selected="selected">mysqldb</option>
                        <option value="1">ORACLE</option>
                    </select>
                数据库名:<input type="text" name="db_name" class="easyui-validatebox" required="true" placeholder="DATABASE" value="dqmm" id="db_name" /></div>
                <div">
                    <di>SQL语句:</div><textarea id="tt" name="sqlstr" rows="10" cols="100" ></textarea>
                </div>
 
            </form>
            <div>
            	  <button type="button" onclick="select_clear();" class="easyui-linkbutton" iconCls="icon-undo" >清除SQL</button>
            	  <button type="button" onclick="$('#dg').datagrid('clearSelections');" class="easyui-linkbutton" iconCls="icon-clear" >取消选中</button>
                <button type="button" onclick="datagrid_load();" class="easyui-linkbutton" iconCls="icon-search" >执行查询</button>
            </div>
        </div>
    </table>
 	<div id="dd" class="easyui-dialog" style="padding:5px;width:400px;height:200px;"
			title="提示" iconCls="icon-ok"
			buttons="#dlg-buttons">
		输入SQL
	</div>
	<div id="dlg-buttons">
		<a href="#" class="easyui-linkbutton" iconCls="icon-ok" onclick="javascript:$('#dd').dialog('close')">Ok</a>
		<a href="#" class="easyui-linkbutton" iconCls="icon-cancel" onclick="javascript:$('#dd').dialog('close')">Cancel</a>
	</div>
</div>
 
 
<script type="text/javascript">
    //动态展示列和数据
    var datagrid_load = function () {
        var columns = new Array();//定义列数组
        var ser = encodeURI($('form').serialize().replace(/(&)/g,'\/').replace(/(=)/g,'\/'));//参数需要跟下面保持一致
     	  if ($("textarea[name='sqlstr']").val() == null || $("textarea[name='sqlstr']").val() == '') {
    	  	ser = ser + "select 1 from dual";
    	  	$('#dd').dialog('open');
    	  }
        $.getJSON('sql/'+ser, null, function (result) {//通过方法获得后台数据，该方法不唯一。
            /*var column = {
                field: 'field', title: '不变的', width: 100, sortable: true, editor: 'textReadonly',
            }
            columns.push(column);//将固定写死的列，存入列数组中*/
            let rslt = {name: name};
            let rsay = [];
            if (Array.isArray(result)) {
            	rslt = result[0];
            	rsay = result;
            } else {
            	rslt = result;
            	rsay.push(result);
            }
            for (var key in rslt) {//遍历获得的后台数据，这是需要动态显示的表头数据源
                var column1 = {
                    field: key, title: key , width: 70, sortable: true, editor: {
                        type: 'numberbox',
                    }
                }// field字段和title值都是根据后台的数据来赋值
                columns.push(column1);//将需要动态显示的列，存入列数组中
            }
            grid = $('#dg').datagrid({ loadFilter: pagerFilter }).datagrid({   //定位到Table标签，Table标签的ID是grid
            //url: 'sql/'+ser,   //指向后台的Action来获取当前菜单的信息的Json格式的数据
            data: rsay,
            title: 'sql查询',
            iconCls: 'icon-view',
//                height: 650,
//                width: '100%',
            fit: true,
            idField: 'ID',
            method: 'get',
            toolbar: '#toolbar',
            pageSize: 50,
            pagination: true,
            rownumbers: true,
            fitColumns: false,    //横向滚动条
            columns: [
                columns,//通过js动态生成，这是关键。
            ],
        });
        });
 
 
    }
 
 
    function initTable(columns) {
//    	  var ser = encodeURI($('form').serialize().replace(/(&)/g,'\/').replace(/(=)/g,'\/'));//参数需要跟下面保持一致
//    	  if ($("textarea[name='sqlstr']").val() == null || $("textarea[name='sqlstr']").val() == '') {
//    	  	ser = ser + "select 1 from dual";
//    	  }
        grid = $('#dg').datagrid({   //定位到Table标签，Table标签的ID是grid
           // url: 'sql/'+ser,   //指向后台的Action来获取当前菜单的信息的Json格式的数据
            title: 'sql查询',
            iconCls: 'icon-view',
//                height: 650,
//                width: '100%',
            fit: true,
            idField: 'ID',
            method: 'get',
            toolbar: '#toolbar',
            rownumbers: true,
            fitColumns: false,    //横向滚动条
            columns: [
                columns,//通过js动态生成，这是关键。
            ],
        });
    };
 
 
    initTable();
    function select_clear() {
        $('#tt').val('');
     }
    function pagerFilter(data) {  
        if (typeof data.length == 'number' && typeof data.splice == 'function') {   // is array  
            data = {  
                total: data.length,  
                rows: data  
            }  
        }  
        var dg = $(this);  
        var opts = dg.datagrid('options');  
        var pager = dg.datagrid('getPager');  
        pager.pagination({  
            onSelectPage: function (pageNum, pageSize) {  
                opts.pageNumber = pageNum;  
                opts.pageSize = pageSize;  
                pager.pagination('refresh', {  
                    pageNumber: pageNum,  
                    pageSize: pageSize  
                });  
                dg.datagrid('loadData', data);  
            }  
        });  
        if (!data.originalRows) {  
            data.originalRows = (data.rows);  
        }  
        var start = (opts.pageNumber - 1) * parseInt(opts.pageSize);  
        var end = start + parseInt(opts.pageSize);  
        data.rows = (data.originalRows.slice(start, end));  
       // console.log(data);
        return data;  
    }
 
</script>
</body>
</html>
```

