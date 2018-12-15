---
title: nodejs node-odbc安装使用
date: 2018-12-04
author: 杨金坤
tags:
  - js
categories:
  - nodejs
thumbnail: img/thumbnail/7.jpg
blogexcerpt:
    使用node-odbc访问数据库
---

# node ODBC
## 安装

```
npm install -g node-odbc
```
[node odbc](https://github.com/ItsClemi/node-odbc)



```
var odbc = require('node-odbc');
let conn = new odbc.Connection().connect('DRIVER={MySQL ODBC 8.0 Unicode Driver};SERVER=localhost;UID=root;PWD=root;DATABASE=dqmm');
let conn = new odbc.Connection().connect('Dsn=mysqldb;UID=root;PWD=root;DATABASE=dqmm');
//返回一行
conn.executeQuery((res, err) => 
    {
        if( err != null )
        {
            console.log( err );
        }
 
        console.log( res); 
    },
    "SELECT * FROM dq_bz_rule"
);

//返回多行
conn.executeQuery(1,(res, err) => 
    {
        if( err != null )
        {
            console.log( err );
        }
        console.log(res);
    },
    "SELECT * FROM dq_bz_rule"
);

//Json遍历key
for(var key in res){
    console.log(key);
}
     
//Json遍历value
for(var key in res){
    console.log(res[key]);
}
```
