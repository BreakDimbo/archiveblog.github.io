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



## 1. 喔！小百科

什么是分布式文件系统？

> Filesystems that manage the storage **across a network of machines** are called distributed filesystems

什么是 HDFS？

> HDFS is a filesystem designed for storing **very large** files with **streaming data access patterns** (write-once, read-many-times), running on clusters of **commodity hardware**.

## 2. 些微进入 HDFS

### 区块

HDFS 也有区块的概念，文件也是按*独立*区块存储的。但是非常大，默认是128M。那么，为何这个区块如此之大?  
*为了最小化寻址时间。*

> If the block is large enough, the time it takes to transfer the data from the disk can be significantly longer than the time to seek to the start of the block.

### Namenodes & Datanodes

HDFS 集群以 Master-Worker 的工作模式运行。分别对应了 Namenodes 和 Datanodes。  

* Namenodes（大脑）: 管理文件系统的*名域*（namespace）。*名域*里面存储了这个文件系统的文件结构以及全部文件和目录的元数据。因此 namenode 可以定位一个给定文件的所有区块所在的 datanodoes。*名域*信息以两种形式永久的保存在本地硬盘上：
  1. namespace image
  2. edit log

**但是，注意！**
  
> however, it does not store block locations persistently, because this information is reconstructed from datanodes when the system starts.

* Datanodes（身体）: 脏活累活全它干！它们负责存储和取回区块内的信息。并且它们定期向namenodes汇报它们存储的区块信息（以 list of blocks 的形式）。

### HDFS 「联邦」

Hadoop 2.x 支持添加多个 namenodes，分别负责文件系统的一部分（*各部分相互独立*），从而增加 Datanodes 的数量（数据存储量），并防止数据全部丢失。

## 3. 一些常用Shell命令

首先，注意以下两个命令的区别：

* hadoop fs - 可用于本地和 HDFS 上的文件
* hdfs dfs - 只可用于操作 HDFS 上的文件

以下是一些命令：  
从本地向 HDFS 拷贝文件

	% hadoop fs -copyFromLocal input/docs/quangle.txt  
	\ hdfs://localhost/user/tom/quangle.txt

创建目录，列出 HDFS 根目录下文件：

	% hadoop fs -mkdir books	% hadoop fs -ls .
	
## 4. Hadoop 文件系统
<figure>
	<a href="http://breakdimbo.github.io/images/Hadoop-Filesystem.png"><img src="http://breakdimbo.github.io/images/Hadoop-Filesystem.png"></a>
</figure>

## 5.Java 接口

### 使用 Hadoop URL 读取数据

使用 java.net.URL 对象来打开一个 stream 并从中读取数据。代码实现如下：

	public class URLCat {		static {			URL.setURLStreamHandlerFactory(newFsUrlStreamHandlerFactory());		}		public static void main(String[] args) 
			throws Exception { InputStream in = null;			try {				in = new URL(args[0]).openStream();				IOUtils.copyBytes(in, System.out, 4096, false); } 
			finally {      			IOUtils.closeStream(in);    		}		} 
	}

注意：需要使用 **URL.setURLStreamFactory()** 方法来获得对 URL 进行设置。该方法对于每个 JVM 只能调用一次。所以一般是静态的，而且*如果你的程序的其他部分调用了这个方法，那么你就不能使用它从 Hadoop 读取数据*。

### 使用 FileSystem API 读写数据

获取 FileSystem 实例：

使用 FileSystem.get() 方法获取 FileSystem instance。方法参数如下：
	
	public static FileSystem get(Configuration conf) throws IOException	public static FileSystem get(URI uri, Configuration conf) throws IOException 
	public static FileSystem get(URI uri, Configuration conf, String user)
		throws IOException
			
其中 URI 类似 Hadoop 的 Path。

#### 读取数据

获取实例后，在实例上调用 .path() 方法，返回 FSDataInputStream 类型。  
.path() 方法参数如下：

	public FSDataInputStream open(Path f) throws IOException	public abstract FSDataInputStream open(Path f, int bufferSize) throws IOException
	
关于 FSDateInputStream 其中包含了两类方法

1. 调整和和获取*流*的位置：
		
		void seek(long pos) throws IOException; 
		long getPos() throws IOException;
		
2. 在一个给定*长度内* (at a given offset) 读取文件的一部分：

		public int read(long position, byte[] buffer, int offset, int length)			throws IOException;		public void readFully(long position, byte[] buffer, int offset, int length)			throws IOException;
		public void readFully(long position, byte[] buffer) throws IOException; 
			
读取数据的代码示例：

	public class FileSystemDoubleCat {		public static void main(String[] args) throws Exception { 
			String uri = args[0];			Configuration conf = new Configuration();			FileSystem fs = FileSystem.get(URI.create(uri), conf);
			FSDataInputStream in = null;			try {				in = fs.open(new Path(uri)); 
				IOUtils.copyBytes(in, System.out, 4096, false); 
				in.seek(0); // go back to the start of the file
				IOUtils.copyBytes(in, System.out, 4096, false);			} 
			finally { 
				IOUtils.closeStream(in);			} 
		}	}
	
#### 写出数据
