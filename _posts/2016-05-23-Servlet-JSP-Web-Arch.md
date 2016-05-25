---
layout: post
Date: 2016-05-24
title: JSP
excerpt: "JSP的6个重要元素。如何写JSP"
java: true
comments: true
tags: [Java, Servlet&JSP]
---

# 总览
1.1 为下列元素写 JSP 代码：  

* 文本格式 
* scripting elements
* standard and custom actions
* expression language elements

1.2 JSP 中的 directives，包括：  

* page
* include
* taglib

1.3 书写 JSP 文档（基于 XML）  

1.4 JSP 页面的生命周期：  

* JSP 页面的翻译
* JSP 页面的编译
* 载入类
* 创建实例
* 调用 jspInit 方法
* 调用 jspService 方法
* 调用 jspDestroy 方法

1.5 JSP 中的内置对象：

* 请求
* 回应
* 输出
* 会话
* 部署
* 应用
* 页面
* 页面内容
* 异常

1.6 使用 DD 设置如下参数：

* 声明一个或多个 tag 库
* 是否使 EL（expression language）无效
* 是否使 scripting 无效


1.7 调用一个来自外部的资源：include  

六个问题：  

> JSP 文件每个部分最终在 servlet 源码中是什么样子的？  > Do you have access to the “servletness” of your JSP page? For example, does a JSP have a concept of a ServletConfig or ServletContext?  > 你能把什么类型的元素放进一个 JSP 中？  > JSP 中不同元素的语法是什么？  > JSP 的生命周期是什么？从翻译到死亡。  > How do the different elements in a JSP interact in the final servlet?  

---

# 1. JSP 中的元素

> Scriptlet:   <% %>  
> Directive:   <%@ %>  
> Expression:  <%= %>  
> Java expressions  
> EL expressions  
> actions  

过程：JSP (translated) -> MyJSP_jsp.java (compiles to) -> MyJSP_jsp.class (is loaded and init as ) -> MyJSP_jsp(**Servlet**)  

***引子：***  
制作一个显示被访问次数的JSP。  
*注意使用 <% ... %> 来写入 Java 代码：（既scripting）*

~~~html
<html><body>The page count is: 
<%out.println(Counter.getCount()); 
%></body></html>
~~~

~~~java
package foo;public class Counter {	private static int count;	public static synchronized int getCount() {		count++;		return count; 
	}}
~~~

上述代码是有问题的。JSP 无法找到 Counter！

解决方法：使用 page ***directive*** 来导入包。

## 1.1 Page ***directive*** 

> **directive** 是一种在翻译页面时向容器给出特殊指示的一种方式。  
> 包括三个种类：
> 
* page  
* include 
* taglib

导入一个包：

~~~html
<%@ page import="foo.*" %><html><body>The page count is: 
<%out.println(Counter.getCount()); 
%></body></html>
~~~

导入多个包：

~~~html
<%@ page import="foo.*, java.util.*" %>
~~~

***注意：带有<%@ 说明是directive***

## 1.2 expression

在 JSP 中可以用 expression 代替out.println()，比如：

~~~java
<% out.println(Counter.getCount()); %>
~~~

可以被如下形式代替：

~~~java
<%= Counter.getCount() %>
~~~

***注意：后面没有分号。%= 之间没有空格。  
想象 = 的含义是将后面的语句当做参数放入 out.println(); 中。***

## 1.3 declaration

~~~html
<html><body><% int count=0; %> 
The page count is now: 
<%= ++count %> 
</body></html>
~~~

在 scriptlet 中声明的变量都是 local variable。因为容器将其中所有的代码都塞进了一个类似 doGet/doPost(*service()*) 的方法里。  

所以我们需要某种办法使得 count 成为实例变量：使用 declaration:!

	<%! int count=0; %>
	
既可以用于变量，也可以用于方法。所有在<%!...%>之间的都被放在类中，而不是 service method。也就是说可以声明静态成员。

<figure>
	<a href="http://breakdimbo.github.io/images/JSP-Declaration.png"><img src="http://breakdimbo.github.io/images/JSP-Declaration.png"></a>
</figure>

---

# 2. JSP implicit object

API | Implicit Object
:-- | :--------------
JspWriter | out
HttpServletRequest|request
HttpServletResponse|response
HttpSession|session
ServletContext|application
ServletConfig|config
Throwable|exception（只能用于“error page”）
PageContext|pageContext(封装了其他 implicit 对象)
Object|page

*小练习：*

~~~html
<%@ page import="java.util.*" %>
<html><body>

The friend who share your hobby of

<%
request.getParameter("hobby");
%>

are<br>

<%
ArrayList al = (ArrayList) request.getAttribute("names");
%>
<%
Iterator it = al.iterator();
while(it.hasNext()){ %>
	<%= it.next() %>
<br>	
<% } %>
<html><body>
~~~

*关于注释：  
<!-- HTML comment -->  
<%-- JSP comment --%>*

---

# 3. JSP 的生命周期

* jspInit()：可以 override。调用自 init()
* jspDestroy()：同上
* _jspService()：不能 override。调用自 service()

>javax.servlet.jsp.JspPage  
>javax.servlet.jsp.HttpJspPage

*下划线意味着不能 override*

## jspInit()

配置 JSP 的初始化参数：与 servlet 类似，唯一的区别就是需要在 xml 里的<servlet>项下添加**\<jsp-file>**

~~~xml
<web-app ...> 
	<servlet>		<servlet-name>MyTestInit</servlet-name>		<jsp-file>/TestInit.jsp</jsp-file>		<init-param>			<param-name>email</param-name>
			<param-value>ikickedbutt@wickedlysmart.com</param-value>		</init-param> 
	</servlet> 
	<servlet-mapping>		<servlet-name>MyTestInit</servlet-name>		<url-pattern>/TestInif.jsp</url-pattern> 
	</servlet-mapping></web-app>
~~~

重写 jspInit()：

~~~java
<%!public void jspInit() {	ServletConfig sConfig = getServletConfig();	String emailAddr = sConfig.getInitParameter(“email”);	ServletContext ctx = getServletContext();	ctx.setAttribute(“mail”, emailAddr); 
}%>

~~~

---

## 4. JSP 中的属性

属性的获得，如下表对应所示：

   |In a Servlet|In a JSP(使用内置对象)
---|------------|-------------------------------
Application|**getServletContext**.setAttribute();|**application**.setAttribute();
Request|**request**.setAttribute();|**request**.setAttribute();
Session|**request.getSession()**.setAttribute();|**session**.setAttribute();
Page|Does note apply!|**pageContext**.setAttribute();


### 使用 PageContext 获取其他所有的属性。

其会取一个 int 参数作为判别使用的是哪个 scope 的属性。如下为 PageContext API：

<figure>
	<a href="http://breakdimbo.github.io/images/PageContext.png"><img src="http://breakdimbo.github.io/images/PageContext.png"></a>
</figure>

**使用范例：**

* 设置一个 page-scoped 属性：

~~~java
<% Float one = new Float(42.5); %><% pageContext.setAttribute(“foo”, one); %>
~~~

* 获取一个 page-scoped 属性：

~~~java
<%= pageContext.getAttribute(“foo”); %>
~~~

* 使用 pageContext 设置一个 session-scoped 属性：

~~~java
<% Float two = new Float(22.4); %>
<% pageContext.setAttribute("foo", two, PageContext.SESSION_SCOPE); %>
~~~

* 使用 pageContext 获取一个 session-scoped 属性：

~~~java
<%= pageContext.getAttribute(“foo”, PageContext.SESSION_SCOPE) %> 
// (Which is identical to: <%= session.getAttribute(“foo”) %> )
~~~

* 使用 pageContext 获取一个 application-scoped 属性：

~~~java
Email is:
<%= pageContext.getAttribute(“mail”, PageContext.APPLICATION_SCOPE) %> 

// Within a JSP, the code above is identical to:
Email is:
<%= application.getAttribute(“mail”) %>
~~~

* 当你不知道一个属性属于哪个 scope 时，使用 pageContext 的 find 进行查找：

~~~java
<%= pageContext.findAttribute(“foo”) %>
~~~

*查找顺序是： pageContext->request->session->application*

---

# 5. 三个 directives

* page directive  
定义了特定页面的*属性*。  
比如字符编码，这个页面返回的内容类型（Content Type）这个页面是否应该有内置 session 对象，导入包等等多达13个属性。

> HD p349

~~~java
<%@ page import="foo.*" session="false" %>
~~~

属性|简介
---|---
import|Defines the Java import statements that’ll be added to the generated servlet class. You get some imports for free (by default): java.lang (duh), javax.servlet, javax.servlet.http, and javax.servlet.jsp.
isThreadSage|Defines whether the generated servlet needs to implement the SingleThreadModel, which, as you know, is a Spectacularly Bad Thing. The default value is...”true”, which means, “My app is thread safe, so I do NOT need to implement SingleThreadModel, which I know is inherently evil.” The only reason to speci y this attribute is if you need to set the attribute value to “false”, which means that you want the generated servlet to use the SingleThreadModel, but you never will.
contentType|Defines the MIME type (and optional character encoding) for the JSP response. You know the default.
isELgnored|Defines whether EL expressions are ignored when this page is translated. We haven’t talked about EL yet; that’s coming in the next chapter. For now, just know that you might choose to ignore EL syntax in your page, and this is one of the two ways you can tell the Container.
isErrorPage|Defines whether the current page represents another JSP’s error page. The default value is “false”, but if it’s true, the page has access to the implicit exception object (which is a reference to the offending Throwable). If false, the implicit exception object is not available to the JSP.
errorPage|De nes a URL to the resource to which uncaught Throwables should be sent. If you de ne a JSP here, then that JSP will have an isErrorPage=”true” attribute in its page directive.

*其他：language; extends; session; buffer; autoFlush; info; pageEncoding.*


* taglib directive
定义了 JSP 可利用的 tag 库。

~~~java
<%@ taglib tagdir=”/WEB-INF/tags/cool” pre x=”cool” %>
~~~

* include directive
定义了在"翻译"时需要被添加进当前页面中的文本和代码。可以让你重复利用某些功能，比如标准页面标题，导航栏，等等。

~~~java
<%@ include  le=”wickedHeader.html” %>
~~~

---

# 6. 在 JSP 中去 java 化－EL

> EL - expression language.  
> 将 java 代码写在某处，使用EL调用它。

## 6.1 初探：

EL expression：  
Please contact: ${applicationScope.mail}

等价于：  
Java expression：  
Please contact: <%= application.getAttribute(“mail”) %>

> EL 的格式一般是 ${ }.

## 6.2 使 JSP 中的 scripting 无效

在 DD 中使用<scripting-invalid>使 JSP 中的 scripting 无效

~~~xml
<web-app ...> 
	...		<jsp-config>			<jsp-property-group> 
			<url-pattern>*.jsp</url-pattern> 
			<scripting-invalid>
				true 
			</scripting-invalid>
			</jsp-property-group> 
		</jsp-config>	... 
</web-app>~~~

## 6.3 忽略 Expression Language

通过 DD 或者 page directive 忽略 EL。既将EL当做正常文本显示。

使用 DD（一般用于全局）：

~~~xml
<web-app ...> 
	...	<jsp-config>		<jsp-property-group> 
		<url-pattern>*.jsp</url-pattern> 
		<el-ignored>
			true 
		</el-ignored>
		</jsp-property-group> 
	</jsp-config>
	... 
</web-app>
~~~

使用 page directive（一般用于特定页面。强于 DD 设置）：

~~~html
<%@ page isELIgnored="true" %>
~~~

---

# 7 Action

标准形式：

	<jsp:include page=”wickedFooter.jsp” />
	
非标准形式：

	<c:set var=”rate” value=”32” />





