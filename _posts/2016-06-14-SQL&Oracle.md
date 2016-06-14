---
layout: post
date: 2016-06-14
title: SQL&Oracle
excerpt: 
java: true
comments: true
tag: [Java, SQL]
---

# lesson 2

登陆：sqlplus  
解锁用户：alter user outln account unlock

# lesson 3

### 四大语句  
1. 查询语句：select  
2. dml 语句：数据操作语句  
3. ddl 语句：  
4. 事物控制语句

select ［选择的表名］, 12\*sal from emp

~~~sql
select ename, 12\*sal anuual_sal from emp
~~~

# lesson 4 
任何含有空值的表达式都是空值，注意空值的计算

字符串连接符||  
select ename||sal from emp;

SQL 使用两个单引号引起来表示字符  

~~~sql
select ename || 'sjajshdask' from emp;  

--使用两个单引号表示一个单引号
select ename || 'skadhsk''sjahd' from emp
~~~

# lesson 5 

~~~sql
--将重复的值消掉
select distinct deptno from emp;  
--去掉重复的组合  
select distinct deptno, job from emp;
~~~

# lesson 6

### where：过滤条件。  

~~~sql
select * from emp where deptno=10;  
select * from emp where ename = 'CLARK'   
~~~

找出薪水大于1500的人  
找出部门编号不是10点  
找出薪水在800到1500的人  

between and:  

~~~sql
select ename from emp where sal between 800 and 1500;
--包含800和1500
~~~

### 空值的处理  
选出（非）空值  

~~~sql
select ename, sal, comm from emp where comm is (not)null; 
~~~

### in 语句：  

~~~sql
select ename, sal, comm from emp where sal in (800, 1500, 2000);  
select ename, sal, comm from emp where ename in('djsks', 'skdjska');
~~~

### Data 日期的处理： 

~~~sql 
select ename, sal, comm from emp where hiredate > '10-2-81' --按照指定格式
~~~

### and or not 的使用：  

~~~sql
select ename, sal, from emp where depton = 10 and sal > 1000;
~~~

### 模糊查询：  

~~~sql
select ename, sal, from emp where ename like '%ALL%'

select ename, sal, from emp where ename like '_A%'

select ename, sal, from emp where ename like '%\%%';

select ename, sal, from emp where ename like '%$%' escape '&';  
~~~

# lesson 7 排序

默认按照插入时的顺序排序

~~~sql
--默认为升序
select * from dept order by dept;  

--如果要降序，加上desc
select * from dept order by deptno desc;
select empno, ename from emp order by empno asc;
~~~

结合where语句

~~~sql
select empno, ename from emp where depto <> 10 order by empno asc;
~~~

使用两个关键字 进行排序  

~~~sql
select empno, ename from emp order by deptno asc, ename desc;
(先按照deptno排序，相同的情况下再按照ename排序)
~~~

总结：取出名字和年薪，命名，其中名字的第二个字母不是A且薪水大于800，结果按倒序排列。

~~~sql
select ename, sal*12 annual_sal from emp
	where ename not like '_A%' and sal > 800
		order by sal desc;
~~~

# lesson 8 常用的单行 SQL 函数

**lower: 转换成小写  
upper：转换成大写**  

~~~sql
select lower(ename) from emp;  

--取出第二个字母是a（不论大小写）的名字：
select ename from emp 
	where lower(ename) like '_a%';
~~~

**substr:**

~~~sql
select substr(ename, 2, 3) from emp;

--chr:(将一个数字装换成对应的字符)
select chr(65) from dual;

select ascii(A) from dual;
~~~

**round:（四舍五入）**

~~~sql
select round(23.652) from dual;

select round(23.652, 2) from dual;

select round(23.652, -1) from dual;
~~~

***重点：to_char***

~~~sql
select to_char(sal, 'L99,999,9999') from emp;
select to_char(sal, 'L00000.0');

--对日期进行转换：
select to_char(hiredate , 'YYYY-MM-DD HH:MI:SS') from emp;
select to_char(sysdate, 'YYYY-MM-DD HH24:MI:SS') from dual;
~~~

# lesson 9 常用的 SQL 函数


**to_date:将字符串转换成日期的格式**

~~~sql
select ename, hiredate from emp   
	where hiredate > '1981-2-20 12:22:23'; --这样会报错

select ename, hiredate from emp
	where hiredate > to_date('1982-2-20 12:22:23', 'YYYY-MM-DD, HH24:MI:SS');
~~~

**to_number: 将字符串转换成数字：** 

~~~sql
select sal from emp where sal > L1928738; --报错

select sal from emp where sal > to_number('L1,234,345', 'L99,999,9999');
~~~

**nvl 处理空值的问题**：

~~~sql
select ename sal*12 + nvl(comm, 0) from emp;  
--如果是空值，则用0替代它
~~~

# lesson 10 组函数-5个（必须记住）

很多条输入，一条输出

~~~sql
select max(sal) from emp;

select min(sal) from emp;

select avg(sal) from emp;

select to_char(arg(sal), '9999999,99') from emp;

select round(avg(sal), 2) from emp;

select sum(sal) from emp;

--求出表中有多少条记录(count 的不是空值的字段有几个)
count(*): 

select count(*) from emp；

select count(*) from emp where deptno = 10;

select count(ename) from emp;

select count(comm) from emp;

select count(distinct deptno) from emp;
~~~

# lesson 11 group_by 语句

如何按照部门进行分类，求部门的平均的薪水。

~~~sql
select sal, deptno from emp;  
select avg(sal), deptno from emp group by deptno;

select max(sal) from emp group deptno, job;


select avg_sal, dname from 
(select avg(sal) avg_sal, deptno from emp group by emp.deptno)e
join (select dname, deptno from dept) d
on (e.deptno = d.deptno)

~~~

求出在所有人里，薪水值最高的人的薪水值。  
再求出挣这么多钱的人的薪水。

~~~sql
select max(sal), ename from emp; 报错。  
~~~

因为可能会有很多个挣这么多的钱的人。而组函数的输出只有一个。ename和max（sal）的结果无法对应。

~~~sql
select ename from emp where sal = (select max(sal) from emp);
~~~

再按部门分类

**(如果存在组函数，出现在select 列表里面的字段，如果没有出现在组函数里，必须出现在group by 里)**  
注意下面两种的区别：

~~~sql
select ename, max(sal) from emp group by deptno;
 
select deptno, max(sal) from emp group by deptno;
~~~



# lesson 12 having
使用having对分组进行限制  
where 只能对**单条**记录进行过滤，所以where语句不能对group进行操作


~~~sql
select avg(sal), deptno from emp group by deptno having avg(sal) > 2000;
~~~

**复习：执行顺序，不能错**

~~~sql
select * from emp
	where sal > 1000
	group by deptno
	having
	order by
~~~


薪水大于1200的雇员按照部门编号进行分组，这些人分组后的平均薪水大于1500，查询分组之内的平均工资，按照工资的倒序进行排序：

~~~sql
select deptno,  avg(sal)
	from emp
	where sal > 1200
	group by deptno
	having avg(sal) > 1500
	order by avg(sal) desc;
~~~

# lesson 14 (自连接)

找出雇员中哪些人是经理人

求出这个人的名字－》对应的经理人编号－》对应的经理人
为同一张表起两个别名，将其当作两张表来用

~~~sql
select e1.ename, e2.ename from emp e1, emp e2 where e1.mgr = e2.empno;
~~~

# 15 SQL 1999\_table_connection

将过滤条件与连接条件分开

~~~sql
select ename, dname from emp cross join dept; 
产生一个笛卡尔乘积
~~~

连接条件写在on里

~~~sql
select ename, dname from emp 
	join dept 
		on(emp.deptno = dept.deptno);
~~~

等值连接可以使用using

非等值连接

~~~sql
select ename, grade, from emp e 
	join salgrade s 
		on (e.sal between s.losal and s.hisal);

三张表连接
select ename, dname, grade from emp e 
	join dept d on (e.deptno = d.deptno)
	join salgrade s on(e.sal bewteen s.losal and s.hisal)
	where ename not like '_A%';
~~~ 


自连接-找出经理人

~~~sql
select e1.ename, e2.ename from emp e1 
	join emp e2 on (e1.mgr = e2.empno);
~~~

外连接：  
左外连接：会将左边不能被连接的数据显示出来
右....

~~~sql
seleclt e1.ename, e2.ename from emp e1 
	left join emp e2 on (e1.mgr = e2.empno);
	
select ename, dname from emp e 
	right join dept d on (e.deptno = d.deptno);
~~~

全外连接：将两边多余的数据拿出来

~~~sql
select ename, dname from emp e 
	full join dept d on (e.depton = d.deptno)
~~~


# lesson 16-24 子查询＋表连接

*在select语句中套用select语句即为子查询*
  
> * 谁挣的钱最多
> 
~~~sql
select ename from emp 
	where sal = (select max(sal) from emp);
~~~

> * 求出哪些人的工资位于所有人的平均工资之上
> 
~~~sql 
select ename from emp
	where sal > (select avg(sal) from emp)
~~~

> * 请求出，按照部门进行分组之后，每个部门中挣钱最多的那个人的名称，部门编号
> 
~~~sql
--理解子查询的关键：将其理解成一张表
select ename, emp.deptno from emp
	join (select deptno, max(sal) max_sal from emp group by deptno) t
	on (emp.sal = t.max_sal and emp.deptno = t.deptno);
~~~

> * 求每一个部门平均薪水，求出后，求出平均薪水的等级
> 
~~~sql
select avg(sal) avg_sal, deptno from emp group by deptno
	join salgrade s on (emp.avg_sal between s.losal and s.hisal);
~~~
>
~~~sql
select deptno, grade, avg_sal from 
	 (select deptno, avg(sal) avg_sal from emp group by deptno) t
	join salgrade s on (t.avg_sal between s.losal and s.hisal);
~~~

> * 求每个部门的平均薪水等级  
1.求每个人的薪水等级  
2.求每个人的薪水等级的平均值，然后按照部门分类  
>
~~~sql
select avg(grade) from
	(select deptno, ename, grade from emp
		join salgrade s 
			on (emp.sal between s.losal and s.hisal)) t
group by deptno;
~~~

> * 雇员中有哪些是经理人
> 
~~~sql
select ename from emp 
	where empno in(select distinct mgr from emp);
~~~

> * 不用组函数,求最高薪水
> 
~~~sql
--使用自连接，左边的薪水值小于右边的，左边连接不上的就是最大值
select distinct e1.sal from emp e1 
	join emp e2 on (e1.sal < e2.sal);
select distinct sal from emp 
	where sal not in
	(select distinct e1.sal from emp e1 join emp e2 on (e1.sal < e2.sal)); 
~~~

> * 平均薪水最高的部门编号与名称
> 
~~~sql
select deptno, avg_sal from
	(select avg(sal) avg_sal, deptno from emp group by deptno) 
	where avg_sal = 
	(select max(avg_sal) from 
		(select avg(sal) avg_sal, deptno from emp group by deptno));
~~~
~~~sql
--把求出来的东西当作一个值：
slect dname from dept where deptno =
(
	select deptno from
		(select avg(sal) avg_sal, deptno from emp group by deptno) 
		where avg_sal = 
		(select max(avg_sal) from 
			(select avg(sal) avg_sal, deptno from emp group by deptno));
)
~~~

> * 嵌套组函数(最多只能嵌套两层)，改写上面的语句：
> 
~~~sql
slect dname from dept where deptno =
(
	select deptno from
		(select avg(sal) avg_sal, deptno from emp group by deptno) 
		where avg_sal = 
		(select max(avg(sal)), deptno from emp group by deptno);
)
~~~

> * 平均薪水的等级最低的部门名称
> 
~~~sql
select dname, t1.deptno, grade, avg_sal from
	(
	select grade, deptno, avg_sal from
			(select avg(sal) avg_sal, deptno from emp group by deptno) t
	join salgrade s on (t.avg_sal between s.losal and s.hisal)
	) t1
join dept on (t1.deptno = dept.deptno)
where t1.grade = 
(
	select min(grade) from 
		(select grade, deptno from
			(select avg(sal) avg_sal, deptno from emp group by deptno) t
			join salgrade s on (t.avg_sal between s.losal and s.hisal))
)
~~~

> * 使用view去掉重复部分(创建一个视图)-就是一个子查询（表）
> 
~~~sql
create view v$_dept_avg_sal_if as 
select grade, deptno, avg_sal from
			(select avg(sal) avg_sal, deptno from emp group by deptno) t
			join salgrade s on (t.avg_sal between s.losal and s.hisal)
~~~
~~~sql
select dname, t1.deptno, grade, avg_sal from
	v$_dept_avg_sal_if t1
join dept on (t1.deptno = dept.deptno)
where t1.grade = 
(
	select min(grade) from v$_dept_avg_sal_if
)
~~~

> * 求部门经理人中平均薪水最低的的部门名称

~~~sql

~~~

> * 比普通员工的最高薪水还要高的经理人名称

~~~sql
select ename from emp
where empno in (select distinct mgr from emp
		where mgr is not null) and
sal > 
(
select max(sal) from emp
	where empno not in (select distinct mgr from emp
		where mgr is not null)
)	
~~~

~~~sql
select ename from emp
	where empno in (select distinct mgr from emp where mgr is not null) and 
sal >
(
select max(sal) from emp
	where empno not in (select mgr from emp where mgr is not null)
)
~~~





# lesson 25-26 DML语句(数据操作语句)

> insert  
> update  
> delete  

每一个数据库有许多表空间。以及数据库结构。

## 第一部分 添加新用户
*添加另一个新用户，与Scott内容相同*

drop user scott casecade; (删除用户)

1. back up scott  
	新开终端，找到目标目录，执行exp
2. 创建用户：create user  
	create user limbo identified by 123456 default tablespace users quota 10M on users  
	分配权限：  
	grant create session(登陆权限), create table, create view to limbo  
3. 导入数据：   
	进入终端，进入导出文件所在目录，执行imp。  
	第一个用户名是limbo，第二个用户名是scott。 
	
注意，在导入用户时可能会出现如下错误：
另外在SQL中创建用户时需要加上c##

~~~bash
[oracle@oracle Documents]$ imp

Import: Release 12.1.0.2.0 - Production on Wed Jun 8 12:07:57 2016

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

Username: c##limbo
Password: 

IMP-00058: ORACLE error 1034 encountered
ORA-01034: ORACLE not available
ORA-27101: shared memory realm does not exist
Linux-x86_64 Error: 2: No such file or directory
Additional information: 3640
Additional information: 768791467
IMP-00005: all allowable logon attempts failed
IMP-00000: Import terminated unsuccessfully
[oracle@oracle Documents]$ ^C
[oracle@oracle Documents]$ imp c##limbo/tiger@break
~~~

最后一行是解决办法：需要表明你的数据库的globe name

## 第二部分 语句学习

>rollbakc 退回

>备份整张表：

>create table emp2 as select * from emp;  
create table dept2 as select * from dept;  
create table salgrade2 as select * from salgrade;  
create table emp3 as select * from emp;

insert 语句

~~~sql
按照字段排列顺序插入
insert into dept values(50, 'game', 'Beijing');
按照选择字段插入
insert into dept (deptno, dname) values(60, 'game2');
将子查询里的字段插入到一个表中(子查询中拿出的东西必须与原表结构一致)
insert into dept select * from dept2;
~~~

### rownum

row number(伪字段)相当于每个表的一个行号。（1，2，...）  
**只能和小于或者小于等于一块用，不能用大于或者单独的等于**

~~~sql
select empno, ename from emp;

前五行
select empno, ename from emp
	where rownum <= 5;
	
后四行
select ename from
	(slect rownum r, ename from emp)
	where r > 10;
~~~


－－求薪水最高的前五名雇员：  

~~~sql

select ename, sal from
	(select ename, sal from emp order by sal desc)
	where rownum <=5;
~~~

－－ **取薪水最高的第六个到第十个人(重点掌握！)。**

~~~sql
select ename, sal from 
(
select ename, sal, rownum r from
	(select ename, sal from emp order by sal desc)
)
where r between 6 and 10;
~~~

－－ 面试题：

有三个表 S, C, SC  
S(SNO, SNAME) 代表（学号，姓名）  
C(CNO, CNAME, CTEACHER) 代表（课号，课名，教师）  
SC(SNO, CNO, SCGRADE) 代表（学号，课号，成绩）  

问题：  
1. 找出没有选过“黎明”老师的所有学生的姓名  
2. 列出2门以上（含两门）不及格学生姓名及平均成绩  
3. 既学过1号课程又学过2号课程所有学生的姓名  

~~~sql
select sname from s
	join sc on(s.sno = sc.sno)
		join c on (sc.cno = c.cno))
where c.cteacher <> '黎明';
~~~

~~~sql
select sname where sno in 
(select sno from sc where scgrade < 60 group by sno having count(*) >=2 
)
~~~

~~~sql
先选1的再选2的，两张表再连接
select sname from s where sno in
(
select sno from sc where cno = 1 and sno in (select distinct sno from sc where cno = 2);
)
~~~

## lesson 26 update & delete

update

~~~sql
update emp2 set sal = sal*2, ename = ename||'-' where deptno = 10;
~~~

delete

~~~sql
delete from emp2
delete from dept2 where deptno < 25

rollback
~~~

# lesson 26 事物（transaction）控制语句

一系列语句要么同时完成，要么不完成——类似Synchronize。

事物启示于第一条DML语句。  
结束于：   
1. rollback   
2. 用户显示地执行**commit**语句  
3. 执行ddl(create ...)或者dcl（grant ...）语句 事物自动提交  
4. 当用户正常断开连接时——exit。 



# lesson 27 数据库常用对象－表／约束／视图

DDL 数据定义语句（建表，建视图）


create	table t (a varchar2(10));
drop table t;

### 创建一张表：

字段类型  
varchar2(n)-变长字符串  (最大4096字节)  
char(n)－定长字符串－存取效率高－类似数组  
number(8,3)  
date   
long(2G)-可变字符串

建表例子-注意默认值的使用,以及最后一个不加逗号

~~~sql
create table stu
	(
	id number(6),
	name varchar2(20),
	sex number(1),
	age number(3),
	sdate date,
	grade number(2) default 1,
	class number(4),
	email varchar2(50)
	);
~~~

## lesson 28－29 约束条件－5个

非空： not null  

唯一：unique 该字段内只能取唯一值  

主键：（primary key）可以**唯一**标识整条记录的东西（比如id）  
语法上：非空＋唯一。逻辑意义：唯一代表整条记录  

外键：（foreign key）建立在一张表的两个字段上，或者两个表的两个字段上。某一个字段会参考另一个字段的值，如果被参考的字段中不存在这个值，则无法insert这个字段（**被参考的字段必须是主键, 不能删除被参考的字段**）

check：  

举例：  
> 约束：  
名字不能为空值  
email 不能为重复值  
主键是 id
外键：不能插入不存在的班级（当进行插入时，会参考reference的表）


~~~sql
## 字段级约束
create table stu
	(
	id number(6) primary key,
	name varchar2(20) not null,
	sex number(1),
	age number(3),
	sdate date,
	grade number(2) default 1,
	class number(4) references class(id),
	email varchar2(50) unique
	);
	
create table class
	(
	id number(4) primary key,
	name varchar2(20) not null
	);
~~~

约束为对象  
为约束取名字： constraint stu\_name_nn not null;

如何实现name和email的组合不能重复。

~~~sql
## 表级约束。
create table stu
	(
	id number(6),
	name varchar2(20) constraint stu_name_nn not null,
	sex number(1),
	age number(3),
	sdate date,
	grade number(2) default 1,
	class number(4),
	email varchar2(50),
	constraint stu_class_fk foreign key (class) references class(id),
	constraint stu_id_pk primary key(id),
	constraint stu_name_email_uni unique(email, name)
	);
~~~

## lesson 31 修改表结构-alter


~~~sql
alter table stu add(addr varchar2(100));

alter table stu drop (addr);

alter table stu modify(addr varchar2(150));

有数据的情况下，修改后的新的精度必须可以容纳愿有数据


删除或者增加约束条件

alter table stu drop constraint stu_class_fk;

修改约束条件，需要先删除，再增加

删除一张表：
drop table stu;
~~~

## lesson 32 数据字典表－user_tables

desc user_tables
如何查询当前用户下有多少表

~~~sql

select table_name from user_tables;

select view_name from user_views;

select constraint_name from user_contraints;

~~~

有多少张数据字典表？

desc dictionary-装有数据字典表的表。


## lesson 33 索引（重点）
优化要首先考虑索引

为某个字段建立索引：

~~~sql
create index idx_stu_email on stu (email);

为两个字段的组合建立一个索引
create index idx_stu_email on stu (email, class);

drop index idx_stu_email;

select index_name from user_indexes;
~~~

当给字段加主键或者唯一约束时，系统会自动为其添加索引。

索引的作用，增加读取的效率。但是会降低添加和删除的效率。


视图：

~~~sql
create view v$_dept_avg_sal_info

视图的其他功能：隐蔽一些信息

create view v$_stu as select id, name, age from stu;
~~~

问题是，不易维护。
视图可以更新数据，但是不建议用它来进行更新。

## lesson 34 序列

用来产生唯一的，不重复的，数字的序列，一般用于做主键。（内部包含线程同步）

~~~sql
create table article
(
id number,
title varchar2(1024)
cont long
);

如何将帖子插入表中。

create sequence seq;
select seq.nextval;

insert into artical values(seq.nextval, 's', 'b');

从哪开始，每次递增几
start with 1, increament by 1


删除：
drop seq
~~~

# 第四部分 三范式 lesson 35

定义：

范式：数据库设计的规则  
原则：不存在冗余数据（同样的数据不存第二次）

举例：设计一张表存储所有人的信息：学号，姓名，年龄

第一范式  
第一个要求：要有主键  
第二个要求：列不可分  

第二范式 （多对多设计时所需遵循的）
当一张表里有多个字段作为主键，不是主键的字段不能依赖部分主键：不能存在部分依赖。——学生姓名依赖于学号。

解决——分割成多个表。学生表，老师表，学生－老师表


第三范式：


## lesson 36-37 BBS

设计一个bbs数据库表

需求：
1. 论坛需要分板块
2. 板块里有不同的帖子
3. 帖子带有回复功能（任何人可以回复任何人）（树状结构）
4. 只有注册用户才能发表帖子
5. 每一个版块有自己的版主——有删帖的权限

权限（资源－操作：某个人对某个资源有哪些操作）

~~~sql
## board
create table board 
(
id number(3) key primary,
name varchar2(20),
admin varchar2(10),
);

## message
create table message
(
id number(5) key primary,
user_id number(20),
reply_id number(10),
board_id number(10) references board,
title varchar2(50),
cont long,
reply long
);

## user
create table user
(
id number(10) key primary,
user_name varchar2(20) unique,
passwd number(10)
)

~~~

# 第五部分 PL_SQL 语言

## lesson 38

带有分支和循环的语言。

declare(可选，生命变量)
begin(必要，程序开始执行)
exception(异常处理)
end（结束）

~~~sql	
set serveroutput on; ##(进行输出的环境变量)
begin
	dbms_output.put_line('HelloWorld!');
end;
/
~~~

declare 语句

~~~sql
declare
	v_name varchar2(20);
begin
	v_name := 'myname';
	dbms_output.put_line(v_name);
end;
/
~~~

完整语句块

~~~sql
declare
	v_num number := 0;
begin
	v_num := 2/v_num;
		dbms_output.put_line(v_num);
exception
	when others then
			dbms_output.put_line('error');
end;
/
~~~

## lesson 39 变量声明

变量名不能使用保留字  
第一个字符必须是字母  
变量名最多包含30个字符  
不要与数据库的表或者列同名  
每一行只能声明一个变量  


常用变量类型

binary_integer：整数，主要用来计数  
number  
char  
varchar2  
date  
long 最长2G
boolean  一定给一个值


###变量声明，使用 ％type属性
可以是声明的变量与%type前面指定的变量类型一起变化。

~~~sql
declare
v_empno number(4);
v_empno2 emp.empno%type;
v_empno3 v_empno2%type;
~~~


复合变量：

table 相当于 java的数组  
record 相当于java的类

－－Table 变量类型

~~~sql
declare
##声明一个新的table类型
	type type_table_emp_empno is table of emp.empno%type index by binary_integer;
##用数组声明变量
		v_empnos type_table_emp_empno;
begin
	v_empnos(0) := 7369;
		v_empnos(2) := 7839;
		v_empnos(-1) := 9999;
		dbms_output.put_line(v_empnos(-1));
end;
~~~


－－Record 变量类型 - 表结构记录器／生成器

但是下面这种情况，当dept的表结构发生变化时，record变量不会跟着变。

~~~sql
declare
	type type_record_dept is record
		(
			deptno dept.deptno%type,
			dname dept.dname%type,
			loc dept.loc%type,
		);
		v_temp type_record_dept;
begin
	v_temp.deptno := 50;
	v_temp.dname := 'aaaa';
	v_temp.loc := 'bj';
	dbmc_output.put_line(....);
end;
~~~ 

－－ 使用%rowtype 声明 record变量——保持recode变量与被rowtype的变量保持同步

~~~sql
declare
	v_temp dept%rowtype;
~~~

## lesson 43 PL 语句中的SQL语句

注意：slelect  
1. select语句必须返回一条记录，只能返回一条记录。  
2. into 关键字 必须有

完成后记得commit。

~~~sql
declare 
	v_ename emp.ename%type;
	v_sal emp.sal%type;
begin
	select ename, sal into v_ename, v_sal from emp where empno = 7369;
	dbmc_output.put_line(v_ename || '' || v_sal);
end;
/
~~~

inset, update, delete 与之前的一致。

**sql%rowcount**

~~~sql
declare
   v_deptno emp2.deptno%type := 10;
   v_count number;
begin
  --update emp2 set sal = sal/2 where deptno = v_deptno;
  --select deptno into v_deptno from emp2 where empno = 7369;
  select count(*) into v_count from emp2;
  dbms_output.put_line(sql%rowcount || '条记录被影响');
 commit;
end;
~~~

## lesson 44 PL 语句中的DDL语句

execute immediate

~~~sql
begin
	execute immediate 'create table T (nnn varchar2(20) default ''aaaa'')';
end;
/
~~~


### PL 语句中的的条件语句

~~~sql
declare
	v_sal emp.sal%type;
begin
	select sal into v_sal from emp
			where empno = 7369;
	if (v_sal < 1200) then
			dbms_output.put_line('low');
	elsif(v_sal < 2000) then
			dbms_output.put_line('middle');
	else
			dbms_output.put_line('high');
	end if;
end;
/
~~~

循环语句：

loop ＋ end loop

~~~sql
-- do while 循环
declare
	i binary_integer := 1;
begin
	loop
		dbmc_output.put_line(i);
		i := i + 1;
		exit when (i > 11);
	end loop;
end;
/
~~~

~~~sql
--while 循环

declare 
	j binary_integer := 1;
begin
	while j < 11 loop
		dbmc_ouput.put_line(j);
			j := j + 1;
	end loop;
end;
~~~

~~~sql
--for 	循环

begin
 for k in 1..10 loop
 	dbmc_output.put_line(k);
 end loop;
 
 for k in reverse 1..10 loop
 	dbmc_output.put_line(k);
 end loop;
end;
/
~~~

## lesson 46 错误处理

日志

~~~sql
--创建事件日志表
create table errorlog
(
id number primary key,
errcode number,
errmsg varchar2(1024),
errdate date
);

--创建序列
create sequence seq_errorlog_id start with 1 increment by 1 
--实验
declare
   v_deptno dept.deptno%type := 10;
   v_errcode number;
   v_errmsg varchar2(1024);
begin
   delete from dept where deptno = v_deptno;
 commit;
exception
   when others then
      rollback;
         v_errcode := SQLCODE;
         v_errmsg := SQLERRM;
      insert into errorlog values (
      seq_errorlog_id.nextval, v_errcode, v_errmsg, sysdate);
      commit;
end;
~~~

# lesson 47 游标cursor 重点

类似java的iterator。指在select返回的结果集。

~~~sql
declare
   cursor c is
            select * from emp;
   v_temp c%rowtype;
begin
    open c;
    fetch c into v_temp;
    dbms_output.put_line(v_temp.ename);
    close c;
end;
~~~

利用游标和循环进行遍历

~~~sql
-- 注意 notfound
declare
    cursor c is
       select * from emp;
    v_emp c%rowtype;
begin
    open c;
    loop
      fetch c into v_emp;
      exit when (c%notfound);
      dbms_output.put_line(v_emp.ename);
    end loop;
    close c;
end;

-- while
declare
    cursor c is
       select * from emp;
    v_emp c%rowtype;
begin
    open c;
    fetch c into v_emp;
    while (c%found) loop
      dbms_output.put_line(v_emp.ename);
      fetch c into v_emp;
    end loop;
    close c;
end;

-- for loop -- 用得最多的
declare
    cursor c is
       select * from emp;
begin
   for v_emp in c loop
        dbms_output.put_line(v_emp.ename);
    end loop;
end;


--带参数的游标
declare
   cursor c (v_deptno emp.deptno%type, v_job emp.job%type)
   is
     select ename, sal from emp where deptno = v_deptno and job = v_job;
begin
   for v_temp in c(30,'CLERK') loop
      dbms_output.put_line(v_temp.ename);
   end loop;
end;

--可更新的游标
declare
  cursor c
  is
    select * from emp2 for update;
begin
   for v_temp in c loop
      if (v_temp.sal < 2000) then
         update emp2 set sal = sal * 2 where current of c;
      elsif (v_temp.sal = 5000) then
         delete from emp2 where current of c;
      end if;
    end loop;
    commit;
end;
~~~

## lesson 49 存储过程

带有名字的一个PL SQL的程序块。

~~~sql
create or replace procedure p
is
  cursor c
  is
    select * from emp2 for update;
begin
   for v_temp in c loop
      if (v_temp.deptno = 10) then
         update emp2 set sal = sal + 10 where current of c;
      elsif (v_temp.deptno = 20) then
         update emp2 set sal = sal + 20 where current of c;
      else
         update emp2 set sal = sal + 50 where current of c;
      end if;
    end loop;
    commit;
end;
--执行 
exec p;
或
begin;
 p;
end;

--带参数的存储过程
create or replace procedure p
     (v_a in number, v_b number, v_ret out number, v_temp in out number)
is

begin
   if (v_a > v_b) then
      v_ret := v_a;
   else
      v_ret := v_b;
   end if;
   v_temp := v_temp + 1;
end;
／

--实验
declare
 v_a number := 3;
 v_b number := 4;
 v_ret number;
 v_temp number := 5;
begin
 p(v_a, v_b, v_ret, v_temp);
 dbms_output.put_line(v_ret);
 dbms_output.put_line(v_temp);
end;
~~~

~~~sql
--函数
create or replace function sal_tax
  (v_sal number)
  return number
is
begin
   if (v_sal < 2000) then
      return 0.10;
   elsif (v_sal < 2750) then
      return 0.15;
   else
      return 0.20;
   end if;
end；
~~~

## lesson 50－51 trigger
触发器；掌握概念

~~~sql
--触发器
create table emp2_log
(
uname varchar2(20),
action varchar(10),
atime date
)

create or replace trigger trig
  after insert or update or delete on emp2
begin
  if inserting then
     insert into emp2_log values (USER, 'insert', sysdate);
  elsif updating then
     insert into emp2_log values (USER, 'update', sysdate);
  elsif deleting then
     insert into emp2_log values (USER, 'delete', sysdate);
  end if;
end;
~~~

## lesson 51 树状结构的存储与展现

~~~sql

--树状结构的存储与展现
drop table article;
 
create table article
(
id number primary key,
cont varchar2(4000),
pid number,
isleaf number(1), --0代表非叶子节点，1代表叶子节点
alevel number(2)
)

insert into article values (1, '蚂蚁大战大象', 0, 0, 0);
insert into article values (2, '大象被打趴下了', 1, 0, 1);
insert into article values (3, '蚂蚁也不好过', 2, 1, 2);
insert into article values (4, '瞎说', 2, 0, 2);
insert into article values (5, '没有瞎说', 4, 1, 3);
insert into article values (6, '怎么可能', 1, 0, 1);
insert into article values (7, '怎么没可能', 6, 1, 2);
insert into article values (8, '可能性是很大的', 6, 1, 2);
insert into article values (9, '大象进医院了', 2, 0, 2);
insert into article values (10, '护士是蚂蚁', 9, 1, 3);
commit;

蚂蚁大战大象
   大象被打趴下了
      蚂蚁也不好过
      瞎说
         没有瞎说
      大象进医院了
         护士是蚂蚁
   怎么可能
         怎么不可能
         可能性是很大的
         
create or replace procedure p (v_pid article.pid%type, v_level binary_integer) is
  cursor c is select * from article where pid = v_pid;
  v_preStr varchar2(1024) := '';
begin
  for i in 1..v_level loop
    v_preStr := v_preStr || '****';
  end loop;
  for v_article in c loop
    dbms_output.put_line(v_preStr || v_article.cont);
  if (v_article.isleaf = 0)
then
    p (v_article.id, v_level + 1);
  end if;
  end loop;
end;

~~~