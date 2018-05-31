---
title: The Java™ Metadata Interface JMI
date: 2018-05-31
author: 杨金坤
tags:
  - 元数据交换
categories:
  - 元数据
thumbnail: img/thumbnail/1.jpg
blogexcerpt:
    JMI 规范定义了一个动态的，平台无关的基础架构，该架构支持创建、存储、接入、发现和交换元数据。JMI基与Object Management Group (OMG)的Meta Object Facility (MOF) 规范。使用JMI，应用和工具可以根据服从MOF和UML规范的模型自动生成java接口。
---

# The Java™ Metadata Interface JMI

JMI 规范定义了一个动态的，平台无关的基础架构，该架构支持创建、存储、接入、发现和交换元数据。JMI基与Object Management Group (OMG)的Meta Object Facility (MOF) 规范。使用JMI，应用和工具可以根据服从MOF和UML规范的模型自动生成java接口。

JMI包含三个自包：
- javax.jmi.model 模型
- javax.jmi.reflect 反射
- javax.jmi.xmi 交换

JMI的主要面向的如下软件的厂商：
- Metadata based solutions 元数据解决方案
- Data warehouse products 数据仓库产品
- Software integration platforms 软件集成平台
- Software development tools 软件开发工具

## MOF4层架构
![image](https://note.youdao.com/yws/api/personal/file/53200CA37A2B4730AD9568C31D258368?method=download&shareKey=a431b909f44a32514a6994cf24c8c033)
- meta-metamodel-元-元模型 (M3)
- metamodels-元模型 (M2)
- models-模型和元数据 (M1)
- information-信息和数据(M0)

M3 -定义-> M2 -定义-> M1 -定义-> M0

MOF定义了一系列反射API。与JAVA反射类似，MOF提供的接口可以访问内部复杂信息。

JMI简介

JMI是对MOF的java语法表示，任何系统提供对JMI的结构实现提供JMI服务。

JMI支持一下J2EE场景：
一个可以用通用java对象访问元数据的元数据平台。
一个java工具和应用的集成和互操作平台。
集成OMG的建模和元数据架构。


**数据仓库场景**

![image](https://note.youdao.com/yws/api/personal/file/9AD6A454C3FD41E88BA3E4E18080325C?method=download&shareKey=981f0c9792098cf822aaf29d2d1e8ba3)

数据仓库应用场景中，需要面临各种多种多样的内部互操作，通常问题表现出缺乏通用的元数据基础设施。数据仓库处理场景中，需要处理各种不同的数据源和数据格式或数据库。

## MOF Reflective Package

提供包含如下功能的接口：
新建、更新、接入、导航和调用类的代理对象。
查询、更新对象间的关联关系。
导航MOF包的结构。

除此之外还有：
查找一个对象的元模型
查找一个对象的容器和包
测试对象身份ID
删除对象

[oracle-JMI](https://docs.oracle.com/cd/E17802_01/products/products/jmi/jmi-1_0-fr-doc/overview-summary.html)

[netbean-JMI](https://netbeans.org/download/5_5_1/javadoc/org-netbeans-api-mdr/architecture-summary.html)

[Ghub-netbeans](https://github.com/apache/incubator-netbeans)

[mdr-doc](http://kickjava.com/src/org.netbeans.mdr.index.htm)