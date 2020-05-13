---
layout: post
title: sqoop 安装部署与数据迁移
description: ""
categories: [Big Data]
keywords: [Linux，大数据]
---

Sqoop是一个用来将[Hadoop](https://baike.baidu.com/item/Hadoop)和关系型数据库中的数据相互转移的工具，可以将一个关系型[数据库](https://baike.baidu.com/item/数据库)（例如 ： MySQL ,Oracle ,Postgres等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。

## sqoop 安装部署

### 环境准备

- Hadoop
- Zookeeper
- Hbase

### 环境搭建

- 将 mysql 的驱动类复制到 sqoop 的 lib 文件夹下

  ```MYSQL
  [root@master software]# cp mysql-connector-java-5.1.39.jar /usr/local/apps/sqoop/lib/
  ```

- 将 hadoop 的三个 jar 包复制到 sqoop 的lib 文件夹下

  ```mysql
  [root@master lib]# cp /usr/local/apps/hadoop/share/hadoop/common/hadoop-common-2.6.0.jar ./
  [root@master lib]# cp /usr/local/apps/hadoop/share/hadoop/hdfs/hadoop-hdfs-2.7.2.jar ./
  [root@master lib]# cp /usr/local/apps/hadoop/share/hadoop/hadoop-mapreduce-client-core-2.7.2.jar ./
  ```

- 配置 sqoop-env.sh 文件

  ```mysql
  export HADOOP_COMMON_HOME=/usr/local/apps/hadoop
  export HADOOP_MAPRED_HOME=/usr/local/apps/hadoop
  export HBASE_HOME=/usr/local/apps/hbase
  export HIVE_HOME=/usr/local/apps/hive
  ```

### 验证 sqoop 版本

[![YGQLDg.md.png](https://s1.ax1x.com/2020/05/11/YGQLDg.md.png)](https://imgchr.com/i/YGQLDg)

## sqoop 连接数据库

- 开启数据库

```mysql
[root@master conf]# systemctl start mysqld
```

- 进入数据库

```mysql
[root@master conf]# mysql -uroot -p123456
```

- 开启远程访问

```mysql
mysql> grant all privileges on *.* to root@'%' identified by '123456';
```

- 列出mysql数据库中的所有数据库命令

```msyql
sqoop list-databases --connect jdbc:mysql://master:3306/ --username root --password 123456
```

[![YG3MHe.md.png](https://s1.ax1x.com/2020/05/11/YG3MHe.md.png)](https://imgchr.com/i/YG3MHe)

- 连接mysql并列出数据库中的表命令

```
sqoop list-tables -connect jdbc:mysql://master:3306/mysql --username root --password 123456
```

[![YG3lAH.md.png](https://s1.ax1x.com/2020/05/11/YG3lAH.md.png)](https://imgchr.com/i/YG3lAH)

## sqoop 数据迁移

sqoop是apache旗下一款“Hadoop和关系数据库服务器之间传送数据”的工具。
导入数据：MySQL，Oracle导入数据到Hadoop的HDFS、HIVE、HBASE等数据存储系统。
导出数据：从Hadoop的文件系统中导出数据到关系数据库mysql等工作机制：是将导入和导出的命令翻译成mapreduce程序来实现，在翻译出的mapreduce中主要对inputformat和outputformat进行定制。

### Hive 数据导入 MySQL

```
sqoop export --connect jdbc:mysql://master:3306/mst  --username root --password 123456 --table  mst_percent --export-dir /usr/hive/warehouse/mst_percent --input-fields-terminated-by '\001'
```






