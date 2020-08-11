---
title: "Cassandra基础及使用"
date: 2020-05-07T10:46:34+08:00
draft: true
---

## Cassandra基础及使用

---

![image-20200507104335808](https://i.loli.net/2020/05/07/TicC8al4vKUsZnD.png)

#### 什么是Apache Cassandra？

- Apache Cassandra是一个分布式数据库，用于管理许多商品服务器上的大量数据，同时提供高可用性服务，没有单点故障
  - 分布式分区行存储
  - 物理多数据中心本机支持
  - 连续可用性
  - 线性规模性能
  - 操作简便
  - 易于分发数据
  - ...

#### 谁在使用Cassandra？

- eBay，GitHub，GoDaddy，Netflix，Reddit，Instagram，Intuit，The Weather Channel等超过1500多家公司
- 苹果的75,000多个节点存储了超过10 PB的数据
- Netflix的2,500个节点，420 TB数据，每天超过1万亿个请求
- 中国搜索引擎Easou（270个节点，300 TB的数据，每天超过8亿个请求）
- eBay（超过100个节点，250 TB）
- ...

#### 安装并启动

`touch /etc/yum.repos.d/cassandra.repo`

`vi /etc/yum.repos.d/cassandra.repo`

```
[cassandra]
name=Apache Cassandra
baseurl=https://www.apache.org/dist/cassandra/redhat/311x/
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://www.apache.org/dist/cassandra/KEYS
```

`sudo yum install cassandra`

`service cassandra start`

`chkconfig cassandra on # 使Cassandra重新启动后自动启动`

#### Cassandra Keyspaces(键空间)

- 键空间
  - 是Cassandra中数据的最外层容器/分组，例如合理的数据库/架构
  - 定义节点上的数据复制
  - 列族–是行集合的容器，例如有理表
    - 每行包含有序的列
    - 每个键空间至少具有一个/多个列族
  - 复制因子
  - 复制品放置策略
    - 简单策略
    - 网络拓扑策略
- keyspace -> table –> column，对应关系型数据库 database -> table -> column

#### 创建 Cassandra Keyspace

- `cqlsh`

- `CREATE KEYSPACE patient WITH replication = { 'class': 'SimpleStrategy', 'replication_factor': 3 }`

- `DESCRIBE patient;`

> 检查所有keyspaces

- `DESCRIBE keyspaces`

#### Column Family

- Column Family是一个有序的列的集合
  - 列是基本数据结构
  - ![image-20200507104353828](https://i.loli.net/2020/05/07/BQJyK9HrGnEgOSC.png)
  - 超级Column
  - ![image-20200507104358286](https://i.loli.net/2020/05/07/Lzy65E8q4TJHVnt.png)

> 注意：列数，每列的名称和类型逐行变化
> ![image-20200507104404188](https://i.loli.net/2020/05/07/3zWKgUq5JPTlkrV.png)

#### 创建Cassandra表

```
CREATE TABLE patient.exam(id int,patient_id int, date timeuuid, detail text, PRIMARY KEY (id));
```

> id是partition-key

```
CREATE TABLE patient.exam(id int,patient_idint, date timeuuid, detail text, PRIMARY KEY (id, patient_id)) WITH CLUSTERING ORDER BY (patient_idASC);
```

> patient_id 是集群列

`DESCRIBE patient.exam`

#### 静态 Columns

- 使具有单个值的列为 **STATIC**

```
CREATE TABLE teammember_by_team(teamnametext,manager text static,location text static,membernametext,nationality text,position text,PRIMARY KEY ((teamname), membername));
```

```
INSERT INTO teammember_by_team(teamname, manager, location)VALUES (‘Red Bull’, ‘Christian H.’, ‘123 XYZ St.’);
```

```
INSERT INTO teammember_by_team(teamname, membername, nationality, position)VALUES (‘Red Bull’, ‘Ricciardo’, ‘Australian’, ‘driver’);

INSERT INTO teammember_by_team(teamname, membername, nationality, position)VALUES (‘Red Bull’, ‘Kvyat’, ‘Russian’, ‘driver’);
```

- 限制条件:
- 当一个表具有至少一个集群键时，该表只能包含静态列
- 指定为分区键的列不能为静态
- 使用COMPACT STORAGE选项的静态列不能用于表

#### Cassandra 二级索引

- 约束：where子句中使用的任何字段都必须是表的主键，或者必须在其上具有辅助索引
- 次要索引用于使用通常不可查询的列查询表。
  - 二级索引可能会对性能产生重大影响
  - 一般规则：索引基数少的低基数的列

```
CREATE TABLE it21.movies ( year int, name text, director text, rank int, PRIMARY KEY ((year, name), rank) );
```

> `SELECT * FROM it21.movies WHERE year = 2015 & name = ‘Free Willy’;` 正确

> `ELECT * FROM it21.movies WHERE year = 2015 & name = ‘Free Willy’ and rank = 1;`
> 正确

> `SELECT * FROM it21.movies WHERE year = 2015;`
> 错误

> `SELECT * FROM it21.movies WHERE rank = 1;`错误

> `CREATE INDEX idx_yearON it21.movies（year）;`

> `SELECT * FROM it21.movies WHERE year = 2015;`正确

#### Secondary-Index on Performance Impact

- 基础架构：Cassandra上有五台机器
- 表：Users
  - Primary Key: userId
  - Secondary-Index: userEmail
    - 索引表存储在集群中的每个节点上
- 行动：通过用户的电子邮件查询用户
- 性能影响：
  - 每台机器都必须查询自己的用户记录。 一个查询，从磁盘读取五次

#### Cassandra物化视图

- 物化视图与视图
  - 物化视图基于磁盘，并根据查询定义定期更新
  - 视图仅是虚拟的，每次访问时都运行查询定义
- 物化视图的限制：
  - 将所有源表的主键包括在实例化视图的主键中
  - 只能将一个新列添加到实例化视图的主键中
  - 排除实例化视图主键列中具有空值的行

```
CREATE TABLE players (id UUID PRIMARY KEY, name text, birthday date, country text);
SELECT * FROM players WHERE birthday = ‘2011-07-21’; 错误

CREATE MATERIALIZED VIEW mv_birthdayAS SELECT birthday, name, country FROM players WHERE birthday IS NOT NULL AND id IS NOT NULL PRIMARY KEY (birthday, id);

SELECT * FROM mv_birthdayWHERE birthday = ‘2011-07-21’; 正确
```

#### Insert / Update / Delete Data

```
use patient;
INSERT INTO exam (patient_id, id, date, detail) values (1, 1, now(), 'first exam patient 1');
INSERT INTO exam (patient_id, id, date, detail) values (1, 2, now(), 'second exam patient 1');
INSERT INTO exam (patient_id, id, date, detail) values (2, 1, now(), 'first exam patient 2');
INSERT INTO exam (patient_id, id, date, detail) values (3, 1, now(), 'first exam patient 3');

UPDATE exam SET partient_id= 12 WHERE id = 6;

DELETE FROM exam WHERE id = 123;
```

> 由于其分布式特性，Cassandra不支持RDBMS样式连接的内置功能

#### 加载CSV文件

- 启动cqlsh
  - Cassandra cqlsh需要python 2.7
  - 创建keyspace–retail_db
  - 创建categories
  - 执行cqlscripts以加载类别数据
  - 创建表–products，customers，orders，order_items
  - 加载products.csv，customers.csv，orders.csv和 order_items.csv

#### 最常用的Cassandra数据类型

![image-20200507104415956](https://i.loli.net/2020/05/07/aWmXnJDtkjgSyUi.png)

#### CQL 命令–(1)

- 修改KEYSPACE

  ```
  ALTER KEYSPACE events WITH replication = {      'class':'NetworkTopologyStrategy’,'replication_factor' : 3 }
  ```

- 删除KEYSPACE 

  ```
  DROP KEYSPACE cycling;
  ```

#### CQL 命令–(2)

```
CREATE TABLE it21.users (userIDuuid, fnametext,lnametext,email text, address text, zip int, state text, PRIMARY KEY ((userID, fname), state));
```

```
CREATE TABLE it21.events (eventIDuuid, startTimetext, userIDtext,city text, state text, PRIMARY KEY (eventID, startTime));
```

- 复合键和群集
  - 复合主键由分区键和确定群集的一个或多个其他列组成
    - 分区键确定存储数据的节点
    - 其他列确定每个分区的群集
    - 集群是一个存储引擎进程，用于对分区内的数据进行排序

#### CQL 命令–(3)

- 表更改

  ```
  ALTER TABLE it21.teams ALTER ID TYPE uuid;
  ALTER TABLE it21.players ADD first_nametext;
  ALTER TABLE it21.calendar ADD events list<text>;
  ALTER TABLE it21.customers DROP birth_year;
  ```

- 清空表

  ```
  TRUNCATE student;
  ```

- 删除表

  ```
  drop table cycling.cyclist;
  ```

#### CQL 命令–(4)

```
CREATE INDEX user_stateON it21.users (state);
CREATE INDEX ON it21.users (zip);

ALTER TABLE users ADD phones set<text>;

--index on set
CREATE INDEX ON users (phones);
↓
Select * from users where phones CONTAINS '416-xxx-9312'


ALTER TABLE users ADD titles map<text, text>;

--index on map keys
CREATE INDEX ON users (KEYS(titles))
↓
Select * from users where titles CONTAINS KEY 'IT Dept'

```

#### CQL 命令–(5)

```
CREATE TABLE airplanes (
    name text PRIMARY KEY,  manufacturer ascii, year int, machfloat
);

INSERT INTO airplanes(name, manufacturer, year, mach)
  VALUES ('P38-Lightning', 'Lockheed', 1937, 0.7);
COPY airplanes (name, manufacturer, year, mach) TO 'temp.csv';

TRUNCATE airplanes;

COPY airplanes (name, manufacturer, year, mach) FROM 'temp.csv';
```

#### CQL Functions

```
CREATE OR REPLACE FUNCTION cycling.fLog(input double) 
CALLED ON NULL INPUT       --确保该功能将始终被执行
RETURNS double LANGUAGE java AS
'return Double.valueOf(Math.log(input.doubleValue()));';

CREATE FUNCTION IF NOT EXISTS cycling.left(column TEXT, num int)
RETURNS NULL ON NULL INPUT        --确保输入值是否为空
RETURNS text                      --则函数不执行
LANGUAGE javascript AS 
$$ column.substring(0,num) $$;

SELECT left(firstname,1), lastnamefrom cycling.cyclist_name;
```

#### 组件的Cassandra层次结构

![image-20200507104435299](https://i.loli.net/2020/05/07/6rCiwnblDq8TZo3.png)

#### Cassandra 环

- 给定一组键值数据
  - 目标：在群集中的所有服务器之间平均分布数据
  - 要求：卸下/添加服务器时，最大程度地减少数据移动
  - ![image-20200507104441547](https://i.loli.net/2020/05/07/ZSjR1suDhFGE2NH.png)
- 一致的散列
  - 1.将所有服务器均匀地放置在一个圆/ ring上
  - 2.将环划分为n个范围，例如0 –359（0-2Π)
  - 3.哈希每个代码的密钥，将代码修改为n，结果是 键值将被放置
  - 4.顺时针移动该点，直到找到服务器为止，这是键值的目标位置
  - 删除服务器时，该服务器上的所有密钥都将移至下一个服务器（顺时针）
  - 添加服务器时，新服务器位置之前带有＃3结果的任何键都将从下一个服务器移至新服务器
  - 通常会为服务器分配一个令牌范围

#### Cassandra 群集

- Cassandra群集由多个环组成，每个环用于一个具有多个机架的数据中心。
  - 告密者将机器组定义为数据中心和机架
  - 需要定义种子节点以引导Cassandra集群
  - 一个节点启动时，它会寻找种子以获取有关集群中其他节点的信息
  - 在Cassandra群集中使用Gossip协议进行内部通信和故障检测。
  - 节点之间的状态信息每秒进行一次交换（默认情况下)
  - 在启动时，为每个节点分配一个令牌范围，该令牌范围确定其在群集中的位置以及该节点存储的数据范围

#### 数据分区

- 每行分配一个行键
- 每行都基于行键放置在节点上
- ![image-20200507104452973](https://i.loli.net/2020/05/07/xWqYKugXSbdOD4c.png)
- 两种基本的数据分区策略：
  - **随机分区**（默认）–使用行键的MD5哈希将行尽可能均匀地分布在所有节点上。
    - 强烈推荐
  - 有序分区–行按行键的排序顺序分布在Cassandra集群中的各个节点上
    - 热点问题

#### 设置数据分区

- 在cassandra.yaml配置文件中设置partitioneroption

  ```
  # 分区器和令牌选择 
  partitioner : org.apache.cassandra.dht.RandomPartitioner
  ```

  - Murmur3Partitioner（默认）
  - RandomPartitioner
  - ByteOrderedPartitioner

> 注意：使用partitioneroption初始化集群后。 如果不重新加载集群中的所有数据，将无法更改它

#### 数据复制

- 为确保容错能力和无单点故障，数据复制有助于在群集中的分区节点上为列族（表）中的每一行制作一个或多个副本。
  - 副本数由“复制因子”控制。
    - 通常，复制因子不应超过群集中的节点数。
  - 复制是在Cassandra中的键空间级别上控制的
    - `CREATE KEYSPACE patient WITH replication = { ‘class’: ‘SimpleStrategy’, ‘replication_factor’: 3 }`

#### 数据复制-SimpleStrategy

- 简单策略：将原始行放在由分区程序确定的节点上。 无需考虑机架或数据中心位置，其他副本行将在右侧的下一个节点上放置
  - 仅用于单个数据中心和一个机架
  - ![image-20200507104503204](https://i.loli.net/2020/05/07/JTOZl1vhay9RuXK.png)

```
CREATE KEYSPACE patient WITH replication = { ‘class’: ‘SimpleStrategy’, ‘replication_factor’: 3 }
}
```

#### 数据复制-NetworkTopologyStrategy

- 网络拓扑策略允许在数据中心的不同机架之间和/或在多个数据中心之间进行复制
  - 该策略指定每个数据中心中有多少个副本
- 在每个数据中心：
  - 原始行根据分区程序放置
  - 顺时针旋转环放置其他副本，直到找到与先前副本不同机架中的节点
  - 如果没有这样的节点，则会将其他副本放置在同一机架中
    ![image-20200507104509166](https://i.loli.net/2020/05/07/kScbsUqDvamJgiC.png)

```
CREATE KEYSPACE sales WITH replication = { ‘class’: ‘NetworkTopologyStrategy’, ‘Toronto’: 2, ‘Beijing’: 3 }
```

#### 一致性等级

- Cassandra  被设计为CAP的AP，并且是有效的。
- 一致性级别由每个客户端会话定义，可以随时更改
  - **ALL**：所有副本必须存储数据 
  - **ONE**：至少一个副本必须存储数据
  - **QUORUM**：Q个副本必须存储数据
    - Q = sum_rf_all_datacenters/ 2 + 1
  - **EACH_QUORUM**：Q副本必须在每个数据中心中存储数据。
  - **LOCAL_QUORUM**：本地数据中心中的Q个副本必须存储数据。
  - **ANY**：至少一个节点（机架和数据中心中的任何节点）必须存储数据。
    ![image-20200507104520138](https://i.loli.net/2020/05/07/hu1jDbcyAHCpRMF.png)

> 注意：通过Write CL = ALL和Read CL = ONEcqlsh：retail_db>一致性QUORUM可以实现强一致性。

#### Cassandra 写

- 客户希望将数据写入Cassandra集群
  - 客户端通过节俭协议或CQL连接到群集的协调器。
  - 根据分区键和使用的复制策略，协调器将每个变节将突变转发到所有适用的节点
  - 每个节点分别处理请求：
    - 首先将变异写入提交日志
    - 然后写到内存表
  - 协调器将等待满足一致性级别所需的适当数量的节点的响应
  - 协调员通知客户突变成功/失败。
    ![image-20200507104528637](https://i.loli.net/2020/05/07/7xFwHgKp864hmay.png)

#### Cassandra Compaction

- 如果由于节点故障导致内存表中的数据丢失，则提交日志用于回放
- 内存表刷新到磁盘时
  - 它达到了其在内存中的最大分配大小
  - 内存表可以停留在内存中的分钟数
  - 由用户手动刷新
- 内存表刷新为一个不可变的结构，称为SSTable（排序字符串表）
- 超时会创建许多SSTable（磁盘上有3个文件）。 压缩是组合SSTables的过程，以便可以在单个SSTableSSTablesMemTableCommit日志中找到相关数据
  ![image-20200507104533608](https://i.loli.net/2020/05/07/DgFYZbaBomynL2V.png)

#### 写入失败公差

- 提示切换是一项功能，当协调器无法在副本集中写入节点时，它允许协调器在本地存储数据
  - 默认情况下会启用提示的切换
    - Set hinted_handoffs_enabledproperty 以禁用/启用功能
  - 默认情况下，协调器最多可以保留3个小时的本地副本
    - Set  max_hint_window_in_msproperty 更改值
- 当发生故障的节点重新联机时，它会闲聊拥有提示的节点以将数据流式传输到其中。

#### Cassandra 读

- 必须为每个读取操作提供行键
  - 协调器使用行键确定第一个副本
  - 使用具有复制因子的复制策略来确定所有其他适用的副本
- 请求的一致性级别确定协调器在响应客户端之前等待的回复数量
  - 如果连接级别为QUORUM，复制因子为3，则协调器将等待至少两个节点的成功答复。
  - 如果答复的版本不同
    - 最新版本回复给客户端
    - 在旧数据版本的节点上进行了区域修复（导致最新数据版本）

#### 读取一致性级别/读取修复

- 每次读取操作必须提供行键

- 在读取中

  - 如果 CL = ONE 则协调器查询一个副本并返回结果

  - 如果 CL大于ONE，则协调器将比较查询的所有副本:

    - 如果所有副本相同，则协调器返回结果。

    - 如果所有副本均表示相等，则Cassandra将最大的最新版本写入不存在的任何副本节点，然后协调器返回最大的最新版本。

    - > 注意：对于QUORUM，“读取修复”会修复该查询所查询的任何节点，仅修复了查询接触的所有节点，而不是所有节点

> ![image-20200507104541012](https://i.loli.net/2020/05/07/cSfY7a6yPiZKqsj.png)