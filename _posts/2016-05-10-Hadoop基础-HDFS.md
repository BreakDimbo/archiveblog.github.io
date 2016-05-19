---
layout: post
title: "蹲马步"
date: 2016-05-19
excerpt: "关于Hadoop的一些基础知识。"
tags: [Blocks, Namenodes, Datanedes, CMD, Java interface, Data Flow, Distcp－并行拷贝]
bigData: true
comments: true
---

# Hadoop——分布式文件系统



## 喔！小百科
* 什么是分布式文件系统？

> > Filesystems that manage the storage **across a network of machines** are called distributed filesystems

* 	什么是HDFS？

> > HDFS is a filesystem designed for storing **very large** files with **streaming data access patterns** (write-once, read-many-times), running on clusters of **commodity hardware**.

## 些微进入HDFS

### 区块

* 区块：HDFS也有区块的概念，文件也是按*独立*区块存储的。但是非常大，默认是128M。
* 那么，为何这个区块如此之大？为了最小化寻址时间。

> > If the block is large enough, the time it takes to transfer the data from the disk can be significantly longer than the time to seek to the start of the block.

### Namenodes & Datanodes

HDFS 集群以Master-Worker的工作模式运行。分别对应了Namenodes和Datanodes。  

* Namenodes（大脑）: 管理文件系统的*名域*（namespace）。*名域*里面存储了这个文件系统的文件结构以及全部文件和目录的元数据。因此namenode可以定位一个给定文件的所有区块所在的datanodoes。*名域*信息以两种形式永久的保存在本地硬盘上：
  1. namespace image
  2. edit log

  **但是，注意！**
  
> > however, it does not store block locations persistently, because this information is reconstructed from datanodes when the system starts.

* Datanodes（身体）: 脏活累活全它干！它们负责存储和取回区块内的信息。并且它们定期向namenodes汇报它们存储的区块信息（以list of blocks的形式）。

### HDFS 「联邦」

Hadoop 2.x支持添加多个namenodes，分别负责文件系统的一部分（*各部分相互独立*），从而增加Datanodes的数量（数据存储量），并防止数据全部丢失。

## 一些常用Shell命令

首先，注意以下两个命令的区别：

* hadoop fs - 可用于本地和HDFS上的文件
* hdfs dfs - 只可用于操作hdfs上的文件

以下是一些命令：  
从本地向HDFS拷贝文件

	% hadoop fs -copyFromLocal input/docs/quangle.txt  
	\ hdfs://localhost/user/tom/quangle.txt

创建目录，列出HDFS根目录下文件：

	% hadoop fs -mkdir books	% hadoop fs -ls .
	
## Hadoop 文件系统
<figure>
	<a href="http://breakdimbo.github.io/images/Hadoop-Filesystem.png"><img src="http://breakdimbo.github.io/images/Hadoop-Filesystem.png"></a>
</figure>
