---
title: express安装使用
date: 2018-12-06
author: 杨金坤
tags:
  - js
categories:
  - nodejs
thumbnail: img/thumbnail/2.jpg
blogexcerpt:
    express安装使用，数据库访问
---

# express
### 安装express


通过 npm init 命令为你的应用创建一个 package.json 文件。

```
npm init
```
键入 app.js 或者你所希望的名称

```
entry point: (index.js)
```
目录下安装 Express 并将其保存到依赖列表中
```
npm install express --save
```

创建app.js

```
const express = require('express')
const app = express()

app.get('/', (req, res) => res.send('Hello World!'))

app.listen(3000, () => console.log('Example app listening on port 3000!'))
```
运行


```
node app.js
```

浏览器访问http://localhost:3000/

安装node-odbc

```
npm install node-odbc --save
```

实现访问odbc访问mysql
app.js

```
const express = require('express')
const app = express()
const odbc = require('node-odbc');
app.get('/', (req, res) => res.send('Hello World!'))
app.get('/sql', function (req, res) {
	let conn = new odbc.Connection().connect('Dsn=mysqldb;UID=root;PWD=root;DATABASE=dqmm');
	conn.executeQuery((rs, err) => 
    {
        if( err != null )
        {
            console.log( err );
        }
       res.send(rs);
    },
    "SELECT * FROM dq_bz_rule"
	);
})

app.listen(3000, () => console.log('Example app listening on port 3000!'))
```


```
app.use(express.static('public'))
```
