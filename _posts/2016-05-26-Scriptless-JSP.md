---
layout: post
title: Scriptless JSP
date: 2016-05-26
excerpt: "Expression language; <jsp:useBean, getProperty, setProperty>; <jsp:include, forward, param>"
java: true
comments: true
tag: [java, Servlet&JSP]
---

# 目标

* 熟悉 EL 中的内置变量:  
pageScope   
requestScope   
sessionScope   
applicationScope   
param and paramValues   
header and headerValues   
cookies   
initParam

* 熟悉 EL 中的操作符:  
property access(the . operator)   
collection access(the [] operator)   
aritmetic operators   
relational operators   
logical operator

* 熟悉 EL 中的函数：  
创建一个 TLD 文件结构，用于声明一个 EL 函数  
使用代码声明一个 EL 函数：*public static*

* 学会使用标准 Action：  
jsp:useBean(with attributes: 'id', 'scope', 'type', 'class')   
jsp:getProperty   
jsp:setProperty(with all attribute combinations)('name', 'property', 'value/param/nothing')  
jsp:include  
jsp:forward  
jsp:param

* 学会导入位于另一个页面的 JSP 片段：  
include directive  
\<jsp:include/> action

---


# 1. \<jsp:useBean/> 以及相关操作的初步使用 

> 声明一个 bean 对象  
> 给 bean 对象设置 property 的值  
> 获取 bean 对象中的 property 的值  
> 创建 bean 对象  
> bean 多态的使用

## 1.1 如果需要传递一个非 String 类型的属性给 JSP 而不使用 scriptlet，需要怎么操作？

例如：  
使用 scripting 来进行输出：

~~~html
<html><body><% foo.Person p = (foo.Person) request.getAttribute("person"); %> 
Person is: <%= p.getName() %></body></html>
~~~

使用标准 action 来进行输出：

~~~html
<html><body><jsp:useBean id="person" class="foo.Person" scope="request" />Person created by servlet: <jsp:getProperty name="person" property="name" /> 
</body></html>
~~~

对上述代码的解释，由于 Person 是一个 JavaBean，所以使用 bean-realted 标准 action。

> JavaBean  
JavaBean 是公共 Java 类，但是为了编辑工具识别，需要满足至少三个条件：  

> * 有一个 public 默认构造器（例如无参构造器,）
> * 属性使用 public 的 get, set 方法访问，也就是说设置成 private，同时 get, set方法与属性名的大小也需要对应。例如属性 name, get 方法就要写成：
> 
~~~java
public String getName(){}
~~~
> * 需要序列化。这个是框架，工具跨平台反映状态必须的
> * 另外，如果需要使用JSPs 中的标准 action，property 的类型必须是 String 或者 primitive。


其中包含两个重要的 action：  

\<jsp:useBean/> - 用于声明并初始化一个 bean 属性：

	<jsp:useBean id="person" class="foo.Person" scope="request" />

id 类似 Attribute 里的 name。*（request.getAttribute("person");）*
	
\<jsp:getProperty/>  - 获取 bean 属性的 property value：

	<jsp:getProperty name="person" property="name" /> 

name="person"：明确实际的 bean 对象，与 id 的值相对应。
property="name"：明确需要从 Person 获取的值的变量名。

## 1.2 \<jsp:useBean/>也能创建一个 bean

如果\<jsp:useBean/>没有找到 id 指定的对象，它会自动新建一个。

\_jspService() 中实现代码如下：

~~~java
foo.Person person = null; 
synchronized (request) {	person = (foo.Person)\_jspx_page_context.getAttribute("person", PageContext.REQUEST_SCOPE);
	if (person == null){
		person = new foo.Person();
		\_jspx_page_context.setAttribute("person", person, PageContext.REQUEST_SCOPE); 
	}}
~~~

## 1.3 使用\<jsp:setProperty/>设置 attribute 的 property

~~~html
<jsp:useBean id="person" class="foo.Person" scope="request" /> 
<jsp:setProperty name="person" property="name" value="Fred" />
~~~

## 1.4 \<jsp:useBean/>可以有一个 body

~~~html
<jsp:useBean id="person" class="foo.Person" scope="page" >
	<jsp:setProperty name="person" property="name" value="Fred" />
</jsp:useBean >
~~~

**注意：第一行最后没有 /。第二行相当于放在一个条件语句中，既当 id 对应的对象不存在时，执行第二行的代码。**

## 1.5 \<jsp:useBean/>多态的使用

> HD p389

因为 class	＝"foo.Person" 中，foo.Person 既是引用名，也是类名。所以如果将 foo.Person 变为抽象类，并创建一个 Employee 类继承它，那么\<jsp:javaBean/>便无法正常的创建对象了。所以需要使用 type 参数来重新定义引用名（type 下可以是类，抽象类，接口），class 参数是类名：

~~~html
<jsp:useBean id="person" type="foo.Person" class="foo.Employee" scope="page">
~~~

**foo.Person** person = null;// code to get the person attribute   
if (person == null){  	person = new **foo.Employee()**;
}

**注意：所以 class 对应的参数必须是 type 对应的参数的子类或者实现**  

**如果使用 type 而不标明 class，则 bean 一定已经存在**  
**如果使用 class 而不使用 type，则 class 一定不能是抽象类，而且必须有一个无参数构造器。**

***再次强调：  
type == reference type  
class == object type  
type 是你要声明什么  
class 是你要实例化什么***


> Note: *\<jsp:useBean/>和\<jsp:getProperty/>默认的 scope 是 page*


## 1.6 如何直接从 request 到 JSP 而不经过 servlet？

比如有一个 form.html 如下：

~~~html
<html><body><form action="TestBean.jsp">	name: <input type="text" name="userName"> 
	ID#: <input type="text" name="userID"> 
	<input type="submit"></form> 
</body></html>
~~~

常规获取参数的方法是：

~~~html
<jsp:useBean id="person" type="foo.Person" class="foo.Employee">
	<jsp:setProperty name="person" property="name"
value="<%= request.getParameter("userName") %>" />
</jsp:useBean>
~~~

\<jsp:setProperty/>的 **param** 属性可以让你用一个来自 request 的参数设置一个 bean 的 property。只需要 *令 param 的值等于所要获取的参数名字即可。*

~~~html
<jsp:useBean id="person" type="foo.Person" class="foo.Employee"> 
	<jsp:setProperty name="person" property="name" param="userName" />
</jsp:useBean>
~~~

**Get Better**  
此外，如果在 form 中，将 name ="userName" 改成 name ="name" 既与 property 指定的名称相同，则可以省略 param。

> 所谓的 property name 既为 getter 和 setter 方法「对应」的名字：get/set 后面的名称，首字母小写所得到的，与你在类中设置的变量名无关。

**Get Even Better**  
如果使所有的参数名与 bean 的 property names 相符的话。令 property 等于通配符：*  
则容器会自动寻找参数，并赋值给指定属性的对应 property。  
**注意：property 需要是 String 类或者 primitive**

~~~html
<jsp:useBean id="person" type="foo.Person" class="foo.Employee">
	<jsp:setProperty name="person" property="*" />
</jsp:useBean>
~~~

---

# 2. Expression Language

如果特性（property）不是 string 或者 primitive 呢？  
既如果所要使用的 attribute 是一个对象，而它的 property 也是一个对象（比如一个 dog，dog 内有 name），此时就不建议使用\<jsp:javaBean/>了。

如果你还使用\<jsp:javaBean/>，像这样：  

~~~html
<html><body><jsp:useBean id="person" class="foo.Person" scope="request" />Dog’s name is: <jsp:getProperty name="person" property="dog" /> 
</body></html>
~~~

则输出是：

	Dog’s name is: foo.Dog@799338

> 不是我们想要的结果，而你又**不能**这样设置：**property=“dog.name”**

**可以使用 Expression Language。**

~~~html
<html><body>Dog’s name is: ${person.dog.name}  </body></html>~~~

等价于：

~~~html
<%= ((foo.Person) request.getAttribute("person")).getDog().getName() %>
~~~

> ${**person**.dog.name}   
第一个部分是内置 map（映射）对象  
或者是在四个 scope（page, request, appliction, session）里的 attributes。


关于 EL 的简介如下图：

<figure>
	<a href="http://breakdimbo.github.io/images/Expression-Language.png"><img src="http://breakdimbo.github.io/images/Expression-Language.png"></a>
</figure>

## 2.1 使用 . 操作符*获取* property 和 Map key

> ${person.name}

如果表达式又一个变量跟着一个 dot，那么：

* 左边一定是个 Map类型(Java.util.Map)或者 bean
* 右边一定是个 Map key 或者 property。
* 如果左边是 bean，而右边对应的 property name 不存在的话，则会抛出异常

> Note: The pageContext implicit object is a bean—it has getter methods.   
> All other implicit objects are Maps.


## 2.2 使用 ［］操作符

当你使用 dot 操作符时，有很多限制。比如，左边必须是 Map 或者 bean，右边必须符合 java 命名规则(比如不能以数字开头，必须以字母，_，开头)。  
但是使用 ［］左边可以是一个 List 或者 array(of any type)，同时右边可以是数字或者任何与数字兼容的东东，或者一个不符合 java 命名规则的 identifier。

> ${musicList[“something”]}

* 左边可以是 Map，bean，array，List
* 右边可以是 Map key，property，index（1 或者 “1”）

> Music is: ${musicMap[Ambient]}  
**注意有引号和没有引号的区别：  
“” 是 Map key  
无引号是 Attribute name**



**如果没有引号，容器会寻找绑定在 Ambient 下的 Attribute。而不是将其当做 musicMap 的 Map key。**

~~~java
java.util.Map musicMap = new java.util.HashMap(); 
musicMap.put("Ambient","Zero 7"); 
musicMap.put("Surf","Tahiti 80"); 
musicMap.put("DJ","BT");musicMap.put("Indie","Frou Frou"); 
request.setAttribute("musicMap", musicMap);request.setAttribute("Genre","Ambient");
~~~

Work：

~~~html
Music is ${musicMap[Genre]} <!--evaluates to--> Music is ${musicMap["Ambient"]}
~~~
	
Doesn't work：

~~~html
Music is ${musicMap["Genre"]} <!--doesn’t change--> Music is ${musicMap["Genre"]}
~~~

可以嵌套：

**In a servlet**

~~~java
java.util.Map musicMap = new java.util.HashMap(); 
musicMap.put("Ambient", "Zero 7"); 
musicMap.put("Surf","Tahiti 80"); 
musicMap.put("DJ", "BT");musicMap.put("Indie", "Frou Frou"); 
request.setAttribute("musicMap", musicMap);String[] musicTypes = {"Ambient", "Surf", "DJ", "Indie"};
request.setAttribute("MusicType", musicTypes);
~~~

**In JSP**

~~~
Music is ${musicMap[MusicType[0]]} 
等价于： Music is ${musicMap["Ambient"]}
~~~

## 2.3 EL 提供了原生文本输入方式，包括HTML


## 2.4 在 EL 中获取 Request 的参数

> 直接使用 ${param.name} 获取。

一个例子：

~~~html
In the HTML form<form action="TestBean.jsp">	Name: <input type="text" name="name"> 
	ID#: <input type="text" name="empID">	First food: <input type="text" name="food"> 
	Second food: <input type="text" name="food"><input type="submit"> 
</form>
~~~

~~~jsp
Request param name is: ${param.name} <br>Request param empID is: ${param.empID} <br>Request param food is: ${param.food} <br>First food request param: ${paramValues.food[0]} <br> 
Second food request param: ${paramValues.food[1]} <br>Request param name: ${paramValues.name[0]}
~~~

## 2.5 获取其他信息

### 获取 header

~~~html
Host is: ${header["host"]} 
Host is: ${header.host}
~~~

### 获取 Http request method

**注意：requestScope 不是 request object**
**Scope 是用来装 Attribute 的(Map of the request scope attributes)**

使用 pageContext：

~~~html
Method is: ${pageContext.request.method}
~~~

### implicit scope object

注意不要将其与 implicit object 混淆。它们里面只装着 Attributes。

利用 implicit scope object 解决名字冲突问题。

如果一个人将 attribute 设置成下面这样：

	request.setAttribute("foo.person", p);

那么其在 EL 中的调用将出现问题：

	${foo.person.name}
	
容器会去寻找名字为 foo 的属性。

解决办法：implicit scope object + []

	${requestScope["foo.person"].name}
	
### 获取 Cookies 和 init params

**获取 cookies**  
在 scripting 中：

~~~jsp
<% Cookie[] cookies = request.getCookies();
	for (int i = 0; i < cookies.length; i++) {
		if ((cookies[i].getName()).equals("userName")) {
			out.println(cookies[i].getValue()); 
		}	} %>
~~~

而使用 EL：(*注意是cookie， 没有s*)

	${cookie.userName.value}
	
**获取 init params**

首先，在 DD 中部署（注意是 *Context* 的 init param）：

~~~xml
<context-param>	<param-name>mainEmail</param-name> 
	<param-value>likewecare@wickedlysmart.com</param-value></context-param>
~~~

在 scripting中，它是这样的：

	email is: <%= application.getInitParameter("mainEmail") %>
	
而在 EL 中：

	email is: ${initParam.mainEmail}
	

## 2.6 在 EL 中使用函数（类似java method）
	
可以使用 EL 调用 java 中的**静态函数**

写一个掷骰子的 JSP，有以下四个步骤：

* Write a Java class with a **public static** method.
* Write a **Tag Library Descriptor** (TLD) file.（*TDL 提供了一个在 「所要调用的用函数所在的 Java class 与 调用那个函数的 JSP」 之间的映射。这样就可以使用别的名字来调用函数了。*）
* Put a taglib directive in your JSP（*用来定义命名空间：前缀(相当于一个在该页面使用的别名，用来区别来自不同 TDL 的同名函数) + 函数名。*）.
* Use EL to invoke the function（*${prefix:name()}*）.**注意是冒号 ：**

<figure>
	<a href="http://breakdimbo.github.io/images/RollDice.png"><img src="http://breakdimbo.github.io/images/RollDice.png"></a>
</figure>

> KEY: URI: **whatever you decided to name the TLD**

在测试上面的程序时，最后运行时出现异常，截图如下：

<figure>
	<a href="http://breakdimbo.github.io/images/error-WEB-INF.png"><img src="http://breakdimbo.github.io/images/error-WEB-INF.png"></a>
</figure>

原因是.....**卧槽 WEB-INF 文件夹名字弄错了。导致找不到。**


> Dumb Q:  
> 在 EL functions 里， 函数必须要有返回值吗？  
> 容器是如何找到 TLD 的？  
> EL function 能有参数吗？

~~~xml
<function-signature>int rollDice(java.util.Map)</function-signature>And call it with ${mine:rollDice(aMapAttribute)}
~~~

## 2.7 EL 中的计算符号

> HD p430


## 2.8 EL 对 null 对处理

不会扔出异常。
null 被当做 0 或者 false。

---

# 3. Layout Templates——可重复利用组件

> include directive
> 
> \<jsp:include/> standard action

## 3.1 include directive 和 \<jsp:include/>

include directive 告诉容器，**复制**所有被 included 的文件的源码，并将它们粘贴到「这个文件」，「**这个地方**」来。

比如有一个 Header.jsp 网页，我们想在 Contact.jsp 网页中使用 Header 放在开头，需要如下操作：

~~~html
<html><body><%@ include file="Header.jsp"%><br><em>We can help.</em> <br><br> 
Contact us at: ${initParam.mainEmail} 
</body></html>~~~

而如果使用\<jsp:include/> 则如下：

~~~html
<html><body><jsp:include page="Header.jsp" /><br><em>We can help.</em> <br><br> 
Contact us at: ${initParam.mainEmail} 
</body></html>
~~~

两种方法的区别在于：  


**include directive 发生在「翻译」的时候，既由容器将所有需要的代码先行复制到 JSP 源码中；  
使用 file；  
对位置敏感**

  
**而\<jsp:include/>则发生在 runtime，既在运行时将 Header.jsp 产生的 response 插入到当前页面；  
使用 page；  
对位置不敏感**

> HD p441

**注意：不要将<html><body>标签加入到你需要重复利用的页面里**

> Do NOT put opening and closing HTML and BODY tags within your reusable pieces!  > Design and write your layout template chunks (like headers, nav bars, etc.) assuming they will be included in some OTHER page.

## 3.2 定制你的 include 内容——\<jsp:param>

如果你想根据不同的页面动态的更改 Header 的一部分（可变的 subtitle），怎么办？

将 subtitle 信息作为一个新的 request 参数传递给被 included 的页面。——使用\<jsp:param/>

在主 JSP 下的代码：

~~~html
<html><body><jsp:include page="Header.jsp" >	<jsp:param name="subTitle" value="We take the sting out of SOAP." /></jsp:include><br><em>Web Services Support Group.</em> <br><br> Contact us at: ${initParam.mainEmail} 
</body></html>~~~

在 Header 中的代码：

~~~html
<img src="images/Web-Services.jpg" > <br> 
<em><strong>${param.subTitle}</strong></em> <br>
~~~

---

# 4. 跳转至其他页面——\<jsp:forward/>

可以从一个 JSP 跳转至另一个 JSP 或者 servlet 或者其他任何你的 web app 上的资源。

Hello.jsp:

~~~html
<html><body>Welcome to our page!<% if (request.getParameter("userName") == null) { %><jsp:forward page="HandleIt.jsp" /><% } %>Hello ${param.userName}</body></html>
~~~

Handle.jsp

~~~html
<html><body>We’re sorry... you need to log in again.<form action="Hello.jsp" method="get">Name: <input name="userName" type="text"><input name="Submit" type="submit"> </form></body></html>
~~~

执行发现，在进入 Hello.jsp 中时，没有出现 Welcome to our page! 而直接跳转至 Handle.jsp。  
因为\<jsp:forward/>在跳转之前会清空缓存，也就是说，如果要进行跳转，那么你在跳转之前写的一切都会「消失」。

**注意：forward 之前不要 flush（）**


---

最后：. 和 [] 的区别。