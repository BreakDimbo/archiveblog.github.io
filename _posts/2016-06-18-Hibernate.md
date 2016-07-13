---
layout: post
date: 2016-06-18
title: Hibernate
excerpt: "Hibernate"
java: true
comments: true
tag: [Java, Hibernate]
---


# Hibernate

> * HelloWorld
> * Hibernate 原理模拟 —— 什么是 O/R Mapping（Object/Relationship），为什么要有 O/R Mapping
> * 常见的 O/R 框架
> *  hibernate 的基础配置
> *  hibernate 的核心接口介绍
> *  对象的三种状态
> *  ID 生成策略
> *  关系映射
> *  Hibernate 查询（HQL）
> *  在 Struts 基础上继续完善 BBS2009
> *  性能优化


## 1.建立 Annotation 版本的 HelloWorld
(JPA-java persistence application)

1. @Entity(javax.persistence.Entity)——表示这家伙是个实体类，会与数据库中的表相对应。
2. @Id —— 一般放在 get 方法上面——标明这个属性是 ID 字段
3. 在 hibernate.cfg.xml 里声明有一个类被加了映射，是实体类

~~~xml
<mapping class="com.limbo.hibernate.model.Teacher"/>
~~~


## 2.什么是 O/R Mapping

在对象和表之间建立联系
简化编程
跨越数据库平台——更改下配置文件里的方言

Hibernate_0200_OR_Mapping_Simulation —— 微型 Hibernate

使用反射机制

* 如何获取方法名：字符串的操作
* 使用反射

~~~java
// waiting for fill
~~~

## 3.常见的 O/R 框架

*JPA*  
ibatis  
hibernate  
jdo  
toplink  

最重要：JPA——一统天下

## 4.常见配置

~~~xml
<property ... ></property>
~~~

* hbm2ddl.auto: validate|update|create（创建表）|create-drop

create 常用

先建类还是先建表？—— 实践中先建表：需要先对数据库进行优化

使用 powerdesigner 建表。

* 日志：org.hibernate.tool.hbm2ddl

> 学习 log4j
> 
> hibernate + log4j —— slj4j

 
### 4.1 在 hibernate 中使用 JUnit

利用 @BeforeClass 标签初始化单例
@Test 标明测试单元
@AfterClass 类似 finally

### 4.2 annotation 常见配置

@Table（注意是 javax) —— Xml 也可以实现同样功能
当类名和表名不一致时，使用这个 annotation 告知应该使用哪个表。  
或者在生成表时指定表名  

@Id 指定 ID 字段

@Column(name="_name") 但是当字段名和属性名不一致时使用。

@Transient 不将该属性加入表中。

@Enumerated 映射枚举类型

## 6. ID 生成策略

### 6.1 在 Xml 中的配置

* increment : 集群环境不要使用，极少，不要使用
* identity：DB2，MySQL
* sequence：Oracle
* uuid：全世界独一无二的值
* native：数据库自己选择——使用最多的

~~~xml
<id name="id">
<generator class="uuid"/>
</id>
~~~

### 6.2 使用 Annotation 进行配置

* 在@Id 下面  
@GeneratedValue：默认为 auto，相当于 native。

* 生成 Sequence：  
放在类上面
SequenceGenerator ：
name=“生成器的名字”
SequenceName=“数据库里面的对象的 sequence 的名字”

* TableGenerator——跨数据库平台时使用（了解）

~~~java
@Entity
@javax.persistence.TableGenerator(
    name="Teacher_GEN",
    table="GENERATOR_TABLE",
    pkColumnName = "pk_key",
    valueColumnName = "pk_value",
    pkColumnValue="Teacher",
    allocationSize=1
)
~~~

### 6.3 联合主键

#### XML：

单独写一个类PK，将联合主键的属性放进去。
将 PK 放进 student 里

* xml里的设置：

~~~xml
<composite-id name="pk" class="com...StudentPK">
<key-property name="id"/>
<key-property name="name"/>
</composite-id>
~~~

实现序列化： 1）为了集群化数据的传输 2）使用虚拟内存

重写 equals 和 hashcode：保证唯一性（在内存中进行区分）

#### Annotation
三种方法：

* 组件类注解为 @Embeddable 并将组件在类中的属性注解为@Id
* 直接在组件方法上写 @EmbededId - 常用
* 分别在作为主键的方法上加上@Id，在类上标明@IdClass(Teacher.class)


## 7 核心开发接口（重要）+ 三种对象状态

Hibernate API 文档

### 7.1 Configuration
Configuration.configure("kkk.xml") 重载的方法可以指定配置文件名。
配置信息管理 

### 7.2 SessionFactory

每一个 Session 都有一个与数据库的链接。  
SF 维护的最重要的东西是*数据库连接池*

~~~java
sessionFactory.getCurrentSession(); 
//不重复创建（不使用 close）
//注意：当使用 session.commit()后，该 session 就自动关闭了。再拿就是新的了。

sessionFactory.openSeesion(); //永远创建新的（用完 close）
~~~

getCurrentSession（)：
*用途：界定事务边界* 当需要多种操作在同一个事务里时，使用 

**需要在配置文件里配置 current_session_context_class 常用取值：thread**

~~~xml
<!-- Enable Hibernate's automatic session context management -->
<property name="current_session_context_class">thread</property>
~~~
JTA（java transaction api） 简介：将两个数据库的操作放在同一个事务里。

> getCurrentSession 和 openSession 不能混用

### 7.3 对象的三种状态
> Transient-Persistent-Detached


1. Transient——刚刚 new 出来，没有 ID，不存在于缓存中
2. Persistant——session.save()/saveOrUpdate() 之后，内存中有，缓存中有，数据库有
3. Detached——session.close() 之后，内存有，缓存没有，数据库有 

三种状态区分的关键在于：

1. 有没有 ID（注意：session 里有一个缓存，里面有一个 hashMap）
2. ID 在数据库中有没有
3. 在内存中有没有（缓存）

### Session 管理一个数据库的任务单元：增删改查

方法：

* save()
* delete() —— 之后变为 Transient 状态（被删除的对象必须有 ID号）
* load(Class, primary_key) 返回的是代理—— 对象变为 Persistance 状态——什么时候用对象里面的内容时才会发送 SQL
* get(Class, primary_key) —— 返回的是对象，立即查询数据库，发出 SQL 语句

> get 与 load 的区别：
> 
> * load 返回代理对象（理解动态代理），等到真正用到对象的内容时才发出 SQL 语句
> * 不存在对应记录时表现不一样
> *  get 直接从数据库加载，没有延迟

* update 什么情况下进行更新
    - 更新 Detach 的对象（commit 之后）
    - 不能更新 transient 对象
    - 如果 transient 对象被手动设置正确 ID，可以更新
    - p 状态的对象只要某个字段值发生改变，就会自动发生更新
    - 更新部分更改的字段
        + 修改注解（Annotation）@Column(updateble=false)—禁止该字段自动更新（极少用）
        + dynamic-update="true" <mapping class="com.limbo.hibernate.model.Teacher" dynamic-update="true"/>
        + 跨 session 实现只更新改过的字段：merge
    - 直接使用 HQL 语句（推荐）

* saveOrUpdate() 根据状态进行 save 或者 update
* clear() 清空缓存
* flush() 强制让缓存和数据库内容进行同步
    -  FlushMode.ALWAYS/NEVER/AUTO/COMMIT/MANUAL
* SchemaExport：在程序里控制生成建表语句

---

## 8 关系映射（重点）

> 关系映射指的是对象之间的关系，并不是指数据库的关系。主要解决当对象之间处于下列关系之一时，数据库表该如何映射，编程上该如何对待
> 
> 三种关系：一对一，一对多，多对一，多对多（单向，双向）
> 
>     
> 集合映射  
> 继承映射（不重要）  
> 组件映射

简化问题：

1. 怎么写 Annotation
2. 增删改查 CRUD 怎么写

常见表设计：主键关联。单向外键关联。

### 8.1 一对一

> * 单向（主键，外键）
> * 双向（主键，外键）
> * 单向，双向的区别：主要体现在 java 编程上，是否可以从一个对象找到另一个对象，在数据库层面上没有区别

单向外键关联：
@OneToOne 注释：加在需要关联的对象的 getter 上
@JoinColumn（name="") 指定外键名字

双向外键关联（mappedBy 必设 ）：
wife 下
@OneToOne(mappedBy="wife") 告诉 hibernate 外键关联已经在 husband 里的 wife 做完了。（wife 映射的是 husband 里的 getter 的后缀名）

一对一单向主键关联（不重要）
@PrimaryKeyJoinColumn

联合主键：  
@Class(WifePK.class)  
@Id  
@Id  
设置名称：  
@JoinClomns  

注意联合主键类实现 Serializable

### 8.2 组件映射

将一个对象作为另一个对象的一部分，做在一张表里面。
Husband.class:  

@Embedded  
getWife()

### 8.3 多对一&一对多

一对多：用户和用户组的关系

> 三范式：   
> 有主键，列不可分  
联合主键不能存在部分依赖  
不能存在传递依赖  

多对一单向关联

注意不要有冗余，在多方加外键，以及对「一」方的引用

Group & User  
User.class  
@ManyToOne  
getGroup()  

一对多单向关联

「一」方添加 Set 集合装「多」，Set 不能重复。
@OneToMany 默认情况会将其作为多对多处理
需要加上 JoinColumn（name="groupId")

双向关联——注意在「一」的一方添加 mappedBy，以多为主导。

### 8.4 多对多的关系

学生老师的关系

数据库中的设计：使用中间表进行连接

* student
* teacher
* t_s

单向：老师知道有多少学生（集合装学生），但是学生不知道有哪些老师
@ManyToMany

@JoinTable(name="t_s",
    joinColumns={@JoinColumn(name="teacher_id")},
    inversJoinColumns={@JoinColumn(name="student_id")}
) 修改第三张表名字,以及字段名。


双向：老师知道自己教了哪些学生，学生也知道教自己的老师有哪些
记得 mappedBy("students")


## 9.关联关系的 CRUD——casecade（CUD） & fetch（R 读）

需要考虑：当两个对象有关联时，存储/删除/更新/查询一个对象时（A），另一个对象怎么处理（B）。

### 9.1 存储——cascade

#### 只存储 user
默认情况下不会自动保存「关联变量（B）」：需要在 A（User） 里的 @ManyToOne 标签的 cascade(传递) 属性：{ALL（任何情况）, MERGE(merge 方法),PERSIST（persist 方法）, REFRESH, REMOVE（remove）}

@ManyToOne(casecade={CascadeType.ALL})

#### 只存储 group
注意设置 u1 & u2 的 group

铁律：

* 双向关系在程序中要设定双向关联
* 双向要 mappedBy

### 9.2 读取——get/load——fetch 

> 默认：取多自动取一，取一自动不取多

get 取 user 时会将 group 取出来（默认情况下就会都取出，不论是否设置 cascade）
使用 fetch 进行设置{EAGER，LAZY}

@ManyToOne(casecade={CascadeType.ALL}, fetch=FetchType.LAZY)

load 与 get 类似，注意 load 拿出来的是代理对象。 代理与 session 相连。
> 双向关联不要两边都设置为 EAGER

树状结构的设计（至关重要）
a. 在同一个类中使用 One2Many & Many2One

### 9.3 更新

只要更改了对象的属性值，commit 后自动 update。


### 9.4 删除

s.delete(u); 注意 cascade 的设置。
当 group 和 user 的 cascade 都为 ALL 时，如何只删除指定 user: 删除之前打破关联关系 u.setGroup(null)，再进行删除。

或者使用 HQL：s.createQuery("delete from User(类名) u where u.id=1");

当删除 group 时，每个 user 都要设置成 null。

关系映射总结：

1. 面向对象的关系，设计什么样的表，进行什么样的映射
2. CRUD，按照自然理解即可
3. cascade 和 fetch

## 10.集合映射（不甚重要）

* Set
* List——可以指定任何字段进行排序或者多个字段的组合进行排序
    - @OrderBy("name ASC")——指定排序字段
* Map——一般 key 是某个字段（主键），value 是对象 Entity
    - @MayKey(name="id") 在 method 上面注释

## 11.继承映射（不甚重要）

> 三种方式：
> 
> * SingleTable： 都存在一个表中，添加一个区别字段。根据区别字段组合出所需对象的信息。不足：太多冗余
> * TablePerClass: 每个类一张表——不足：表与表之间「主键」 不能相同
> * Join：在一张主表（A）里存储所有共有的属性，每个子类（B，C）存储的内容只有自己特定的内容。 主表与字表之间建立主键关联。不足：添加表连接

### 11.1 Single Table
三个注释：
@Inheritance
@DiscriminatorColumn（区分字段）
@DiscriminatorValue（区分字段的值）

### 11.2 TablePerClass

两个注释：
@Inheritance
@TableGenerator

### 11.3 Join

一个注释：
@Inheritance(strategry="")

子类表不要添加@Id


## 树状映射（至关重要——背过）

先设计表，
再站在面向对象的角度考虑设计类

在同一个类中使用 One2Many 和 Many2One

~~~java
public void testSave() {

    Org o = new Org();
    o.setName("总公司");
    Org o1 = new Org();
    o1.setName("分公司01");
    Org o1_1 = new Org();
    o1_1.setName("分公司01_01");
    Org o2 = new Org();
    o2.setName("分公司02");
    Org o2_1 = new Org();
    o2_1.setName("分公司02_01");
    Org o2_1_1 = new Org();
    o2_1_1.setName("分公司02_01_01");


    o.getChildren().add(o1);
    o.getChildren().add(o2);
    o1.getChildren().add(o1_1);
    o2.getChildren().add(o2_1);
    o2_1.getChildren().add(o2_1_1);

    o1.setParent(o);
    o2.setParent(o);
    o1_1.setParent(o1);
    o2_1.setParent(o2);
    o2_1_1.setParent(o2_1);

    Session session = sf.getCurrentSession();
    session.beginTransaction();

    session.save(o);

    session.getTransaction().commit();
}

@Test
public void testTree() {
    testSave();

    Session session = sf.getCurrentSession();
    session.beginTransaction();

    Org o = (Org) session.load(Org.class, 1);
    print(o, 0);

    session.getTransaction().commit();


}

private void print(Org o, int level) {

    StringBuffer preString = new StringBuffer();

    for (int i = 0; i < level; i++) {
        preString.append("-|-");
    }

    System.out.println(preString + o.getName());

    level++;
    for(Org os : o.getChildren()) {
        print(os, level);
    }
}
~~~

## 学生、课程、分数的设计（重要）

> * 使用联合主键 @Embeded —— 实现 Serializable
> * 不使用联合主键
> * 设计：
>     - 实体类（类）
>     - 导航（为了编程方便）


用程序生成表的意义：生成一个导致的框架，作为数据库表调优的指导。


## 12 HQL 查询（Query Language）—— 面向对象！

查询方式：

1. NativeSQL > HQL > EJBQL > QBC > QBE
2. QL 应该和导航关系结合，共同为查询提供方便


创建 Query 接口：使用 org.hibernate.Query

### EJBQL 参考例子程序

~~~java
// 注意写的是类名，不是表名
Query q = session.createQuery("from Category");
// 返回一个对象
List<Category> categories = (List<Category>) q.list();

q.uniqueResult() 只返回一个结果

// NativeQuery：
SQLQuery q = session.createSQLQuery("select * from category limit 2,4").addEntity(Category.class);
~~~

用 in 还是 exists 但是用 exists ，应为它的效率高

### QBC & QBE

~~~java
Cirteria c =session.createCriteria(Topic.class);
~~~


## 12 性能优化

* 注意 session.clear() 的运用，尤其在不断分页循环的时候
    - 在一个大集合中进行遍历，遍历 msg，取出其中的含有敏感字样的的对象
    - 另外一种形式的内存泄漏 // java 有内存泄漏吗？（注意连接池，文件，等资源的关闭）
* 1+N 的问题：当 FetchType="Eager"，时本来只应该发一条 SQL 语句，但是发了 N 条。解决方案：
    - 将 fetch 设置为 lazy
    - 在 Category 上设置 BatchSize（size=n）
    - join fetch（或者使用 Createcriteria）
* List 和 iterator 
    - iterator 先取出 Id，用到其他信息时，再去取
    - list 一次性取出所有对象
    - iterator 会利用 session 里面的缓存
    - list 不会利用 session 的缓存，再次执行时会再次从数据库读取数据。
    - 使用：第一次取的时候用 list，再使用这些数据时使用 iterator 

* 一级缓存和二级缓存（面试题）
    - 什么是缓存
    - 是么是一级缓存，session 级别的缓存
    - 什么是二级缓存，SessionFactory 级别的缓存，可以跨越 session 存在
        + 经常被访问
        + 不会经常改动
        + 数量有限
    - 打开二级缓存
        +  hibernate.cfg.xml 设定：
~~~xml
<!-- Disable the second-level cache  -->
    <property name="...">true</property>
    <property name="cache.provider_class">org.hibernate.cache.internal.EhCacheProvider</property>

    <!-- 另外需要在 ehcache 的 xml 文件进行配置 -->
~~~
    - load 默认使用二级缓存，iterator 默认使用二级缓存
    - list 默认往二级缓存加数据，但是查询的时候不使用
    - 如果要 query 用二级缓存，需打开查询缓存
    - 要把某个实体类放进二级缓存：@Cache

* 缓存算法：缓存满了之后哪个对象先完蛋（LRU、LFU、FIFO）
    - Least Recently Used
    - Least Frequently Used
    - First in First Out

### 事务隔离机制（并发处理）——乐观锁——悲观锁

#### 事务：ACID
什么是事务：要么都完成，要么都不完成

可能出现的问题：

* 第一类丢失更新（Lost Update）
* dirty read 脏读（*）——你读到了别的事务还么有提交的数据
* 不可重复读（*）—— 同一个事务里对同一个数据前后读两回的值不同
* 第二类丢失更新——在读的时候其他事务插入和删除了数据
* 幻读（*）

#### 数据库的事务隔离机制
四种：read-uncommitted read-commited repeatable non-repeatable

read-uncommiteed: 允许：脏读、不可重复读、幻读
read-committed: 会出现：不可重复读、幻读(一般设置为这个)
repeatable read: 给数据加一把锁、可重复读
serial 解决一切问题

hibernate设定隔离机制级别
hibernate.connection.isolation=2;
用悲观锁-乐观锁解决：repeatable-read

* 用悲观锁解决（依赖于数据库的锁）在读取时加一把锁，加锁后其他事物都不能进行修改——对其他事物悲观
1. select...for update
2. load(xx.class, i, LockMode.Upgrade)
    a) 

* 乐观锁：允许其他事务更新，但是在原事务进行更新前会与原值进行比较，查看是否有修改。

添加一个 version 属性，并在 getter 上加一个注解：@Version，表示该字段专门用来记录版本。不需要合适 version，hibernate 自动设置 version。当出现不可重复读问题时，会报错。


## 总结：

xml 建议查
annotation 查背结合
原理模拟
基础配置
CLOB BLOB
自定义类型
ID 生成策略

核心开发接口
configuration API：buildSessionFactory
SF.openSession/getCurrentSession 注意区别（背过）
Session 重要:save delete get load update clear flush
SchemaExport

HQL

对象状态 小两口

对象之间关系：一对一

继承映射，需要能说出来
树状结构 重要

1+N 很重要
