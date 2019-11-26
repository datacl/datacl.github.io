---
title: drill安装和使用
date: 2019-11-26
author: 杨金坤
tags:
  - drill
categories:
  - 开源
thumbnail: img/thumbnail/7.jpg
blogexcerpt:
    drill安装，独立运行模式和集群模式的安装，并使用libpam4j实现用户安全认证
---


# drill 安装与使用
## 安装
### 1.下载二进制压缩包

[http://drill.apache.org/download/](https://note.youdao.com/)

### 2.安装环境

1. 独立运行：JDK,JAVA_HOME
2. 集群运行：JDK，zookeeper

### 3.独立运行

- 压缩包解压
- 不需要任何配置，到bin目录下执行
- 不要配置drill-override.conf
- 不能设置用户权限
- Linux系统
- 设置hostname，不要使用localhost，不使用127.0.0.1

```
drill-embedded.bat
```

### 4.集群运行

- linux系统
- 设置hostname

```
[root@localhost ~]# hostnamectl set-hostname youhostname
[root@localhost ~]# cat  /etc/sysconfig/network
NETWORKING=yes #表示系统是否使用网络，一般设置为yes。如果设为no，则不能使用网络。
HOSTNAME=youhostname #设置本机的主机名，这里设置的主机名要和/etc/hosts中设置的主机名对应
GATEWAY=192.168.78.2 #设置本机连接的网关的IP地址。
[root@localhost ~]# cat /etc/hosts
#127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
#::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
#192.168.78.131 localhost localhost.localdomain localhost4 localhost4.localdomain4
#127.0.1.1 abc
#127.0.0.1 abc
192.168.78.131 abc
[root@localhost ~]# reboot


```
- 安装zookeeper
- - 下载[https://www.apache.org/dyn/closer.cgi/zookeeper/](https://note.youdao.com/)
- - 解压二级制压缩包
- - 添加配置文件conf/zoo.cfg

```
[root@abc conf]# cat zoo.cfg 
tickTime=2000
dataDir=/opt/apache-zookeeper-3.5.5-bin/data
server.1=youhostname:2888:3888
clientPort=2181
```
- - 启动与连接

```
bin/zkServer.sh start
bin/zkCli.sh -server youhostname:2181
```
- 配置drill-override.conf

```
drill.exec: {
  cluster-id: "drillbits1",
  zk.connect: "192.168.78.131:2181"
} 
```
- 启停drillbit

```
drillbit.sh [--config <conf-dir>] (start|stop|graceful_stop|status|restart|autorestart)
```
- 访问web，测试用户，http://192.168.78.131:8047/
### 5. 使用libpam用户认证
- linux系统，集群模式下
- 登录drill-localhost，SVN用户是linux系统用户，设置为管理员

```
[root@abc bin]# ./drill-localhost
Apache Drill 1.16.0
"A Drill in the hand is better than two in the bush."
apache drill> alter system set security.admin.users = 'svn';
+------+-------------------------------+
|  ok  |            summary            |
+------+-------------------------------+
| true | security.admin.users updated. |
+------+-------------------------------+
1 row selected (1.397 seconds)
```
- 停止drillbit，修改drill-override.conf
```
drill.exec: {
  cluster-id: "drillbits1",
  zk.connect: "192.168.78.131:2181",
  impersonation: {                                   
         enabled: true,                             
         max_chained_user_hops: 3                     
       },
  security.user.auth: {
         enabled: true,
         packages += "org.apache.drill.exec.rpc.user.security",
         impl: "pam4j",
         pam_profiles: [ "sudo", "login" ],
   encryption.sasl.enabled: true,
   encryption.sasl.max_wrapped_size: 65536
  } 
} 
```
- 访问web，测试用户，http://192.168.78.131:8047/
- 添加rdbms数据源

```
{
  "type": "jdbc",
  "driver": "com.mysql.jdbc.Driver",
  "url": "jdbc:mysql://192.168.78.1:3306/dqmm?useUnicode=true&characterEncoding=UTF-8&remarks=true&useInformationSchema=true",
  "username": "root",
  "password": "root",
  "caseInsensitiveTableNames": true,
  "enabled": true
}
```
- 执行查询
   