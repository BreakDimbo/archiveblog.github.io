---
layout: post
date: 2016-05-26
title: 自定义标签
excerpt: 
java: true
comments: true
tag: [Java, JSP&Servlet]
---

# 目标

#### 使用 pageContext API，写一个可以获取 JSP 内置变量和 web application 属性的 tag handler

#### 使用 tag handler 获取 parent tag 和一个 arbitary tag ancestor

#### 了解简单 custom tag 事件模型，事件方法（doTag（））

#### 熟悉 Tag File 模型，了解  web application structure for tag Files; write a tag  le; and explain the constraints on the JSP content in the body of the tag.

---

# 1. Tag Files

可以用来创建 custom tags。但是首先让我们看看它是如何取代 \<jsp:include> 和 \<c:import> 的。

最简单的三部曲：

1. 找到你需要引入的页面，比如（Header.jsp），将扩展名改为.tag

2. 将这个文件放在 WEB-INF 下的 *tags* 目录下

3. 在你的目标页面中加入 taglib directive（with a tagdir 属性），然后唤醒它~

~~~html
<%@ taglib pre x=”myTags” tagdir=”/WEB-INF/tags” %><html><body>
<myTags:Header/>
Welcome to our site. 
</body></html>
~~~ 

## 1.1 如何把参数传给 Header 呢？*attribute directive*

还记得如果使用<jsp:include>时，传递的是 request 的参数：param。但是，使用 Tag Files，你只需要传递 tag attribute。

**Invoking the tag from the JSP**  

*Before (using <jsp:param> to set a request parameter)*

~~~html
<jsp:include page=”Header.jsp”>
<jsp:param name=”subTitle” value=”We take the sting out of SOAP.” />
</jsp:include>
~~~

*After (using a Tag with an attribute)*

~~~html
<myTags:Header subTitle=”We take the String out of SOAP” />
~~~

**Using the attribute in the Tag File**

*Before (using a request param value)*

~~~html
<em><strong>${param.subTitle}</strong></em>
~~~

*After (using a Tag File attribute)(Header.tag)*

~~~html
<em><strong>${subTitle}</strong></em> <br>
~~~

> 有一点需要明确：\<jsp:include> \<jsp:param> 使用的是 request parameter，就是意味着这个参数是在 request scope 下的，任何与这个 request 有关的家伙都能获取这个参数。但是！Tag Files 的 scope 仅限于 tag 本身。


## 1.2 Tag File 使用 attribute directive

这个 directive 是专为 Tag File 使用的。

在 Tag File 中：

~~~html
<%@ attribute name=”subTitle” required=”true” rtexprvalue=”true" %>
<img src=”images/Web-Services.jpg” > <br>
<em><strong>${subTitle}</strong></em> <br>
~~~

在使用那个 tag 的 JSP 中：

~~~html
<%@ taglib pre x=”myTags” tagdir=”/WEB-INF/tags” %> <html><body><myTags:Header subTitle=”We take the String out of SOAP” /><br>Contact us at: ${initParam.mainEmail} </body></html>
~~~

## 1.3 当你的 tag 的属性值太大时使用 <jsp:doBody>

可以将其放进 tag 的 body里，而不需要使用 attribute value。

在 Tag File：

~~~html
<img src=”images/Web-Services.jpg” > <br> <em><strong><jsp:doBody/></strong></em> <br>
~~~

在使用那个 tag 的 JSP 中： 

~~~html
<%@ taglib pre x=”myTags” tagdir=”/WEB-INF/tags” %> <html><body><myTags:Header>We take the sting out of SOAP. OK, so it’s not Jini,<br> but we’ll help you get through it with the least<br> frustration and hair loss.</myTags:Header><br>Contact us at: ${initParam.mainEmail} </body></html>
~~~

> 但是如何声明一个 body 的内容的属性呢？

## 1.4 为 Tag File 声明一个 body-content: tag directive

所谓的 tag directive 类似于 JSP 中的 page directive，它的大部分属性类似于 page directive，除了一个特别的地方：**body-content。**

body-content 的默认值是 scriptless，既你不能在里面添加 scripting 的元素（<%..%>, <%=..%>, <%!..%>）

body-content 的另外两个值是 empty（不允许在 tag body 里添加任何东西），tagdependent（将 body 里面的东西都当做为 plain text）。

> evaluated：生效。

举个例子：

在 Tag File（Header.tag）里的情况：

~~~html<%@ attribute name=”fontColor” required=”true” %>
<%@ tag body-content=”tagdependent” %>
<img src=”images/Web-Services.jpg” > <br><em><strong><font color=”${fontColor}”><jsp:doBody/></font></strong></em> <br>
~~~

在 JSP 中的傻样：

~~~html
<%@ taglib pre x=”myTags” tagdir=”/WEB-INF/tags” %> <html><myTags:Header fontColor=”#660099”>We take the sting out of SOAP. OK, so it’s not Jini,<br> but we’ll help you get through it with the least<br> frustration and hair loss.</myTags:Header><br>Contact us at: ${initParam.mainEmail} </body></html>
~~~

## 1.5 容器去哪找 Tag FIles 呢？

> 如果一个 Tag File 是被放在一个 Jar 中的话，那么它必须要有一个 TLD 来部署它。如果是直接放在 web app 中的话，那就不需要了。

### 四个地方：

1. 直接在 WEB-INF/tags

2. 在 WEB-INF/tags 的下一级目录里

3. 在 JAR 里的 META-INF/tags （JAR 位于 WEB-INF/lib）

4. 在 META-INF/tags 的下一级目录里

5. **如果一个 Tag File 是被放在一个 Jar 中的话，那么它必须要有一个 TLD 来部署它**

> 目录结构: HD p543

## 1.6 关于 Tag File 的一些蠢问题

### Tag File 是否可以获取 request 和 response 内置对象？ 
是的，基本都可以。除了Servlet Context，Tag File 使用 JspContext。


# 2. Tag Handler

类似于 EL function，但是比他更强大，因为 EL function 只能使用静态方法，但是 tag handler：

> a tag handler class has access to tag attributes, the tag body, and even the page context so it can get scoped attributes and the request and response.

Tag Handler 有两种：传统的和简单的（Classic&simple）。

## 2.1 制作一个 Simple tag handler

1. 写一个 class extends SimpleTagHandler：

~~~java
package foo;import javax.servlet.jsp.tagext.SimpleTagSupport; // more imports neededpublic class SimpleTagTest1 extends SimpleTagSupport { // tag handler code here}
~~~

2. 重写 doTag() 方法:

~~~java
public void doTag() throws JspException, IOException { getJspContext().getOut().print(“This is the lamest use of a custom tag”);}
~~~

3. 为这个 tag 创建一个 TLD：

~~~xml
<taglib ...> <tlib-version>1.2</tlib-version> <uri>simpleTags</uri><tag><description>worst use of a custom tag</description> <name>simple1</name> <tag-class>foo.SimpleTagTest1</tag-class> <body-content>empty</body-content>
</tag></taglib>
~~~

4. 部署 tag handler 和 TLD：  
将 TLD 放进 WEB-INF，将 tag handler 放进 WEB-INF classes。

5. 最后写一个 JSP 使用它

~~~html
<%@ taglib pre x=”myTags” uri=”simpleTags” %>
<html><body><myTags:simple1/></body></html>
~~~

> 如果要使用一个有 body 的 tag 的话，需要修改三个地方：JSP， doTag() 中添加 getJspBody().invoke(null)，TLD 中设置 <body-content>。


### Simple Tag API

<figure>
	<a href="http://breakdimbo.github.io/images/SimpleTag-API.png"><img src="http://breakdimbo.github.io/images/SimpleTag-API.png"></a>
</figure>

### LifeCircle

<figure>
	<a href="http://breakdimbo.github.io/images/LifeOfSimpleTagHandler.png"><img src="http://breakdimbo.github.io/images/LifeOfSimpleTagHandler.png"></a>
</figure>

## 2.2 如果 tag body 「动态」里使用了一个 expression for an attribute？

也就是说你的 tag body 是依赖于 tag handler 来设置
属性的。

一个小例子：

在 JSP 中调用 tag：

~~~html:
<myTags:simple3>Message is: ${message}</myTags:simple3>
~~~

在 tag handler 中：

~~~java
public void doTag() throws JspException, IOException {
	getJspContext().setAttribute(“message”, “Wear sunscreen.”);
	getJspBody().invoke(null);
}
~~~

## 2.3 一个拥有动态数据表的 tag：iterating body

JSP：

~~~html
<table> <myTags:simple4><tr><td>${movie}</td></tr> </myTags:simple4></table>
~~~

tag Handler:

~~~html
String[] movies = {“Monsoon Wedding”, “Saved!”, “Fahrenheit 9/11”};public void doTag() throws JspException, IOException { for(int i = 0; i < movies.length; i++) {getJspContext().setAttribute(“movie”, movies[i]);getJspBody().invoke(null); }}
~~~

## 2.4 一个带有属性的 Simple tag

