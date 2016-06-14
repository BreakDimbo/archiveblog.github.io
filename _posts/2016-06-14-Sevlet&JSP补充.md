---
layout: post
date: 2016-06-14
title: Sevlet&JSP 补充
excerpt: 
java: true
comments: true
tag: [Java, JSP&Servlet]
---

# Sevlet&JSP

404 页面找不到
403 禁止访问
500 服务器内部错误

## Web Application

复合j2ee web application的标准——就是目录

必须有的：
WEB-INF 
文件：	web.xml
目录：	lib
		classes
		
静态页面，放到WEB-INF下面。注意访问路径。

## servlet 广义上跑在服务器的范围内

我们需要继承 Servlet —— GenericServlet —— HttpServlet

### Servlet 的生命周期

加载：classloader
实例化：new
初始化：init（ServletConfig）
处理请求：service doGet／doPost
最后：Destory

ServletConfig——保存了这个Servlet的配置信息

使用doGet 和 doPost 处理请求

response.setContextype("text/html");

trick: 在doPost里调用doGet（）


## lesson 8 cookies

http 无连接性。

解决办法：

1. 服务器可以向客户端写内容
2. 只能是文本内容
3. 客户端可以阻止服务器写入
4. 只能拿自己的webapp写入的东西
5. 每个浏览器有一个独一无二的编号

所谓cookie：服务器端写到客户端的文本信息：名值对（key／value）＋一个独一无二的对应浏览器的编码

两种cookie：

1. 一种存在硬盘上
2. 一种存在内存里（存活时间为－1）——只属于这个窗口或者其子窗口

~~~java
//写cookie
Cookie cookie = new Cookie("","");
response.setMaxAge(3600); // 设置存活周期
response.addCookie(cookie);

//拿到cookie
Cookie[] cookies = request.getCookies();
if(cookie != null) {
	Cookie cookie;
	for(int i=0; i<cookies.length; i++) {
		cookie = cookies[i];
		String s1 = cookie.getName();
		String s2 = cookie.getValue();
	}
}

~~~

## lesson 10 关于cookie的两个实验

一个servlet/jsp设置的cookie能够被同一个路径下面的或者是子路径下面的servlet/jsp都读到。父路径的无法读到。——路径值URL（xml里配置的路径！＝真实文件路径）

## lesson 11 Session 会话跟踪

1. 与Cookie最大的区别——Session记录在服务器端
2. Session（服务器在内存中开辟空间创建session）与浏览器的窗口或者其子窗口关联（其他浏览器或者另一个浏览器的窗口不行）
3. Session 如何识别浏览器——浏览器第一次访问时，为浏览器窗口设置一个独一无二的号码（session id）并映射给对应的session
4. **session ID 的两种实现方式：1）通过cookie。2）通过URL重写实现**

规则：

1. 如果浏览器支持cookie，创建session时会将session的id放到cookie里(存在内存里)
2. 如果浏览器不支持cookie，必须自己编程，使用URL重写的方式实现：
	1. response.encodeURL()
	   1. 转码
	   2. URL后面加入session id。
3. Session 不像Cookie拥有路径的问题，同一个Application下的servlet／jsp可以共享同一个session，前提是同一个客户端窗口。（lesson 14）

## lesson 13 Session

~~~java
HttpSession mySession = request.getSession(true); // true：没有就创建，有就用原来的

mySession.isNew();
mySession.getId();
mySeesion.geCreationTime();
mySession.getLastAccessedTime(); //根据最后访问时间确认session何时结束。 注意session过期问题－可以在web.xml下配置

request.getRequestedSessionId();
request.isRequestedSessionIdFromCookie();
request.isRequestedSessionIdFromURL();
request.isRequestedSessionIdValid();

//向session里放／获取值
Integer accessCount = (Integer) session.getAttribute("accessCount");

//Session 里的名字永远是String，value－Object
session.setAttribute("accessCount", accessCount);

//session.getCreationTime 返回long
new Date(session.getCreationTime());
~~~

## lesson 15 application

用于保存整个webapp生命周期内所有部分（客户端）都可以访问的数据。—— ServletContext。

如何拿到ServletContext：

~~~java
	ServlectContext context = this.getServletContext();
	context.getAttribute(String name, Object value);
	context.setArribute(String name, Object value)
~~~

放在包里的servlet，记得将servlet-class的名字加上包名。

## lesson 18 数据库的处理 JavaBean（咖啡豆）

组键：一系列的类综合在一起，共同提供服务。 

javabean：一个普通的的java类。

1. 属性名第一个字母必须小写，并且是private
2. 属性具有getter 和 setter 方法


例子：

1. 直接连接数据库／
2. 在servlet中使用javabean连接数据库。 
	
## lesson 19 JSP

JSP 就是一个servlet。
直接在html中内嵌jsp代码，无需配置。
不能放置在WEB-INF目录下。

## lesson 20 JSP 语法

Declaration
Scriptlet
Exception
Comment
Action 指令
内置对象

Declaration：

~~~html
//！ 声明成员变量
<%!
	int accessCount = 0;
%>
//声明局部变量
<%
	int accessCount2 = 0;
%>

<%= ++accessCount %>
~~~

Scriptlet

~~~html
<%
	// java 命令
%>

request jsp 可以直接拿来用。
session对应于：httpSession
~~~

## Lesson 20 Directive 编译指令

**注意：相当于在编译期间得指令**

	<%@Directive 属性＝“属性值%>
	
常见的有：
page
include
taglib

page：指明与JSP Container的沟通方式（指明页面特点）

	<%@page import="java.util.*, ...%> - 用于引入包
	<%@errorPage="errorPageUrl"%>
	<%@isErrorPage="true"%>
	<%@contentType=".."%>

### lesson 21 include

	<%@include file="fileURL"%>
	限制：不能向fileURL中传递参数。
将file先放进来，再编译，再执行。—— 编译期间包含。编译指令里不能传参数。该指令用于包含静态页面

## lesson 22 Action —— 动作指令

运行期间指令：

jsp:useBean
	jsp:setProperty
	jsp:getProperty

jsp:include
jsp:forward
	jsp:param
	
### jsp:include
用于动态包含jsp程序或者html文件
除非这个指令被执行到，都则不会被编译

格式，没有％

	<jsp:include page="URLSpec"/>
	<jsp:include page"URLSepc>
		<jsp:param name="ParamName" value="paramValue"/>

注意<jsp:include page=""/> 与 <%include file=""%> 的区别

### jsp:forward/jsp:param-跳转

用于将一个jsp的内容传送到page所指定的jsp程序或者servlet中处理（URL）

格式：

	<jsp:forward page=""/>
	<jsp:forward page="">
		<jsp:param name="paramName" value=""/>
		
	// 另一种跳转
	response.sendRedirect("URL")

两者的区别需要记住：lesson 24
	
	<jsp:forward> 跳转发生在服务器端，地址栏仍然是原来的页面地址，request对象是同一个request。
	response.sendRedirect 是不同的 request。相当于发起了两次请求。 
	
### lesson 25 jsp:useBean

通过jsp:useBean, 可以在JSP中使用定义好的Bean

1. 属性名第一个字母必须小写，并且是private
2. 属性具有getter 和 setter 方法
3. 有一个无參构造器

基本用法：
	
	//这句话的意思是，new一个对象，名字是id，类是class。相当于 <% CounterBean beanName = new CounterBean();%>
	
	<jsp:useBean id="beanName" scope="page|request|session|application" class="CounterBean" type=" "/>
	
	<jsp:useBean...>
		<jsp:setProperty..> // 调用set方法
		<jsp:getProperty..> // 调用get方法
	</jsp:useBean>

注意：在jsp中使用bean，不要使用裸体类，要放在包里。
scope 指在哪个个范围内有效。 type指把它当作那个类。

### lesson 26 scope

### leason 27 jsp:set/getProperty

	<jsp:useBean id="entry" class="bean.SaleEntry">
		<jsp:setProperty
			name="entry"
			proptery="numItems"
			param="numItes"/> // 自动转换类型
		<jsp:getProperty..>
	</jsp:useBean>
	
		<jsp:setProperty name="hello" proptery="*"> // 前一个页面将request传给他，但是需要属性name需要与bean里的属性name一一对应（用的不多）

JSP 调用getter和setter方式使用了反射的机制。

### lesson 28 characterEncoding

	request.setCharacterEncodeing(“GBK”)
	
	new String(name.getBytes("ISO85588_1"), "GBK");
	
当页面显示乱码时，一定要分析清楚显示的内容经过了哪些环节。

### lesson 29 JSP的9个内置对象

out  
request  
response  
pageContext - 不常用  
session  
application  
config － web.xml （不常用）  
exception  
page - 不常用  

**写出JSP常用的内置对象以及常用的方法**

out:  
out.println() 常用  
out.newLine() 常用  
out.close()  
out.flush()  
out.clearBuffer()  
out.clear()  
out.getBuffersize()  

request：  
getMethod()  
getParameter(String name)    
getParameterNames()
getParameterValues(String name)
getRequestURI()



