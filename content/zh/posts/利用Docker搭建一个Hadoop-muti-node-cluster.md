---
keywords:
- "Docker"
- "Hadoop"
- "大数据"
title: "利用Docker搭建一个Hadoop Mutinode Cluster"
date: 2021-11-09T11:09:02+08:00
lastmod: 2021-11-09T11:09:02+08:00
description: "利用Docker搭建一个多节点服务器集群"
draft: false 
author: Calvin Haynes
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
tags:
- "Docker"
- "Hadoop"
categories: "大数据"
img: https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/Wallpaper/server-cluster.3p8zh61kpbm0.png
---
## 前言

最近上了一门关于云计算&大数据的课，课堂上有留一些大作业，所以我也想借此简单的入门一下这个领域，并记下此文作为总结和回顾，同时也方便让大家一起简单认识一下这门技术。

这次大作业一的要求就是搭建一个Hadoop的多节点服务器集群，要求是三个节点，一个master和两个slave，老师的建议是虚拟机搭集群，但是由于跑虚拟机占用资源太多（虚拟机加上图形界面本身就很大很吃资源，然后还要创建多个虚拟机，很麻烦），而且Docker容器化虚拟技术的诞生本就是更容易也更适合做服务器集群的（**之前有幸学了一点Docker真是赚到了笑死**）

但是仅仅按照步骤搭建这个环境就没什么意思了，所以我也想简单理解一下这些配置文件和一些名词，和大数据的一些思想，在文章的末尾我会添加一个版块专门写我查找的很多优质资料以及我的一些学习见解。

**那么下面我们首先开始搭建步骤的详解，毕竟这才是这个文章的主线！**

## 搭建思路的顺序图

![Docker-Hadoop-Muti-Node-Cluste](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/Docker-Hadoop-Muti-Node-Cluste.6fl283tfznc0.png)

上图可能有点小，而且未来会有变动，建议大家也在下面这个网站上查看这个思维导图的完整版（感谢知犀思维导图的在线查看的功能，真的很感谢这个开源的思维导图软件）

**链接：https://www.zhixi.com/view/dca65eb3 密码：2426**

> **知犀思维导图的官方推荐文案，感兴趣的小伙伴试一试吧，是真的良心！！**
>
> 发现一个好用的思维导图软件，知犀思维导图（免费）。有几千个免费优质模板，不用开会员，节点数不限制，直接导出高清无水印图片，还支持导出PDF，Word。很强大很良心，颜值还挺高，知犀有在线版、电脑版，还有手机App，多端云同步，推荐试试：https://www.zhixi.com/

## 1 - 写基础环境的Dockerfile并build一个image

**Dockerfile如下：**

```dockerfile
# 实验一利用Hadoop搭建服务器集群的基础环境Dockerfile
FROM ubuntu:20.04
ARG arch_name=amd64

# 1.备份源列表
RUN cp /etc/apt/sources.list /etc/apt/sources.backup.list
# 2.把本目录下的sources.list中的镜像源添加到Docker中，下载速度起飞
COPY sources.list /etc/apt/sources.list
# 3.更新源
RUN apt-get update 

# 翻墙代理
# RUN export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*') && \
# export https_proxy="http://${hostip}:7890" && \
# export http_proxy="http://${hostip}:7890"

# 设置一些环境变量
ENV TZ=Asia/Shanghai \
    LANG=en_US.utf8 \
    LANGUAGE=en_US.UTF-8 \
    DEBIAN_FRONTEND=noninteractive

# 安装一些Hadoop集群需要的基本环境和辅助程序
RUN apt-get install -y openjdk-8-jdk sudo vim ssh openssl wget openssh-server openssh-client net-tools iputils-ping

# pdsh全称是parallel distributed shell，可以并行执行对远程目标主机的操作，利于解决批量执行命令或分发任务的运维需求。
# 适用于大批量服务器的配置，部署，文件复制等运维操作。
RUN apt-get install -y pdsh && \
    echo ssh >> /etc/pdsh/rcmd_default

# SSH配置
RUN ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa && \
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
    chmod 600 ~/.ssh/authorized_keys && \
    chmod 700 ~/.ssh && \
    echo "service ssh start" >> ~/.bashrc 

# 安装Hadoop
RUN wget https://dlcdn.apache.org/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz && \
    tar -xzf hadoop-3.2.2.tar.gz && \
    mv hadoop-3.2.2 /usr/local/hadoop && \
    rm hadoop-3.2.2.tar.gz 

# 安装Scala
RUN wget  https://downloads.lightbend.com/scala/2.12.6/scala-2.12.6.tgz && \
    tar -xzf scala-2.12.6.tgz && \
    mv scala-2.12.6 /usr/local/scala && \
    rm scala-2.12.6.tgz

# 安装Spark
RUN wget https://dlcdn.apache.org/spark/spark-3.2.0/spark-3.2.0-bin-hadoop3.2-scala2.13.tgz && \
    tar -xzf spark-3.2.0-bin-hadoop3.2-scala2.13.tgz && \
    mv spark-3.2.0-bin-hadoop3.2-scala2.13 /usr/local/spark && \
    rm spark-3.2.0-bin-hadoop3.2-scala2.13.tgz

# 创建一些目录，一会儿配置Hadoop文件要用到
RUN mkdir -p /usr/local/hadoop/hadoop_data/hdfs && \
    mkdir -p /usr/local/hadoop/hadoop_data/hdfs/tmp && \
    mkdir -p /usr/local/hadoop/hadoop_data/hdfs/namenode && \
    mkdir -p /usr/local/hadoop/hadoop_data/hdfs/datanode && \
    mkdir -p /usr/local/hadoop/hadoop_data/hdfs/edits && \
    mkdir -p /usr/local/hadoop/hadoop_data/hdfs/checkpoints && \
    mkdir -p /usr/local/hadoop/hadoop_data/hdfs/checkpoints/edits

# 开放Hadoop相关端口，方便外部进行访问
EXPOSE 8088 50090 9864 19888 50070 8080

CMD ["/bin/bash"]
```

> 从上述Dockerfile中不难看出这步仅仅是安装一些必备的环境、配置SSH、声明开放一些端口、创建一些Hadoop的HDFS需要用到的目录，当然Spark和Scala的安装是第二个大作业的要求，我会在下一篇博文分享

**利用此`Dockerfile`去`build`一个`image`：**（不要忽略后面的**“.”**）

```powershell
docker build -t="hadoop-mutinode-cluster" .
```

## 2 - 配置Docker桥接网络

> **因为服务器集群相互之间需要连通，所以必须进行网络的配置。**
>
> Docker网络bridge桥接模式，是创建和运行容器时默认模式。这种模式会为每个容器分配一个独立的网卡，桥接到默认或指定的bridge上，同一个Bridge下的容器下可以互相通信的。我们也可以创建自定义bridge以满足个性化的网络需求。

- 创建`Docker`桥接网络

```powershell
docker network create --driver bridge hadoop-br
```

以上命令创建了一个名为`hadoop-br`的bridge类型的网络

- 创建三个`Container`，也就是三个服务器，并指定网络和端口映射关系

```powershell
# master节点创建命令如下：
docker run -itd --network hadoop-br --name hadoop_master --hostname master --ip 172.18.0.2 --add-host slave1:172.18.0.3 --add-host slave2:172.18.0.4 -p 8088:8088 -p 8080:8080 -p 50090:50090 -p 9864:9864 -p 19888:19888 -p 50070:50070 calvinhaynes412/hadoop-mutinodes-cluster:v1.2.0
# slave1
docker run -itd --network hadoop-br --name hadoop_slave1 --hostname slave1 --ip 172.18.0.3 --add-host master:172.18.0.2 --add-host slave2:172.18.0.4 calvinhaynes412/hadoop-mutinodes-cluster:v1.2.0
# slave2
docker run -itd --network hadoop-br --name hadoop_slave2 --hostname slave2 --ip 172.18.0.4 --add-host master:172.18.0.2 --add-host slave1:172.18.0.3 calvinhaynes412/hadoop-mutinodes-cluster:v1.2.0
```

- 查看网络情况

```powershell
docker network inspect hadoop-br 
```

**执行上述命令会得到类似下面的结果：**

```json
[
    {
        "Name": "hadoop-br",
        "Id": "273d82ad790f475ef14a78bd5f203d3e441f446683a89e01da5b8240aae731a9",
        "Created": "2021-11-05T07:04:49.300557Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "1440ea1055bf14ded8c756bea2ad3f185b00e5663e76b1cf382dd08574be4965": {
                "Name": "hadoop_master",
                "EndpointID": "f193c3bc481b9e447fcffefa50b10519ee6488a34862b51a1f77be52b1d2c922",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "627843643b974c89063fdd8659c6f2f3b00c1e5ad232b56a3a6513d50d30ce06": {
                "Name": "hadoop_slave2",
                "EndpointID": "1366b734f7579ae61bae3f41ba53a1f2433bd8df9edc115364dc83e7926b0461",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "62a874539d08a4074374cf74f75832cb2c2245a7c95e809f36cbf1250e8a49ac": {
                "Name": "hadoop_slave1",
                "EndpointID": "324e045b4bc61b44967c246fe7ee363582f5cc7c3b78c1327270e3f504ce6ab6",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

**从以上输出不难看出三台机器的ip**：

```powershell
172.18.0.2 master
172.18.0.3 slave1
172.18.0.4 slave2
```

**可以登录三个Container，尝试互相可不可以ping的通：**

```powershell
# 在每个Container中分别测试能不能ping通另外两台机器，如果ping的通代表网络没问题了
docker exec -it master bash
docker exec -it slave1 bash
docker exec -it slave2 bash
```

- 测试互相SSH免密登录

```bash
ping master 
ping slave1
ping slave2
ssh master
ssh slave1
ssh slave2
```

如果上述均没问题，那么到此为止修改Hadoop配置之前的预备操作就到此结束了

## 3 - 配置环境变量

设置一下Hadoop必须用到的环境变量：（将下面这些环境变量配置的键值对写入`~/.bashrc`）

```bash
#Hadoop Variables
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPPED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HODOOP_HOME
export CLASSPATH=$CLASSPATH:/usr/local/hadoop/lib/*:.
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native:$JAVA_LIBRARY_PATH
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
#Hadoop Variables
```

**更新bash配置**：

```bash
source ~/.bashrc
```

## 4 - 配置Hadoop配置文件（核心部分）

**在hadoop集群中，需要配置的文件主要包括四个，分别是core-site.xml、hdfs-site.xml、mapred-site.xml和yarn-site.xml，这四个文件分别是对不同组件的配置参数**：

![img](https://pic3.zhimg.com/80/v2-dff2be66297ddbf26c60cab2b3e1d3be_720w.jpg)

具体参数含义和参数值可以看如下的四个官方文档：（**不同版本只要修改docs的下一级/中的版本号即可**）

- code-site.xml：https://hadoop.apache.org/docs/r3.2.2/hadoop-project-dist/hadoop-common/core-default.xml
- hdfs-site.xml：https://hadoop.apache.org/docs/r3.2.2/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml
- mapred-site.xml：https://hadoop.apache.org/docs/r3.2.2/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml
- yarn-site.xml：https://hadoop.apache.org/docs/r3.2.2/hadoop-yarn/hadoop-yarn-common/yarn-default.xml

**我的参数配置如下：**

- **core-site.xml**

```xml
<configuration>
<!--配置namenode的地址-->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://master:9000</value>
  </property>
<!-- 指定hadoop运行时产生文件的存储目录 -->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/tmp</value>
 </property>        
</configuration>
```

- **hdfs-site.xml**

```xml
<configuration>

  <!--上传的文件的副本数(数据备份数，缺省值为3)-->
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
    
  <!-- HDFS的webapp允许使用 -->
  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>
    
  <!-- namenode web应用端口地址 -->
  <property>
    <name>dfs.namenode.http-address</name>
    <value>master:50070</value>
  </property>
  <!--存放namenode所使用的元数据的路径（文件名、副本的数量、每一个块的Datanode的位置）-->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/namenode</value>
  </property>
  <!--namenode 存放 edits 的路径（edits中存储的是HDFS操作的日志记录）-->
  <property>
    <name>dfs.namenode.edits.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/edits</value>
  </property>

  <!-- SecondaryNameNode的作用是合并fsimage和edits文件。-->
  <!--但是如果NameNode执行了很多操作的话，就会导致edits文件会很大，那么在下一次启动的过程中，
      就会导致NameNode的启动速度很慢，慢到几个小时也不是不可能，所以出现了SecondNameNode。-->
  <!-- secondary namenode web 监听端口 -->
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>master:50090</value>
  </property>
  <!-- secondary namenode 节点存储 checkpoint 目录-->
  <property>
    <name>dfs.namenode.checkpoint.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/checkpoints</value>
  </property>
  <!-- secondary namenode 节点存储 edits 目录-->
  <property>
    <name>dfs.namenode.checkpoint.edits.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/checkpoints/edits</value>
  </property>
  
  <!--DataNode所使用的元数据保存路径-->
  <property>
    <name>dfs.datanode.name.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/datanode</value>
  </property>
  <!--DataNode 节点存储 edit 文件-->
  <property>
    <name>dfs.datanode.edits.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/edits</value>
  </property>

</configuration>
```

- **mapred-site.xml**

```xml
<configuration>

  <!--上传的文件的副本数-->
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>

  <!-- namenode web 监听端口 -->
  <property>
    <name>dfs.namenode.http-address</name>
    <value>master:50070</value>
  </property>
  <!--namenode 所使用的元数据保存路径设置-->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/namenode</value>
  </property>
  <!--namenode 存放 edits 的路径（edits中存储的是HDFS操作的日志记录）-->
  <property>
    <name>dfs.namenode.edits.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/edits</value>
  </property>

  <!-- SecondaryNameNode的作用是合并fsimage和edits文件。-->
  <!--但是如果NameNode执行了很多操作的话，就会导致edits文件会很大，那么在下一次启动的过程中，
      就会导致NameNode的启动速度很慢，慢到几个小时也不是不可能，所以出现了SecondNameNode。-->
  <!-- secondary namenode web 监听端口 -->
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>master:50090</value>
  </property>
  <!-- secondary namenode 节点存储 checkpoint 文件目录-->
  <property>
    <name>dfs.namenode.checkpoint.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/checkpoints</value>
  </property>
  <!-- secondary namenode 节点存储 edits 文件目录-->
  <property>
    <name>dfs.namenode.checkpoint.edits.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/checkpoints/edits</value>
  </property>
  
  <!--DataNode所使用的元数据保存路径-->
  <property>
    <name>dfs.datanode.name.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/datanode</value>
  </property>
  <!--DataNode 节点存储 edit 文件-->
  <property>
    <name>dfs.datanode.edits.dir</name>
    <value>file:/usr/local/hadoop/hadoop_data/hdfs/edits</value>
  </property>

</configuration>
```

- **yarn-site.xml**

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
  
  <!-- 指定nodeManager组件在哪个机子上跑 -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.shuffleHandler</value>
  </property>
  
  <!-- 指定resourcemanager组件在哪个机子上跑 -->
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>master</value>
  </property> 
  <!--resourcemanager web地址-->
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>master:8088</value>
  </property>

</configuration>
```

- **workers**

```
master
slave1
slave2
```

## 5 - 将master中的Hadoop配置文件复制到两个slave中

在`master`的`container`中命令行敲入以下命令实现将Hadoop配置文件复制到两个slave中：

```bash
scp -r /usr/local/hadoop/etc/hadoop root@slave2:/usr/local/hadoop/etc
scp -r /usr/local/hadoop/etc/hadoop root@slave1:/usr/local/hadoop/etc
```

## 6 - 启动所有服务&访问相应的Web服务

1. 格式化NameNode

```bash
bin/hdfs namenode -format
```

2. 启动HDFS

```bash
sbin/start-all.sh
```

3. JPS命令查看各节点HDFS运行情况

```bash
jps
```

```bash
20528 NodeManager
20368 ResourceManager
20065 SecondaryNameNode
19636 NameNode
19791 DataNode
27423 Jps
```

4. 通过浏览器访问HDFS对外提供的web服务端口

- 首先要查看端口映射关系，因为创建容器的时候`-P`将容器开放的端口随机映射到宿主机的一个随机空闲端口上

```powershell
docker container ls
```

- 然后就可以根据其中的端口映射关系访问对应的`webapp`服务（大部分在配置文件中都能找到相应的设置）：

  - `namenode information`的`webapp`：master(ip):50070  ====> http://localhost:50070

  ![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.2fpglw0gi3fo.png)

  - `secondnamenode information` 的 webapp：master(ip):50090 ====> http://localhost:50090/

  ![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.3elnxyw5fam0.png)

  - `datanode information` 的 `webapp`：workers(ip):9864  ====> http://localhost:9864/

  ![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.7dfy48s625k0.png)

  - `YARN`的`Webapp`：**resourcemanager url: master(ip):8088** ====> http://localhost:8088/

  ![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.3g8g4r14p5q0.png)

  - 启动`JobHistoryService`：`bin/mapred --daemon start historyserver` ----> **JobHistory url: master(ip):19888**  ====> http://localhost:19888/

  ![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.1m3ajupuk7k0.png)

## 7 - 简单测试一些基本的HDFS操作

- 新建一个文件夹

```bash
bin/hdfs dfs -mkdir /testHDFS
```

- 上传一个文件（**为了方便我上传的就是hadoop的README文件**）

```bash
bin/hdfs dfs -put /usr/local/hadoop/README.txt /testHDFS
```

- `Webapp`查看新建的文件夹和文件

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.2zctrgfe9zg0.png)

![image](https://cdn.jsdelivr.net/gh/CalvinHaynes/ImageHub@main/BlogImage/image.5uxgmau6hj00.png)

## 8 - 对Hadoop的初步学习

### （一）什么是Hadoop？

> The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

根据上述官网的说明可以看出Hadoop就是一个分布式处理大数据集的一个框架，他把单服务器存储、处理数据扩展到了多台服务器集群分布式存储、计算数据，减轻了单台服务器的负载，并且提高了性能和稳定性（多台服务器集群互相备份也方便有服务器宕机的时候可以顶上去），并且提供了管理多台服务器集群的手段。

### （二）为什么出现了Hadoop？

其实不难思考，现代世界最重要的就是数据，面对海量的数据，老旧的单机存储和处理数据的手段已经不适用海量的甚至是无关联的数据集，分布式存储计算已经成为解决这些问题的核心手段，其实仔细思考，现在技术圈比较火的几个名词：微服务，云原生，NoSQL，大数据，分布式，云计算，人工智能，归根结底这些技术都离不开海量数据的存储，计算，分析等等，举例来说，Google搜索引擎每天要存储海量的网页，并且执行自己的分析算法，随着数据量越来越大，就要采用分布式的办法，所以他们先后提出了GFS、NGFS、BigTable，而Hadoop也一步步推出自己相关的解决方案，如下图：

![img](https://pic3.zhimg.com/v2-5ce9db6d9b9b17c005cc8f127ccfe8be_b.jpg)

### （三）初探Hadoop

Hadoop的核心架构，就是HDFS和MapReduce。其中HDFS（Hadoop File System）是为了**存储**海量的数据来存在的，而MapReduce是为海量数据提供了一个**计算**框架

#### 1 - HDFS

![img](https://pic4.zhimg.com/80/v2-12bac7206f243ab217e58a23a555da47_720w.jpg)

HDFS主要有三个要素：NameNode、DataNode、Client

- **NameNode**：Master节点，相当于所有DataNode的管理者，Namenode会将文件系统的一系列元数据（元数据这个词非常常见，它的意思就是表示数据的数据）存在其中，包括每个文件Block在DataNode中的信息。
- **DataNode**：是Slave节点（从节点），是文件存储的基本单元，它将Block存储在本地文件系统中，保存了Block的Meta-data，同时周期性地将所有存在的Block信息发送给NameNode。
- **Client：**切分文件；访问HDFS；与NameNode交互，获得文件位置信息；与DataNode交互，读取和写入数据。 
- **Block（块）**：Block是HDFS中的基本读写单元；HDFS中的文件都是被切割为block（块）进行存储的；这些块被复制到多个DataNode中；块的大小（通常为64MB）和复制的块数量在创建文件时由Client决定。

#### 2 - MapReduce

MapReduce就是总结了大数据算法的通用特点之后造的轮子，关于MapReduce的介绍和讲解网上的例子都大差不差，我推荐下面这几个介绍和讲解

- 官网：https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html
- Google的MapReduce论文讲解：https://zhuanlan.zhihu.com/p/34849261
- 五分钟搞懂Hadoop：https://zhuanlan.zhihu.com/p/20176725

## **结语**

感谢您看到最后，如果本文对您有所帮助的话，还希望给我一个一键三连（狗头保命），如果对于我和我的文章感兴趣的话，欢迎点一个关注，您会收到我回答和文章的更新通知，也欢迎加入我建立的技术交流群QQ：725133797 讨论交流。

最后附上我的个人博客站：[https://blog.calvinhaynes.top/](https://link.zhihu.com/?target=https%3A//blog.calvinhaynes.top/)，欢迎访问和交流

## 参考资料

https://zhuanlan.zhihu.com/p/25472769

https://www.cnblogs.com/upupfeng/p/13616125.html

https://hadoop.apache.org/docs/r3.2.2/

https://zhuanlan.zhihu.com/p/54994736



