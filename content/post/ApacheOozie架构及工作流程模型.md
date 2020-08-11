---
title: "ApacheOozie架构及工作流程模型"
date: 2020-05-07T10:57:14+08:00
draft: true
---

# Apache Oozie架构及工作流程模型

## 一 Oozie概述

### 1 Oozie是什么

- Hadoop的工作流运行引擎和调度程序
  - MapReduce作业
  - Spark（Streaming）作业
  - Hive 作业
- Apache顶级项目
  - 打包主要在Hadoop发行版中
  - http://oozie.apache.org/docs/4.2.0/index.htm
- 提供工作流管理与协调器
- 管理Actions的有向无环图（DAG）

### 2 Oozie架构

![image-20200507105750081](https://i.loli.net/2020/05/07/Bo9HtzRrIeWjC26.png)

- 运行HTTP服务
  - Clients通过提交workflows与服务进行交互
  - 工作流程立即执行或以后执行
- 通过XML定义工作流程
  - 无需编写实现工具接口和扩展Configured类的Java代码
- 四种访问Oozie的方式
  - CML
  - Java API
  - Rest API
  - Web UI

### 3 Oozie组件

- 工作流定义：这些被表示为有向非循环图（DAG），以指定要执行的动作序列
  - workflow.xml
- Property file：用于workflow的参数
  - job.properties
- 库：在lib文件夹中（可选）
- 协调器：调度程序（可选）
  - coordinator.xml
- 捆绑包：协调程序和工作流程的包（可选）

#### 3.1 组件之间的关系

- 一个bundle job可以可以对应多个协调器作业
- 一个协调器作业可以对应多个workflow作业
- 一个workflow作业可以对应多个actions
- 一个workflow流程可以有零个或多个子workflow

#### 3.2 view层中的Oozie组件

![image-20200507105808165](https://i.loli.net/2020/05/07/qavrLNnTEpG8YDO.png)

### 4 Oozie优缺点

#### 4.1 Oozie优点

- 强大的社区支持
- 与HUE，Cloudera Manager，Apache Ambari集成
- HCatalog集成
- SLA警报（Oozie 4中的新增功能）
  - Workflow, Action 等级
- 生态系统支持：M/R，Hive，Spark，Sqoop，Java等
- 非常详细的文档
- 启动器作业作为map tasks

#### 4.2 Oozie 缺点

- 启动器作业作为map tasks
- UI界面不友好—但是HUE，oozie-web（和良好的API）可以补充
- 令人困惑的对象模型（bundles, coordinators, workflows）
- 安装困难—extjs, hadoop proxy user, RDBMS
- XML！

## 二 Oozie Workflow

- 一个工作流由Action节点和Control节点组成

  - Action节点：执行工作流程的任务
  - Control节点：管理工作流程执行

  ![image-20200507105821028](https://i.loli.net/2020/05/07/ubsIq9A46lrfBeG.png)



### 1 Workflow定义

![image-20200507105825645](https://i.loli.net/2020/05/07/3QxplfmXKbyuHBe.png)

### 2 Workflow Actions概述

- Oozie Workflow Actions是异步的
  - Work工作流操作中的任务是异步执行的
  - flow工作流程会等到所有操作完成后再过渡到下一个节点
  - fs action 是一个例外，它被视为同步操作
- Actions具有两个转换：OK 或 Error 
- 动作恢复
  - 如果action已经开始，则不会触发任何重试
  - 如果故障是短暂错误，例如网络错误，Oozie将在预定时间间隔后重试采取的行动；
  - 如果不是临时错误而导致失败，Ozzie将中止工作流程作业，此后将需要人工干预

### 3 Actions

#### 3.1 MapReduce Action

在应用程序目录中包含.jar文件的目录应命名为lib

![image-20200507105833174](https://i.loli.net/2020/05/07/9lU7BKmayNHjepf.png)

#### 3.2 FS (HDFS) Action

- fs操作允许在HDFS中操作文件和目录：移动，删除，mkdir，chmod，touchz和chgrp
- FS命令是从FS Action中同步执行的，工作流作业将等待直到指定的文件命令完成后再继续执行下一个操作

```xml
<workflow-app name="sample-wf" xmlns="uri:oozie:workflow:0.5">
...
<action name="hdfscommands">
<fs>
<delete path='hdfs://foo:8020/usr/tucu/temp-data'/>
<mkdir path='archives/${wf:id()}'/>
<move source='${jobInput}' target='archives/${wf:id()}/processed-input'/>
<chmod path='${jobOutput}' permissions='-rwxrw-rw-' dir-files='true'><recursive/></chmod>
<chgrp path='${jobOutput}' group='testgroup' dir-files='true'><recursive/></chgrp>
</fs>
<ok to="myotherjob"/>
<error to="errorcleanup"/>
</action>
...
</workflow-app>
```

#### 3.3 Java Action

![image-20200507105846645](https://i.loli.net/2020/05/07/pRKfyhWSZQz4en8.png)

- Java操作将执行指定的主Java类的 public static void main（String [] args）方法
- Java应用程序在Hadoop集群中作为map-reduce任务与单个Mapper任务一起执行

#### 3.4 Shell Action

![image-20200507105851921](https://i.loli.net/2020/05/07/pdzhNrlGbgA9oyT.png)

- 该动作将执行shell命令
- 输出可以限制为2k

#### 3.5 Hive Action

##### 3.5.1 script

![image-20200507105857541](https://i.loli.net/2020/05/07/2bwUT6SOJNP38mg.png)

- Script：一个element，其中包含要运行的Hive脚本的名称和相对于工作流目录的路径。 Hive脚本与应用程序打包在工作流目录下。 
- Query：v0.6中的支持

##### 3.5.2 jdbc

![image-20200507105902371](https://i.loli.net/2020/05/07/5EZjIPtmVby84HB.png)

- Jdbc-url：JDBC用户名，用于提供对JDBC驱动程序的远程访问

#### 3.6 DISCP Action

![image-20200507105909415](https://i.loli.net/2020/05/07/dKSg49oAxmJ6P2D.png)

- arg：disp命令的参数
  - -update：用于从源复制目标上不存在或与目标版本不同的文件
  - -skipcrccheck：是否在源路径和目标路径之间跳过CRC检查

#### 3.7 Spark Action

![image-20200507105915146](https://i.loli.net/2020/05/07/3XDfQo9IhJORrz8.png)

#### 3.8 Sub-Workflow Action

子工作流程作业与父工作流程作业的在同一Oozie系统实例中运行

```xml
<workflow-app name="sample-wf" xmlns="uri:oozie:workflow:0.1">
...
<action name="a">
<sub-workflow>
<app-path>child-wf</app-path>
<configuration>
<property>
<name>input.dir</name>
<value>${wf:id()}/second-mr-output</value>
</property>
</configuration>
</sub-workflow>
<ok to="end"/>
<error to="kill"/>
</action>
...
</workflow-app>
```

### 4 Workflow状态转换

![image-20200507105926109](https://i.loli.net/2020/05/07/EsHITXyjSzlF4dn.png)

#### 4.1 Start, End, Kill

##### 4.1.1 Start Control Node  

```xml
<workflow-app name="[WF-DEF-NAME]" xmlns="uri:oozie:workflow:0.1">
<start to="[NODE-NAME]"/>
...
</workflow-app>
```

##### 4.1.2 End Control Node 

```xml
<workflow-app name="[WF-DEF-NAME]" xmlns="uri:oozie:workflow:0.1">
...
<end name="[NODE-NAME]"/>
</workflow-app>
```

##### 4.1.3 Kill Control Node

```xml
<workflow-app name="foo-wf" xmlns="uri:oozie:workflow:0.1">
...
<kill name="killBecauseNoInput">
<message>Input unavailable</message>
</kill>
...
</workflow-app>
```

#### 4.2 Decision

![image-20200507105934642](https://i.loli.net/2020/05/07/7EUCzj3MLBiZ6bn.png)

#### 4.3 Fork & Join

![image-20200507105939198](https://i.loli.net/2020/05/07/5KYcGjIMxkniWTV.png)

```xml
<fork name="forking">
<path start="mrjob"/>
<path start="hivejob"/>
</fork>
<action name="mrjob">
…
<ok to="joining"/>
<error to="kill"/>
</action>
<action name="hivejob">
…
<ok to="joining"/>
<error to="kill"/>
</action>
<join name="joining" to="nextaction"/>
```

### 5 Workflow应用程序的部署和执行

1. 应用程序文件夹位于HDFS中

2. 必须定义工作流程定义在 workflow.xml中

3. 任何库都必须存储在lib子文件夹中

4. job.properties

   ```properties
   nameNode=hdfs://sandbox.hortonworks.com:8020
   jobTracker=sandbox.hortonworks.com:8050
   queueName=default
   oozie.wf.application.path=${nameNode}/application-folder
   oozie.use.system.libpath=true
   oozie.libpath=${nameNode}/user/oozie/share/lib
   ```

5. 执行

   ```sh
   oozie job --oozie http://sandbox.hortonworks.com:11000/oozie -config job.properties -run
   ```

## 三 Oozie协调器–作业调度

Oozie根据以下内容执行工作流程作业

- 时间依赖性/频率
- 数据依赖

![image-20200507105945937](https://i.loli.net/2020/05/07/Jf4nFsimWaMIV3E.png)

### 1 协调器定义概述

![image-20200507105950529](https://i.loli.net/2020/05/07/M19izVwgEP2QkWj.png)

### 2 Coordinator Details

![image-20200507105954829](https://i.loli.net/2020/05/07/uHJd5oFybXa2VlS.png)

### 3 定义

```xml
<coordinator-app name="[NAME]"
frequency="[FREQUENCY]" start="[DATETIME]" end="[DATETIME]" timezone="[TIMEZONE]"
xmlns="uri:oozie:coordinator:0.1">
<!-- … -->
</coordinator-app>

```

- start：作业的开始时间
- end：作业的结束日期时间
- timezone：协调器应用程序的时区。
- frequency：以分钟为单位的actions频率

### 4 Controls

```xml
<controls>
<timeout>[TIME_PERIOD]</timeout>
<concurrency>[CONCURRENCY]</concurrency>
<execution>[EXECUTION_STRATEGY]</execution>
</controls>

```

- timeout：实例化操作将等待其他条件以分钟为单位的最长时间
- concurrency：并发运行的实例的最大数量
- execution：指定执行顺序：
  - FIFO（最旧的优先）默认值
  - LIFO（最新的优先）
  - ONLYLAST（只有最后的）

### 5 Datasets

```xml
<datasets>
<dataset name="logs" frequency="${coord:days(1)}"
initial-instance="2009-01-02T08:00Z" timezone="America/Los_Angeles">
<uri-template>hdfs://bar:9000/app/logs/${YEAR}${MONTH}/${DAY}/data</uri-template>
<done-flag></done-flag>
</dataset>
<dataset name=“feeds" frequency="${coord:days(1)}"
initial-instance="2009-01-02T08:00Z" timezone="America/Los_Angeles">
<uri-template>hdfs://bar:9000/app/stats/${YEAR}/${MONTH}/${DAY}/data</uri-template>
</dataset>
</datasets>

```

- 数据集协调器应用程序使用
- 完成标志
  - 如果未指定，oozie将检查文件夹中的_SUCCESS文件
  - 如果设置为空，则oozie检查文件夹是否存在
  - 如果设置了文件名，请检查文件夹中文件是否存在

### 6 Input-Events

```xml
<input-events>
<data-in name="[NAME]" dataset="[DATASET]">
<instance>[INSTANCE]</instance> ...
</data-in>
<data-in name="[NAME]" dataset="[DATASET]">
<start-instance>[INSTANCE]</start-instance>
<end-instance>[INSTANCE]</end-instance>
</data-in>
</input-events>

```

- input-events：协调器作业输入事件
- data-in：定义一个可以解析为一个或多个实例数据集的作业输入条件

### 7 Output-Events

```xml
<output-events>
<data-out name="[NAME]" dataset="[DATASET]">
<instance>[INSTANCE]</instance> </data-out>
</output-events>

```

- input-events: 协调器作业输出事件
- data-in: 定义一个可以解析为一个或多个实例数据集的作业输出

### 8 协调器通用EL功能

数据集频率：使用下面的计划或按时间表计划

![image-20200507110004736](https://i.loli.net/2020/05/07/KotgdOSc48G97fn.png)

数据集事件时间：

- ${coord：current（int n）}表示n x数据集实例频率偏移
- ${coord：offset（int n，String timeUnit）}表示n个时间单位的频偏
- $ {coord：latest（int n）}表示第n个最新的当前可用实例



### 9 Data Pipeline

具有两个协调器应用程序的数据管道

- 计划每小时运行一次
- 另一个计划每天运行

![image-20200507110009578](https://i.loli.net/2020/05/07/cfLjohE2gMY5Vba.png)



### 10 协调器应用程序：部署和执行

1. 协调器文件夹位于HDFS中

2. 协调器必须定义在coordinator.xml中

3. 任何库都必须存储在lib子文件夹中

 ![image-20200507110014926](https://i.loli.net/2020/05/07/bS1N9hnPcW6Xdoj.png)

4. 相关的工作流程应该已经定义并部署

5. 执行

   ```sh
   oozie job --oozie http://sandbox.hortonworks.com:11000/oozie -config job.properties -run
   
   ```

## 四 Oozie捆绑包

### 1 Bundle概述

- **Oozie Bundle system**允许用户定义和执行一堆通常称为数据管道的协调器应用程序。捆绑软件中的协调器应用程序之间没有明确的依赖关系。 但是，用户可以使用协调程序应用程序的数据依赖性来创建隐式数据应用程序管道
- 用户将能够在捆绑软件级别中启动/停止/暂停/继续/重新运行，从而实现更好，更轻松的操作控制

- 在协调器顶部的新抽象层
- 用户可以定义和执行一系列协调器应用程序
- 捆绑包是可选的

![image-20200507110021371](upload\image-20200507110021371.png)



### 2 Bundle定义

![image-20200507110025967](https://i.loli.net/2020/05/07/iPvsSBZ7CENdMx2.png)

## 五 Oozie常用命令

-  oozie version
-  oozie job –info 
-  oozie job –kill 
-  oozie job –log 
-  oozie job –run|submit|start
-  oozie admin –status
-  oozie validate workflow.xml

## 六 Oozie安装

### 1 上传和解压

下载oozie和ext-2.2.zip，上传至/software下，解压至/opt

```sh
# tar zxvf oozie-4.1.0-cdh5.14.2.tar.gz -C /opt/

```

### 2 配置环境变量

编辑环境变量

```sh
# vi /etc/profile
------------------------------------
# oozie
export OOZIE_HOME=/opt/oozie-4.1.0-cdh5.14.2
export PATH=$OOZIE_HOME/bin:$PATH

```

刷新环境变量

```sh
# source /etc/profile

```

### 3 数据库中建表

这里提到的数据库是关系型数据库，用来存储Oozie的数据。Oozie自带一个Derby，不过Derby只能拿来实验的，这里我选择mysql作为Oozie的数据库

```sql
mysql -uroot -proot

mysql> create database oozie;
mysql> create user 'oozie' identified by 'oozie';
mysql> grant all privileges on oozie.* to 'oozie'@'localhost' identified by 'oozie' with grant option;
mysql> grant all privileges on oozie.* to 'oozie'@'%' identified by 'oozie' with grant option;
mysql> flush privileges;

```

### 4 编辑配置文件

编辑 oozie-site.xml 配置mysql的连接属性

```sh
# cd /opt/oozie-4.1.0-cdh5.14.2/conf/
# cp oozie-site.xml oozie-site.xml.bak
# vi oozie-site.xml

```

```xml
<?xml version="1.0"?>
<configuration>
    <property>
		<name>oozie.services</name> 
		<value>
			org.apache.oozie.service.SchedulerService,
            org.apache.oozie.service.InstrumentationService,
            org.apache.oozie.service.MemoryLocksService,
            org.apache.oozie.service.UUIDService,
            org.apache.oozie.service.ELService,
            org.apache.oozie.service.AuthorizationService,
            org.apache.oozie.service.UserGroupInformationService,
            org.apache.oozie.service.HadoopAccessorService,
            org.apache.oozie.service.JobsConcurrencyService,
            org.apache.oozie.service.URIHandlerService,
            org.apache.oozie.service.DagXLogInfoService,
            org.apache.oozie.service.SchemaService,
            org.apache.oozie.service.LiteWorkflowAppService,
            org.apache.oozie.service.JPAService,
            org.apache.oozie.service.StoreService,
            org.apache.oozie.service.SLAStoreService,
            org.apache.oozie.service.DBLiteWorkflowStoreService,
            org.apache.oozie.service.CallbackService,
            org.apache.oozie.service.ActionService,
            org.apache.oozie.service.ShareLibService,
            org.apache.oozie.service.CallableQueueService,
            org.apache.oozie.service.ActionCheckerService,
            org.apache.oozie.service.RecoveryService,
            org.apache.oozie.service.PurgeService,
            org.apache.oozie.service.CoordinatorEngineService,
            org.apache.oozie.service.BundleEngineService,
            org.apache.oozie.service.DagEngineService,
            org.apache.oozie.service.CoordMaterializeTriggerService,
            org.apache.oozie.service.StatusTransitService,
            org.apache.oozie.service.PauseTransitService,
            org.apache.oozie.service.GroupsService,
            org.apache.oozie.service.ProxyUserService,
            org.apache.oozie.service.XLogStreamingService,
            org.apache.oozie.service.JvmPauseMonitorService,
            org.apache.oozie.service.SparkConfigurationService
		</value>
	</property>
	<property>
		<name>oozie.service.HadoopAccessorService.hadoop.configurations</name>
		<value>*=/opt/hadoop-2.6.0-cdh5.14.2/etc/hadoop</value>
	</property>
	<property>
		<name>oozie.service.JPAService.create.db.schema</name>
		<value>true</value>
	</property>
	<property>
		<name>oozie.service.JPAService.jdbc.driver</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>
	<property>
		<name>oozie.service.JPAService.jdbc.url</name>
		<value>jdbc:mysql://singleNode:3306/oozie?createDatabaseIfNotExist=true</value>
	</property>
	<property>
		<name>oozie.service.JPAService.jdbc.username</name>
		<value>oozie</value>
	</property>
	<property>
		<name>oozie.service.JPAService.jdbc.password</name>
		<value>oozie</value>
	</property>
	<property>
		<name>oozie.processing.timezone</name>
		<value>GMT+0800</value>
	</property>
</configuration>

```

可以在oozie-env.sh中进行参数修改，比如修改端口号，默认端口号为11000

### 5 创建扩展目录

oozie根目录创建libext文件夹，复制mysql的driver压缩包到libext文件夹中,也可以做一个软连接(libext是lib的扩展目录，好多软件都有)

```sh
# cd /opt/oozie-4.1.0-cdh5.14.2/
# mkdir libext
# cp /software/mysql-connector-java-5.1.31.jar ./libext/

```

### 6 创建oozie需要的表

创建oozie需要的表结构，命令：

```sh
ooziedb.sh create -sqlfile oozie.sql -run

```

使用mysql客户端查看是否生成oozie的数据库和表

![image-20200507110036916](https://i.loli.net/2020/05/07/346Jc1na9Yr5mRp.png)

### 7 设置hadoop代理用户

设置hadoop代理用户。`hadoop.proxyuser.root.hosts&hadoop.proxyuser.root.groups`启动或者重新启动HDFS+YARN

```xml
# vi /opt/hadoop-2.6.0-cdh5.14.2/etc/hadoop/core-site.xml
添加如下配置
-----------------------------------------------------
<property>
    <name>hadoop.proxyuser.root.hosts</name>
    <value>*</value>
</property>
<property>
    <name>hadoop.proxyuser.root.groups</name>
    <value>*</value>
</property>

```

### 8 在hdfs上创建公用文件夹

#### 8.1 执行命令

```sh
# oozie-setup.sh sharelib create -fs hdfs://singleNode:9000 -locallib /opt/oozie-4.1.0-cdh5.14.2/oozie-sharelib-4.1.0-cdh5.14.2-yarn.tar.gz

```

#### 8.2 创建war文件

**需安装unzip和zip** `yum-y install unzip;yum -y install zip`,   执行

```sh
# addtowar.sh -inputwar $OOZIE_HOME/oozie.war -outputwar $OOZIE_HOME/oozie-server/webapps/oozie.war -hadoop 2.6.0 $HADOOP_HOME -jars $OOZIE_HOME/libext/mysql-connector-java-5.1.31.jar -extjs /software/ext-2.2.zip

```

或者将hadoop相关包，mysql相关包和ext压缩包放到libext文件夹中，运行`oozie-setup.sh prepare-war`也可以创建war文件

### 9 启动oozie

运行：**oozied.sh run** 或者 **oozied.sh start**(前者在前端运行，后者在后台运行)

```
oozied.sh start

```

jps 看见bootstrap说明oozie启动起来了###使用 oozied.sh stop 停止服务

![image-20200507110048861](https://i.loli.net/2020/05/07/QDtodcmfWbJHxpO.png)

查看web界面&查看状态`oozie admin -oozie http://singleNode:11000/oozie -status `##显示normal属于正常

![image-20200507110054760](https://i.loli.net/2020/05/07/W6Z3hRFAEVSMGDI.png)

web页面查看地址：http://singleNode:11000/oozie

![image-20200507110108176](https://i.loli.net/2020/05/07/XesgSypDCYvUcRb.png)

## Oozie使用

### 1 示例一

#### 1.1 编写job.properties

```properties
nameNode=hdfs://singleNode:9000
jobTracker=singleNode:8032
queueName=default
examplesRoot=myoozie

oozie.wf.application.path=${nameNode}/user/${examplesRoot}/fs/workflow.xml

```

#### 1.2 编写workflow.xml

```xml
<workflow-app xmlns="uri:oozie:workflow:0.4" name="fs">
    <start to="fs-delete" />

    <action name="fs-delete">
        <fs>
            <delete path="${nameNode}/oozie.txt" />
            <delete path="${nameNode}/bin" />
        </fs>
        <ok to="fs-mkdir" />
        <error to="fail" />
    </action>


    <action name="fs-mkdir">
        <fs>
            <mkdir path="${nameNode}/kgc" />
        </fs>
        <ok to="fs-move" />
        <error to="fail" />
    </action>

    <action name="fs-move">
        <fs>
            <move source="${nameNode}/123" target="${nameNode}/456" />
        </fs>
        <ok to="end" />
        <error to="fail" />
    </action>

    <kill name="fail">
        <message>fs action failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>

    <end name="end" />
</workflow-app>

```

#### 1.3 在HDFS创建目录

```sh
# hdfs dfs -put /etc/passwd /oozie.txt
# hdfs dfs -mkdir /bin
# hdfs dfs -touchz /123
# hdfs dfs -mkdir /456

```

#### 1.4 上传workflow.xml

```sh
# hdfs dfs -mkdir -p /user/myoozie/fs
# hdfs dfs -put workflow.xml /user/myoozie/fs

```

#### 1.5 开启日志聚集

```sh
# mr-jobhistory-daemon.sh start historyserver

```

#### 1.6 检查workflow是否有效

```sh
# oozie validate workflow.xml
如果报如下异常，需要导入OOzie的url
------------------------------
java.lang.IllegalArgumentException: Oozie URL is not available neither in command option or in the environment
        at org.apache.oozie.cli.OozieCLI.getOozieUrl(OozieCLI.java:702)
        at org.apache.oozie.cli.OozieCLI.createXOozieClient(OozieCLI.java:907)
        at org.apache.oozie.cli.OozieCLI.validateCommand(OozieCLI.java:1951)
        at org.apache.oozie.cli.OozieCLI.processCommand(OozieCLI.java:676)
        at org.apache.oozie.cli.OozieCLI.run(OozieCLI.java:617)
        at org.apache.oozie.cli.OozieCLI.main(OozieCLI.java:218)
Oozie URL is not available neither in command option or in the environment
---------------------------------------
# export OOZIE_URL=http://singleNode:11000/oozie

```

#### 1.7 执行程序

```sh
# oozie job -oozie http://singleNode:11000/oozie -config job.properties -run

```

#### 1.8 查看web界面

![image-20200507110116505](https://i.loli.net/2020/05/07/yEcsGfTPYz84Dnx.png)

## 七 常见问题

### sandbox安装oozie扩展

```sh
wget http://public-repo-1.hortonworks.com/HDP-UTILS-GPL-1.1.0.22/repos/centos7-ppc/extjs/extjs-2.2-1.noarch.rpm

rpm -ivh extjs-2.2-1.noarch.rpm

rm /usr/hdp/current/oozie-server/.prepare_war_cmd

# 重启Oozie

```