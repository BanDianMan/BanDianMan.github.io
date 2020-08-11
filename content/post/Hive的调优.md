---
title: "Hive的调优"
date: 2020-05-07T10:02:47+08:00
draft: true
---

### 表的优化

1. 在表的优化中, 当数据量较大的时候常用的手段就是拆分表, 大表拆小表, 分区表, 临时表, 外部表

2. 小表和大表的join, 要把数据量小的表放在join的左边, 先进行缓存, 这样会减少表join的时候内存的消耗量

### 数据倾斜

数据倾斜产生的原因为分区之后某一个reduce运算的数据量比较小, 而某一个reduce运行的数据量比较大, 造成两个reduce处理数据不平等

#### 合理设置map数量

##### 可以影响map的数量的因素

在input文件夹中, 每一个文件就是一个map. 而input文件的数量, input文件的大小都会影响map的数量, 在mapreduce任务中, 一个切片就是一个map任务, 在Driver中设置如下:

```
FileInputFormat.setMaxInputSplitSize(job, size);
FileInputFormat.setMinInputSplitSize(job, size);
```
#### 合理设置reduce数量

设置reduce个数:

```
hive (default)> set mapreduce.job.reduces;
mapreduce.job.reduces=-1
//默认为-1, 就是不设置reduce的个数
```

根据业务自定分区规则

### 并行执行

并行执行与java多线程的异步和同步概念差不多, 在MR运行任务中, 存在很多的MR任务可以进行执行, 有些MR任务和下一个MR任务存在依赖关系, 但是有些MR任务没有依赖关系. 例如: 存在依赖关系的MR, 一个MR任务的输出就是下一个MR任务的输入. 对于没有依赖关系的MR任务可以使用并行执行, 在同一时间运行多个MR任务, 这样在运行的过程中效率就会得到提升.

可以通过以下参数来设置

1. 开启并行任务

```
hive (default)> set hive.exec.parallel;
hive.exec.parallel=false
---------------------------------------
set hive.exec.parallel=true;
```

2. 设置多少个任务可以同时运行

```
hive (default)> set hive.exec.parallel.thread.number;
hive.exec.parallel.thread.number=8
//默认值为8个任务可以同时执行
```

### 严格模式

hive中提供有严格模式, 为了防止一些查询出现不好的影响, 例如笛卡尔积, 在严格模式下是不能运行的.

默认的严格模式设置:

```
<property>
    <name>hive.mapred.mode</name>
    <value>nonstrict</value>
    <description>
      The mode in which the Hive operations are being performed. 
      In strict mode, some risky queries are not allowed to run. They include:
        Cartesian Product.
        No partition being picked up for a query.
        Comparing bigints and strings.
        Comparing bigints and doubles.
        Orderby without limit.
    </description>
  </property>
  //默认值为非严格模式 : nonstrict
```

开启严格模式 : strict

开启了严格模式会对查询语句进行一些限制:

1. 对于分区表: 必须存在where语句对分区表中的分区字段进行条件过滤, 否则不允许执行该查询.
2. 对于使用order by: 当使用orderby语句时, 必须使用limit进行限定, 由于orderby之后所有的数据都会被分到一个reduce中, 这样reduce操作的数据量太大, 可能时间过长, 导致卡死, 所以为了防止出现这种情况, 在orderby的时候必须给定limit限制, 减少reduce处理的数据量
3. 笛卡尔积查询: 在多表join中会出现笛卡尔积, 笛卡尔积灰造成内存的加大消耗, 为了防止这种情况, 禁止使用笛卡尔积查询, 同时防止误操作

### JVM重用

在hive执行计算任务的时候, 会把执行计划上传到YARN集群中进行提交, 运行MR任务, 每次进行任务的运行的时候都会开启一个JVM进程运行MR任务, 如果提交任务频繁过多, 就会造成JVM频繁的开启和关闭, 在JVM的开启和关闭的过程中会造成大量的资源浪费.

在处理小文件的时候, 由于map任务较多, 所以JVM或频繁的开启和关闭, 所以对于小文件的处理优化, 主要减少JVM开启的次数

在mapred-default.xml配置文件中有如下参数:

```
<property>
  <name>mapreduce.job.jvm.numtasks</name>
  <value>10</value>
  <description>How many tasks to run per jvm. If set to -1, there is
  no limit. 
  </description>
</property>
```

在hive中临时设置JVM重用任务的数量

```
hive (default)> set mapreduce.job.jvm.numtasks;
mapreduce.job.jvm.numtasks=1
```

### 推测执行

由于集群中的资源分配不均等, 或者说每个集群中节点硬件性能会导致某个任务运行的时间快, 或者某个任务运行时间慢, 或者某个任务运行时直接卡死.

为了防止某些任务在运行过程中拖慢了整个MR任务的进度, 在运行慢的任务节点上, 开启相同的任务, 如果时间比原来的任务运行的快, 则直接输出推测的任务.

**注意** : 推测执行分为map端的推测执行以及reduce端的推测执行

#### map端

设置开启map端推测执行的参数:

```
<property>
  <name>mapreduce.map.speculative</name>
  <value>true</value>
  <description>If true, then multiple instances of some map tasks
               may be executed in parallel.</description>
</property>
```

在hadoop中默认开启推测执行, 推测执行不是说一卡死就开启推测任务, 而是必须要运行到5%以上才开启推测执行

在hive中通过set设置

```
hive (default)> set mapreduce.map.speculative;
mapreduce.map.speculative=true
```

#### reduce端

设置开启reduce端推测执行的参数:

```
<property>
  <name>mapreduce.reduce.speculative</name>
  <value>true</value>
  <description>If true, then multiple instances of some reduce tasks
               may be executed in parallel.</description>
</property>
```
在hive中通过set设置

hive中提供可以查看hql语句的执行计划 , 在执行计划中会生成抽象语法树, 在语法树中会显示hql语句之间的以来关系以及执行过程. 通过这些执行的过程和以来关系可以对hql语句进行优化

```
explain + 执行语句
------------------------------------------------
hive (default)> explain select * from emp;
OK
Explain
STAGE DEPENDENCIES:
  Stage-0 is a root stage

STAGE PLANS:
  Stage: Stage-0
    Fetch Operator
      limit: -1
      Processor Tree:
        TableScan
          alias: emp
          Statistics: Num rows: 2 Data size: 653 Basic stats: COMPLETE Column stats: NONE
          Select Operator
            expressions: empno (type: int), ename (type: string), job (type: string), mgr (type: int), edate (type: string), sal (type: double), deptno (type: int)
            outputColumnNames: _col0, _col1, _col2, _col3, _col4, _col5, _col6
            Statistics: Num rows: 2 Data size: 653 Basic stats: COMPLETE Column stats: NONE
            ListSink

Time taken: 0.127 seconds, Fetched: 17 row(s)
```

一般来说都会把复杂语句简单化处理, 例如多表的join