---
layout: post
title: "蹲马步: Hadoop——分布式文件系统"
date: 2016-05-19
excerpt: "关于Hadoop的一些基础知识。"
tags: [Blocks, Namenodes, Datanedes, CMD, Java interface, Data Flow, Distcp－并行拷贝]
bigData: true
comments: true
---




# 1. 喔！小百科

什么是分布式文件系统？

> Filesystems that manage the storage **across a network of machines** are called distributed filesystems

什么是 HDFS？

> HDFS is a filesystem designed for storing **very large** files with **streaming data access patterns** (write-once, read-many-times), running on clusters of **commodity hardware**.

# 2. 些微进入 HDFS

## 2.1 区块

HDFS 也有区块的概念，文件也是按*独立*区块存储的。但是非常大，默认是128M。那么，为何这个区块如此之大?  
*为了最小化寻址时间。*

> If the block is large enough, the time it takes to transfer the data from the disk can be significantly longer than the time to seek to the start of the block.

## 2.2 Namenodes 和 Datanodes

HDFS 集群以 Master-Worker 的工作模式运行。分别对应了 Namenodes 和 Datanodes。  

* Namenodes（大脑）: 管理文件系统的*名域*（namespace）。*名域*里面存储了这个文件系统的文件结构以及全部文件和目录的元数据。因此 namenode 可以定位一个给定文件的所有区块所在的 datanodoes。*名域*信息以两种形式永久的保存在本地硬盘上：
  1. namespace image
  2. edit log

**但是，注意！**
  
> however, it does not store block locations persistently, because this information is reconstructed from datanodes when the system starts.

* Datanodes（身体）: 脏活累活全它干！它们负责存储和取回区块内的信息。并且它们定期向namenodes汇报它们存储的区块信息（以 list of blocks 的形式）。

## 2.3 HDFS 「联邦」

Hadoop 2.x 支持添加多个 namenodes，分别负责文件系统的一部分（*各部分相互独立*），从而增加 Datanodes 的数量（数据存储量），并防止数据全部丢失。

# 3. 一些常用Shell命令

首先，注意以下两个命令的区别：

* hadoop fs - 可用于本地和 HDFS 上的文件
* hdfs dfs - 只可用于操作 HDFS 上的文件

以下是一些命令：  
从本地向 HDFS 拷贝文件

~~~java
% hadoop fs -copyFromLocal input/docs/quangle.txt  
\ hdfs://localhost/user/tom/quangle.txt
~~~

创建目录，列出 HDFS 根目录下文件：

~~~java
% hadoop fs -mkdir books% hadoop fs -ls .
~~~
	
# 4. Hadoop 文件系统
<figure>
	<a href="http://breakdimbo.github.io/images/Hadoop-Filesystem.png"><img src="http://breakdimbo.github.io/images/Hadoop-Filesystem.png"></a>
</figure>

# 5.Java 接口
> URL；FileSystem; FSDataInputStream; FSDataOutputStream; FileStatus; PathFilter

## 5.1 使用 Hadoop URL 读取数据

使用 java.net.URL 对象来打开一个 stream 并从中读取数据。代码实现如下：

~~~java
public class URLCat {	static {		URL.setURLStreamHandlerFactory(newFsUrlStreamHandlerFactory());	}	public static void main(String[] args) 
		throws Exception { InputStream in = null;		try {			in = new URL(args[0]).openStream();			IOUtils.copyBytes(in, System.out, 4096, false); } 
		finally {     		IOUtils.closeStream(in);   		}	} 
}
~~~

注意：需要使用 **URL.setURLStreamFactory()** 方法来获得对 URL 进行设置。该方法对于每个 JVM 只能调用一次。所以一般是静态的，而且*如果你的程序的其他部分调用了这个方法，那么你就不能使用它从 Hadoop 读取数据*。

## 5.2 使用 FileSystem API 读写数据

获取 FileSystem 实例：

使用 FileSystem.get() 方法获取 FileSystem instance。方法参数如下：

~~~java
public static FileSystem get(Configuration conf) throws IOExceptionpublic static FileSystem get(URI uri, Configuration conf) throws IOException 
public static FileSystem get(URI uri, Configuration conf, String user) throws IOException
~~~
			
其中 URI 类似 Hadoop 的 Path。

### 读取数据

获取 **FileSystem** 实例后，在实例上调用 .path(Path p) 方法，返回 FSDataInputStream 类型。  
.path() 方法参数如下：

~~~java
public FSDataInputStream open(Path f) throws IOExceptionpublic abstract FSDataInputStream open(Path f, int bufferSize) throws IOException
~~~
	
关于 FSDateInputStream 其中包含了两类方法

1. 调整和和获取*流*的位置：

		void seek(long pos) throws IOException; 
		 long getPos() throws IOException;

		
2. 在一个给定*长度内* (at a given offset) 读取文件的一部分：
	
		public int read(long position, byte[] buffer, int offset, int length) throws IOException;		 public void readFully(long position, byte[] buffer, int offset, int length) throws IOException;
		 public void readFully(long position, byte[] buffer) throws IOException; 
	
读取数据的代码示例：

~~~java
public class FileSystemDoubleCat {	public static void main(String[] args) throws Exception { 
		String uri = args[0];		Configuration conf = new Configuration();		FileSystem fs = FileSystem.get(URI.create(uri), conf);
		FSDataInputStream in = null;		try {			in = fs.open(new Path(uri)); 
			IOUtils.copyBytes(in, System.out, 4096, false); 
			in.seek(0); // go back to the start of the file
			IOUtils.copyBytes(in, System.out, 4096, false);		} 
		finally { 
			IOUtils.closeStream(in);		} 
	}}
~~~
	
### 写出数据

同样，获取 **FileSystem** 实例后，在实例上有三个主要相关方法可以调用：

1. 创建新文件并写入，create()方法会自动创建父目录，需要注意：

		public FSDataOutputStream create(Path f) throws IOException
	
2. 在已有文件中进行写入：

		public FSDataOutputStream append(Path f) throws IOException
	
3. 回调函数，当数据被写入 datanodes 时，你的程序会得到通知：

		public void progress()
	
具体实现与写入类似。

## 5.3 建立目录

获取 **FileSystem** 实例后，可以使用如下方法创建目录：

~~~java
public boolean mkdirs(Path f) throws IOException
~~~
这个方法会创建所有的父目录，如果它们不存在。

## *5.4 文件系统的查询*

### 文件/目录状态查询——FileStatus类

可以查询文件/目录的文件长度，副本数量，修改时间，拥有者，权限信息，区块大小。代码示例如下：

~~~java
FileSystem fs = FileSystem.get(new Configuration());
Path file = new Path("/dir/file");
//判断文件是否存在
fs.exists(file);					
FileStatus stat = fs.getFileStatus(file);
stat.getPath();
stat.isDirectory();	
//返回 long		
stat.getLen();		
//返回 Millis			
stat.getModificationTime();	
//返回 short
stat.getReplication();			
//返回 long
stat.getBlockSize();		
//返回	String	
stat.getOwner();					
stat.getGroup();					
stat.getPermission();			
~~~

### 列出目录内容——listStatus 方法

该方法主要有三种使用方法：

1. 直接输入 Path 作为参数，如果 Path 指向文件，则返回该文件的 FileStatus，如果指向目录，则返回目录内所有文件/目录的 FileStatus 对象组成的数组。

		public FileStatus[] listStatus(Path f) throws IOException
		
2. 参数加上 PathFilter 类，可以进行文件目录匹配的限制。

		public FileStatus[] listStatus(Path f, PathFilter filter) 
			throws IOException	
	
3. 参数为 Path[ ]，常用于构建来自文件系统不同部分的文件整合处理。

		public FileStatus[] listStatus(Path[] files) throws IOException		 public FileStatus[] listStatus(Path[] files, PathFilter filter) 
			throws IOException
	
代码示例：

~~~java
public class ListStatus {	public static void main(String[] args) throws Exception { 
		String uri = args[0];		Configuration conf = new Configuration();		FileSystem fs = FileSystem.get(URI.create(uri), conf);
		Path[] paths = new Path[args.length]; 
		for (int i = 0; i < paths.length; i++) {			paths[i] = new Path(args[i]); }		FileStatus[] status = fs.listStatus(paths); 
		//注意 stat2Paths 的使用
		Path[] listedPaths = FileUtil.stat2Paths(status); 
		for (Path p : listedPaths) {    		System.out.println(p);    	}
   	}
}
~~~### 对一系列文件进行相同操作——globStatus() 方法

globStatus() 方法结合通配符进行使用。参数如下：

~~~java
public FileStatus[] globStatus(Path pathPattern) throws IOException 
public FileStatus[] globStatus(Path pathPattern, PathFilter filter) 	throws IOException
~~~

通配符如下表：

通配符 |  含义
:---- |:------------
*     |含有0个或更多任意字符
?     |一个任意字符
[ab]  |符合 a 或 b 任意==一个==*字符*
[^ab] |任意==一个==不是 a 或 b 的*字符*(闭区间)
[a-b] |任意==一个==a-b 之间的字符
[^a-b]|任意==一个==不在 a-b 之间的字符
{a,b} |符合 a 或 b 任意一个的 *expression*
\c    |相符的转意符

一个🌰如下：
假设有如下目录结构存储的日志。
<figure>
	<a href="http://breakdimbo.github.io/images/DirectoryH-globStatus.png"><img src="http://breakdimbo.github.io/images/DirectoryH-globStatus.png"></a>
</figure>

相对应的的通配符如下表：
<figure>
	<a href="http://breakdimbo.github.io/images/Wildcard-globStatus.png"><img src="http://breakdimbo.github.io/images/Wildcard-globStatus.png"></a>
</figure>


### 路径过滤器——PathFilter Interface

当你想要更 Customized 过滤掉一些文件，就需要用到 PathFilter 接口。 其中定义了一个 accept(Path path) 方法，可以进行实现。这个接口类似于 java.io.FileFilter。  

以下是示例：

~~~java
public class RegexExcludePathFilter implements PathFilter {	private final String regex;	public RegexExcludePathFilter(String regex) { 
		this.regex = regex;	}	public boolean accept(Path path) { 
		return !path.toString().matches(regex);	} 
}
~~~
	
**注意：**路径过滤器仅能对在**文件名**上进行操作。

### 删除文件或者目录

~~~java
public boolean delete(Path f, boolean recursive) throws IOException
~~~

# 6. 数据流
> 输入流；输出流；Coherence Model

## 6.1 文件的读取
