---
layout:     post
title:      "「原创」Hadoop 一篇精通MapReduce"
subtitle:   "HDFS概述、Shell/API操作、读/写流程、NN/2NN工作机制、DN工作机制"
date:       2021-11-29 12:00:00
author:     "NI"
header-img: "img/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
  - Hadoop
  - MapReduce
  - 原创
---

## 第1章 MapReduce概述

### 1.1 MapReduce定义

### 1.2 MapReduce优缺点

#### 1.2.1 优点

##### 1）MapReduce易于编程

##### 2）良好的扩展性

##### 3）高容错性

##### 4）适合PB级别以上海量数据的离线处理

#### 1.2.2 缺点

##### 1）不擅长实时计算

##### 2）不擅长流式计算

##### 3）不擅长DAG（有向无环图）计算

### 1.3 MapReduce核心思想

### 1.4 MapReduce进程

### 1.5 官方WordCount源码

### 1.6 常用数据序列化类型

### 1.7 MapReduce编程规范

### 1.8 WordCount案例实操

## 第2章 Hadoop序列化

### 2.1 序列化概述

### 2.2 自定义bean对象实现序列化接口（Writable）

### 2.3 序列化案例实操

## 第3章 MapReduce框架原理

### 3.1 InputFormat数据输入

#### 3.1.1 切片与MapTask并行度决定机制

#### 3.1.2 提交流程源码和切片源码详解

#### 3.1.3 FileInputFormat切片机制

#### 3.1.4 TextInputFormat

#### 3.1.5 CombineTextInputFormat切片机制

#### 3.1.6 CombineTextInputFormat案例实操

### 3.2 MapReduce工作流程

### 3.3 Shuffle机制

#### 3.3.1 Shuffle机制

#### 3.3.2 Partition分区

#### 3.3.3 Partition分区案例实操

#### 3.3.4 WritableComparable排序

#### 3.3.5 WritableComparable排序案例实操（全排序）

#### 3.3.6 WritableComparable排序案例实操（区内排序）

#### 3.3.7 Combiner合并

#### 3.3.8 Combiner合并案例实操

### 3.4 OutputFormat数据输出

#### 3.4.1 OutputFormat接口实现类

#### 3.4.2 自定义OutputFormat案例实操

### 3.5 MapReduce内核源码解析

#### 3.5.1 MapTask工作机制

#### 3.5.2 ReduceTask工作机制

#### 3.5.3 ReduceTask并行度决定机制

#### 3.5.4 MapTask & ReduceTask源码解析

### 3.6 Join应用

#### 3.6.1 Reduce Join

#### 3.6.2 Reduce Join案例实操

#### 3.6.3 Map Join

#### 3.6.4 Map Join案例实操

### 3.7 数据清洗（ETL）

### 3.8 MapReduce开发总结

## 第4章 Hadoop数据压缩

### 4.1 概述

### 4.2 MR支持的压缩编码

### 4.3 压缩方式选择

#### 4.3.1 Gzip压缩

#### 4.3.2 Bzip2压缩

#### 4.3.3 Lzo压缩

#### 4.3.4 Snappy压缩

#### 4.3.5 压缩位置选择

### 4.4 压缩参数配置

### 4.5 压缩实操案例

#### 4.5.1 Map输出端采用压缩

#### 4.5.1 Reduce输出端采用压缩

## 第4章 常见错误及解决方案














