---
title: oracle进程管理
date: 2014-04-16
author: 杨金坤
tags:
  - SQL
categories:
  - Oracle
thumbnail: img/thumbnail/1.jpg
blogexcerpt:
    查找ORACLE被锁表，存储过程SQL，解除锁进程SQL。
---

## 查找被锁表
Select s.username, i.type, decode(i.type,'TM','TABLE LOCK','TX','ROW LOCK',NULL) LOCK_LEVEL,O.OWNER,O.OBJECT_NAME,O.OBJECT_TYPE,S.SID,S.SERIAL#,S.TERMINAL,S.MACHINE,S.PROGRAM,S.OSUSER,S.STATUs from v$session s,V$LOCK i,DBA_OBJECTS O
where i.sid = s.sid and
i.id1 = o.object_id(+)
and s.username is not null order by i.type

## 杀死锁表进程
alter system kill session '24,37651'

## 查找用户进程
Select * from v$session s where username = 'BBPT' and  UPPER(ACTION) LIKE '%BBPT_NEWCUSTRISK_VERIFY'

## 查找被锁存储过程
select * from v$db_object_cache where type like 'PROC%' AND LOCKS >0 AND PINS > 0