---
layout: post
title: Hadoop 搭建 hdfs 分布式集群
description: ""
categories: [Linux]
keywords: [Linux，大数据]

---

之前一周在学习 Linux，这周要开始学习大数据了，感觉又要开始写博客了。

2019.12.2

## 基础环境

我的操作系统是 Mac OS，虚拟机软件用的是 Virtual Box，远程连接用的就是 Mac 自带的终端，文件上传工具使用的是 Secure FX ，总共配置了四台服务器，系统为 CentOS 7.7，Hadoop 的版本是 2.6.0，JDK 版本是 1.8.0。我给服务器添加了 Host 和 Net 两块网卡，前者用于虚拟机之间的互联，后者用于实现连接以太网。

首先准备四台服务器，1 个 namenode 节点和 3 个 datanone 节点。通过编辑网卡，配置服务器的 ip 地址，网关，子网掩码，以实现服务器之间的互通。我的服务器的 ip 通过链接复制之后，它的 ip 都是在同一个网段的，直接就可以 ping 通，我估计是因为 host 模式，自动将 ip 配置好了。（记下来明天问老师）

### 环境准备

1. 配置 host 和主机名

修改主机名：```vi /etc/sysconfig/network```

```shell
NETWORKING=yes
HOSTNAME=master
```

修改 hosts 文件：```vi /etc/hosts```

```shell
192.168.56.101 master
192.168.56.102 slave1
192.168.56.103 slave2
192.168.56.104 slave3
```

这里凡是你要用到的服务器，ip 和 主机名都必须写上去，保存退出后重启系统，可以通过`ping hostname`来查看是否配置成功。

```shell
scp /etc/hosts slave1:/etc/
scp /etc/hosts slave2:/etc/
scp /etc/hosts slave3:/etc/
```

2. 给服务器安装 jdk

我这里是根据视频，用的 Secure Fx 软件，连接的 master 服务器，然后上传到 /root 目录下，然后解压到 apps 目录下，最后通过 `scp -r jdk1.8.0/ slave1:/root/apps/`来给每个服务器安装 jdk。当然也可以通过`yum -y install java-1.8.0-openjdk*`的命令来安装。

安装完成之后，在 apps/ 目录下解压，然后配置环境变量，修改配置文件`vi /etc/profile`文件。

```shell
export JAVA_HOME=/root/apps/jdk1.8.0_141
export HADOOP_HOME=/root/apps/hadoop-2.6.0
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

然后通过`scp`给每一台服务器配置环境变量。

```
scp /etc/profile slave1:/etc/profile
scp /etc/profile slave2:/etc/profile
scp /etc/profile slave3:/etc/profile
```

保存退出后，使用`source /etc/ profile`让文件立刻生效。

### 免密登陆

1. 关闭防火墙

```shell
iptables -F
```

2. 配置 ssh 免密登陆服务器
   - ssh-keygen -t rsa
   - ssh-copy-id hostname

通过这两个步骤就可以生成 ssh 密钥和配对密钥就能够实现免密登陆。如果要实现 master 登陆 salve1 服务器，只需要再次执行第二个步骤，就可以实现 master 免密登陆 salve1。

```shell
[root@master ~]# ssh slave1
Last login: Mon Dec  2 22:43:36 2019 from 192.168.56.1
[root@slave1 ~]# ssh master
Last login: Mon Dec  2 22:44:30 2019 from 192.168.56.1
```

## Hadoop 环境搭建

### 配置 hadoop-master 的 hadoop 环境

1. 将 hadoop 压缩包上传至服务器，并解压缩至 apps/ 文件夹。

我这里用的是老师在教学服务器上传的 hadoop 文件，版本为 2.6.0，依旧是通过 Secure Fx 上传至服务器。

```shell
tar -xzvf hadoop-2.6.0.tar.gz -C /root/apps/
```

2. 配置 hadoop-master 的 hadoop 环境变量。修改配置文件`vi /etc/profile`

```shell
export HADOOP_HOME=/root/apps/hadoop-2.6.0
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

然后使用`source /etc/profile`命令使文件立即生效。

3. 修改配置文件

进入 /root/apps/hadoop/etc/hadoop 编辑配置文件。

- vi hadoop-env.sh

  ```
  export JAVA_HOME=/root/apps/jdk1.8.0_141
  ```

- 配置 core-site.xml

  修改 Hadoop 核心配置文件```/root/apps/hadoop/etc/hadoop/core-site.xml```，通过```fs.default.name```指定 NameNode 的IP地址和端口号，通过```hadoop.tmp.dir```指定 hadoop 数据存储的临时文件夹。

  ```shell
  <configuration>
      <property>
          <name>hadoop.tmp.dir</name>
          <value>/root/apps/hadoop-2.6.0/tmp</value>
      </property>
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://master:9000</value>
      </property>
  </configuration>
  ```

  **特别注意：如没有配置 hadoop.tmp.dir 参数，此时系统默认的临时目录为：/tmp/hadoo-hadoop。而这个目录在每次重启后都会被删除，必须重新执行 format 才行，否则会出错。**

- 配置 hdfs-site.xml

  修改HDFS核心配置文件`/root/apps/hadoop/etc/hadoop/hdfs-site.xml`，通过`dfs.replication`指定HDFS的备份因子为3，通过`dfs.name.dir`指定namenode节点的文件存储目录，通过`dfs.data.dir`指定datanode节点的文件存储目录。

  ```
  <configuration>
      <property>
          <name>dfs.replication</name>
          <value>3</value>
      </property>
      <property>
          <name>dfs.name.dir</name>
          <value>/root/apps/hadoop2.6.0/hdfs/name</value>
      </property>
      <property>
          <name>dfs.data.dir</name>
          <value>/root/apps/hadoop2.6.0/hdfs/data</value>
      </property>
  </configuration>
  ```

- 配置 mapper-site.xml 

  原本是没有这个文件的，通过`cp mapper-site.xml.template mapper-site.xml`创建

  ```shell
  <configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
     <property>
        <name>mapred.job.tracker</name>
        <value>http://hadoop-master:9000</value>
    </property>
  </configuration>
  ```

- 配置 yarn-site.xml

  ```
  <configuration>
      <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
      </property>
      <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>hadoop-master</value>
      </property>
  </configuration>
  ```

- 配置 master 文件

  修改`vi /root/apps/hadoop-2.6.0/etc/hadoop/master`，该文件指定namenode节点所在的服务器机器。删除localhost，添加namenode节点的主机名hadoop-master；不建议使用IP地址，因为IP地址可能会变化，但是主机名一般不会变化。

  ```shell
  master
  ```

- 配置 salves 文件

  修改`/root/apps/hadoop-2.6.0/etc/hadoop/slaves`，该文件指定哪些服务器节点是datanode节点。删除locahost，添加所有datanode节点的主机名，如下所示。

  ```shell
  slave1
  slave2
  ```

### 配置 salve 的 hadoop 环境

1. 复制 hadoop 到 slave 节点

   ```shell
   scp -r /root/apps/hadoop-2.6.0 slave1:/root/apps
   ```

2. 登陆到 salve 服务器，删除 slaves 文件

   ```shell
   rm -rf slaves
   ```

3. 配置环境变量

   ```shell
   vi /etc/profile
   export HADOOP_HOME=/root/apps/hadoop-2.6.0
   export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   ```

   装载文件，立即生效

   ```shell
   source /etc/profile
   ```

### 启动集群

1. 格式化 HDFS 文件系统

```shell
hdfs namenode -format
```

2. 启动 hadoop

```shell
start-all.sh
```

3. 使用 jps 查看运行情况

```shell
4165 NameNode
4395 SecondaryNameNode
4669 Jps
```

## 错误

第二天重新开启服务器，查看网页面板的时候发现上不去了，原因防火墙关闭后还会自动开启，执行以下代码。

```shell
iptables -F
systemctl disable firewalld.service 
```



