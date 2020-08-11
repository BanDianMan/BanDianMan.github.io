---
title: "Hadoop生态常用数据模型"
date: 2020-05-07T10:26:30+08:00
draft: true
---

## Hadoop常用数据模型

### TextFile
- 文本文件通常采用CSV,JSON等固定长度的纯文本格式
- 优点
  - 便于与其他应用程序(生成或读取分隔文件)或脚本进行数据交换
  - 易读性好,便于理解
- 缺点
  - 数据存储量非常庞大
  - 查询效率不高
  - 不支持块压缩
---

### SequenceFile
- 按行存储键值对为二进制数据格式，以<Key,Value>形式序列化为二进制文件，HDFS自带
  - 支持压缩和分割
  - Hadoop中的小文件合并
  - 常用于在MapReduce作业之间传输数据
- SequenceFile中的Key和Value可以是任意类型的Writable(org.apache.hadoop.io.Writable)
- Java API : org.apache.hadoop.io.SequenceFile
>记录级压缩是如何存储的 ？

A：记录级仅压缩value数据，按byte的偏移量索引数据。每个记录头为两个固定长度为4的数据量，一个代表本条Record的长度，一个代表Key值的长度，随后依次存储key值和压缩的value值。

模拟读取过程如下，首先偏移4获得本条记录长度r，然后偏移4获得key类型的长度k，再偏移k读入key的值，最后偏移到位置r，获取压缩后的value值，本条记录读完。
> 块级压缩是如何存储的 ?

A：块级压缩同时压缩key和value，相同特性的记录将会被归为同一块。块头为定长4byte的代表本块所含的记录数目n、定长4byte的代表压缩后的key的长度的k、压缩后的key值组、定长4byte的达标压缩后的value的长度的v、压缩后的value值组。（n,k_length,k, v_length,v）

模拟读取过程如下，首先偏移4byte，获取当前块存储的记录数目n；偏移4byte，获取压缩后的key类型的长度k，再偏移n*k读入多个key值分别存储；偏移4byte，获取压缩后的value类型的长度v，再偏移n*v读入多个value值分别对应key值。
#### 读写操作
- 读写操作
  - SequenceFile.Writer （指定为块压缩）
  - SequenceFile.Reader（读取时能够自动解压）
- 在Hive中使用SequenceFile 
- 方式一
  - STORED AS sequencefile
- 方式二显示指定
```scala
STORED AS INPUTFORMAT   'org.apache.hadoop.mapred.SequenceFileInputFormat' 
OUTPUTFORMAT  'org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat'
```
- 在Spark中使用SequenceFile
```scala
val rdd=sc.sequenceFile[Int,String]("/tmp/myseqfile.seq")		//装载
rdd.saveAsSequenceFile("/tmp/seq")					//存储
```
---

### Avro
#### 特性
Apache Avro是一个数据系列化系统,数据定义以JSON格式存储,数据内容以二进制格式存储
- 丰富的数据结构,被设计用于满足Schema Evolution
- Schema和数据分开保存
- 基于行存储
- 快速可压缩的二进制数据格式
- 容器文件用于持久化数据
- 自带远程过程调用RPC
- 动态语言可以方便地处理Avro数据
#### 优点
- 高扩展的Schema,为Schema Evolution而生
- 数据压缩快
#### 数据类型
基本数据类型 : Null,Boolean,Int,Long,Float,Double,Bytes,String
复杂数据类型 : Record,Enum,Aaary,Map,Union,Fixed
#### Arvo 应用
> user.json
```json
{"name": "Alyssa", "favorite_number": 256, "favorite_color": "black"}
{"name": "Ben", "favorite_number": 7, "favorite_color": "red"}
{"name": "Charlie", "favorite_number": 12, "favorite_color": "blue"}
```
> user.avsc : 定义了User对象的Schema
```acsc
{
"namespace": "example.avro",
 "type": "record",
 "name": "User",
 "fields": [
    {"name": "name", "type": "string"},
     {"name": "favorite_number",  "type": "int"},
     {"name": "favorite_color", "type": "string"}
 ]}
```
> 使用Schema+data生成avro文件
```linux
java -jar avro-tools-1.8.2.jar fromjson --schema-file user.avsc user.json > user.avro
java -jar avro-tools-1.8.2.jar fromjson --codec snappy --schema-file user.avsc user.json > user.avro
```
> avro转json 读取
```linux
java -jar avro-tools-1.8.2.jar tojson user.avro
java -jar avro-tools-1.8.2.jar tojson user.avro --pretty
```
> 获取avro元数据
```linux
java -jar avro-tools-1.8.2.jar getmeta user.avro
```
#### 在hive中使用arvo
```hive
create table CUSTOMERS 
row format serde 'org.apache.hadoop.hive.serde2.avro.AvroSerDe' 
stored as inputformat 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat' 
outputformat 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat' 
tblproperties ('avro.schema.literal'='{ 
	     "name": "customer", "type": "record", 
              "fields": [ 
	    {"name":"firstName", "type":"string"}, {"name":"lastName", "type":"string"}, 
              {"name":"age", "type":"int"}, {"name":"salary", "type":"double"}, 
              {"name":"department", "type":"string"}, {"name":"title", "type":"string"},
              {"name": "address", "type": "string"}]}'); 
```
> 外部表
```scala
create external table user_avro_ext(name string,favorite_number int,favorite_color string) 
stored as avro 
location '/tmp/avro'; 
```
#### 在Spark中使用Avro
>拷贝spark-avro_2.11-4.0.0.jar包到Spark目录下的jars目录下。或使用IDEA导依赖
```scala
//需要spark-avro
import com.databricks.spark.avro._
//装载
val df = spark.read.format("com.databricks.spark.avro").load("input dir")
//存储  
df.filter("age > 5").write.format("com.databricks.spark.avro").save("output dir")
```


### Parquet

### RC&ORC