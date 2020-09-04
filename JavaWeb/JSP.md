# JSP
## 一、JSP介绍
JSP全称是（Java server pages），和servlet技术一样，都是开发动态web资源技术。
JSP最大的特点是，JSP代码和html相同，但html只能为用户提供静态资源，而JSP技术可以在页面中嵌套Java代码为用户提供动态资源。

## 二、JSP原理
客户端在向服务器发送请求时，不管访问什么资源，都是在请求一个servlet，所以在访问JSP页面时，也是在访问servlet。在服务器一执行，就会把JSP文件翻译成一个servle的class文件，然后在通过浏览器访问这个被翻译过后的JSP页面。
翻译过后的JSP页面是继承HttpJspBase，而HttpJspBase又是HttpServlet的子类，所以JSP页面本质上也是一个servlet。

## 三、JSP基础语法
### 1、JSP表达式
作用：JSP表达式用于将程序数据传输到客户端
语法：`<% = 变量或表达式>`
```jsp
<%-- 向客户端输出当前时间 --%>
<%= new Date()%>
```
注意：JSP表达式的变量或表达式后面不能有分号。

### 2、JSP脚本片段
作用：JSP脚本片段用于在JSP页面中写多行Java代码
语法：
```jsp
<%
    int a = 1;
    for (int i = 0; i < 10; i++) {
        a += i;
    }
    out.print(a);
%>
```
注意：
* JSP脚本片段只能出现Java代码，在翻译JSP页面时，会把片段里的Java代码原封不动的放到servlet上。
* JSP脚本片段必须遵循Java语法规则。
* 在一个JSP页面中可以有多个JSP脚本片段，中间可以嵌入文本、HTML或其他JSP元素。
* 多个JSP脚本片段可以互相访问。如果单个JSP脚本片段不完整，但多个JSP脚本片段组合起来是完整的也是可以的。

### 3、JSP声明
作用：在JSP页面中写的代码，默认会写到service方法中，而JSP声明中的代码被写service方法外面，全局使用。
JSP声明可用于定义JSP页面转换成的Servlet程序的静态代码块、成员变量和方法.
多个静态代码块、变量和函数可以定义在一个JSP声明中，也可以分别单独定义在多个JSP声明中.
语法：
```java
<%!
    static {
        System.out.println("jsp声明的静态代码");
    }
    private int globalVar = 10;
    public void method(){
        System.out.println("method");
    }
%>
```

## 四、JSP指令介绍
JSP指令（directive）是为JSP引擎而设计的，它们并不直接产生任何可见输出，而只是告诉引擎如何处理JSP页面中的其余部分。
三种JSP指令
* page指令
* Include指令
* taglib指令

JSP指令基本语法：<%@ 指令 属性名="值" %>

### 1、Page指令
page指令用于指定JSP页面的各种属性，无论page指令出现在jsp页面中哪个地方，它的作用都是整个jsp页面。

#### 1.1、page的import属性
在jsp中，会自动导入下面的包
```jsp
<%@ page import="java.util.Date" %>
```

#### 1.2、page的errorPage属性
* errorPage属性的设置值必须使用相对路径，如果以“/”开头，表示相对于当前Web应用程序的根目录(注意不是站点根目录)，否则，表示相对于当前页面
* 可以在web.xml文件中使用<error-page>元素为整个Web应用程序设置错误处理页面。
* <error-page>元素有3个子元素，<error-code>、<exception-type>、<location>
* <error-code>子元素指定错误的状态码，例如：<error-code>404</error-code>
* <exception-type>子元素指定异常类的完全限定名，例如：<exception-type>java.lang.ArithmeticException</exception-type>
* <location>子元素指定以“/”开头的错误处理页面的路径，例如：<location>/ErrorPage/404Error.jsp</location>
* 如果设置了某个JSP页面的errorPage属性，那么在web.xml文件中设置的错误处理将不对该页面起作用。

在jsp的page配置错误页面
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%-- 指向错误页面 --%>
<%@ page errorPage="error/error.jsp" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%
    int x = 1 / 0;
%>
</body>
</html>
```

在web.xml中配置错误页面
```xml
<error-page>
    <error-code>404</error-code>
    <location>/error/error.jsp</location>
  </error-page>
```

### 2、include指令
include指令用于引入其它JSP页面，如果使用include指令引入了其它JSP页面，那么JSP引擎将把这两个JSP翻译成一个servlet。所以include指令引入通常也称之为静态引入。
```jsp
<%--引入其他jsp--%>
<%@include file="common/header.jsp"%>
网站主体
<%@include file="common/footer.jsp"%>
```

## 五、JSP的9大内置对象

| NO   | 内置对象    | 类型                                   |
| ---- | ----------- | -------------------------------------- |
| 1    | pageContext | javax.servlet.jsp.PageContext          |
| 2    | request     | javax.servlet.http.HttpServletRequest  |
| 3    | response    | javax.servlet.http.HttpServletResponse |
| 4    | session     | javax.servlet.http.HttpSession         |
| 5    | application | javax.servlet.ServletContext           |
| 6    | config      | javax.servlet.ServletConfig            |
| 7    | out         | javax.servlet.jsp.JspWriter            |
| 8    | page        | java.lang.Object                       |
| 9    | exception   | java.lang.Throwable                    |

### 1、内置对象使用说明
#### 1.1、page对象
page对象表示当前的一个jsp页面，把jsp当作一个对象来看。
#### 1.2、out对象
out对象用于向客户端发送文本数据
#### 1.3、pageContext对象
pageContext对象是JSP技术中最重要的一个对象，它代表JSP页面的运行环境，这个对象不仅封装了对其它8大隐式对象的引用，它自身还是一个域对象(容器)，可以用来保存数据。并且，这个对象还封装了web开发中经常涉及到的一些常用操作，例如引入和跳转其它资源、检索其它域对象中的属性等。

1. 通过pageContext获取其他对象
    * getException方法返回exception隐式对象
    * getPage方法返回page隐式对象
    * getRequest方法返回request隐式对象
    * getResponse方法返回response隐式对象
    * getServletConfig方法返回config隐式对象
    * getServletContext方法返回application隐式对象
    * getSession方法返回session隐式对象
    * getOut方法返回out隐式对象

2. pageContext作为域对象

常用方法
```java
public void setAttribute(java.lang.String name,java.lang.Object value)
public java.lang.Object getAttribute(java.lang.String name)
public void removeAttribute(java.lang.String name)
public java.lang.Object findAttribute(java.lang.String name)
```

其中findAttribute方法可以查找各个域中的属性，当要查找某个属性时，findAttribute方法查找顺序"page→request→session→application"在这四个对象中去查找，找到就返回属性值，没找到就返回null。在使用el表达式时，找不到会返回一个空字符。

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%
    pageContext.setAttribute("name1", "a"); //数据只在一个页面有效
    request.setAttribute("name2", "b");     //数据在一次请求中有效
    session.setAttribute("name3", "c");     //数据在一次会话中有效
    application.setAttribute("name4", "d"); //数据在服务器中有效
%>
<%
    String name1 = (String) pageContext.findAttribute("name1");
    String name2 = (String)pageContext.findAttribute("name2");
    String name3 = (String)pageContext.findAttribute("name3");
    String name4 = (String)pageContext.findAttribute("name4");
%>
<h1>pageContext.findAttribute查找到的值</h1>
<h3>${name1}</h3>
<h3>${name2}</h3>
<h3>${name3}</h3>
<h3>${name4}</h3>
</body>
</html>
```
pageContext封装了访问其他域的方法
```java
public java.lang.Object getAttribute(java.lang.String name,int scope)
public void setAttribute(java.lang.String name, java.lang.Object value,int scope)
public void removeAttribute(java.lang.String name,int scope)
```
其中scope表示域的常量
```
PageContext.APPLICATION_SCOPE
PageContext.SESSION_SCOPE
PageContext.REQUEST_SCOPE
PageContext.PAGE_SCOPE 
```
pageContext访问其他域
```jsp
<%
//此时的设置属性等同于 session.setAttribute();
pageContext.setAttribute("name1", "a",PageContext.SESSION_SCOPE);
%>
<%
    //使用pageContext获取session对象属性
    String name1 = (String) pageContext.getAttribute("name1",PageContext.SESSION_SCOPE);
    //使用session对象获取属性
    String name11 = (String) session.getAttribute("name1");
%>
<h1>pageContext.getAttribute查找到的值</h1>
<h3><%=name1%></h3>
<h3><%=name11%></h3>
```


3. PageContext引入和跳转到其他资源
PageContext类中定义了一个forward方法(用来跳转页面)和两个include方法(用来引入页面)来分别简化和替代RequestDispatcher.forward方法和include方法。
```jsp
<%    pageContext.forward("/demo.jsp");%>
```

## 八、JSP标签
JSP标签也称之为Jsp Action(JSP动作)元素，它用于在Jsp页面中提供业务逻辑功能，避免在JSP页面中直接编写java代码，造成jsp页面难以维护。
JSP常用标签：
* <<jsp:include>>
* <<jsp:forward>>
* <<jsp:param>>  

### 1、<<jsp:include>>标签
用于把另外一个资源的输出内容插入进当前JSP页面的输出内容之中，这种在JSP页面执行时的引入方式称之为动态引入
语法：
<jsp:include page="relativeURL | <%=expression%>" flush="true|false" />　　
page属性用于指定被引入资源的相对路径，它也可以通过执行一个表达式来获得。　　
flush属性指定在插入其他资源的输出内容时，是否先将当前JSP页面的已输出的内容刷新到客户端。
```jsp
<%--引入外部资源--%>
<jsp:include page="common/header.jsp"/>
<h1>页面主体</h1>
<jsp:include page="common/footer.jsp"/>
```
#### 1.1、<<jsp:include>>标签和include指令的区别
<<jsp:include>>标签是动态引入，被引入的两个资源都会被翻译为servlet，在执行时进行合并。而include指令是静态引入，被引入的jsp页面先合并再被翻译为一个servlet。

### 2、<<jsp:forward>>标签
<jsp:forward>标签用于把请求转发给另一个资源

### 3、<<jsp:param>>标签
当使用<jsp:include>和<jsp:forward>标签引入或将请求转发给其它资源时，可以使用<jsp:param>标签向这个资源传递参数。

## 七、JSTL标签库
JSTL标签库是为了弥补html标签的不足。

### 1、核心标签库
在使用核心标签库前，需要在jsp页面引入
><%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
>核心标签库分为四类
1. 表达控制标签
2. 流程控制标签
3. 循环标签
4. URL操作标签

#### 1.1、表达控制标签
1. <c:out>标签功能
<c:out>标签主要是用来向页面输出数据或对象。
2. <c:set>标签功能
<c:set>标签用于把某一个对象存在指定的域范围内，或者将某一个对象存储到Map或者JavaBean对象中
3. <c:remove>标签的功能
<c:remove>标签主要用来从指定的JSP范围内移除指定的变量
4. <c:catch>标签的功能
<c:catch>标签用于捕获嵌套在标签体中的内容抛出的异常。

#### 1.2、流程控制标签
1. <c:if>标签的功能
<c:if>标签和程序中的if语句作用相同，用来实现条件控制。
2. <c:choose>、<c:when>和<c:otherwise>标签的功能
<c:choose>、<c:when>和<c:otherwise>这3个标签通常情况下是一起使用的，<c:choose>标签作为<c:when>和<c:otherwise>标签的父标签来使用。　　
使用<c:choose>，<c:when>和<c:otherwise>三个标签，可以构造类似 “if-else if-else” 的复杂条件判断结构。

#### 1.3、循环控制标签
1. <c:forEach>标签的功能
该标签根据循环条件遍历集合（Collection）中的元素。
2. <c:forTokens>标签的功能
该标签用于浏览字符串，并根据指定的字符将字符串截取。

#### 1.4、URL操作标签
1. <c:import>标签的功能
该标签可以把其他静态或动态文件包含到本JSP页面，与<jsp:include>的区别为：<jsp:include>只能包含同一个web应用中的文件。而<c:import>可以包含其他web应用中的文件，甚至是网络上的资源。
2. <c:url>标签的功能
<c:url>标签用于在JSP页面中构造一个URL地址，其主要目的是实现URL重写。
3. <c:redirect>标签的功能
该标签用来实现请求的重定向。同时可以配合使用<c:param>标签在url中加入指定的参数。

#### 1.5、<c:param>标签
在JSP页面进行URL的相关操作时，经常要在URL地址后面附加一些参数。<c:param>标签可以嵌套在<c:import>、<c:url>或<c:redirect>标签内，为这些标签所使用的URL地址附加参数。


## 八、EL表达式
EL表达式（Expression Language），主要作用：
* 获取数据
* 执行运算
* 获取web开发常用对象
* 调用Java方法