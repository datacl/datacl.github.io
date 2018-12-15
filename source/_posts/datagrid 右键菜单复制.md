---
title: datagrid 右键菜单复制
date: 2018-12-08
author: 杨金坤
tags:
  - esayui
categories:
  - nodejs
thumbnail: img/thumbnail/4.jpg
blogexcerpt:
    esayui datagrid 右键菜单复制
---


# datagrid 右键菜单复制
### 概述
clipboard.min.js实现easyui datagrid 表格上选中行的复制，数据以\t分隔，\n换行；并且复制数据带表头和记录序号。
### js插件

```
    <link rel="stylesheet" type="text/css" href="jquery-easyui-1.5/themes/gray/easyui.css">
    <link rel="stylesheet" type="text/css" href="jquery-easyui-1.5/themes/icon.css">
    <script type="text/javascript" src="jquery-easyui-1.5/jquery.min.js"></script>
    <script type="text/javascript" src="jquery-easyui-1.5/jquery.easyui.min.js"></script>
    <script type="text/javascript" src="jquery-easyui-1.5/locale/easyui-lang-zh_CN.js"></script>
    <script type="text/javascript" src="clipboard.min.js"></script>
```

### easyui 右键菜单


```
<div id="mm" class="easyui-menu" style="width:120px;">
	<div class="copy" onclick="javascript:clipRow()">复制</div>
	<div iconCls="icon-save">
		<span>保存</span>
		<div style="width:150px;">
			<div iconCls="icon-filexls" onclick="javascript:saveRow('xls')"><b>Excel</b></div>
			<div iconCls="icon-filecsv" onclick="javascript:saveRow('csv')"><b>CSV</b></div>
			<div iconCls="icon-filetxt" onclick="javascript:saveRow('txt')"><b>TXT</b></div>
			<div iconCls="icon-filejson" onclick="javascript:saveRow('json')"><b>JSON</b></div>
		</div>
	</div>
	<div class="menu-sep"></div>
	<div>退出</div>
</div>
```
### 右键复制js

```
function clipRow() {
    	//console.log(JSON.stringify($('#dg').datagrid('getSelections')));
    	var datajson = $('#dg').datagrid('getSelections');
    	var data ='';
    	var cnt = 1;
    	for(var key in datajson[0]){
        data = data + '\t' + key;
			};
			data = data + '\n';
    	datajson.forEach(function(record) {
    		data = data + cnt++;
    		for(var key in record){
    			data = data + '\t' + record[key];
				};
				data = data + '\n';
    	})
			var clipboard = new ClipboardJS('.copy', {
			    text: function() {
			      return data;
			    }
			});
			clipboard.on('success', function(e) {
        $('#dd').dialog({title:'提示',content:'<span style="color:black; text-align:center;font-size:200%;">已复制</span>',iconCls:'icon-tip',buttons:'#dlg-buttons',resizable:true}).dialog('open');
    	});
			///clipboard.destroy();
    }
```
![右键菜单复制](https://note.youdao.com/yws/api/personal/file/CBD1B42141B44BBAB7DE03FE8B523E75?method=download&shareKey=5b65c534318c99c556fc317181329a24)
