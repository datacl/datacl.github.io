---
title: Calcite SQL解析与验证
date: 2020-06-12
author: 杨金坤
tags:
  - 常用
categories:
  - calcite
thumbnail: img/thumbnail/2.jpg
blogexcerpt:
    Apache Calcite是一个动态数据管理框架，它具备很多典型数据库管理系统的功能，比如SQL解析、SQL校验、SQL查询优化、SQL生成以及数据连接查询等，但是又省略了一些关键的功能，比如Calcite并不存储相关的元数据和基本数据，不完全包含相关处理数据的算法等。
---
# Calcite SQL解析与验证
## Calcite 主要功能
Apache Calcite是一个动态数据管理框架，它具备很多典型数据库管理系统的功能，比如SQL解析、SQL校验、SQL查询优化、SQL生成以及数据连接查询等，但是又省略了一些关键的功能，比如Calcite并不存储相关的元数据和基本数据，不完全包含相关处理数据的算法等。

使用Calcite作为SQL解析与优化引擎的又Hive、Drill、Flink、Phoenix和Storm，Calcite凭借其优秀的解析优化能力，会有越来越多的数据处理引擎采用Calcite作为SQL解析工具。
## Calcite SQL解析
### 解析配置SqlParser.Config

```
SqlParser.Config config = SqlParser.configBuilder()
                .setLex(Lex.MYSQL) //使用mysql 语法
                .setCaseSensitive(false)
                .build();
```
### 创建解析语句

```
SqlParser sqlParser = SqlParser
                .create("insert into newstu select id --主键\r\n,name,age FROM (select id,name,age FROM stu) T left join T2 on T.id = T2.rid where age<20", config);
```
### 生产SQL语法结构树

```
SqlNode sqlNode = null;

        try {
            sqlNode = sqlParser.parseStmt();
        } catch (SqlParseException e) {
            throw new RuntimeException("", e);
        }
```

![sqlNode](https://note.youdao.com/yws/api/personal/file/FA927B9292084F4EB4EDFF65FE80C538?method=download&shareKey=6831d6961581fae5bce3c4d07fa76f35)

### SqlNode继承关系

SqlNode

```
graph LR
SqlCall-->Sqlnode
SqlInsert-->SqlCall
SqlSelect-->SqlCall
SqlDelete-->SqlCall
SqlUpdate-->SqlCall
SqlJoin-->SqlCall
SqlBasicCall-->SqlCall

```
### 获取SqlNode属性

```
    /**
     *
     * @param sn 解析sql语法结构树
     * @param ijn Jackson objectnode ,递归调用
     * @return Jackson objectnode ,递归调用
     */
private static ObjectNode nodeTree (SqlNode sn, ObjectNode ijn) {
        ObjectNode jn = ijn;
        Method[] mtd = sn.getClass().getDeclaredMethods();
        for (int i=0; i<mtd.length; i++) {
            try {
                if (!(mtd[i].getName().startsWith("get") && mtd[i].getParameterCount()==0) || mtd[i].getName().equals("getModifierNode") ||  mtd[i].invoke(sn) == null) continue;
                if (sn.getClass().getName().equals("org.apache.calcite.sql.SqlBasicCall")) {
                    try {
                        Object[] obj = (Object[]) mtd[i].invoke(sn);
                        ObjectNode njn = new JsonNodeFactory(false).objectNode();
                        if (obj[0].getClass().getName().equals("org.apache.calcite.sql.SqlIdentifier")) jn.put("getTable", obj[0].toString());
                        else jn.putPOJO("getSubSelect", nodeTree((SqlNode) obj[0], njn));
                        jn.put("getAlias", obj[1].toString());
                    } catch (ClassCastException e) {
//                    e.printStackTrace();
                    }
                }
                Object obj = mtd[i].invoke(sn);
                String nm = obj.getClass().getName();
                String op = "";
                if (nm.equals("org.apache.calcite.sql.SqlBasicCall")) {
                    Method mt = obj.getClass().getMethod("getOperator");
                    op = mt.invoke(obj).toString();
                    System.out.println(mtd[i].invoke(sn).getClass().getName());
                }
                if (nm.equals("org.apache.calcite.sql.SqlKind") || nm.equals("org.apache.calcite.sql.SqlLiteral") || (nm.equals("org.apache.calcite.sql.SqlBasicCall") && !op.equals("AS"))|| nm.equals("org.apache.calcite.sql.SqlIdentifier") || nm.equals("org.apache.calcite.sql.SqlNodeList")) jn.put(mtd[i].getName(), obj.toString());
                else {
                    ObjectNode njn = new JsonNodeFactory(false).objectNode();
                    jn.putPOJO(mtd[i].getName(), nodeTree((SqlNode) obj, njn));
                }
            } catch (IllegalAccessException e) {
//                e.printStackTrace();
            } catch (InvocationTargetException e) {
//                e.printStackTrace();
            } catch (ClassCastException e) {
//                  e.printStackTrace();
            } catch (NoSuchMethodException e) {
//                e.printStackTrace();
            }
        }
        return  jn;
    }
```
## SQL验证
### 数据源连接

```
        Properties info = new Properties();
        info.setProperty("lex", "JAVA");
        CalciteConnection calciteConnection = null;
        Statement stat = null;
        try {
            Class.forName("org.apache.calcite.jdbc.Driver");
            Connection connection =
                    DriverManager.getConnection("jdbc:calcite:", info);
            calciteConnection =
                    connection.unwrap(CalciteConnection.class);
            SchemaPlus rootSchema = calciteConnection.getRootSchema();
//创建Mysql的数据源schema
            Class.forName("com.mysql.jdbc.Driver");
            BasicDataSource dataSource = new BasicDataSource();
            dataSource.setUrl("jdbc:mysql://localhost:3306/dqmm?useUnicode=true&characterEncoding=UTF-8&remarks=true&useInformationSchema=true&useSSL=false");
            dataSource.setUsername("root");
            dataSource.setPassword("root");
            Schema schema = JdbcSchema.create(rootSchema, "mysqlTest", dataSource,
                    null, null);
            rootSchema.add("mysqlTest", schema);
            stat = calciteConnection.createStatement();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
```
### inline数据源连接

```
Properties info = new Properties();
        calcateConnectionDatasource(info);
        info.put("lex", "mysql");
        info.put("caseSensitive", "false");
        info.put("model",
                "inline:"
                        + "{\n"
                        + "  version: '1.0',\n"
                        + "  defaultSchema: 'dbtest',\n"
                        + "  schemas: [\n"
                        + "    {\n"
                        + "      name: 'dbtest',\n"
                        + "      type: 'custom',\n"
                        + "      factory: 'org.apache.calcite.adapter.jdbc.JdbcSchema$Factory',\n"
                        + "      operand: {\n"
                        + "        jdbcDriver: 'com.mysql.jdbc.Driver',\n"
                        + "        jdbcUrl:'jdbc:mysql://localhost:3306/dqmm?useUnicode=true&characterEncoding=UTF-8&remarks=true&useInformationSchema=true&useSSL=false',\n"
                        + "        jdbcUser: 'root',\n"
                        + "        jdbcPassword: 'root'\n"
                        + "      }\n"
                        + "    }\n"
                        + "  ]\n"
                        + "}");
        Connection  connection = null;
        CalcitePrepare.Context prepareContext = null;
        try {
            Class.forName("com.mysql.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:calcite:", info);
            Statement stat = connection.createStatement();
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
```
### 执行验证


```
         CalcitePrepare.Context prepareContext = null;
        try {
            CalciteServerStatement cstat = stat.unwrap(CalciteServerStatement.class);
            prepareContext = cstat.createPrepareContext();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        //这里需要注意大小写问题，否则表会无法找到
        Properties properties = new Properties();
        properties.setProperty(CalciteConnectionProperty.CASE_SENSITIVE.camelName(),
                String.valueOf(config.caseSensitive()));

        SqlTypeFactoryImpl factory = new SqlTypeFactoryImpl(RelDataTypeSystem.DEFAULT);
        CalciteCatalogReader calciteCatalogReader = new CalciteCatalogReader(
                prepareContext.getRootSchema(),
                prepareContext.getDefaultSchemaPath(),
                factory,
                new CalciteConnectionConfigImpl(properties));
        // 校验（包括对表名，字段名，函数名，字段类型的校验。）
        DataLineageFromSqlValidator validator = new DataLineageFromSqlValidator(SqlStdOperatorTable.instance(),
                calciteCatalogReader, factory,SqlValidator.Config.DEFAULT
        );
        // 校验后的SqlNode
        SqlNode validateSqlNode = validator.validate(sqlNode);
```
### SqlValidator简介
![SqlValidator](https://note.youdao.com/yws/api/personal/file/C6BC87642F1B4DE380ED6161E1C402F2?method=download&shareKey=24d319daea39b8e375647d4d0b554475)

将[元数据]转换成 SqlValidator 内部的 对象 进行表示，也就是 SqlValidatorScope 和 SqlValidatorNamespace 两种类型的对象：

- SqlValidatorNamespace：a description of a data source used in a query，它代表了 SQL 查询的数据源，它是一个逻辑上数据源，可以是一张表，也可以是一个子查询；
- SqlValidatorScope：describes the tables and columns accessible at a particular point in the query，代表了在某一个程序运行点，当前可见的字段名和表名。


### 总结
- 利用calcite可以实现SQL解析，生产语法树
- 验证SQL的正确性，生产数据血缘关系