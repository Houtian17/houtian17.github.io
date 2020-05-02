---
layout: post
title: MySql 和 Hive 环境搭建
description: ""
categories: [Linux]
keywords: [Linux，大数据,hadoop，mysql,hive]
---

## 安装 Mysql

- 删除系统自带的 mariadb 安装包

  ```shell
  [root@hadoop01 software]# rpm -qa | grep mariadb
  mariadb-libs-5.5.64-1.el7.x86_64
  [root@hadoop01 software]# yum remove mariadb-libs-5.5.64-1.el7.x86_64
  ```

- 解压 mysql 安装包

  ```shell
  [root@hadoop01 software]# tar -xvf mysql-5.7.12-1.el6.x86_64.rpm-bundle.tar 
  ```

- 安装 mysql 组件

  ```shell
  [root@hadoop01 software]# rpm -ivh mysql-community-common-5.7.12-1.el6.x86_64.rpm 
  [root@hadoop01 software]# rpm -ivh mysql-community-libs-5.7.12-1.el6.x86_64.rpm 
  [root@hadoop01 software]# rpm -ivh mysql-community-client-5.7.12-1.el6.x86_64.rpm 
  [root@hadoop01 software]# rpm -ivh mysql-community-server-5.7.12-1.el6.x86_64.rpm --force --nodeps
  ```

- 查看 mysql 运行状态

  ```shell
  [root@hadoop01 software]# systemctl status mysqld
  ```

- 关闭数据库运行状态

  ```shell
  [root@hadoop01 software]# systemctl stop mysqld
  ```

- 初始化数据库

  ```shell
  [root@hadoop01 software]# mysqld --initialize --console
  ```

- 给数据库目录授权

  ```shell
  [root@hadoop01 software]# chown -R mysql:mysql /var/lib/mysql
  ```

- 启动数据库

  ```shell
  [root@hadoop01 software]# systemctl start mysqld
  ```

- 查看数据库临时密码

  ```shell
  [root@hadoop01 software]# grep 'temporary password' /var/log/mysqld.log
  ```

- 得到临时密码，进入数据库配置密码

  ```mysql
  mysql> alter USER 'root'@'localhost' IDENTIFIED BY '123456';
  ```

- 设置访问权限

  ```mysql
  grant all privileges on *.* to root@'%' identified by '123456';
  ```

## 安装 Hive

- 解压 Hive 安装包到相应目录下

  ```shell
  [root@hadoop01 software]# tar -zxvf apache-hive-1.1.0-bin.tar.gz -C /usr/local/apps/
  ```

- 进入到 Hive 的 conf 目录下，创建 hive-site.xml 文件并配置

  ```shell
  [root@hadoop01 conf]# touch hive-site.xml
  ```

  

  ```shell
  <configuration>
  
  	<property>
  		<name>javax.jdo.option.ConnectionURL</name>
  		<value>jdbc:mysql://hadoop01:3306/hive?createDatabaseIfNotExist=true</value>
  	</property>
  	
  	<property>
  		<name>javax.jdo.option.ConnectionDriverName</name>
  		<value>com.mysql.jdbc.Driver</value>
  	</property>
  	
  	<property>
  		<name>javax.jdo.option.ConnectionUserName</name>
  		<value>root</value>
  	</property>
  	
  	<property>
  		<name>javax.jdo.option.ConnectionPassword</name>
  		<value>123456</value>
  	</property>
  </configuration>
  ```

- 将 mysql-connector-java-5.1.39.jar 文件拷贝到 hive 的 lib 目录下

  ```shell
  [root@hadoop01 software]# cp mysql-connector-java-5.1.39.jar /usr/local/apps/hive/lib/
  ```

- 删除 hadoop 中的低版本 jline.jar，并复制 hive 的 lib 目录下的 jline.jar 过去

  ```shell
  [root@hadoop01 /]# cd /usr/local/apps/hadoop/share/hadoop/yarn/lib/
  [root@hadoop01 lib]# rm -rf jline-0.9.94.jar 
  [root@hadoop01 lib]# cp jline-2.12.jar /usr/local/apps/hadoop/share/hadoop/yarn/lib/
  ```

- 运行 hive

  ```shell
  [root@hadoop01 bin]# ./hive
  ```

  ![lq9tcd.png](https://s2.ax1x.com/2020/01/14/lq9tcd.png)

## 几个常见的错误

1. 没有启动 hdfs。
2. 目录没有授权。
3. 配置文件修改错误。
