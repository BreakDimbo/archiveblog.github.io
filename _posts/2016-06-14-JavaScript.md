---
layout: post
date: 2016-06-14
title: JavaScript
excerpt: 
java: true
comments: true
tag: [JavaScript]
---

# JavaScript

## lesson 1

三部分语法：

* 基础语法
* DOM —— 将文件当作对象
* BOM —— 将整个浏览器当作一个对象

在html 中调用javascript的方式：

1: 直接在html 中使用：
2: 从外部调用：src＝“...js”

~~~html
<!DOCTYPE html>
<html>
<head>
	<title>Javascript test</title>
	<script type="text/javascript">
		alert("Hello Javascript");
		alert("Hello");
	</script>
</head>
<body>
	<pre><script type="text/javascript" src="program.js"></script></pre>
 	<pre><script type="text/javascript">
		document.writeln("this is a Javascript output.")
	</script></pre>
</body>
</html>
~~~

javascript 可以出现在html的任何地方，一般函数定义出现在head。操作出现在body。

一个**页面**如果有几个javascript的话，不同部分的方法和变量可以共享。



javascript 调试方法：打印，删一段改一段。


## lesson 2 基本语法

弱类型，所有类型都使用 var 定义。
变量建议先定义，再使用，并且注释变量是做什么的。

数组：长度可以定义，但是不重要。

~~~html
var arr = new Array(3);
arr[0] = 1;
arr[1] = 2;
arr[2] = 3;
arr[3] = 4;
~~~

### 事件处理：重点
 
 onFocus: 当用户为了输入而选择 select text textarea等
 
 onBlur: 在 select text password textarea 失去焦点时
 
 onChange：在上述项目等值发生改变而且失去焦点时
 
 onClick：当一个对象被鼠标点中时
 
 onLoad：第一次将文档再入窗口时，一般用于body里。
 
 onUnload：当用户退出一个文档时
 
 onMouseOver：鼠标被移动到一个对象上时
 
 onMouseOut：移开
 
 onSelect：当form对象中的内容被选中时
 
**onSubmit：用户通过提交按钮提交一个表单时,（提交之前，还没有提交的时候。当onsubmit="return false"时，不能进行提交，使用这个方法进行表单的验证）**

~~~js
<!DOCTYPE html>
<html>
<head>
	<title>Javascript test</title>
	<script type="text/javascript">
		function check () {
			 if (document.test.username.value == "") {
			 	alert("空串不允许");
			 	return false;
			 } 
			 return true;
		}
	</script>
</head>
<body>
	<form name="test" action="1.jsp" onsubmit="return check()">
		<input type="text" name="username">
		<input type="submit" name="OK">
	</form>
</body>
</html> 
~~~

# lesson 3 DOM & BOM

alert 和 prompt

~~~js
var username = prompt("请输入名字：");
	document.write("你好" + uername);
~~~

confirm: 确认框

点击删除时可以弹出。

~~~js
<!DOCTYPE html>
<html>
<head>
	<title>Javascript test</title>
	<script type="text/javascript">
		function confirmt () {
			 if(confirm("confirm delete?")) {
			 	document.test.submit();
			 }
		}
	</script>
</head>
<body>
	<!-- 确认框 -->
	<form name="test" action="1.jsp" method="post">
		<input type="button" name="delete" value="confirm delete" onclick="confirmt()">
	</form>
</body>
</html>
~~~

内置对象：

1. this:当前对象
2. for..in->for(x in arr)->x是arr中属性的名字，注意
3. with 为一段代码设置默认值：

~~~
with(document) {
	write(...);
	write(...);
	write(...);
}
~~~

4. new Date();


BOM:

window: 当前窗口

~~~js
<script language="Javascript">
window.status="hekk";  
</script>
~~~

~~~js
<!--弹出新窗口-->
window.open("1.html");

<!--显示url地址-->
window.location

<!--转向文档-->
window.location="33.html"
~~~

setTimeOut（show（），delay） setInterval 动画常用函数

## lesson 5 实用效果

* 图片下拉选择器

* relative的写法要记住。效果，一个下拉条改变时，另一个跟着变。下拉条的联动。

* drawer 菜单效果

* 日历 重点
* 判断
* **表单验证 重点**

document.getElementById();通过id获得对象
innerHTML 动态指定div中间的内容

iframe 