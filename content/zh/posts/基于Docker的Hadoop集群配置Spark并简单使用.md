---
keywords:
- "Docker"
- "Hadoop"
- "大数据"
- "Spark"
title: "基于Docker的Hadoop集群配置Spark并简单使用"
date: 2021-11-16T14:12:23+08:00
lastmod: 2021-11-16T14:12:23+08:00
description: "基于Docker的Hadoop集群配置Spark并简单使用"
draft: false 
author: Calvin Haynes 
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
tags:
- "Hadoop"
- "服务器集群"
- "Docker"
- "Spark"
categories: "大数据"
img: https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/Wallpaper/server-cluster.3p8zh61kpbm0.png
---

## 前言

上一篇文章我写了[如何利用Docker搭建一个Hadoop-muti-node-cluster](https://zhuanlan.zhihu.com/p/430943139)，从中我们得知Hadoop可以通过MapReduce机制实现一些计算任务，但是由于MapReduce任务需要跑很多次而且需要多次迭代，每次迭代计算结果都要存到HDFS中，而HDFS本质上就是硬盘，相当于把每次运算结果写入硬盘，而且还要考虑备份的问题，所以在MapReduce的传统计算中存储占用了绝大部分时间。而Spark不同，它是将中间计算结果存储在内存中并直接在内存中执行迭代计算，速度会更快（CPU直接访问内存速度远远快于访问磁盘），是现在非常流行的通用云计算引擎/框架。

> **官网描述：**Apache Spark 是用于大规模数据处理的统一分析引擎。它提供了 Java、Scala、Python 和 R 中的高级 API，以及支持通用执行图的优化引擎。它还支持一组丰富的更高级别的工具，包括[SparkSQL](https://spark.apache.org/docs/latest/sql-programming-guide.html)用于SQL和结构化数据的处理，[MLlib](https://spark.apache.org/docs/latest/ml-guide.html)机器学习，[GraphX](https://spark.apache.org/docs/latest/graphx-programming-guide.html)用于图形处理，以及[结构化流](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)的增量计算和流处理。

## 1 - 搭建前注意事项

### 官网说明：

Spark 应用程序作为集群上的独立进程集运行，由`SparkContext` 主程序（称为*驱动程序*）中的对象协调。

具体来说，为了在集群上运行，SparkContext 可以连接到多种类型的*集群管理器* （**Spark 自己的独立集群管理器、Mesos、YARN 或 Kubernetes**），它们在应用程序之间分配资源。连接后，Spark 会在集群中的节点上获取*执行*程序，这些进程为您的应用程序运行计算和存储数据。接下来，它将您的应用程序代码（由传递给 SparkContext 的 JAR 或 Python 文件定义）发送到执行程序。最后，SparkContext 将*任务*发送给执行程序以运行。

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.fbwnal6nc28.png)

**Spark支持的集群管理器如下：**

- [Standalone](https://spark.apache.org/docs/latest/spark-standalone.html)– Spark 附带的简单集群管理器，可以轻松设置集群。
- [Apache Mesos](https://spark.apache.org/docs/latest/running-on-mesos.html) – 一个通用的集群管理器，也可以运行 Hadoop MapReduce 和服务应用程序。（已弃用）
- [Hadoop YARN](https://spark.apache.org/docs/latest/running-on-yarn.html) – Hadoop 2 中的资源管理器。
- [Kubernetes](https://spark.apache.org/docs/latest/running-on-kubernetes.html) – 一个开源系统，用于自动部署、扩展和管理容器化应用程序。

> 由以上说明在不同的集群模式（即选用不同的集群管理器）下，Spark有不同的部署方法，本文选用最简单的Standalone模式部署（其实Docker和Kubernetes搭建更好，最近没时间学，之后再写相关文章吧）

## 2 - 下载Spark&Scala

> 要安装 Spark Standalone 模式，您只需在集群的**每个节点**上放置一个编译版本的 Spark。

- 关于Spark的下载，一定要寻找对应Hadoop版本的Spark，否则可能会出现奇奇怪怪的问题。
  - Spark下载网址在这里：https://spark.apache.org/downloads.html
- 按照Spark下载网站上的说明下载对应的Scala版本（比如下面这个版本官网的说明就是建议下载Scala2.12版本）
  - Scala下载网址在这里：https://www.scala-lang.org/download/


![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.1po6ki27ev34.png)

> 关于二者的下载，如果看过我上篇文章的读者，应该已经发现在上篇文的[Dockerfile](https://gist.github.com/CalvinHaynes/6f872a1fb4ba6717a293011f6bfb63a2#file-Hadoop%E9%9B%86%E7%BE%A4Dockerfile)中我已经下载了相应版本的Spark和Scala

## 3 - 配置环境变量

设置一下Spark必须用到的环境变量：（将下面这些环境变量配置的键值对写入`~/.bashrc`）

```bash
#SCALA Variables
export PATH=${JAVA_HOME}/bin:${PATH}
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
export SCALA_HOME=/usr/local/scala
export PATH=$PATH:$SCALA_HOME/bin
export PATH=$PATH:$SCALA_HOME/sbin
#SCALA Variables
#SPARK Variables
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin
export SPARK_DIST_CLASSPATH=/usr/local/hadoop/bin/hadoop
export PYTHONPATH=$SPARK_HOME/python/:$SPARK_HOME/python/lib/py4j-0.10.6-src.zip:$PYTHONPATH
export PATH=$PATH:$SPARK_HOME/sbin
#SPARK Variables
```

**更新bash配置**：

```bash
 source ~/.bashrc
```

## 4 - 配置Spark配置文件（conf目录）

### 1 - **spark-env.sh文件**

- 将下面这些配置写入`$SPARK_HOME/conf`目录下的`spark-env.sh`配置文件中（其中**SPARK_MASTER_HOST**出换成你**master**的ip地址，或者直接写master应该也可以）

- 注意，当Spark安装时，conf/spark-env.sh默认是不存在的。你可以复制conf/spark-env.sh.template创建它（`cp spark-env.sh.template spark-env.sh`），**其他配置文件也同理。**

```sh
export SPARK_DIST_CLASSPATH=/usr/local/hadoop/bin/hadoop
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export SCALA_HOME=/usr/local/scala
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export SPARK_MASTER_HOST=172.18.0.2
export SPARK_HOME=/usr/local/spark
export SPARK_WORKER_CORES=1
export SPARK_WORKER_MEMORY=512m
export SPARK_WORKER_INSTANCES=2
```

### 2 - Workers

```workers
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# A Spark Worker will be started on each of the machines listed below.
# 在这里写上包括master节点在内的所有工作节点
master
slave1
slave2
```

### 3 - log4j.properties

- 打印Spark运行日志，方便后期排错和检查运行情况

```properties
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Set everything to be logged to the console
log4j.rootCategory=WARN, console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n

# Set the default spark-shell/spark-sql log level to WARN. When running the
# spark-shell/spark-sql, the log level for these classes is used to overwrite
# the root logger's log level, so that the user can have different defaults
# for the shell and regular Spark apps.
log4j.logger.org.apache.spark.repl.Main=WARN
log4j.logger.org.apache.spark.sql.hive.thriftserver.SparkSQLCLIDriver=WARN

# Settings to quiet third party logs that are too verbose
log4j.logger.org.sparkproject.jetty=WARN
log4j.logger.org.sparkproject.jetty.util.component.AbstractLifeCycle=ERROR
log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=INFO
log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=INFO
log4j.logger.org.apache.parquet=ERROR
log4j.logger.parquet=ERROR

# SPARK-9183: Settings to avoid annoying messages when looking up nonexistent UDFs in SparkSQL with Hive support
log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL
log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR

# For deploying Spark ThriftServer
# SPARK-34128：Suppress undesirable TTransportException warnings involved in THRIFT-4805
log4j.appender.console.filter.1=org.apache.log4j.varia.StringMatchFilter
log4j.appender.console.filter.1.StringToMatch=Thrift error occurred during processing of message
log4j.appender.console.filter.1.AcceptOnMatch=false

```

## 5 - 启动Spark

- 可以使用以下 shell 脚本启动或停止集群，基于 Hadoop 的部署脚本，并在`$SPARK_HOME/sbin`以下位置可用：
  - `sbin/start-master.sh` - 在执行脚本的机器上启动主实例。
  - `sbin/start-workers.sh`- 在`conf/workers`文件中指定的每台机器上启动一个工作实例。
  - `sbin/start-worker.sh` - 在执行脚本的机器上启动一个工作实例。
  - `sbin/start-all.sh` - 如上所述启动一个主节点和多个工作节点。
  - `sbin/stop-master.sh`- 停止通过`sbin/start-master.sh`脚本启动的 master 。
  - `sbin/stop-worker.sh` - 停止执行脚本的机器上的所有工作实例。
  - `sbin/stop-workers.sh`- 停止`conf/workers`文件中指定的机器上的所有工作实例。
  - `sbin/stop-all.sh` - 如上所述停止主节点和工作节点。

- 使用`jps`就可以看到现在正在运行的进程了

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.21bfx40q5ppc.png)

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.2t5etunougi0.png)

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.301n10rxy4s0.png)

## 6 - 启动spark-shell并测试运行一个简单的Scala字数计算程序

- 启动Spark-shell：

```
spark-shell
```

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.34xsi5bfzj60.png)

- 完整代码如下：

```scala
# 接续上次文章中上传的文件
val textFile = sc.textFile("hdfs:///calvinhaynes/README.txt")
textFile.first()
val wordCount = textFile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey((a,b) => a+b)
wordCount.collect()
textFile.count()
textFile.take(10)
:quit
```

- 运行效果如下：

![Spark-shell](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/Spark-shell.2pgae0eh5y60.jpg)

![Spark-shell2](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/Spark-shell2.4dysyv8fgns0.jpg)

## 结语

作为上一篇文章的补充，简单了解一下Spark这个现在流行的通用云计算框架。

感谢您看到最后，如果本文对您有所帮助的话，还希望给我一个一键三连（狗头保命），如果对于我和我的文章感兴趣的话，欢迎点一个关注，您会收到我回答和文章的更新通知，也欢迎加入我建立的技术交流群QQ：725133797 讨论交流。

最后附上我的个人博客站：[https://blog.calvinhaynes.top/](https://link.zhihu.com/?target=https%3A//blog.calvinhaynes.top/)，欢迎访问和交流

