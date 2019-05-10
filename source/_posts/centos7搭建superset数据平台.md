---
title: centos7搭建superset数据平台
date: 2019-05-10
author: 杨金坤
tags:
  - 工具
categories:
  - 开源
thumbnail: img/thumbnail/7.jpg
blogexcerpt:
    centos7搭建superset数据平台，Superset 是一款由 Airbnb 开源的“现代化的企业级 BI（商业智能） Web 应用程序”，其通过创建和分享 dashboard，为数据分析提供了轻量级的数据查询和可视化方案。Superset 的前端主要用到了 React 和 NVD3/D3，而后端则基于 Python 的 Flask 框架和 Pandas、SQLAlchemy 等依赖库。
---


# centos7搭建superset数据平台
## 1.安装完安装一些基础包：

```
yum -y install perl gd gd-devel libpng libpng-devel libjpeg libjpeg-devel zlib zlib-devel pcre-devel gcc gcc-c++ make cmake autoconf openssl openssl-devel ncurses-devel patch libxml2 libxml2-devel curl-devel openldap openldap-devel libevent libevent-devel bison icu libicu-devel libtool readline-devel net-snmp-devel bzip2-devel freetype-devel vim
```
## 2.安装mysql

### 下载mysql
https://dev.mysql.com/downloads/mysql/5.5.html#downloads


```
useradd -s /sbin/nologin -M mysql
tar zxvf mysql-5.5.62-linux-glibc2.12-x86_64.tar.gz
mv mysql-5.5.62-linux-glibc2.12-x86_64 mysql-5.5.62
cd mysql-5.5.62
```
### 配置mysql


```
[root@localhost mysql-5.5.62]# pwd
/opt/mysql-5.5.62
[root@localhost mysql-5.5.62]# ls
bin      data  include         lib  my.cnf        mysql-test  scripts  sql-bench
COPYING  docs  INSTALL-BINARY  man  mysql.server  README      share    support-files
[root@localhost mysql-5.5.62]# scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql
[root@localhost mysql-5.5.62]# cp support-files/my-medium.cnf my.cnf
[root@localhost mysql-5.5.62]# cp support-files/mysql.server mysql.server
[root@localhost mysql-5.5.62]# chmod 777 /tmp/mysql.sock
```

my.cnf找到对应位置添加以下内容：

```
[mysqld]
port=3306
socket=/tmp/mysql.sock
basedir=/opt/mysql-5.5.62
datadir=/data/mysql
user=mysql
character_set_server=utf8
lower_case_table_names=1
```

mysql.server添加以下内容：

```
basedir="/opt/mysql-5.5.62"
datadir=/data/mysql
```
添加软连接：
```
ln -s /opt/mysql-5.5.62/bin/mysql /usr/bin
ln -s /opt/mysql-5.5.62/bin/mysqladmin /usr/bin
```

### mysql启停

```
/opt/mysql-5.5.62/mysql.server start
/opt/mysql-5.5.62/mysql.server stop
/opt/mysql-5.5.62/mysql.server status
```
初始化密码：


```
mysqladmin -uroot password
```
### 存储库

```
show variables like "%char%";
create database superset
use superset
--这里如果不设置数据库为utf8，在后面初始化数据库时会报  Specified key was too long; max key length is 767 bytes 的错误
alter database superset character set utf8;
```
## 3.安装superset平台
依赖包安装；

```
yum upgrade python-setuptools
yum install gcc gcc-c++ libffi-devel python-devel python-pip python-wheel openssl-devel libsasl2-devel openldap-devel
yum install mysql-devel
```
libsasl2-devel 这个包找不到，后面也没影响
### 在virtualenv中安装 superset
```
[root@localhost mysql-5.5.62]# pip install virtualenv
[root@localhost mysql-5.5.62]# virtualenv supersetenv
[root@localhost opt]# pwd
/opt
[root@localhost opt]# ls
mysql-5.5.62  supersetenv
[root@localhost opt]# cd supersetenv/
[root@localhost opt]# source ./bin/activate
(supersetenv) [root@localhost supersetenv]# pip install --upgrade setuptools pip
(supersetenv) [root@localhost supersetenv]# pip install mysqlclient
(supersetenv) [root@localhost bin]# pwd
/opt/supersetenv/bin
(supersetenv) [root@localhost bin]# vi superset_config.py
```
superset_config.py 的内容：

```
#-*- coding: utf-8 -*-
#===============superset_config.py开始================
#使用python2.7，如果下面三行不加的话，使用中文时会出问题。
import sys # import sys package, if not already imported
reload(sys)
sys.setdefaultencoding('utf-8')

#---------------------------------------------------------
#Superset specific config
#---------------------------------------------------------
ROW_LIMIT = 5000
SUPERSET_WORKERS = 4
SUPERSET_WEBSERVER_PORT = 8088

#---------------------------------------------------------
#Flask App Builder configuration
#---------------------------------------------------------
#Your App secret key
SECRET_KEY = '\2\1thisismyscretkey\1\2\e\y\y\h'

#元数据存储默认使用的是sqlite。SQLALCHEMY_DATABASE_URI = 'sqlite:////path/to/superset.db'
#我这里改成mysql
#mysql://用户名:密码@192.168.1.162/数据库名?charset=utf8
SQLALCHEMY_DATABASE_URI = 'mysql://root:root@localhost/superset?charset=utf8&unix_socket=/tmp/mysql.sock'

#Flask-WTF flag for CSRF
WTF_CSRF_ENABLED = True

#Set this API key to enable Mapbox visualizations
MAPBOX_API_KEY = ''

#汉化
BABEL_DEFAULT_LOCALE='zh'
LANGUAGES = {
'zh': {'flag': 'cn', 'name': 'Chinese'},
'en': {'flag': 'us', 'name': 'English'}
}

#=============== superset_config.py结束===============
```
### 安装superset

选择版本0.26.3
```
(supersetenv) [root@localhost bin]# pip install superset==0.26.3
(supersetenv) [root@localhost bin]# pip install "markdown<3.0.0" superset
```
创建admin用户


```
(supersetenv) [root@localhost bin]# fabmanager create-admin --app superset
```

然后需要输入：

```
Username [admin]: admin
User first name [admin]: admin
User last name [user]: admin
Email [admin@fab.org]: admin@163.com
Password: admin
Repeat for confirmation: admin
```
安装出错 

```
cannot import name _maybe_box_datetimelike from pandas.core.common
```
查看当前 pandas 版本
```
(supersetenv) [root@localhost bin]#  pip list | grep pandas
pandas  0.24.2
```
安装低版本 pandas

```
(supersetenv) [root@localhost bin]#  pip install pandas==0.23.4
```
然后重新运行

```
(supersetenv) [root@localhost bin]# fabmanager create-admin --app superset
```
有其他错误的对比Package版本
```
(supersetenv) [root@localhost bin]# pip list
DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that date. A future version of pip will drop support for Python 2.7.
Package          Version    
---------------- -----------
alembic          1.0.10     
amqp             2.4.2      
asn1crypto       0.24.0     
attrs            19.1.0     
Babel            2.6.0      
backports-abc    0.5        
billiard         3.6.0.0    
bleach           3.1.0      
boto3            1.4.7      
botocore         1.7.48     
cchardet         2.1.4      
celery           4.3.0      
certifi          2019.3.9   
cffi             1.12.3     
chardet          3.0.4      
click            6.7        
colorama         0.4.1      
contextlib2      0.5.5      
cryptography     2.6.1      
docutils         0.14       
enum34           1.1.6      
et-xmlfile       1.0.1      
Flask            0.12.4     
Flask-AppBuilder 1.10.0     
Flask-Babel      0.11.1     
Flask-Caching    1.7.1      
Flask-Compress   1.4.0      
Flask-Login      0.2.11     
Flask-Migrate    2.4.0      
Flask-OpenID     1.2.5      
Flask-Script     2.0.6      
Flask-SQLAlchemy 2.1        
Flask-Testing    0.7.1      
Flask-WTF        0.14.2     
flower           0.9.3      
functools32      3.2.3.post2
future           0.16.0     
futures          3.2.0      
geographiclib    1.49       
geopy            1.19.0     
gunicorn         19.9.0     
humanize         0.5.1      
idna             2.8        
ijson            2.3        
ipaddress        1.0.22     
isodate          0.6.0      
itsdangerous     1.1.0      
jdcal            1.4.1      
Jinja2           2.10.1     
jmespath         0.9.4      
jsonlines        1.2.0      
jsonschema       3.0.1      
kombu            4.5.0      
linear-tsv       1.1.0      
Mako             1.0.9      
Markdown         2.6.11     
MarkupSafe       1.1.1      
mysqlclient      1.4.2.post1
numpy            1.16.3     
openpyxl         2.4.11     
pandas           0.23.4     
parsedatetime    2.4        
pathlib2         2.3.3      
pip              19.1.1     
polyline         1.3.2      
pycparser        2.19       
pydruid          0.5.2      
PyHive           0.6.1      
pyrsistent       0.15.1     
python-dateutil  2.8.0      
python-editor    1.0.4      
python-geohash   0.8.5      
python-openid    2.2.5      
pytz             2019.1     
PyYAML           5.1        
requests         2.21.0     
rfc3986          1.3.1      
s3transfer       0.1.13     
sasl             0.2.1      
scandir          1.10.0     
setuptools       41.0.1     
simplejson       3.16.0     
singledispatch   3.4.0.3    
six              1.12.0     
SQLAlchemy       1.3.3      
SQLAlchemy-Utils 0.33.11    
sqlparse         0.3.0      
superset         0.26.3     
tableschema      1.4.1      
tabulator        1.20.0     
thrift           0.11.0     
thrift-sasl      0.3.0      
tornado          5.1.1      
unicodecsv       0.14.1     
Unidecode        1.0.23     
urllib3          1.24.3     
vine             1.3.0      
webencodings     0.5.1      
Werkzeug         0.15.2     
wheel            0.33.2     
WTForms          2.2.1      
xlrd             1.2.0
```
## 4.运行Superset

初始化数据

```
(supersetenv) [root@localhost bin]# superset db upgrade
```
启动superset
```
(supersetenv) [root@localhost bin]# superset runserver
```
或者指定端口
```
(supersetenv) [root@localhost bin]# superset runserver -p 8388 &
```
登录Superset

```
http://IP:8088/
admin/admin
```
## 5.Superset SQLlab权限

```
can list on DatabaseView,
can show on DatabaseView,
can list on DatabaseAsync,
can show on DatabaseAsync,
can results on Superset,
can tables on Superset,
can datasources on Superset,
can table on Superset,
can csv on Superset,
can fetch datasource metadata on Superset,
can stop query on Superset,
can sqllab on Superset,
can schemas on Superset,
can sql json on Superset,
can list on QueryView,
can list on TableModelView
```

## 6.其他事项

```
--防火墙
关闭防火墙：systemctl stop firewalld.service
开机禁用防火墙：systemctl disable firewalld.service
启动防火墙：systemctl start firewalld.service
开启启动防火墙： systemctl enable firewalld.service

--虚拟环境
启动虚拟环境：source /opt/supersetenv/bin/activate
(supersetenv) [root@localhost supersetenv]# 
退出虚拟环境：deactivate
```

## 7.参考文献
- [centos7搭建superset数据平台](https://blog.51cto.com/14033037/2331657)
- [Superset安装出错](https://www.jianshu.com/p/d1d3946a426f)
- [superset权限管控](https://blog.csdn.net/Alongpo/article/details/89210885)