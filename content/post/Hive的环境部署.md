---
title: "Hive的环境部署"
date: 2020-05-07T10:03:48+08:00
draft: true
---

# Hive安装文档

## hive的安装部署

1. 由于hive是依赖于hadoop的, 所以先把hadoop相关的服务启动
2. 配置hive
   -->解压 tar -zxvf apache-hive-1.2.1-bin.tar.gz -C [一个安装目录]
   -->创建目录用于保存hive的所有数据, 便于管理

  ```
  	bin/hdfs dfs -mkdir       /tmp
  	bin/hdfs dfs -mkdir       /user/hive/warehouse
  ```

  -->修改权限

  ```
  	bin/hdfs dfs -chmod g+w   /tmp
  	bin/hdfs dfs -chmod g+w   /user/hive/warehouse
  ```

4. hive在HDFS上的默认路径

   ```
   	<property>
   	  <name>hive.metastore.warehouse.dir</name>
   	  <value>/user/hive/warehouse</value>
   	  <description>location of default database for the warehouse</description>
   	</property>
   ```

5. 修改hive-env.sh(改名)

  ```
  	# Set HADOOP_HOME to point to a specific hadoop install directory
  	HADOOP_HOME=/opt/moduels/hadoop-2.5.0
  	# Hive Configuration Directory can be controlled by:
  	export HIVE_CONF_DIR=/opt/moduels/hive-0.13.1-bin/conf
  ```

6. 启动hive

  ```
  	${HIVE_HOME}/bin/hive
  ```

## 安装配置mysql

由于hive中默认的元数据保存在derby中只能单用户访问hive , 则另一用户无法访问, 会出现以下错误信息:

```
Caused by: ERROR XSDB6: Another instance of Derby may have already booted the database /opt/app/apache-hive-1.2.1-bin/metastore_db.
```

为了解决以上的问题, 可以把hive的元数据保存在mysql中.

### mysql的安装步骤

1. 在Linux系统中,可能存在mysql的安装包, 所以第一步先检查是否安装过mysql

```
[hadoop@hadoop apache-hive-1.2.1-bin]$ rpm -qa | grep -i mysql
mysql-libs-5.1.73-5.el6_6.x86_64
```

执行该命令可以查看是否安装mysql

2. 卸载已有的mysql安装包

```
[hadoop@hadoop apache-hive-1.2.1-bin]$ sudo rpm -e --nodeps mysql-libs-5.1.73-5.el6_6.x86_64
[sudo] password for hadoop:  
```

3. 查看是否卸载成功

```
[hadoop@hadoop apache-hive-1.2.1-bin]$ rpm -qa | grep -i mysql  
[hadoop@hadoop apache-hive-1.2.1-bin]$ 
```



4. mysql分为server端和client端

```
 MySQL-client-5.5.47-1.linux2.6.x86_64.rpm
 MySQL-server-5.5.47-1.linux2.6.x86_64.rpm
```
 ``` 
 依次运行
 yum install numact1
 yum install libaio
 yum install perl
 ```


    > 解压Mysql包
    ```
    tar xvf MySQL-5.6.26-1.linux_glibc2.5.x86_64.rpm-bundle.tar -C /opt/
    ```

5. 安装mysql软件

   通过rpm安装server

```
 rpm -ivh MySQL-server-5.5.47-1.linux2.6.x86_64.rpm 
```

	通过rpm安装client

```
 rpm -ivh MySQL-client-5.5.47-1.linux2.6.x86_64.rpm
```

6. 查看mysql的运行状态

```
sudo service mysql status
```

7. 启动mysql服务

```
[root@hadoop mysql]# service mysql start
Starting MySQL.. SUCCESS!
```

8. 再次查看mysql运行状态

```
 SUCCESS! MySQL running (5094)
```

### 设置密码,远程授权

#### 设置密码

1. mysql安装好之后进入mysql

```
mysql -uroot
```

2. 查询数据库


```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.01 sec)
```

3. 切换mysql数据库

```
mysql> use mysql;
Database changed
```

4. 查看user, host, passWord信息

```
mysql> select user,host,password from user;
+------+-----------+----------+
| user | host      | password |
+------+-----------+----------+
| root | localhost |          |
| root | hadoop    |          |
| root | 127.0.0.1 |          |
| root | ::1       |          |
|      | localhost |          |
|      | hadoop    |          |
+------+-----------+----------+
6 rows in set (0.00 sec)
```

5. 设置mysql密码

```
mysql> update user set password=PASSWORD('root') where user='root';
Query OK, 4 rows affected (0.00 sec)
Rows matched: 4  Changed: 4  Warnings: 0
```

6. 修改密码之后, 查询user表内容如下,说明在本地已经成功设置好了密码

```
mysql> select user,host,password from user;
+------+-----------+-------------------------------------------+
| user | host      | password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| root | hadoop    | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| root | 127.0.0.1 | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| root | ::1       | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
|      | localhost |                                           |
|      | hadoop    |                                           |
+------+-----------+-------------------------------------------+
6 rows in set (0.00 sec)
```

#### 设置远程授权

1. 通过新设置的密码登录mysql, 发现遇到如下问题, 说明用户名或密码不正确

```
[root@hadoop mysql]# mysql -uroot -proot
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

	在user表中存在字段**host** , 该字段表示可以访问mysql的路径地址, 从哪个节点可以访问, 有这个字段来决定

2. 所以要授权远程登录, 则需要修改host字段, 增加一条信息, 表示任意节点都可以访问mysql, 用%来表示任意

```
mysql> update user set host='%' where user='root' and host='127.0.0.1';
```

3. 完成以上语句后, 需要对修改的user进行刷新来生效语句操作

```
mysql> flush privileges;
```

4. 完成以上操作之后验证mysql用户登录,可以登录成功

```
[root@hadoop mysql]# mysql -uroot -proot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.5.47 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## 配置hive元数据保存在mysql

需要在hive-site.xml配置文件总进行配置

1. 设置hive链接mysql

```
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://192.168.91.100:3306/metastore?createDatabaseIfNotExist=true</value>
    <description>JDBC connect string for a JDBC metastore</description>
  </property>
```

	metastore: 默认保存hive中的元数据, 是一个数据库的名字

2. 设置jdbc的驱动类

```
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>
```

3. 设置mysql的用户名

```
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
    <description>Username to use against metastore database</description>
  </property>
```

4. 设置mysql的密码

```
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>root</value>
    <description>password to use against metastore database</description>
  </property>
```

5. 完成以上的配置之后, 需要在hive/lib下边存放jdbc的驱动包, 上传好驱动包之后最好修改权限

```

```

6. 将驱动包拷贝到hive目录下的lib文件夹

```
cp mysql-connector-java-5.1.31.jar /opt/app/apache-hive-1.2.1-bin/lib/
```

7. 到hive的lib下检查是否拷贝成功

8. 配置完成, 退出hive重新进入, 检查mysql中是否创建了metastore数据库, 如果创建成功, 则说明配置成功

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| metastore          |
| mysql              |
| performance_schema |
| test               |
+--------------------+
```

## hiveserver2

### 1. beeline方式的连接

相当于在hive中启动一个服务器端, 客户端可以远程连接该hive, hiveserver2不用安装, 直接在hive/bin目录下启动

```
bin/hiveserver2
```

hiveserver2的服务启动之后, 可以通过bin/beeline客户端进行连接

官方实例:

```
!connect jdbc:hive2://localhost:10000 scott tiger
```

按照官方提供实例, 连接hiveserver2 测试能否连接成功

```
!connect jdbc:hive2://hadoop:10000 hadoop 123456
```
```
./beeline -u jdbc:hive2://hadoop:10000 -n hadoop 123456

```

### 2. jdbc的方式连接

```
package org.hive.server;

import java.sql.SQLException;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
import java.sql.DriverManager;

public class HiveJdbcClient {

	 private static String driverName = ""org.apache.hive.jdbc.HiveDriver"";
	 
	  /**
	   * @param args
	   * @throws SQLException
	   */
	  public static void main(String[] args) throws SQLException {
	      try {
	      Class.forName(driverName);
	    } catch (ClassNotFoundException e) {
	      // TODO Auto-generated catch block
	      e.printStackTrace();
	      System.exit(1);
	    }
	    //replace ""hive"" here with the name of the user the queries should run as
	    Connection con = DriverManager.getConnection(""jdbc:hive2://10.0.152.235:10000/default"", ""hive"", """");
	    Statement stmt = con.createStatement();
	    
	    String sql = ""show tables"";
	    System.out.println(""Running: "" + sql);
	    ResultSet res = stmt.executeQuery(sql);
	    if (res.next()) {
	      System.out.println(res.getString(1));
	    }
	 }
}
```