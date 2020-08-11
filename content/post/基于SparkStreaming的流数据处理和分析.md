---
title: "基于SparkStreaming的流数据处理和分析"
date: 2020-05-06T20:11:44+08:00
draft: true
---

# 基于Spark Streaming的流数据处理和分析

## 一 Spark Streaming

### 1 Spark Streaming概述

#### 1.1 实时数据处理的动机

- 以前所未有的速度创造数据
  - 来自移动，网络，社交，物联网的指数数据增长...
  - 联网设备：2012年为9B，到2020年将达到50B
  - 到2020年，超过1万亿个传感器
- 我们如何实时利用数据的价值？
  - 价值会迅速下降→立即获取价值
  - 从被动分析到直接运营
  - 解锁新的竞争优势
  - 需要全新的方法

#### 1.2 跨行业的用例

![image-20200322183944594](https://i.loli.net/2020/05/06/2C6k9gL8jyb1pHw.png)

#### 1.3 什么是Spark Streaming？

Apache Spark核心API的扩展，用于流处理。该框架提供具有**良好的容错能力、可扩展性、高通量、低延迟**的优点



![image-20200322184037733](https://i.loli.net/2020/05/06/zdUueB3I5rHYZTQ.png)

#### 1.4 流引擎对比

![image-20200322190351215](https://i.loli.net/2020/05/06/TxOeXtEFHR1bqK3.png)

#### 1.5 流处理架构

![image-20200322184057591](https://i.loli.net/2020/05/06/4cmkMbWVePyqYLj.png)

#### 1.6 微批量架构

- 传入数据作为离散流（DStream）
- 流被细分为微批。从Spark 2.3.1起延迟可达到1毫秒（在此之前大约100毫秒）
- 每个微批处理都是一个RDD –可以在批处理和流之间共享代码

![image-20200322184204654](https://i.loli.net/2020/05/06/sj9ZiQLy4RzNpUT.png)

### 2 Spark Streaming 操作

#### 2.1 Streaming Context

 Streaming Context消费Spark中的数据流，数据流输入后， Streaming Context会将数据流分成批数据

- 一个JVM中只能激活一个StreamingContext
- StreamingContext在停止后无法重新启动，但可以重新创建

![image-20200322184247007](https://i.loli.net/2020/05/06/4LKgO9wuxhI68o2.png)

#### 2.2 DStream

Discretized Stream（离散流）或DStream是Spark Streaming提供的基本抽象

![image-20200322184315079](https://i.loli.net/2020/05/06/p9hbCWfLU3KsvIz.png)

##### 2.2.1 Input DStreams 和 Receivers

 Streaming Context只能在Driver端，Receiver可以在Executor端

![image-20200322184404471](https://i.loli.net/2020/05/06/kFX7bDJMd32GQSc.png)

- Input DStreams 表示从streaming sources接收的输入数据流
- 每个Input DStreams（文件流除外）与一个Receiver对象相关联，该对象从源接收数据并将其存储在Spark的内存中以进行处理，**可以并行处理后使用union，将分开的数据集在进行合并**
- 可以在同一StreamingContext下创建多个输入DStream
- Streaming Sources
  - 基本Sources。可从Streaming API获得
    - sc.fileStream
    - sc.socketStream
  - 高级Sources
    - Kafka, , Flume, etc.

##### 2.2.2 InputStream的重点

- 在本地运行Spark-Streaming程序时，请始终使用“ local [n]”作为主URL，其中==n>receivers==，需要留下来一个线程用于处理数据
- 在集群上运行时，分配给Spark Streaming应用程序的==核心数必须大于接收者数==

##### 2.2.3 Spark Streaming的Sources

- Spark StreamingContext具有以下两种内置的创建Streaming Sources的方式

  - 第一种

  ```scala
  def textFileStream(directory: String): DStream[String]
  // Process files in directory – hdfs://namenode:8020/logs/
  ```

  - 第二种

  ```scala
  def socketTextStream(hostname: String, port: Int, storageLevel: StorageLevel
  StorageLevel.MEMORY_AND_DISK_SER_2): ReceiverInputDStream[String]
  // Create an input stream from a TCP source
  ```

- Flume Sink for Spark Streaming

  ```scala
  val ds = FlumeUtils.createPollingStream(streamCtx, [sink hostname], [sink port]);
  ```

- Kafka Consumer for Spark Streaming

  ```scala
  val ds = KafkaUtils.createStream(streamCtx, zooKeeper, consumerGrp, topicMap);
  ```

##### 2.2.4 示例Wordcount

- 使用nc打开socket连接

  ```sh
  nc -lk 9999
  ```

- 使用spark-shell输入代码

  ```scala
  import org.apache.spark._
  import org.apache.spark.streaming._
  
  // 创建SparkConf配置
  val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWC")
  // shell中只能有一个context，所以需要停掉原来的
  sc.stop()
  // 创建StreamingContext
  val ssc = new StreamingContext(conf,Seconds(3))
  // 执行Wordcount
  val lines = ssc.socketTextStream("localhost",9999)
  val words = lines.flatMap(_.split("\\s+"))
  val pairs = words.map(w=>(w,1))
  val wordCounts = pairs.reduceByKey(_+_)
  // 打印输出（默认前10行）
  wordCounts.print()
  // 启动程序
  ssc.start()
  ```

##### 2.2.5 foreachRDD

使用foreachRDD算子创建数据库连接时，创建的连接是在Driver端的，如果直接使用rdd.foreach去进行发送消息是发送不出去的，因为RDD会执行在不同的Executor端，所以需要使用foreachPartition查找到每个RDD分区所在的Executor，然后再每个Executor端上创建连接，之后再进行操作

![image-20200322184741272](https://i.loli.net/2020/05/06/tfaZyhv5mdbJV4M.png)

#### 2.3 Transformation

##### 2.3.1 Transformations 算子

1. map, flatMap
2. filter
3. count, countByValue
4. repartition
5. union, join, cogroup（cogroup会对结果进行分组，即结果中相同的key只会出现一次，value值会都放在一个组里，join却不会）
6. reduce, reduceByKey
7. transform
8. updateStateByKey

![image-20200322184618632](https://i.loli.net/2020/05/06/H2jbwEPqJeUpCNh.png)



##### 2.3.2 Transform操作

Transform操作（以及它的诸如transformWith之类的变体）==允许将任意RDD-to-RDD函数应用于Dstream==

![image-20200322184652572](https://i.loli.net/2020/05/06/qFlLyG5gn3RM6jE.png)

##### 2.3.3 UpdateStateByKey操作

updateStateByKey操作允许维持任意状态，同时不断用新信息更新它

- 定义状态-状态可以是任何数据类型
- 
- 定义状态更新功能


![image-20200322184719826](https://i.loli.net/2020/05/06/vTEgZ8ahm4cBit3.png)





#### 2.4 DataFrames和SQL操作

在流数据上使用DataFrames和SQL操作时， 一个SparkSession需要通过使用StreamingContext使用的SparkContext创建

![image-20200322184844595](https://i.loli.net/2020/05/06/YoF2DWLM7SITRl3.png)


SQL查询可以在来自不同线程的流数据定义的表上执行。因为StreamingContext不了解任何异步SQL查询，它将在查询完成之前删除旧的流数据，所以在使用SQL时，StreamingContext需要记住足够的流数据

```scala
// 最后5分钟在批处理之间共享数据
streamingContext.remember(Minutes(5))
```

#### 2.5 窗口操作

##### 2.5.1 窗口操作概述

- 任何窗口操作都需要指定两个参数
  - 窗口长度-窗口的持续时间
  - 滑动间隔-窗口操作的间隔
- countByWindow
- reduceByWindow
- reduceByKeyAndWindow
- countByValueAndWindow

![image-20200322184954897](https://i.loli.net/2020/05/06/VdAHjPReTCXqKxf.png)

##### 2.5.2 窗口操作说明

![image-20200322185021726](https://i.loli.net/2020/05/06/aGj238SeXPtYdl5.png)



#### 2.6 DStream的输出操作

- DStream的print（）方法在运行流应用程序的驱动程序节点上打印DStream中每批数据的前十个元素。这对于开发和调试很有用。
- saveAsTextFiles
- saveAsHadoopFiles
- saveAsObjectFiles
  - 每个批次间隔保存一次文件

#### 2.7 DStream缓存和持久化

DStream中的数据可以保留在内存中

- 调用persist（）方法将RDD持久保存在DStream中
- 基于窗口或基于状态的操作生成的DStream会自动保存在内存中
- 与RDD不同，DStream的默认持久性级别将数据序列化在内存中

#### 2.8 Spark Streaming 检查点

Spark Streaming Checkpointing是一种容错机制

- 检查点将保存DAG和RDD，当Spark应用程序从故障中重新启动时，它将继续进行计算
- 有两种检查点的数据
  - 元数据检查点
    - 配置-用于创建流应用程序的配置，当程序挂掉之后，从checkpoint中加载配置信息创建Streaming context
    - Stream DStream操作-定义流应用程序的DStream操作集
    - 不完整的批次-作业排队但尚未完成的批次
  - 数据检查点
    - 将生成的RDD保存到可靠的存储中
    - 有状态转换的中间RDD定期检查点到可靠的存储（例如HDFS），以切断依赖关系链

##### 2.8.1 何时启用检查点

- 使用状态转换。**如果使用updateStateByKey或reduceByKeyAndWindow，则必须提供检查点目录以允许定期的RDD检查点**
- 从运行应用程序的驱动程序故障中恢复-元数据检查点用于恢复进度信息

##### 2.8.2 如何配置检查点

```scala
streamingContext.checkpoint(checkpointDirectory)
dstream.checkpoint(checkpointInterval)
```

通常，DStream的5-10个滑动间隔的检查点间隔是一个不错的尝试

![image-20200322185222764](https://i.loli.net/2020/05/06/Hm6qQbuwxRKefJU.png)

##### 2.8.3 累加器，广播变量

累加器，广播变量无法从Spark Streaming中的检查点恢复

![image-20200322185246950](https://i.loli.net/2020/05/06/fwGv61YEQhN5eVo.png)



## 二 Spark Structured Streaming

- Structured Streaming是基于Spark SQL引擎构建的可伸缩且容错的流处理引擎
- Spark SQL引擎将负责连续不断地运行流计算，并在流数据继续到达时更新最终结果
- 系统通过检查点和预写日志来确保端到端的一次容错保证

![image-20200322185327859](https://i.loli.net/2020/05/06/TcYmO9iIPaZwC43.png)

```scala
val lines = spark.readStream.format("socket").option("host", "localhost").option("port", 9999).load()  // 这里返回的是一个DataFrame
// Split the lines into words
val words = lines.as[String].flatMap(_.split(" "))  // as[String]会将DataFrame转化为DataSet[DataSet]
// Generate running word count
val wordCounts = words.groupBy("value").count()
```



### 1 程式设计模型

- 在每个触发时间间隔内，新行都会追加到input table中
- 输出模式
  - 完整模式（Complete Mode）：整个更新结果将被写入外部存储器
  - 追加模式（Append Mode--默认）：自最后一次触发以来，只会在结果表中追加新行
  - 更新模式（Update Mode）：仅写入自上次触发以来结果表中已更新的行

![image-20200322185448310](https://i.loli.net/2020/05/06/GqyBJ9mKwbQ41Yr.png)

#### 1.1 Input Table 是无界的

![image-20200322185345185](https://i.loli.net/2020/05/06/vtQgIN4uARz6smD.png)

#### 1.2 In Action

![image-20200322185512326](https://i.loli.net/2020/05/06/LwiYsFCaz8tH3Kn.png)

```scala
// nc -lk 9999 // 打开netcat

val lines = spark.readStream.format("socket").option("host", "localhost").option("port", 9999).load()
// Split the lines into words
val words = lines.as[String].flatMap(_.split(" "))
// Generate running word count
val wordCounts = words.groupBy("value").count()
// Start running the query that prints the running counts to the console
val query = wordCounts.writeStream.outputMode("complete").format("console").start()
query.awaitTermination()
```

### 2 处理 Event-time

#### 2.1 Event-time概述

- 事件时间是数据本身产生的时间，而不是接收数据的时间。事件时间是每一行中的列值
- 基于窗口的聚合

![image-20200322185538325](https://i.loli.net/2020/05/06/2QjPTAxJErnR46p.png)



#### 2.2 示例

![image-20200322185558711](https://i.loli.net/2020/05/06/U7yBm6jR3D9FrEV.png)

```scala
val lines = spark.readStream.format("socket").option("host", "localhost").option("port", 9999).load()
// Split the lines into words
val words = lines.as[String].flatMap(_.split(" ")).withColumnRenamed("value", "word").withColumn("timestamp", current_timestamp())
val windowedCounts = words.groupBy(
window($"timestamp", "10 minutes", "5 minutes"), $"word").count()
// Start running the query that prints the running counts to the console
val query = windowedCounts.writeStream.outputMode("complete").format("console").start()
query.awaitTermination()
```



### 3 处理Late Data

- 事件时间模型允许处理晚于预期到达的数据
  - 有较晚的数据时更新旧的聚合
  - 清理旧的聚合
- Watermarking
  - 一个时间阈值，用于指示如何仍可处理晚期数据

![image-20200322185703576](https://i.loli.net/2020/05/06/uekXidP4yGWYjav.png)



### 4 使用 DataFrame 和 Dataset

![image-20200322185823704](https://i.loli.net/2020/05/06/vmNzkWPGn4FLd2l.png)

### 5 Join 操作

#### 5.1 Join Operations

![image-20200322185842801](https://i.loli.net/2020/05/06/SkaTDlPhYFR3GNe.png)

#### 5.2 Legal Join Types

![image-20200322185907535](https://i.loli.net/2020/05/06/ZPwk6EM8aHz31FK.png)

1. join可以级联--df1.join(df2, ...).join(df3, ...)
2. 从Spark 2.3开始，仅当查询处于Append输出模式时才能应用join
3. 从Spark 2.3开始，无法在联接之前使用非类map操作
   - 加入前不能使用流式聚合
   - 联接之前，不能在更新模式下使用mapGroupsWithState和flatMapGroupsWithState

### 6 Streaming去重

#### 6.1 Streaming Deduplication

可以使用事件中的唯一标识符将重复的记录放入数据流中

![image-20200322185932920](https://i.loli.net/2020/05/06/7A5cWhNekrHaCxO.png)

#### 6.2 不支持/更改的操作

- 不支持的操作
  - 多个流聚合（即流DF上的聚合链）
  - Limit 或者 take 前N行
  - streaming对流数据集的distinct操作
  - streaming流数据集上很少类型的outer join
  - 仅在聚合之后且在“完整输出”模式下，流数据集才支持排序操作
- 改变使用方式
  - count() → ds.groupBy().count 
  - foreach() → ds.writeStream.foreach( … ) 
  - show() → use the console sink

### 7 结构化流的输入源

- File source

  ![image-20200322190027640](https://i.loli.net/2020/05/06/CpmlEVL2sM9Q3n6.png)

- Kafka source 

- Socket source (test only)

- Rate source (test only)



## 三 Spark Streaming集成Kafka

- 自从Kafka 0.10或更高版本，仅支持Direct DStream
- Kafka分区和Spark分区之间的一一对应

![image-20200322190055025](https://i.loli.net/2020/05/06/MGiQdfw46tvYpBy.png)



### 1 区位策略

- 新的Kafka consumer API 将messages预取到缓冲区中

- Spark集成将缓存的consumers保留在执行程序上
  - LocationStrategies.PreferConsistent：在可用的Spark执行程序之间平均分配分区
  - LocationStragegies.PreferBrokers：Spark执行程序与Kafka brokers位于同一主机上
  - LocationStrategies.PreferFixed：固定的Kafka分区到Spark executors之间的映射

### 2 消费者策略

使用ConsumerStrategies，即使从检查点重新启动后，Spark仍可以获取配置正确的使用者

- ConsumerStrategies.Subscribe：允许订阅固定的主题集合

- ConsumerStrategies.Assign：允许指定固定的分区集合


![image-20200322190153235](https://i.loli.net/2020/05/06/pDdu7tNY3eExHh4.png)



### 3 存储Offset

- 通过启用Spark检查点。由于重复输出，因此输出操作必须是幂等的
- 通过调用Kafka偏移提交API

![image-20200322190233593](https://i.loli.net/2020/05/06/WPOIAhd1KBaSt7b.png)



### 4 Structured Streaming与Kafka 

#### 4.1 使用Structured Streaming查询Kafka 

![image-20200322190249386](https://i.loli.net/2020/05/06/j9cBEDrLQTuWfAa.png)



#### 4.2 使用Structured Streaming写入Kafka

![image-20200322190306188](https://i.loli.net/2020/05/06/fdktnTylKbwJ3C9.png)

```scala
// 相当于StreamingContext(sparkContext,"2 seconds")
wordCounts.writeStream.trigger(processingTime = '2 seconds') 
```

### Kafka消费模型

#### 高级API（High Level Consumer API）

1. 不需要自己管理offset
2. 默认实现最少一次消息传递语义（At least once）

#### 低级API（Low Level Consumer API）

1. 需要自己手动管理Offset
2. 可以实现各种消息传递语义



### Receiver 和 Direct

#### Receiver 

Kafka的topic分区和Spark Streaming中生成的RDD分区**没有关系**。 在KafkaUtils.createStream中增加
分区数量只会增加单个receiver的线程数， 不会增加Spark的并行度
可以创建多个的Kafka的输入DStream， 使用不同的group和topic， 使用多个receiver并行接收数据。
如果启用了HDFS等有容错的存储系统， 并且启用了写入日志，则接收到的数据已经被复制到日志中。
因此，输入流的存储级别设置StorageLevel.MEMORY_AND_DISK_SER（即使用
KafkaUtils.createStream（...，StorageLevel.MEMORY_AND_DISK_SER））的存储级别

#### Direct

**简化的并行性**：不需要创建多个输入Kafka流并将其合并。 使用directStream，Spark Streaming将创建
与使用Kafka分区一样多的RDD分区，这些分区将全部从Kafka并行读取数据。 所以在Kafka和RDD分
区之间有一对一的映射关系。
**效率**：在第一种方法中实现零数据丢失需要将数据存储在预写日志中，这会进一步复制数据。 这实际
上是效率低下的，因为数据被有效地复制了两次 - 一次是Kafka，另一次是由预先写入日志（Write
Ahead Log）复制。 这个第二种方法消除了这个问题，因为没有接收器，因此不需要预先写入日志。
只要Kafka数据保留时间足够长。
**正好一次（Exactly-once）的语义**：第一种方法使用Kafka的高级API来在Zookeeper中存储消耗的偏移
量。传统上这是从Kafka消费数据的方式。虽然这种方法（结合预写日志）可以确保零数据丢失
（即至少一次语义），但是在某些失败情况下，有一些记录可能会消费两次。发生这种情况是因为
<u>Spark Streaming可靠接收到的数据与Zookeeper跟踪的偏移之间的不一致</u>。因此，在第二种方法中，
我们可以不使用Zookeeper的简单Kafka API。<u>在其检查点内，Spark Streaming跟踪偏移量</u>。这消除了
Spark Streaming和Zookeeper / Kafka之间的不一致，因此Spark Streaming每次记录都会在发生故障的
情况下有效地收到一次。为了实现输出结果的一次语义，将数据保存到外部数据存储区的输出操作必须
是幂等的，或者是保存结果和偏移量的原子事务。

### Kafka Offset管理

#### Checkpoint管理

1. 启用Spark Streaming的checkpoint是存储偏移量最简单的方法。
2. 流式checkpoint专门用于保存应用程序的状态， 比如保存在HDFS上，在故障时能恢复。
3. Spark Streaming的checkpoint无法跨越应用程序进行恢复。
4. Spark 升级也将导致无法恢复。
5. 在关键生产应用， 不建议使用spark检查点的管理offset方式。

```scala
/**
  * 用checkpoint记录offset
  * 优点：实现过程简单
  * 缺点：如果streaming的业务更改，或别的作业也需要获取该offset，是获取不到的
  */
import kafka.serializer.StringDecoder
import org.apache.spark.SparkConf
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.streaming.{Duration, Seconds, StreamingContext}

object StreamingWithCheckpoint {
  def main(args: Array[String]) {
    //val Array(brokers, topics) = args
    val processingInterval = 2
    val brokers = "singleNode:9092"
    val topics = "mytest1"
    // Create context with 2 second batch interval
    val sparkConf = new SparkConf().setAppName("ConsumerWithCheckPoint").setMaster("local[2]")
    // Create direct kafka stream with brokers and topics
    val topicsSet = topics.split(",").toSet
    val kafkaParams = Map[String, String]("metadata.broker.list" -> brokers, "auto.offset.reset" -> "smallest")
    val checkpointPath = "hdfs://singleNode:9000/spark_checkpoint1"

    def functionToCreateContext(): StreamingContext = {
      val ssc = new StreamingContext(sparkConf, Seconds(processingInterval))
      val messages = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc, kafkaParams, topicsSet)

      ssc.checkpoint(checkpointPath)
      messages.checkpoint(Duration(8 * processingInterval.toInt * 1000))
      messages.foreachRDD(rdd => {
        if (!rdd.isEmpty()) {
          println("################################" + rdd.count())
        }

      })
      ssc
    }

    // 如果没有checkpoint信息，则新建一个StreamingContext
    // 如果有checkpoint信息，则从checkpoint中记录的信息恢复StreamingContext
    // createOnError参数：如果在读取检查点数据时出错，是否创建新的流上下文。
    // 默认情况下，将在错误上引发异常。
    val context = StreamingContext.getOrCreate(checkpointPath, functionToCreateContext _)
    context.start()
    context.awaitTermination()
  }
}
// 以上案例测试过程：
// 模拟消费者向mytest1插入10条数据，
// 强制停止streaming，
// 再插入20条数据并启动streaming查看读取的条数为20条
```

#### Zookeeper管理

```
1. 路径：
   val zkPath = s"{kakfaOffsetRootPath}/{groupName}/{o.topic}/{o.partition}"
2. 如果Zookeeper中未保存offset,根据kafkaParam的配置使用最新或者最旧的offset
3. 如果 zookeeper中有保存offset,我们会利用这个offset作为kafkaStream的起始位置
```

```scala
import kafka.common.TopicAndPartition
import kafka.message.MessageAndMetadata
import kafka.serializer.StringDecoder
import org.apache.curator.framework.CuratorFrameworkFactory
import org.apache.curator.retry.ExponentialBackoffRetry
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.InputDStream
import org.apache.spark.streaming.kafka.{HasOffsetRanges, KafkaUtils, OffsetRange}
import org.apache.spark.streaming.{Seconds, StreamingContext}

import scala.collection.JavaConversions._

object KafkaZKManager  extends Serializable{
  /**
    * 创建zookeeper客户端
    */
  val client = {
    val client = CuratorFrameworkFactory
      .builder
      .connectString("node01:2181/kafka0.9") // zk中kafka的路径
      .retryPolicy(new ExponentialBackoffRetry(1000, 3)) // 重试指定的次数, 且每一次重试之间停顿的时间逐渐增加
      .namespace("mykafka") // 命名空间:mykafka
      .build()
    client.start()
    client
  }

  val kafkaOffsetRootPath = "/consumers/offsets"

  /**
    * 确保zookeeper中的路径是存在的
    * @param path
    */
  def ensureZKPathExists(path: String): Unit = {
    if (client.checkExists().forPath(path) == null) {
      client.create().creatingParentsIfNeeded().forPath(path)
    }
  }

  def storeOffsets(offsetsRanges:Array[OffsetRange], groupName:String) = {
    for (o <- offsetsRanges) {
      val zkPath = s"${kafkaOffsetRootPath}/${groupName}/${o.topic}/${o.partition}"
      ensureZKPathExists(zkPath)
      // 保存offset到zk
      client.setData().forPath(zkPath, o.untilOffset.toString.getBytes())
    }
  }

  /**
    * 用于获取offset
    * @param topic
    * @param groupName
    * @return
    */
  def getFromOffsets(topic : String,groupName : String): (Map[TopicAndPartition, Long], Int) = {
    // 如果 zookeeper中有保存offset,我们会利用这个offset作为kafkaStream 的起始位置
    var fromOffsets: Map[TopicAndPartition, Long] = Map()
    val zkTopicPath = s"${kafkaOffsetRootPath}/${groupName}/${topic}"
    // 确保zookeeper中的路径是否存在
    ensureZKPathExists(zkTopicPath)
 	// 获取topic中，各分区对应的offset
    val offsets: mutable.Buffer[(TopicAndPartition, Long)] = for {
      // 获取分区
      p <- client.getChildren.forPath(zkTopicPath)
    } yield {
      //遍历路径下面的partition中的offset
      val data = client.getData.forPath(s"$zkTopicPath/$p")
      //将data变成Long类型
      val offset = java.lang.Long.valueOf(new String(data)).toLong
      println("offset:" + offset)
      (TopicAndPartition(topic, Integer.parseInt(p)), offset)
    }

    if(offsets.isEmpty) {
      (offsets.toMap,0)
    }else{
      (offsets.toMap,1)
    }
  }

  def main(args: Array[String]): Unit = {
    val processingInterval = 2
    val brokers = "singleNode:9092"
    val topic = "mytest1"
    val sparkConf = new SparkConf().setAppName("KafkaZKManager").setMaster("local[2]")
    // Create direct kafka stream with brokers and topics
    val topicsSet = topic.split(",").toSet
    val kafkaParams = Map[String, String]("metadata.broker.list" -> brokers,
      "auto.offset.reset" -> "smallest")

    val ssc = new StreamingContext(sparkConf, Seconds(processingInterval))

    // 读取kafka数据
    val messages = createMyDirectKafkaStream(ssc, kafkaParams, topic, "group01")

    messages.foreachRDD((rdd,btime) => {
      if(!rdd.isEmpty()){
        println("==========================:" + rdd.count() )
        println("==========================btime:" + btime )
      }
      // 消费到数据后，将offset保存到zk
      storeOffsets(rdd.asInstanceOf[HasOffsetRanges].offsetRanges, "group01")
    })

    ssc.start()
    ssc.awaitTermination()
   }

  def createMyDirectKafkaStream(ssc: StreamingContext, kafkaParams: Map[String, String], topic: String, groupName: String): InputDStream[(String, String)] = {
    // 获取offset
    val (fromOffsets, flag) = getFromOffsets( topic, groupName)
    var kafkaStream : InputDStream[(String, String)] = null
    if (flag == 1) {
      // 这个会将kafka的消息进行transform,最终kafak的数据都会变成(topic_name, message)这样的tuple
      val messageHandler = (mmd : MessageAndMetadata[String, String]) => (mmd.topic, mmd.message())
      println("fromOffsets:" + fromOffsets)
      kafkaStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder, (String, String)](ssc, kafkaParams, fromOffsets, messageHandler)
    } else {
      // 如果未保存,根据kafkaParam的配置使用最新或者最旧的offset
      kafkaStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc, kafkaParams, topic.split(",").toSet)
    }
    kafkaStream
  }

}
```

启动zk命令：

```shell
zkCli.sh  -timeout 5000  -r  -server  singleNode:2181
```

![](https://i.loli.net/2020/05/06/JhKgDZ9ank6UQle.png)

