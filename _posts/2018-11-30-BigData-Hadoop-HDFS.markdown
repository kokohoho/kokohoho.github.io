---
layout:     post
title:      "「原创」Hadoop 一篇精通HDFS"
subtitle:   "HDFS概述、Shell/API操作、读/写流程、NN/2NN工作机制、DN工作机制"
date:       2021-11-29 12:00:00
author:     "NI"
header-img: "img/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
  - Hadoop
  - HDFS
  - 原创
---

<font color='red'></font>

## 第1章 HDFS概述

### 1.1 HDFS产出背景及定义

#### 1)HDFS产出背景

随着数据量越来越大，在一个操作系统存不下所有的数据，那么就分配到更多的操作系
统管理的磁盘中，但是不方便管理和维护，迫切<font color='red'>需要一种系统来管理多台机器上的文件</font>，这
就是分布式文件管理系统。<font color='red'>HDFS 只是分布式文件管理系统中的一种。</font>

#### 2)HDFS定义

HDFS（Hadoop Distributed File System），它是一个文件系统，用于存储文件，<font color='red'>通过目
录树来定位文件；其次，它是分布式的</font>，由很多服务器联合起来实现其功能，*集群中的服务
器有各自的角色。

<font color='red'>HDFS 的使用场景：适合一次写入，多次读出的场景。</font>一个文件经过创建、写入和关闭
之后就不需要改变。

### 1.2 HDFS优缺点

#### <font color='red'>HDFS优点</font>

 1)高容错性

> 数据自动保存<font color='red'>多个副本</font>。它通过增加副本的形式，提高容错性。

> 某一个副本丢失以后，它可以自动恢复。

 2)适合处理大数据

> 数据规模：能够处理数据规模达到GB、TB、甚至<font color='red'>PB</font>级别的数据。

> 文件规模：能够处理<font color='red'>百万</font>规模以上的文件数量，数量相当之大。

 3)可构建在廉价机器上，通过多副本机制，提高可靠性

 <font color='red'>HDFS缺点</font>

 1)<font color='red'>不适合低延时数据访问</font>，例如毫秒级存储数据，是做不到的

 2)<font color='red'>无法高效的对大量小文件进行存储</font>

> 存储大量小文件的话，它会占用NameNode大量的内存来存储文件目录和块信息。
> 因为NameNode的物理内容是有限的，所以大量小文件会占用大量可用内存。

> 小文件存储的寻址时间会超过读取时间，它违反了HDFS的设计目标。

 3)不支持并发写入、文件随机修改

> 一个文件只能有一个写，不允许多个线程同时写。

> <font color='red'>仅支持数据追加append</font>，不支持文件的随机修改。


### 1.3 HDFS组成架构

#### <font color='purple'>1) NameNode（nn）：就是Master，它是一个主管、管理者。</font>

(1) 管理HDFS的名称空间；

(2) 配置副本策略；

(3) 管理数据块（Block）映射信息；

(4) 处理客户端读写请求；

#### <font color='purple'>2) DataNode（dn）：就是Slave、Worker。NameNode下达命令，DataNode执行实际操作。</font>

 (1) 存储实际的数据块；

 (2) 执行数据块读/写操作；


#### <font color='purple'>3) Client：就是客户端。</font>

 (1) 文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传；
 
 (2) 与NameNode交互，获取文件的位置信息；
 
 (3) 与DataNode交互，读取或者写入数据；
 
 (4) Client提供一些命令来管理HDFS，比例NameNode格式化；
 
 (5) Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作；

#### <font color='purple'>4) SecondaryNameNode（2nn）：并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务</font>

 (1) 辅助NameNode，分担其工作量，比如定期合并fsimage和Edits，并推送给NameNode；
 
 (2) 在紧急情况下，可辅助恢复NameNode；

### 1.4 HDFS文件块大小（面试重点）

HDFS中的文件在物理上是分块存储（Block），块的大小可以通过配置参数
( dfs.blocksize）来规定，<font color='red'>默认大小在Hadoop2.x/3.x版本中是128M，1.x版本中是64M。</font>

> <font color='red'>如果寻址时间约为10 ms，即查到目标Bolck的时间为10ms</font>

> <font color='red'>寻址时间为传输时间的1%时，则为最佳状态。（专家）因此，传输时间 = 10ms/0.01 = 1000ms =1s </font>

> <font color='red'>而目前磁盘的传输速率普遍为100MB/s </font>

思考：为什么块的大小不能设置太小，也不能设置太大？

（1）HDFS的块设置太小，会增加寻址时间，程序一直在找块的开始位置；

（2）如果块设置的太大，从磁盘传输数据的时间会明显大于定位这个块开
始位置所需的时间。导致程序在处理这块数据时，会非常慢。

总结：HDFS块的大小设置主要取决于磁盘传输速率。机械硬盘128MB，固态硬盘256MB。


## 第2章 HDFS的Shell操作（开发重点）

## 第3章 HDFS的API操作

## 第4章 HDFS的读写流程（面试重点）

### 4.1 HDFS写数据流程

#### 4.1.1 剖析文件写入

![avatar](/img/bigdata/hadoop/hdfs/write.png)

 （1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在

  （2）NameNode返回是否可以上传。

 （3）客户端请求第一个Block上传到哪几个DataNode服务器上。

 （4）NameNode返回3个DataNode节点，分别为dn1，dn2，dn3。

 （5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。

 （6）dn1，dn2，dn3逐级应答客户端。

 （7）客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存）。以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个Packet会放入一个应答队列等待应答

 （8）当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。

#### 4.1.2 网络拓扑-节点距离计算

![avatar](/img/bigdata/hadoop/hdfs/choose.png)

在 HDFS 写数据的过程中，NameNode 会选择距离待上传数据最近距离的 DataNode 接
收数据。那么这个最近距离怎么计算呢？
节点距离：两个节点到达最近的共同祖先的距离总和。

#### 4.1.3 机架感知（副本存储节点选择）

![avatar](/img/bigdata/hadoop/hdfs/replication.png)

第一个副本首先选择本机架的节点，优选选择客户端所在节点；第二个副本选择另外一个机架中的节点；第三个副本选择第二个副本所在
机架的另一个节点

### 4.2 HDFS读数据流程

![avatar](/img/bigdata/hadoop/hdfs/read.png)

  （1）客户端通过 DistributedFileSystem 向 NameNode 请求下载文件，NameNode 通过查询元数据，找到文件块所在的 DataNode 地址。

  （2）挑选一台 DataNode（就近原则，然后随机）服务器，请求读取数据。

  （3） DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以 Packet 为单位来做校验）。

  （4）客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。


## 第5章 NameNode和SecondaryNameNode

### 5.1 NN和2NN工作机制

### 5.2 Fsimage和Edits解析

### 5.3 CheckPoint时间设置

## 第6章 DataNode

### 6.1 DataNode工作机制

### 6.2 数据完整性

### 6.3 掉线时限参数设置
























