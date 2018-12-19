# apache-hive-1.2.1-bin 安装

## 更多资源:https://github.com/opensourceteams

## 技能标签
- 下载apache hive 安装包
- 进行apache-hive-1.2.1-bin.tar.gz安装
- 配置mysql存储元数据
- 配置HDFS存储 /user/hive/warehouse
- 该Hive版本1.2.1默认集成在(Spark1.6 到 Spark2.4)

## 下载
- 下载安装包apache-hive-1.2.1-bin.tar.gz: https://archive.apache.org/dist/hive/hive-1.2.1
- 由于spark-2.4.0-bin-hadoop2.7 预编译好的捆绑hive版本为hive-1.2.1,为了方便所以选用版本apache-hive-1.2.1-bin
- 如果spark支持其它版本的hive,需要重新手动编译spark源码也可以实现



## 安装
### 解压安装包

```
tar -zxvf apache-hive-1.2.1-bin.tar.gz -C /opt/module/bigdata/
```

### 配置环境变量

```
export HIVE_HOME=/opt/module/bigdata/apache-hive-1.2.1-bin
export PATH=$HIVE_HOME/lib:#PATH

```

### HDFS新建目录

```
bin/hadoop fs -mkdir -p  /user/hive/warehouse #创建目录
bin/hadoop fs -chmod -R 777 /user/hive/warehouse #新建的目录赋予读写权限
bin/hadoop fs -mkdir -p /tmp/hive/#新建/tmp/hive/目录
bin/hadoop fs -chmod -R 777 /tmp/hive #目录赋予读写权限
#用以下命令检查目录是否创建成功
bin/hadoop fs -ls /user/hive
bin/hadoop fs -ls /tmp/hive

```
### 连接mysql数据库
- 复制jdbc连接驱动

```
cp mysql-connector-java-8.0.13.jar  $HIVE_HOME/lib/
```


### 新建hive-site.xml数据库相关的配置
- touch hive-site.xml文件

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	 <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>  
  </property>
  
  <property>
    <name>hive.exec.scratchdir</name>
    <value>/tmp/hive</value>  
  </property>
  
  <property>
    <name>javax.jdo.option.ConnectionURL</name> 
    <value>jdbc:mysql://macbookmysql.com:3306/hive?createDatabaseIfNotExist=true</value>
    <description>JDBC connect string for a JDBC metastore</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.cj.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>admin</value>
    <description>username to use against metastore database</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>000000</value>
    <description>password to use against metastore database</description>
  </property>


</configuration>	

```
### 新建hive-env.sh
- cp hive-env.sh.template

```
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Set Hive and Hadoop environment variables here. These variables can be used
# to control the execution of Hive. It should be used by admins to configure
# the Hive installation (so that users do not have to set environment variables
# or set command line parameters to get correct behavior).
#
# The hive service being invoked (CLI/HWI etc.) is available via the environment
# variable SERVICE


# Hive Client memory usage can be an issue if a large number of clients
# are running at the same time. The flags below have been useful in 
# reducing memory usage:
#
# if [ "$SERVICE" = "cli" ]; then
#   if [ -z "$DEBUG" ]; then
#     export HADOOP_OPTS="$HADOOP_OPTS -XX:NewRatio=12 -Xms10m -XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=15 -XX:+UseParNewGC -XX:-UseGCOverheadLimit"
#   else
#     export HADOOP_OPTS="$HADOOP_OPTS -XX:NewRatio=12 -Xms10m -XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=15 -XX:-UseGCOverheadLimit"
#   fi
# fi

# The heap size of the jvm stared by hive shell script can be controlled via:
#
# export HADOOP_HEAPSIZE=1024
#
# Larger heap size may be required when running queries over large number of files or partitions. 
# By default hive shell scripts use a heap size of 256 (MB).  Larger heap size would also be 
# appropriate for hive server (hwi etc).


# Set HADOOP_HOME to point to a specific hadoop install directory
# HADOOP_HOME=${bin}/../../hadoop

# Hive Configuration Directory can be controlled by:
# export HIVE_CONF_DIR=

# Folder containing extra ibraries required for hive compilation/execution can be controlled by:
# export HIVE_AUX_JARS_PATH=




HADOOP_HOME=/opt/module/bigdata/hadoop-2.9.2
export HIVE_CONF_DIR=/opt/module/bigdata/apache-hive-1.2.1-bin/conf
export HIVE_AUX_JARS_PATH=/opt/module/bigdata/apache-hive-1.2.1-bin/lib

```

### 初使化元数据数据库

```
schematool -initSchema -dbType mysql

```


### 启动hive

```
hive
```
- hive启动说明

![](https://github.com/opensourceteams/apache-hive-1.2.1-bin/blob/master/md/images/hive.png)



### 启动hive metastore服务
- 启动该服务会开一个 9083端口
- netstat -ntlup 可以查看服务端占用的端口

```
hive --service metastore & 
```

![](https://github.com/opensourceteams/apache-hive-1.2.1-bin/blob/master/md/images/netstat.jpg)



### 停止hive metastore服务
```
ps -ef|grep hive

kill -9 pid
```

### 启动hiveserver服务 (该服务可以不启动)
```
hive --service hiveserver2 & 
```


## 操作
### 启动 hive
```
hive
```

### 退出 hive命令
```
exit;
```
### 创建database
```
create database;
```

### 显示所有的database
```
show databases;
```

### 显示所有的表
```
show tables;
```
### 创建表
```
CREATE TABLE IF NOT EXISTS employee (  name String, salary String) COMMENT 'Employee details' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LINES TERMINATED BY '\n' STORED AS TEXTFILE;
```

### 插入数据
- 数据文件 employee.txt 
 
```
小明    100000
小王    50000
小李    60000
```

- 插入数据文件到hive上 

```
LOAD DATA LOCAL INPATH '/home/liuwen/temp/employee.txt' OVERWRITE INTO TABLE employee;
```

### 查看表数据
```
 select * from employee;
```

## 问题处理

### 无法访问spark-assembly-*.jar

```

[liuwen@standalone lib]$ schematool -initSchema -dbType mysql
ls: 无法访问/opt/module/bigdata/spark-2.4.0-bin-without-hadoop/lib/spark-assembly-*.jar: 没有那个文件或目录
Metastore connection URL:        jdbc:mysql://macbookmysql.com:3306/hive?createDatabaseIfNotExist=true
Metastore Connection Driver :    com.mysql.cj.jdbc.Driver
Metastore connection User:       admin
Starting metastore schema initialization to 1.2.0
Initialization script hive-schema-1.2.0.mysql.sql
Error: Duplicate key name 'PCS_STATS_IDX' (state=42000,code=1061)
org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
*** schemaTool failed ***


```
- 处理方法
- 处理$HIVE_HOME/bin/hive

```
  #sparkAssemblyPath=`ls ${SPARK_HOME}/lib/spark-assembly-*.jar`
  sparkAssemblyPath=`ls ${SPARK_HOME}/jars/*.jar`
```
end