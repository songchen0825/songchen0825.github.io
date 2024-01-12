---
title: 安装Hadoop-yarn集群
cover: false
date: 2024-01-04 11:42:58
tags:
categories:
 - [软件工程,大数据]
top:
coverImg:
---
spark下载速度太慢清华下载地址
https://mirrors.tuna.tsinghua.edu.cn/apache/spark/

Hadoop下载速度太慢清华下载地址
https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.3.5/

## Hadoop 安装环境搭建

修改主机名 hostnamectl set hostname ,本地服务器ip信息如下：

| Server name | Ip           |
| ----------- | ------------ |
| ser03       | 192.168.0.62 |
| ser04       | 192.168.0.63 |
| ser05       | 192.168.0.64 |


## 设置Linux环境变量 

每一台采用同意的配置环境 

```bash
export JAVA_HOME=/home/centos/jdk1.8.0_381
export HADOOP_HOME=/home/centos/hadoop/hadoop-3.3.5
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop

export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

## 针对Linux环境修改配置信息

### Hadoop的配置文件信息

#### core-site.xml

```xml
<configuration>
    <!--用来指定hdfs的老大-->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ser03:9000</value>
    </property>
    <!--用来指定hadoop运行时产生文件的存放目录-->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/centos/hadoop/temp</value>
    </property>
</configuration>
```

#### hdfs-site.xml

```xml
<configuration>
    <!--设置名称节点的目录-->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/centos/hadoop/temp/namenode</value>
    </property>
    <!--设置数据节点的目录-->
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/centos/hadoop/temp/datanode</value>
    </property>
    <!--设置辅助名称节点-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>ser03:50090</value>
    </property>
    <!--hdfs web的地址，默认为9870，可不配置-->
    <!--注意如果使用hadoop2，默认为50070-->
    <property>
        <name>dfs.namenode.http-address</name>
        <value>0.0.0.0:9870</value>
    </property>
    <!--副本数，默认为3，可不配置-->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <!--是否启用hdfs权限，当值为false时，代表关闭-->
    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>
</configuration>
```

#### workers


- ser03
- ser04
- ser05


#### mapred-env.sh

```bash
export JAVA_HOME=/home/centos/jdk1.8.0_381
```
#### mapred-site.xml

```xml
<configuration>
    <!--配置MR资源调度框架YARN-->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
</configuration>
```

#### yarn-env.sh

```bash
export JAVA_HOME=/home/centos/jdk1.8.0_381
export HADOOP_HOME=/home/centos/hadoop/hadoop-3.3.5
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

#### yarn-site.xml

```xml
<configuration>
    <!--配置资源管理器：集群master-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>ser03</value>
    </property>
    <!--配置节点管理器上运行的附加服务-->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!--关闭虚拟内存检测，在虚拟机环境中不做配置会报错-->
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
</configuration>
```

### 修改start-dfs.sh stop-dfs.sh 

在两个文件头部添加
```
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```

### 修改start-yarn.sh，stop-yarn.sh
```
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```


## Hadoop集群启动

### 格式化NameNode

```shell
hdfs namenode -format
```

看到以下日志代表成功
```log
INFO common.Storage: Storage directory /home/centos/hadoop/temp/namenode has been successfully formatted.
```

### 启动HDFS：`start-dfs.sh`

{% asset_img image.png 配置图 %}

### 启动Yarn：`start-yarn.sh`

{% asset_img image1.png 配置图 %}

