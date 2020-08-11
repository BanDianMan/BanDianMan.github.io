---
title: "ApacheFulme基础及案例"
date: 2020-05-07T10:49:38+08:00
draft: true
---

### Apache Flume 基础及使用案例

#### 什么是Flume ? 

- Flume 是用于从多个源将日志流到Hadoop和其他目标的服务。
- 一种分布式的、可靠的、可用的服务，用于有效地收集、聚合和移动大量的流数据到Hadoop分布式文件系统(HDFS)。

- Apache Flume具有简单灵活的基于流数据流的体系结构;并且具有可调的故障转移和恢复可靠性机制，具有健壮性和容错性。

#### Apahe Flume是做什么的

- 流数据
  - 从多个源获取流式数据到Hadoop中存储和分析
- 隔离系统
  - 缓冲存储平台的暂态峰值，当传入数据的速率超过数据可以写入目的地的速率

- 保证数据交付
  - 使用基于通道的事务来保证可靠的消息传递。
- 规模水平
  - 根据需要摄取新的数据流和额外的数据量。

####  简单的流动 

			Apache Flume最初是由Cloudera开发的，目的是提供一种快速、可靠地将web服务器生成的大量日志文件流到Hadoop中的方法。

![image-20200507105158243](https://i.loli.net/2020/05/07/Qgus9J1zGxrba8D.png)

#### Apahe Flume 架构

-  Flume作为一种或多种agents部署; 
-  Flume agents 是一个JVM进程，它承载组件，事件通过组件从外部源流向下一个目的地。  
-  每个代理包含三个组件
   - Source(s) , Channel(s) and Sink

![image-20200507105204939](https://i.loli.net/2020/05/07/hAWpEbPJZ1Q8TIy.png)

#### 多个代理架构

![image-20200507105214267](https://i.loli.net/2020/05/07/19IiXuGoDWTMKmB.png)

#### 事务

- Transaction接口是Flume可靠性的基础
- 事务在通道中实现。
  - 连接到通道的每个源和接收器都必须获得一个事务对象

![image-20200507105224278](https://i.loli.net/2020/05/07/IP31ubUVlz7ySoX.png)

#### Apache Flume 事件

-  事件是通过系统传递的单个数据包(源-->信道-->接收器); 

-  在日志文件术语中，事件是后跟新行字符的文本行。 

#### Apache Flume 源

- 一个Apache Flume源
  -  侦听来自外部应用程序的事件，如web服务器位置收集器、传感器: 
     - 读取数据
     - 翻译活动
     - 处理失败 
  -  不存储事件
  -  向通道发送事件

- 内置api的资源
  -  Avro源码、Spooling-Directory源码、Syslog源码、HTTP源码、Twitter sNetcat源码等。 
  -  Exec source—在STDOUT上生成数据的Unix命令 

####  假脱机目录源 

-  假脱机目录源 
   -  监视新文件的指定目录; 
   -  解析新文件中出现的事件;
   -  在一个文件被完全处理之后，它会被重命名以指示完成(或可选地删除)

#### Apache Flume 通道

- 一个通道
  -  是代理之间的通信桥梁之间的通信桥梁。 
  -  存储事件，直到它们被Flume消耗 
- 与built-upport渠道: 
  -  内存通道 
  -  文件通道(带检查点) 
  -  JDBC通道
  -  Spill-able内存通道 
  -  伪事务通道 

#### Apache Flume Sink

- 一个Sink
  - 从通道中移除事件
  - 将其放入外部存储库(如HDFS)或将其转发到流中的下一个Flume代理的Flume源 
- 给定代理中的Flume源和接收器与通道中暂存的事件异步运行 
- 内置支持的Sink
  -  		HDFS Sink, Logger Sink, Avro Sink, Thrift Sink, File-Roll Sink, Null Sink, HBaseSink, ElasticSearch Sink, IRC Sink, MorphlineSolr Sink, etc  

#### HDFS Sink

- HDFS Sink :
  - 将事件写入到HDFS
  - 支持多种文件格式-文本，Avro等。
- 翻转属性:

![image-20200507105234382](https://i.loli.net/2020/05/07/5XZP4UQC6avwVHi.png)

#### Flume + Kafka (1)

- Kafka Channels
  - 在代理中，Kafka(一个主题)用作通道

![image-20200507105240196](https://i.loli.net/2020/05/07/lsWf8uyFzgbHqpE.png)

- Bottleneck : HDFS Sink

![image-20200507105250080](https://i.loli.net/2020/05/07/9cNrRmhgkTz6yCM.png)

#### Flume + Kafka(2)

- Kafka Souce % Sink
  - 在Flume 中，Kafka被用作源和/或接收器

![image-20200507105259461](https://i.loli.net/2020/05/07/y4YDHW7wU1CaS25.png)

- Bottleneck - Kafka Source & HDFS Sink

![image-20200507105305256](https://i.loli.net/2020/05/07/rjzSw8OgIdCJikK.png)

#### Apache Flume 拦截器

- 拦截器可以根据任何条件修改甚至删除事件。
- Flume 支持链接的拦截器。
- 拦截器是指将。apache。flum。interceptor。interceptor 

#### Flume 自定义组件

-  除了内置组件外，还可以用Java创建自定义源、hannel和sink。 

![image-20200507105315540](https://i.loli.net/2020/05/07/HvDz7pI2jghlVai.png)

#### Sink处理器

- Sink 处理器 :
  - 接收组允许用户将多个接收分组到一个实体中。
  - 接收器处理器可用于在组内所有接收器上提供负载平衡功能，或在临时故障时实现从一个接收器到另一个接收器的故障转移

![image-20200507105321596](https://i.loli.net/2020/05/07/cD4CjoMKsvp3Vyk.png)

#### 当Flume不合适  ? 

- 非常大的事件
  - 事件不能大于代理机器上的内存或磁盘
- 罕见的健硕的负载 
  - 其他工具可能更好，例如HDFS文件Slurper

#### 适用场景

-  在工厂里，有许多机器，每台机器生产大量的原木。Flume可以用来收集机器状态，产品质量分析的日志。 
-  在电子商务中，Flume可以用来收集web日志，以了解客户的浏览/购物行为。
-  在社交网络中，Flume可用于收集tweet、聊天、信息以进行情感分析。

#### 运行机制

Flume系统中核心的角色是**agent**，agent本身是一个Java进程，一般运行在日志收集节点。
 每一个agent相当于一个数据传递员，内部有三个组件：
 Source：采集源，用于跟数据源对接，以获取数据；

Sink：下沉地，采集数据的传送目的，用于往下一级agent传递数据或者往最终存储系统传递数据；
 Channel：agent内部的数据传输通道，用于从source将数据传递到sink；

在整个数据的传输的过程中，流动的是**event**，它是Flume内部数据传输的最基本单元。event将传输的数据进行封装。如果是文本文件，通常是一行记录，event也是事务的基本单位。event从source，流向channel，再到sink，本身为一个字节数组，并可携带headers(头信息)信息。event代表着一个数据的最小完整单元，从外部数据源来，向外部的目的地去。

一个完整的event包括：event headers、event body、event信息，其中event信息就是flume收集到的日记记录。

#### 环境搭建

> 1. 上传安装包

![image-20200507105330616](https://i.loli.net/2020/05/07/BKRPaiyq8UnIWGg.png)

> 2. 解压 并 设置环境变量

```shell
tar -zxvf apache-flume-1.8.0-bin.tar.gz -C /opt
```

```shell
vi /etc/profile
```

```shell
# Flume
export FLUME_HOME=/opt/apache-flume-1.8.0-bin
export PATH=$PATH:$FLUME_HOME/bin
```

```shell
source /etc/profile
```

> 3. 设置JAVA_HOME

```shell
cp flume-env.sh.template flume-env.sh
```

![image-20200507105336960](https://i.loli.net/2020/05/07/CBkWAmxKYqoQG3L.png)

> 4. 查看版本号

```shell
flume-ng version
```

![image-20200507105342243](https://i.loli.net/2020/05/07/SFDod24RxB5AMLN.png)

####  Flume监听端口  netcat

安装telnet

```shell
yum search telnet
yum install telnet.x86_64
```

![image-20200507105352845](https://i.loli.net/2020/05/07/Vani8NpAFCxTvSc.png)

写配置文件  flumejob_telnet.conf 

```shell
#smple.conf: A single-node Flume configuration

# Name the components on this agent 定义变量方便调用 加s可以有多个此角色
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source 描述source角色 进行内容定制
# 此配置属于tcp source 必须是netcat类型
a1.sources.r1.type = netcat 
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink 输出日志文件
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory（file） 使用内存 总大小1000 每次传输100
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel 一个source可以绑定多个channel 
# 一个sinks可以只能绑定一个channel  使用的是图二的模型
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

>  放置在flume/conf/下 

启动

```
flume-ng agent --conf conf/ --name a1 --conf-file conf/flumejob_telnet.conf -Dflume.root.logger=INFO,console
```

发送数据

```shell
telnet localhost 44444
```

![image-20200507105400992](https://i.loli.net/2020/05/07/abgyxK9wip1L6WQ.png)

查看数据

![image-20200507105406182](https://i.loli.net/2020/05/07/gZ6XKqfoeBrS1wn.png)

####  实时的采集文件到HDFS  exec sourece

 写配置文件 flumejob_hdfs.conf 

```shell
# Name the components on this agent 
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source 
# exec 执行一个命令的方式去查看文件 tail -F 实时查看
a1.sources.r1.type = exec
# 要执行的脚本command tail -F 默认10行 man tail  查看帮助
a1.sources.r1.command = tail -F /tmp/root/hive.log
a1.sources.r1.channels = c1

# Describe the sink 
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://192.168.229.100:9000/flume/%Y%m%d/%H
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = logs-
#是否按照时间滚动文件夹
a1.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹  秒 （默认30s）
a1.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位（每小时滚动一个文件夹）
a1.sinks.k1.hdfs.roundUnit = minute
#是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个 Event 才 flush 到 HDFS 一次
a1.sinks.k1.hdfs.batchSize = 500
#设置文件类型，可支持压缩
a1.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件 秒
a1.sinks.k1.hdfs.rollInterval = 30
#设置每个文件的滚动大小 字节（最好128M）
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与 Event 数量无关
a1.sinks.k1.hdfs.rollCount = 0
#最小冗余数(备份数 生成滚动功能则生效roll hadoop本身有此功能 无需配置) 1份 不冗余
a1.sinks.k1.hdfs.minBlockReplicas = 1

# Use a channel which buffers events in memory 
a1.channels.c1.type = memory 
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

启动

```shell
flume-ng agent --conf conf/ --name a1 --conf-file conf/flumejob_hdfs.conf
```

操作Hive端

![image-20200507105414554](https://i.loli.net/2020/05/07/JVmfCXyw2MeOoZp.png)

 报错日志将存储到hdfs中 

![image-20200507105420468](https://i.loli.net/2020/05/07/MkV5L6tEXw9CsFK.png)

再次查看

```
hdfs dfs -cat /flume/20200214/13/logs-.1581657539898.tmp
```

![image-20200507105429432](https://i.loli.net/2020/05/07/bfOZkJ6FiEqoAN1.png)

#### Exec source 

```shell
agent.sources = s1
agent.channels = c1
agent.sinks = sk1
# 配置source为exec,命令为tail -F
agent.sources.s1.type = exec
agent.sources.s1.command = tail -F /root/b.txt
agent.sources.s1.channels = c1
# 配置channel为内存
agent.channels.c1.type = memory
# 配置sink为logger形式，使用的channel为c1
agent.sinks.sk1.type = logger
agent.sinks.sk1.channel = c1
```

```shell
flume-ng agent --name agent -f flumeExec.cof -Dflume.root.logger=INFO,console
```

#### spooling directory source

```shell
agent.sources = s1
agent.channels = c1
agent.sinks = sk1
# 配置source为exec,命令为tail -F
agent.sources.s1.type = spooldir
agent.sources.s1.spoolDir = /root/spooldir
agent.sources.s1.channels = c1
# 配置channel为内存
agent.channels.c1.type = memory
# 配置sink为logger形式，使用的channel为c1
agent.sinks.sk1.type = logger
agent.sinks.sk1.channel = c1

----------------------------------
flume-ng agent --name agent -f flumeExec2.cof -Dflume.root.logger=INFO,console
```



 

#### 实时监听文件夹 

   写配置文件 flumejob_dir.conf 

```shell
# 定义
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = spooldir
# 监控的文件夹
a1.sources.r1.spoolDir = /root/spooldir
# 上传成功后显示后缀名 
a1.sources.r1.fileSuffix = .COMPLETED
# 如论如何 加绝对路径的文件名 默认false
a1.sources.r1.fileHeader = true

#忽略所有以.tmp 结尾的文件（正在被写入），不上传
# ^以任何开头 出现无限次 以.tmp结尾的
a1.sources.r1.ignorePattern = ([^ ]*\.tmp)

# Describe the sink 
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://192.168.229.100:9000/flume/spooldir/%Y%m%d/%H
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = spooldir-
#是否按照时间滚动文件夹
a1.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹
a1.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个 Event 才 flush 到 HDFS 一次
a1.sinks.k1.hdfs.batchSize = 50

#设置文件类型，可支持压缩
a1.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件
a1.sinks.k1.hdfs.rollInterval = 600
#设置每个文件的滚动大小大概是 128M 
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与 Event 数量无关
a1.sinks.k1.hdfs.rollCount = 0
#最小副本数
a1.sinks.k1.hdfs.minBlockReplicas = 1

# Use a channel which buffers events in memory 
a1.channels.c1.type = memory 
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1 
a1.sinks.k1.channel = c1
```

 创建/root/spooldir文件夹 

```shell
cd /root
mkdir spooldir
```

启动

```shell
flume-ng agent --conf conf/ --name a1 --conf-file conf/flumejob_dir.conf
```

将/root下的b.txt复制到spooldir目录下

```
cp -rf /root/b.txt /root/spooldir/
```

![image-20200507105441028](https://i.loli.net/2020/05/07/HIsWjREgJQpdz8x.png)

 然后查看hdfs 

![image-20200507105446080](https://i.loli.net/2020/05/07/DpAQCzPFZYT5nS9.png)

此时/flume/spooldir/20181125/20/spooldir-.1543147878160.tmp 文件中的内容就是a.txt文件中的内容，

如果此时关闭监听命令，那么spooldir-.1543147878160.tmp文件就变成spooldir-.1543147878160文件持久化到hdfs中。

![image-20200507105455732](https://i.loli.net/2020/05/07/DmlG5azCHtyF9Nj.png)
#### Flume + Kafka 

配置文件 kafka_netcat.conf

```shell
#example.conf: A single-node flume configuration
#Test Kafka Sink in netcat Source
 
#Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1
 
#Describe/configue the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444
 
#Describe the sink
#设置kafkaSink 注意大小写
a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
#设置kafka的主题topic
a1.sinks.k1.topic = kafka_netcat_test
#设置kafka 的 broker地址以及端口号
a1.sinks.k1.kafka.bootstrap.servers = localhost:9092
#设置kafka序列化方式
a1.sinks.k1.serializer.class = kafka.serializer.StringEncoder
 
#use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
 
#Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

 创建kafka topic kafka_netcat_test，保证测试的数据是发送到指定的topic中 

```shell
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafka_netcat_test
```

![image-20200507105503080](https://i.loli.net/2020/05/07/bEZiPhXcdeWCsMu.png)

 启动Kafka Consumer，指定topic是kafka_netcat_test 

```shell
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic kafka_netcat_test --from-beginning
```

 启动Flume 

```shell
flume-ng agent --conf conf/ --name a1 --conf-file conf/kafka_netcat.conf -Dflume.root.logger=INFO,console
```

向44444端口发送信息

```shell
nc localhost 4444
```

![image-20200507105512656](https://i.loli.net/2020/05/07/HXN2olwDJhMQAU5.png)

检查Kafka consummer端能不能获取到信息

![image-20200507105523096](https://i.loli.net/2020/05/07/TFixWy8Db5XZzhH.png)