---
title: "Hive笔记(一)"
date: 2020-05-07T09:34:52+08:00
draft: true
---

## Hive概述
#### 什么是Hive ?
- Hives是基于Hadoop构建的一个数据仓库工具
- 可以将结构化的数据映射为一张数据库表
- 提供HQL(HiveSQL)查询功能
- 由Facebook实现并开源
- 底层数据存储在HDFS上
- Hive的本质是将SQL语句转换为MapReduce任务运行
- 使不熟悉MapReduce的用户很方便的利用HQL处理和计算HDFS上的结构化数据,适用于离线的批量数据计算 

> Hive 依赖于HDFS存储数据,Hive将HQL转换成MapReduce执行,所以说Hive是基于Hadoop的一个数据仓库工具,实质就是一款基于HDFS的MapReduce的计算框架,对存储在HDFS中的数据进行分析和管理
> ![image-20200507093523719](https://i.loli.net/2020/05/07/lxPpMBcqtogFKJA.png)
#### 为什么使用Hive ?
- 友好的接口 : 操作接口采用类似SQL的语法,提供快速开发的能力
- 低学习成本 : 避免了写MapReduce,减少开发人员的学习成本
- 好的扩展性 : 可自由的扩展集群规模而无需重启服务,支持用户自定义函数
#### Hive的特点 
##### 优点 :
1. 可扩展性
1. 简化MR开发
1. 自定义函数,格式
1. 庞大活跃的社区
##### 缺点 :
1. 不支持记录级别的增删改查操作
1. 查询延时严重
1. 不支持事物

#### Hive与RDBMS 的对比
![image-20200507093735420](https://i.loli.net/2020/05/07/JYEKA4lmayqt8HG.png)

#### 数据表(Tables)
- 分为内部表和外部表
- 内部表(管理表)
  - HDFS中为所属数据库目录下的子文件夹
  - 数据完全由Hive管理,删除表(元数据) 会删除数据
- 外部表 (External Tables)
   - 数据保存在指定位置的HDFS路径中
   - Hive不完全管理数据,删除表(元数据)不会删除数据
#### 内部表和外部表的区别
- 删除内部表，删除表元数据和数据
- 删除外部表，删除元数据，不删除数据

#### 内部表和外部表的使用选择
如果数据的所有处理都在 Hive 中进行，那么倾向于 选择内部表，但是如果 Hive 和其他工具要针对相同的数据集进行处理，外部表更合适。

#### 总结
Hive 具有 SQL 
> 数据库的外表，但应用场景完全不同，==Hive 只适合用来做海量离线数 据统计分析，也就是数据仓库。==

## Hive 基本操作
> beeline方式启动 : 
```
./hiveserver2 &
beeline -u jdbc:hive2://localhost:10000
```
#### 显示所有数据库
```
0: jdbc:hive2://hadoop:10000> show databases;
OK
+----------------+--+
| database_name  |
+----------------+--+
| default        |
+----------------+--+
1 row selected (0.046 seconds)
0: jdbc:hive2://hadoop:10000> 
```
#### 创建数据库
```
0: jdbc:hive2://hadoop:10000> create database test;
OK
No rows affected (0.24 seconds)
0: jdbc:hive2://hadoop:10000> 
```
#### 查看数据库描述
```
0: jdbc:hive2://hadoop:10000> desc database myhivebook;
OK
+-------------+----------+----------------------------------------------------------------+-------------+-------------+-------------+--+
|   db_name   | comment  |                            location                            | owner_name  | owner_type  | parameters  |
+-------------+----------+----------------------------------------------------------------+-------------+-------------+-------------+--+
| myhivebook  |          | hdfs://192.168.239.100:9000/user/hive/warehouse/myhivebook.db  | anonymous   | USER        |             |
+-------------+----------+----------------------------------------------------------------+-------------+-------------+-------------+--+
1 row selected (0.036 seconds)
0: jdbc:hive2://hadoop:10000> 

```
#### 删除数据库
```
0: jdbc:hive2://hadoop:10000> drop database if exists test;
OK
No rows affected (0.274 seconds)
0: jdbc:hive2://hadoop:10000> 
```
#### 显示当前数据库
```
0: jdbc:hive2://hadoop:10000> select current_database();
+----------+--+
|   _c0    |
+----------+--+
| default  |
+----------+--+
1 row selected (6.289 seconds)
0: jdbc:hive2://hadoop:10000> 

```
```
hive (default)> set hive.cli.print.current.db=true;
hive (default)> show databases;
OK
default
myhivebook
Time taken: 0.011 seconds, Fetched: 2 row(s)
hive (default)> 
```
> 效果: hive(default)> 会把数据库名称显示在右边

#### 创建表
```
CREATE TABLE event_attendees
            (
                    event string,yes string,maybe string,invited string
            )
            ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
            WITH SERDEPROPERTIES 
            (
               "separatorChar" = ",",
               "quoteChar"     = "\""
            )       
            TBLPROPERTIES ("skip.header.line.count"="1");
            
            
示例 : 
CREATE EXTERNAL TABLE IF NOT EXISTS employee_external (
name string,
work_place ARRAY<string>,
sex_age STRUCT<sex:string,age:int>,
skills_score MAP<string,int>,
depart_title MAP<STRING,ARRAY<STRING>>
)
COMMENT 'This is an external table'
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':'
STORED AS TEXTFILE
LOCATION '/user/root/employee'; 

comment : 可选 描述
row format delimited 
fields terminated by '|' : 如何分割列(字段)
collection items terminated by ',' : 如何分割集合和映射
if not exists : 可选,如果表存在,则忽略
external : 外部表
location : 数据存储路径(HDFS)
stored as testfile : 文件存储格式
```


```
0: jdbc:hive2://hadoop:10000> create table student(id int,name string )row format delimited fields terminated by ',' stored as textfile;
No rows affected (2.822 seconds)
0: jdbc:hive2://hadoop:10000> show tables;
+-----------+--+
| tab_name  |
+-----------+--+
| student   |
+-----------+--+
1 row selected (0.104 seconds)
0: jdbc:hive2://hadoop:10000> select * from student;
+-------------+---------------+--+
| student.id  | student.name  |
+-------------+---------------+--+
+-------------+---------------+--+
No rows selected (2.021 seconds)
```
## Hive 高级查询

#### MapJoin
- 小表关联大表
- 可进行不等值连接
```
set hive.auto.convert.join = true(默认值)
```
##### MAPJOIN 操作不支持:
- 在UNION ALL,LATERAL VIEW,GROUP BY/JOIN/SORT BY/CLUSTER BY/DISTRIBUTE BY等操作后面
- 在UNION,JOIN,以及其他MAOJOIN之前

#### 插入数据
> 向hive中加载数据(local模式) 几种方法
```
load data local inpath '/home/hadoop/data/employees.txt' into table employee;
```
#### 数据交互
- import和export用于数据导入和到处
  - 常用于数据迁移创景
  - 除数据库,可导入导出所有数据和元数据
> 使用export导出数据
```
export table 表名 TO '/tmp/output3';
```
> 使用import导入数据
```
improt table 表名 from '/tmp/output3';
```