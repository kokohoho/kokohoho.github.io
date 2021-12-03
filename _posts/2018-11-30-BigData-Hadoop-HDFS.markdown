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

思考：NN中的元数据是存储在哪里的？

首先，我们做个假设，如果存储在 NameNode 节点的磁盘中，因为经常需要进行随机访
问，还有响应客户请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在
内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的
FsImage。

这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新 FsImage，就会导
致效率过低，但如果不更新，就会发生一致性问题，一旦 NameNode 节点断电，就会产生数
据丢失。因此，引入 Edits 文件（只进行追加操作，效率很高）。每当元数据有更新或者添
加元数据时，修改内存中的元数据并追加到 Edits 中。这样，一旦 NameNode 节点断电，可
以通过 FsImage 和 Edits 的合并，合成元数据。

但是，如果长时间添加数据到 Edits 中，会导致该文件数据过大，效率降低，而且一旦
断电，恢复元数据需要的时间过长。因此，需要定期进行 FsImage 和 Edits 的合并，如果这
个操作由NameNode节点完成，又会效率过低。因此，引入一个新的节点SecondaryNamenode，
专门用于 FsImage 和 Edits 的合并。

![avatar](/img/bigdata/hadoop/hdfs/namenode.png)

#### 1）第一阶段：NameNode启动

（1）第一次启动 NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。

（2）客户端对元数据进行增删改的请求。

（3）NameNode 记录操作日志，更新滚动日志。

（4）NameNode 在内存中对元数据进行增删改。

#### 2）第二阶段：Secondary NameNode工作

（1）Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接带回 NameNode
是否检查结果。

（2）Secondary NameNode 请求执行 CheckPoint。

（3）NameNode 滚动正在写的 Edits 日志。

（4）将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。

（5）Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。

（6）生成新的镜像文件 fsimage.chkpoint。

（7）拷贝 fsimage.chkpoint 到 NameNode。

（8）NameNode 将 fsimage.chkpoint 重新命名成 fsimage。


### 5.2 Fsimage和Edits解析

NameNode被格式化之后，将在/opt/module/hadoop-3.1.3/data/tmp/dfs/name/current目录中产生如下文件

````
fsimage_0000000000000000000
fsimage_0000000000000000000.md5
seen_txid
VERSION
````

（1）Fsimage文件：HDFS文件系统元数据的一个永久性的检查点，其中包含HDFS文件系统的所有目
录和文件inode的序列化信息。

（2）Edits文件：存放HDFS文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先
会被记录到Edits文件中。

（3）seen_txid文件保存的是一个数字，就是最后一个edits_的数字

（4）每次NameNode启动的时候都会将Fsimage文件读入内存，加 载Edits里面的更新操作，保证内存
中的元数据信息是最新的、同步的，可以看成NameNode启动的时候就将Fsimage和Edits文件进行了合并。

#### 1)oiv查看Fsimage文件

（1）查看 oiv 和 oev 命令

````
[kokohoho@hadoop102 current]$ hdfs
oiv apply the offline fsimage viewer to an fsimage
oev apply the offline edits viewer to an edits file
````

（2）基本语法

hdfs oiv -p 文件类型 -i 镜像文件 -o 转换后文件输出路径

（3）案例实操

````
[kokohoho@hadoop102 current]$ pwd
/opt/module/hadoop-3.1.3/data/dfs/name/current
[kokohoho@hadoop102 current]$ hdfs oiv -p XML -i
fsimage_0000000000000000025 -o /opt/module/hadoop-3.1.3/fsimage.xml
[kokohoho@hadoop102 current]$ cat /opt/module/hadoop-3.1.3/fsimage.xml
````

将显示的 xml 文件内容拷贝到 Idea 中创建的 xml 文件中，并格式化。部分显示结果如
下。

````xml
<inode>
    <id>16386</id>
    <type>DIRECTORY</type>
    <name>user</name>
    <mtime>1512722284477</mtime>
    <permission>kokohoho:supergroup:rwxr-xr-x</permission>
    <nsquota>-1</nsquota>
    <dsquota>-1</dsquota>
</inode>
<inode>
    <id>16387</id>
    <type>DIRECTORY</type>
    <name>kokohoho</name>
    <mtime>1512790549080</mtime>
    <permission>kokohoho:supergroup:rwxr-xr-x</permission>
    <nsquota>-1</nsquota>
    <dsquota>-1</dsquota>
</inode>
<inode>
    <id>16389</id>
    <type>FILE</type>
    <name>wc.input</name>
    <replication>3</replication>
    <mtime>1512722322219</mtime>
    <atime>1512722321610</atime>
    <perferredBlockSize>134217728</perferredBlockSize>
    <permission>kokohoho:supergroup:rw-r--r--</permission>
    <blocks>
        <block>
            <id>1073741825</id>
            <genstamp>1001</genstamp>
            <numBytes>59</numBytes>
        </block>
    </blocks>
</inode >
````

思考：可以看出，Fsimage 中没有记录块所对应 DataNode，为什么？

在集群启动后，要求 DataNode 上报数据块信息，并间隔一段时间后再次上报。

#### 2)oev查看Edits文件

（1）基本语法

````
hdfs oev -p 文件类型 -i 编辑日志 -o 转换后文件输出路径
````

（2）案例实操

````
[kokohoho@hadoop102 current]$ hdfs oev -p XML -i
edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop3.1.3/edits.xml
[kokohoho@hadoop102 current]$ cat /opt/module/hadoop-3.1.3/edits.xml
````

将显示的 xml 文件内容拷贝到 Idea 中创建的 xml 文件中，并格式化。显示结果如下。

````xml

<?xml version="1.0" encoding="UTF-8"?>
<EDITS>
    <EDITS_VERSION>-63</EDITS_VERSION>
    <RECORD>
        <OPCODE>OP_START_LOG_SEGMENT</OPCODE>
        <DATA>
            <TXID>129</TXID>
        </DATA>
    </RECORD>
    <RECORD>
        <OPCODE>OP_ADD</OPCODE>
        <DATA>
            <TXID>130</TXID>
            <LENGTH>0</LENGTH>
            <INODEID>16407</INODEID>
            <PATH>/hello7.txt</PATH>
            <REPLICATION>2</REPLICATION>
            <MTIME>1512943607866</MTIME>
            <ATIME>1512943607866</ATIME>
            <BLOCKSIZE>134217728</BLOCKSIZE>
            <CLIENT_NAME>DFSClient_NONMAPREDUCE_-1544295051_1
            </CLIENT_NAME>
            <CLIENT_MACHINE>192.168.10.102</CLIENT_MACHINE>
            <OVERWRITE>true</OVERWRITE>
            <PERMISSION_STATUS>
                <USERNAME>kokohoho</USERNAME>
                <GROUPNAME>supergroup</GROUPNAME>
                <MODE>420</MODE>
            </PERMISSION_STATUS>
            <RPC_CLIENTID>908eafd4-9aec-4288-96f1-e8011d181561
            </RPC_CLIENTID>
            <RPC_CALLID>0</RPC_CALLID>
        </DATA>
    </RECORD>
    <RECORD>
        <OPCODE>OP_ALLOCATE_BLOCK_ID</OPCODE>
        <DATA>
            <TXID>131</TXID>
            <BLOCK_ID>1073741839</BLOCK_ID>
        </DATA>
    </RECORD>
    <RECORD>
        <OPCODE>OP_SET_GENSTAMP_V2</OPCODE>
        <DATA>
            <TXID>132</TXID>
            <GENSTAMPV2>1016</GENSTAMPV2>
        </DATA>
    </RECORD>
    <RECORD>
        <OPCODE>OP_ADD_BLOCK</OPCODE>
        <DATA>
            <TXID>133</TXID>
            <PATH>/hello7.txt</PATH>
            <BLOCK>
                <BLOCK_ID>1073741839</BLOCK_ID>
                <NUM_BYTES>0</NUM_BYTES>
                <GENSTAMP>1016</GENSTAMP>
            </BLOCK>
            <RPC_CLIENTID></RPC_CLIENTID>
            <RPC_CALLID>-2</RPC_CALLID>
        </DATA>
    </RECORD>
    <RECORD>
        <OPCODE>OP_CLOSE</OPCODE>
        <DATA>
            <TXID>134</TXID>
            <LENGTH>0</LENGTH>
            <INODEID>0</INODEID>
            <PATH>/hello7.txt</PATH>
            <REPLICATION>2</REPLICATION>
            <MTIME>1512943608761</MTIME>
            <ATIME>1512943607866</ATIME>
            <BLOCKSIZE>134217728</BLOCKSIZE>
            <CLIENT_NAME></CLIENT_NAME>
            <CLIENT_MACHINE></CLIENT_MACHINE>
            <OVERWRITE>false</OVERWRITE>
            <BLOCK>
                <BLOCK_ID>1073741839</BLOCK_ID>
                <NUM_BYTES>25</NUM_BYTES>
                <GENSTAMP>1016</GENSTAMP>
            </BLOCK>
            <PERMISSION_STATUS>
                <USERNAME>kokohoho</USERNAME>
                <GROUPNAME>supergroup</GROUPNAME>
                <MODE>420</MODE>
            </PERMISSION_STATUS>
        </DATA>
    </RECORD>
</EDITS>
````

思考：NameNode 如何确定下次开机启动的时候合并哪些 Edits？

### 5.3 CheckPoint时间设置

#### 1）通常情况下，SecondaryNameNode 每隔一小时执行一次。

[hdfs-default.xml]

````xml
<property>
<name>dfs.namenode.checkpoint.period</name>
<value>3600s</value>
</property>
````

#### 2）一分钟检查一次操作次数，当操作次数达到 1 百万时，SecondaryNameNode 执行一次。

````xml
<property>
<name>dfs.namenode.checkpoint.txns</name>
 <value>1000000</value>
<description>操作动作次数</description>
</property>
<property>
 <name>dfs.namenode.checkpoint.check.period</name>
 <value>60s</value>
<description> 1 分钟检查一次操作次数</description>
</property>
````

## 第6章 DataNode

### 6.1 DataNode工作机制

![avatar](/img/bigdata/hadoop/hdfs/datanode.png)

（1）一个数据块在 DataNode 上以文件形式存储在磁盘上，包括两个文件，一个是数据
本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

（2）DataNode 启动后向 NameNode 注册，通过后，周期性（6 小时）的向 NameNode 上
报所有的块信息。

DN 向 NN 汇报当前解读信息的时间间隔，默认 6 小时；

````xml
<property>
    <name>dfs.blockreport.intervalMsec</name>
    <value>21600000</value>
    <description>Determines block reporting interval in milliseconds.</description>
</property>
````

DN 扫描自己节点块信息列表的时间，默认 6 小时

````xml
<property>
    <name>dfs.datanode.directoryscan.interval</name>
    <value>21600s</value>
    <description>Interval in seconds for Datanode to scan data directories and reconcile the difference between blocks in memory and on the disk. Support multiple time unit suffix(case insensitive), as described in dfs.heartbeat.interval.
    </description>
</property>
````
（3）心跳是每 3 秒一次，心跳返回结果带有 NameNode 给该 DataNode 的命令如复制块
数据到另一台机器，或删除某个数据块。如果超过 10 分钟没有收到某个 DataNode 的心跳，
则认为该节点不可用。

（4）集群运行中可以安全加入和退出一些机器。

### 6.2 数据完整性

思考：如果电脑磁盘里面存储的数据是控制高铁信号灯的红灯信号（1）和绿灯信号（0），
但是存储该数据的磁盘坏了，一直显示是绿灯，是否很危险？同理 DataNode 节点上的数据
损坏了，却没有发现，是否也很危险，那么如何解决呢？

如下是 DataNode 节点保证数据完整性的方法。

（1）当 DataNode 读取 Block 的时候，它会计算 CheckSum。

（2）如果计算后的 CheckSum，与 Block 创建时值不一样，说明 Block 已经损坏。

（3）Client 读取其他 DataNode 上的 Block。

（4）常见的校验算法 crc（32），md5（128），sha1（160）

（5）DataNode 在其文件创建后周期验证 CheckSum。

![avatar](/img/bigdata/hadoop/hdfs/checksum.png)


### 6.3 掉线时限参数设置

1、DataNode进程死亡或
者网络故障造成DataNode
无法与NameNode通信

2、NameNode不会立即把该节点判定
为死亡，要经过一段时间，这段时间
暂称作超时时长。

3、HDFS默认的超时时长为10分钟+30秒。

4、如果定义超时时间为TimeOut，则超时时长的计算公式为：

> TimeOut = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval。

而默认的dfs.namenode.heartbeat.recheck-interval 大小为5分钟，dfs.heartbeat.interval默认为3秒。

需要注意的是 hdfs-site.xml 配置文件中的 heartbeat.recheck.interval 的单位为毫秒，
dfs.heartbeat.interval 的单位为秒。

````xml
<property>
    <name>dfs.namenode.heartbeat.recheck-interval</name>
    <value>300000</value>
</property>
<property>
    <name>dfs.heartbeat.interval</name>
    <value>3</value>
</property>
````



















