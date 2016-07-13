---
layout: post
date: 2016-06-14
title: 设计模式
excerpt: 
java: true
comments: true
tag: [Java]
---

# 设计模式

## 1.责任链
理解 filter 和 tomcat

真实案例：在网上发表信息——需要检查信息

扩展 Myprocess。动态指定过滤规则->创建一个 Filter interface->创建其他的 Filter class 实现这个接口->将 Filter 放进数组里面构成过滤链，对数组迭代-> 依次执行。

好处：

1. 可以给过滤规则排序，
2. 灵活添加其他规则
3. 灵活删除其他规则
4. 利用配置文件随便配置


如果已经有其他链条存在，如何进行合并。——

新建另一个类FilterChain，装新的过滤器们，将这个类当做一个整个的 Filter。控制其他所有的 Filter 的添加和执行操作。
Myprocessor 调用 FilterChain 的方法进行处理

再让 FilterChain 实现 Filter 接口。

增强过滤器的功能——如何实现同时处理进出的消息的过滤器。

新增两个类——封装请求和反馈

重新设计 Filter 接口内部的的 doFilter 方法。

利用递归调用 chain 的 doFilter。设置一个 index 索引记录当前经历的 filter 的位置，及时跳出递归。巧妙之处在于 index 的设置！

## 2 Iterater

ArrayList 和 LinkedList 底层实现不一样，所以遍历方式不同。

那么如何实现遍历方式的统一：使用Iterator接口，内部定义其他容器必须提供的方法。由不同的容器自己来实现自己不同的遍历方式。

## 3 动态代理

没有源码：

* 修改代码的时候
* 检测代码，日志记录

两种普通代理的实现方式：

1. 使用继承，重写父类方法
2. 实现接口，实现借口方法，调用 Tank t 的 move 方法（聚合：一个类里有另外一个对象）

聚合的好处：当需要多种代理组合使用时（功能叠加），可以灵活组合。在代理类里存储接口类型的对象。*重点在于：实现同一个接口*——静态代理

### 3.1 动态代理

增加一个stop()方法，如何在不增加重复代码的前提下，在stop() 方法里也计算下该方法的运行时间。

如何实现将 TankTimeProxy 变换成 TimeProxy——即可以计算任意一个对象中方法的运行时间。

不需要代理类的名字，直接生成一个代理，直接使用。

在 newProxyInstance 里，保存一段字符串（内部是一段生成对象的代码）—— 如果可以做到动态生成这段代码的话，就可以动态生成所需的代理对象。

那么，如何动态编译这段代码。

* 使用JDK6 的Compiler API
* CGLib
* ASM

如何使用JDK6 的Compiler API（lesson－03）来编译字符串里的代码从而动态生成.class文件。

lesson 04

然后 load 到内存，并生成新对象。
使用 普通的ClassLoader只能将bin 文件里的class文件载入内存。
我们使用 URLClassLoader——去指定的地址寻找class

lesson 05

开始生成对象：c.getConstructor 拿到构造方法。

lesson 6

如何产生实现任意接口的动态代理——把接口传给 newProxyInstance（Class infce)

实现方法的动态生成 

但是方法里的逻辑是固定的。如何动态制定方法内容。增加一个参数表示使用者指定的方法的执行过程。 

lesson 7

在代理中源码如何调用所需方法。

lesson 9 AOP aspect oriented programing

不用修改原来的实现的代码，就能在原来的代码的基础之上插入新的逻辑。

逻辑可插拔。
逻辑可叠加。

### 3.2 复习


## 4. 工厂模式