---
layout: post
data: 2016-05-21
title: "Hadoop 开发环境的搭建"
excerpt: "单机/伪分布式以及集群式安装配置攻略"
bigData: true
comments: true
tags: [Hadoop, single node, cluster, Pseudo-Distributed]
---

# 1. 准备工作

	搭建环境： 
	系统环境为：Ubuntu 16.04 64位 
	基于 Hadoop 2.7.1
	
## 1.1 创建用户

新增名为 hadoop 的测试用户，并使用 /bin/bash 作为 shell：

	$ sudo useradd -m hadoop -s /bin/bash
	
为 hadoop 用户设置密码：

	$ sudo passwd hadoop
	
为 hadoop 用户增加管理权限，方便后续测试：

	$ sudo adduser hadoop sudo

## 1.2 更新 apt

为保证后续安装其他软件没有问题，需要更新一下 apt：

	$ sudo apt-get update
	
安装 vim：

	$ sudo apt-get install vim

## 1.3 安装 SSH，配置 SSH 无密码登陆。

Ubuntu 默认已经安装了 SSH client。还需要安装 SSH server：

	$ sudo apt-get install openssh-server

测试一下：

	$ ssh localhost

配置 SSH 无密码登陆：利用 ssh-keygen 生成密钥，并将密钥加入到授权中：

	$ cd ~/.ssh
	$ ssh-keygen -t rsa
	$ cat ./id_rsa.pub >> ./authorized_keys
	$ chmod 0600 ~/.ssh/authorized_keys

	
关于 SSH 的更多解释，可以参考 [**archlinux-SSH keys**](https://wiki.archlinux.org/index.php/SSH_keys_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

## 1.4 安装 Java 环境

Java 环境可以选择 Oracle 的 JDK，或者 OpenJDK。本攻略以 OpenJDK 为例。  
直接运行 apt-get 安装 OpenJDK：

	$ sudo apt-get install openjdk-8-jre openjdk-8-jdk
	
接下来配置 JAVA_HOME 环境变量。  
首先找到 OpenJDK 安装路径：

	$ dpkg -L openjdk-8-jdk | grep '/bin/javac'
	
然后在 ~/.bashrc 中设置环境变量，打开文件：

	$ vim ~/.bashrc
	
输入 OpenJDK 的安装路径到JAVA_HOME（注意，等号两边没有空格）：

	export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
	
保存退出，使配置生效：
	
	$ source ~/.bashrc

最后，检验一下配置是否成功：

	$ echo $JAVA_HOME
	$ java -version
	$ $JAVA_HOME/bin/java -version
	
> java -version 与 $JAVA_HOME/bin/java -version 输出到版本信息应该一致。

*关于设置Linux环境变量的方法和区别*

> 首先是设置全局环境变量，对所有用户都会生效：

> * etc/profile: 此文件为系统的每个用户设置环境信息。当用户登录时，该文件被执行一次，并从 /etc/profile.d 目录的配置文件中搜集shell 的设置。一般用于设置所有用户使用的全局变量。

> * /etc/bashrc: 当 bash shell 被打开时，该文件被读取。也就是说，每次新打开一个终端 shell，该文件就会被读取。

> 接着是与上述两个文件对应，但只对单个用户生效：

> * ~/.bash_profile 或 ~/.profile: 只对单个用户生效，当用户登录时该文件仅执行一次。用户可使用该文件添加自己使用的 shell 变量信息。另外在不同的LINUX操作系统下，这个文件可能是不同的，可能是 ~/.bash_profile， ~/.bash_login 或 ~/.profile 其中的一种或几种，如果存在几种的话，那么执行的顺序便是：~/.bash_profile、 ~/.bash_login、 ~/.profile。比如 Ubuntu 系统一般是 ~/.profile 文件。

> * ~/.bashrc: 只对单个用户生效，当登录以及每次打开新的 shell 时，该文件被读取。

> 此外，修改 /etc/environment 这个文件也能实现环境变量的设置。/etc/environment 设置的也是全局变量，从文件本身的作用上来说， /etc/environment 设置的是整个系统的环境，而/etc/profile是设置所有用户的环境。有几点需注意：

> * 系统先读取 etc/profile 再读取 /etc/environment（还是反过来？）
> * /etc/environment 中不能包含命令，即直接通过 VAR="..." 的方式设置，不使用 export 。
> * 使用 source /etc/environment 可以使变量设置在当前窗口立即生效，需注销/重启之后，才能对每个新终端窗口都生效。


## 1.5 安装 Hadoop

首先通过官网下载稳定版本 [**hadoop-2.7.2.tar.gz**](http://mirror.bit.edu.cn/apache/hadoop/common/)。建议选择编译好的，如果需要使用源文件重新编译，比较麻烦，查阅 src 包内的安装文档。

将下载好的文件解压至 /usr/local/ 目录下，并修改文件名为 hadoop，修改文件权限：

	$ sudo tar -zxf ~/Downloads/hadoop-2.7.2.tar.gz -C /usr/local
	$ cd /usr/local/
	$ sudo mv ./hadoop-2.7.2/ ./hadoop 
	$ sudo chown -R hadoop ./hadoop
	
设置 Hadoop 环境变量：

	$ cd /usr/local/hadoop/etc/hadoop
	$ sudo vim hadoop-env.sh

定义参数如下：

	 # set to the root of your Java installation
  	export JAVA_HOME=/usr/java/latest
  	
运行检测：
	
	$ cd /usr/local/hadoop
	$ ./bin/hadoop -version
	
至此，准备工作全部完成。进入配置阶段。

---

# 2. Hadoop 单机配置

上述工作做完后，默认已经可以进行单机操作。下面使用 Hadoop 自带的 grep 程序测试一下。将 input 文件夹中的所有文件作为输入，筛选其中符合正则表达式 'dfs[a-z.]+' 的单词并统计出现次数，最终将结果输入到 /output 文件夹中。


	$ cd /usr/local/hadoop
	$ mkdir ./input
	$ cp ./etc/hadoop/*.xml ./input
	$ ./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar
		grep ./input ./output 'dfs[a-z.]+'
	$ cat output/*


**注意：***Hadoop 默认不会覆盖结果文件，因此再次运行上面实例会提示出错，需要先将 ./output 删除。*

---

# 3. Hadoop 伪分布式配置

Hadoop 可以在单节点上以伪分布式的方式运行，Hadoop 进程以分离的 Java 进程来运行，节点既作为 NameNode 也作为 DataNode，同时，读取的是 HDFS 中的文件。

Hadoop 的配置文件位于 /usr/local/hadoop/etc/hadoop/ 中，伪分布式需要修改2个配置文件 **core-site.xml** 和 **hdfs-site.xml** 。Hadoop的配置文件是 xml 格式，每个配置以声明 property 的 name 和 value 的方式来实现。

## 3.1 配置 core-site.xml

修改配置文件 core-site.xml:

~~~xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://localhost:9000</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>file:/usr/local/hadoop/tmp</value>
	</property>	
</configuration>
~~~

## 3.2 配置 hdfs-site.xlm

修改配置文件 hdfs-site.xml：

~~~xml
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:/usr/local/hadoop/tmp/dfs/name</value>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:/usr/local/hadoop/tmp/dfs/data</value>
	</property>
</configuration>
~~~

*Hadoop配置文件说明：*  

> Hadoop 的运行方式是由配置文件决定的（运行 Hadoop 时会读取配置文件），因此如果需要从伪分布式模式切换回非分布式模式，需要删除 core-site.xml 中的配置项。

> 此外，伪分布式虽然只需要配置 fs.defaultFS 和 dfs.replication 就可以运行（官方教程如此），不过若没有配置 hadoop.tmp.dir 参数，则默认使用的临时目录为 /tmp/hadoo-hadoop，而这个目录在重启时有可能被系统清理掉，导致必须重新执行 format 才行。所以我们进行了设置，同时也指定 dfs.namenode.name.dir 和 dfs.datanode.data.dir，否则在接下来的步骤中可能会出错。

## 3.3 扫荡边角

格式化文件系统（HDFS）,开启守护进程：

	$ ./bin/hdfs namenode -format
	$ ./bin/start-dfs.sh

运行 jps 检测服务是否启动。需要出现三个进程：“NameNode”、”DataNode” 和 “SecondaryNameNode”。

*Hadoop无法正常启动的解决方法 ：* 

> 一般可以查看启动日志来排查原因，注意几点：

> * 启动时会提示形如 “DBLab-XMU: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hadoop-namenode-DBLab-XMU.out”，其中 DBLab-XMU 对应你的机器名，但其实启动日志信息是记录在 /usr/local/hadoop/logs/hadoop-hadoop-namenode-DBLab-XMU.log 中，所以应该查看这个后缀为 .log 的文件。

> * 每一次的启动日志都是追加在日志文件之后，所以得拉到最后面看，对比下记录的时间就知道了。
一般出错的提示在最后面，通常是写着 Fatal、Error、Warning 或者 Java Exception 的地方。
可以在网上搜索一下出错信息，看能否找到一些相关的解决方法。

> 此外，若是 DataNode 没有启动，可尝试如下的方法（注意这会删除 HDFS 中原有的所有数据，如果原有的数据很重要请不要这样做）：

	$ ./sbin/stop-dfs.sh   # 关闭
	$ rm -r ./tmp     # 删除 tmp 文件，注意这会删除 HDFS 中原有的所有数据
	$ ./bin/hdfs namenode -format   # 重新格式化 NameNode
	$ ./sbin/start-dfs.sh  # 重启

成功启动后，可以访问 Web 界面 [**http://localhost:50070**](http://localhost:50070) 查看 NameNode 和 Datanode 信息，还可以在线查看 HDFS 中的文件。

## 3.4 运行实例
  
上面的单机模式，grep 例子读取的是本地数据，伪分布式读取的则是 HDFS 上的数据。要使用 HDFS，首先需要在 HDFS 中创建用户目录，后续文件操作 hdfs 会默认在 /user/hadoop 下进行：

	$ ./bin/hdfs dfs -mkdir -p /user/hadoop

接着将 ./etc/hadoop 中的 xml 文件作为输入文件复制到分布式文件系统中，即将 /usr/local/hadoop/etc/hadoop 复制到分布式文件系统中的 /user/hadoop/input 中。我们使用的是 hadoop 用户，并且已创建相应的用户目录 /user/hadoop ，因此在命令中就可以使用相对路径如 input，其对应的绝对路径就是 /user/hadoop/input:

	$ ./bin/hdfs dfs -mkdir input
	$ ./bin/hdfs dfs -put ./etc/hadoop/*.xml input

复制完成后，可以通过如下命令查看文件列表：

	$ ./bin/hdfs dfs -ls input
	
伪分布式运行 MapReduce 作业的方式跟单机模式相同，区别在于伪分布式读取的是HDFS中的文件（可以将单机步骤中创建的本地 input 文件夹，输出结果 output 文件夹都删掉来验证这一点）。

	$ ./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar
		grep input output 'dfs[a-z.]+'

查看运行结果的命令（查看的是位于 HDFS 中的输出结果）：

	$ ./bin/hdfs dfs -cat output/*

我们也可以将运行结果取回到本地：

	$ rm -r ./output    # 先删除本地的 output 文件夹（如果存在）
	$ ./bin/hdfs dfs -get output ./output     # 将 HDFS 上的 output 文件夹拷贝到本机
	$ cat ./output/*

Hadoop 运行程序时，输出目录不能存在，否则会提示错误 “org.apache.hadoop.mapred.FileAlreadyExistsException: Output directory hdfs://localhost:9000/user/hadoop/output already exists” ，因此若要再次执行，需要执行如下命令删除 output 文件夹:

	$ ./bin/hdfs dfs -rm -r output    # 删除 output 文件夹

关闭 hadoop

	$ ./sbin/stop-fds.sh
	
## 3.5 启动YARN

新版的 Hadoop 使用了新的 MapReduce 框架（MapReduce V2，也称为 YARN，Yet Another Resource Negotiator）。

YARN 是从 MapReduce 中分离出来的，负责资源管理与任务调度。YARN 运行于 MapReduce 之上，提供了高可用性、高扩展性。

通过 ./sbin/start-dfs.sh 启动 Hadoop，仅仅是启动了 MapReduce 环境，我们可以启动 YARN ，让 YARN 来负责资源管理与任务调度。

首先修改配置文件 mapred-site.xml，需要先进行重命名：

	$ mv ./etc/hadoop/mapred-site.xml.template ./etc/hadoop/mapred-site.xml

然后再进行编辑

~~~xml
<configuration>
        <property>
             <name>mapreduce.framework.name</name>
             <value>yarn</value>
        </property>
</configuration>
~~~

接着修改配置文件 yarn-site.xml：

~~~xml
<configuration>
        <property>
             <name>yarn.nodemanager.aux-services</name>
             <value>mapreduce_shuffle</value>
            </property>
</configuration>
~~~

然后就可以启动 YARN 了（需要先执行过 ./sbin/start-dfs.sh）：

	$ ./sbin/start-yarn.sh      # 启动YARN
	$ ./sbin/mr-jobhistory-daemon.sh start historyserver  
	  # 开启历史服务器，才能在Web中查看任务运行情况

开启后通过 jps 查看，可以看到多了 NodeManager 和 ResourceManager 两个后台进程。  

关闭 YARN 的脚本如下：

	$ ./sbin/stop-yarn.sh
	$ ./sbin/mr-jobhistory-daemon.sh stop historyserver


启动 YARN 之后，运行实例的方法还是一样的，仅仅是资源管理方式、任务调度不同。观察日志信息可以发现，不启用 YARN 时，是 “mapred.LocalJobRunner” 在跑任务，启用 YARN 之后，是 “mapred.YARNRunner” 在跑任务。启动 YARN 有个好处是可以通过 Web 界面查看任务的运行情况：[**http://localhost:8088/cluster**](http://localhost:8088/cluster)。  


**注意：***不启动 YARN 需重命名 mapred-site.xml：  
如果不想启动 YARN，务必把配置文件 mapred-site.xml 重命名，改成 mapred-site.xml.template，需要用时改回来就行。否则在该配置文件存在，而未开启 YARN 的情况下，运行程序会提示 “Retrying connect to server: 0.0.0.0/0.0.0.0:8032” 的错误，这也是为何该配置文件初始文件名为 mapred-site.xml.template。*

*配置PATH环境变量：*  

> 在这里额外讲一下 PATH 这个环境变量（可执行 echo $PATH 查看，当中包含了多个目录）。例如我们在主文件夹 ~ 中执行 ls 这个命令时，实际执行的是 /bin/ls 这个程序，而不是 ~/ls 这个程序。系统是根据 PATH 这个环境变量中包含的目录位置，逐一进行查找，直至在这些目录位置下找到匹配的程序（若没有匹配的则提示该命令不存在）。

> 上面的攻略中，我们都是先进入到 /usr/local/hadoop 目录中，再执行 sbin/hadoop，实际上等同于运行 /usr/local/hadoop/sbin/hadoop。我们可以将 Hadoop 命令的相关目录加入到 PATH 环境变量中，这样就可以直接通过 start-dfs.sh 开启 Hadoop，也可以直接通过 hdfs 访问 HDFS 的内容，方便平时的操作。

> 同样我们选择在 ~/.bashrc 中进行设置（vim ~/.bashrc，与 JAVA_HOME 的设置相似），在文件最前面加入如下单独一行:

>
	export PATH=$PATH:/usr/local/hadoop/sbin:/usr/local/hadoop/bin

> 添加后执行 source ~/.bashrc 使设置生效，生效后，在任意目录中，都可以直接使用 hdfs 等命令了。

---

# 4. Hadoop集群安装配置

> 环境：使用两个节点作为集群环境: 一个作为 Master 节点，局域网 IP 为 192.168.1.121；另一个作为 Slave 节点，局域网 IP 为 192.168.1.122。

## 4.1 配置流程

Hadoop 集群的安装配置大致为如下流程:

1. 选定一台机器作为 Master
2. 在 Master 节点上配置 hadoop 用户、安装 SSH server、安装 Java 环境
3. 在 Master 节点上安装 Hadoop，并完成配置
4. 在其他 Slave 节点上配置 hadoop 用户、安装 SSH server、安装 Java 环境
5. 将 Master 节点上的 /usr/local/hadoop 目录复制到其他 Slave 节点上
6. 在 Master 节点上开启 Hadoop

## 4.2 网络配置

首先，配置 Master 节点的网络参数。  
为方便管理，修改主机名为 Master：

	$ sudo vim /etc/hostname

然后执行如下命令修改自己所用节点的IP映射：

	$ sudo vim /etc/hosts
	
	192.168.1.121   Master
	192.168.1.122   Slave1
	
对 Slave 进行类似的配置，重启机器。

配置好后需要在各个节点上执行如下命令，测试是否相互 ping 得通，如果 ping 不通，后面就无法顺利配置成功：

	$ ping Master -c 3   # 只ping 3次，否则要按 Ctrl+c 中断
	$ ping Slave1 -c 3
	
## 4.3 SSH无密码登陆节点

这个操作是要让 Master 节点可以无密码 SSH 登陆到各个 Slave 节点上。

首先生成 Master 节点的公匙，在 Master 节点的终端中执行（因为改过主机名，所以还需要删掉原有的再重新生成一次）：

	$ cd ~/.ssh               # 如果没有该目录，先执行一次ssh localhost
	$ rm ./id_rsa*            # 删除之前生成的公匙（如果有）
	$ ssh-keygen -t rsa       # 一直按回车就可以

让 Master 节点需能无密码 SSH 本机，在 Master 节点上执行：

	$ cat ./id_rsa.pub >> ./authorized_keys
	$ chmod 0600 ~/.ssh/authorized_keys

完成后可执行 ssh Master 验证一下（可能需要输入 yes，成功后执行 exit 返回原来的终端）。接着在 Master 节点将上公匙传输到 Slave1 节点：

	$ scp ~/.ssh/id_rsa.pub hadoop@Slave1:/home/hadoop/

scp 是 secure copy 的简写，用于在 Linux 下进行远程拷贝文件，类似于 cp 命令，不过 cp 只能在本机中拷贝。执行 scp 时会要求输入 Slave1 上 hadoop 用户的密码(hadoop)，输入完成后会提示传输完毕。

接着在 Slave1 节点上，将 ssh 公匙加入授权：

	$ mkdir ~/.ssh       # 如果不存在该文件夹需先创建，若已存在则忽略
	$ cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
	$ rm ~/id_rsa.pub    # 用完就可以删掉了
	$ chmod 0600 ~/.ssh/authorized_keys

如果有其他 Slave 节点，也要执行将 Master 公匙传输到 Slave 节点、在 Slave 节点上加入授权这两步。

这样，在 Master 节点上就可以无密码 SSH 到各个 Slave 节点了，可在 Master 节点上执行如下命令进行检验，如下所示：

	ssh Slave1
	
## 4.4 配置PATH变量

需要在 Master 节点上进行配置。首先执行 

	$ vim ~/.bashrc
加入一行：

	export PATH=$PATH:/usr/local/hadoop/bin:/usr/local/hadoop/sbin
	
保存后执行 source ~/.bashrc 使配置生效。

## 4.5 配置集群/分布式环境

集群/分布式模式需要修改 /usr/local/hadoop/etc/hadoop 中的5个配置文件，更多设置项可查看官方说明，这里仅设置了正常启动所必须的设置项： slaves、core-site.xml、hdfs-site.xml、mapred-site.xml、yarn-site.xml 。

### 4.5.1 文件 slaves

slaves 将作为 DataNode 的主机名写入该文件，每行一个，默认为 localhost，所以在伪分布式配置时，节点即作为 NameNode 也作为 DataNode。分布式配置可以保留 localhost，也可以删掉，让 Master 节点仅作为 NameNode 使用。

本攻略让 Master 节点仅作为 NameNode 使用，因此将文件中原来的 localhost 删除，只添加一行内容：Slave1。

### 4.5.2 core-site.xml 配置

~~~xml
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://Master:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/usr/local/hadoop/tmp</value>
                <description>Abase for other temporary directories.</description>
        </property>
</configuration>
~~~

### 4.5.3 hdfs-site.xml 配置

dfs.replication 一般设为 3，但我们只有一个 Slave 节点，所以 dfs.replication 的值还是设为 1

~~~xml
<configuration>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>Master:50090</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>1</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/usr/local/hadoop/tmp/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/usr/local/hadoop/tmp/dfs/data</value>
        </property>
</configuration>
~~~

### 4.5.4  mapred-site.xml 配置

~~~xml
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>Master:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>Master:19888</value>
        </property>
</configuration>
~~~

### 4.5.5  yarn-site.xml 配置

~~~xml
<configuration>
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>Master</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
</configuration>
~~~

## 4.6 扫荡边角

配置好后，将 Master 上的 /usr/local/Hadoop 文件夹复制到各个节点上。因为之前有跑过伪分布式模式，建议在切换到集群模式前先删除之前的临时文件。  
在 Master 节点上执行：

	$ cd /usr/local
	$ sudo rm -r ./hadoop/tmp     # 删除 Hadoop 临时文件
	$ sudo rm -r ./hadoop/logs/*   # 删除日志文件
	$ tar -zcf ~/hadoop.master.tar.gz ./hadoop   # 先压缩再复制
	$ cd ~
	$ scp ./hadoop.master.tar.gz Slave1:/home/hadoop

在 Slave1 节点上执行：

	$ sudo rm -r /usr/local/hadoop    # 删掉旧的（如果存在）
	$ sudo tar -zxf ~/hadoop.master.tar.gz -C /usr/local
	$ sudo chown -R hadoop /usr/local/hadoop

同样，如果有其他 Slave 节点，也要执行将 hadoop.master.tar.gz 传输到 Slave 节点、在 Slave 节点解压文件的操作。

首次启动需要先在 Master 节点执行 NameNode 的格式化：

	$ hdfs namenode -format       # 首次运行需要执行初始化，之后不需要

接着可以启动 hadoop 了，启动需要在 Master 节点上进行：

	$ start-dfs.sh
	$ start-yarn.sh
	$ mr-jobhistory-daemon.sh start historyserver

通过命令 jps 可以查看各个节点所启动的进程。正确的话，在 Master 节点上可以看到 NameNode、ResourceManager、SecondrryNameNode、JobHistoryServer 进程

在 Slave 节点可以看到 DataNode 和 NodeManager 进程。

缺少任一进程都表示出错。另外还需要在 Master 节点上通过命令 hdfs dfsadmin -report 查看 DataNode 是否正常启动，如果 Live datanodes 不为 0 ，则说明集群启动成功。

*伪分布式、分布式配置切换时的注意事项：*  

> * 从分布式切换到伪分布式时，不要忘记修改 slaves 配置文件；

> * 在两者之间切换时，若遇到无法正常启动的情况，可以删除所涉及节点的临时文件夹，这样虽然之前的数据会被删掉，但能保证集群正确启动。所以如果集群以前能启动，但后来启动不了，特别是 DataNode 无法启动，不妨试着删除所有节点（包括 Slave 节点）上的 /usr/local/hadoop/tmp 文件夹，再重新执行一次 hdfs namenode -format，再次启动试试。


至此，就可以搭建一整套相对“完整”对 Hadoop 环境了。


*本文主要参考了以下资料：*  
  
[官方教程](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

[Hadoop安装教程：单机/伪分布式配置_Hadoop2.6.0/Ubuntu14.04](http://www.powerxing.com/install-hadoop/)

[Hadoop集群安装配置教程](http://www.powerxing.com/install-hadoop-cluster/)
