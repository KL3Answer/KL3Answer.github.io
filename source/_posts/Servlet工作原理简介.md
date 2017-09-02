---
title: Servlet工作原理简介
date: 2014-10-14 11:56:40
tags: 
	- 笔记
---
### 一、Servlet的三大方法
 Servlet是在javax.serlvet包中定义的一个接口，使用了单例设计模式，其本身是无状态的。它声明了servlet生命周期中必不可少的三个方法-init()、service()和destroy()。

1. init()方法在servlet生命周期的初始化阶段被调用。它传递一个实现了javax.servlet.ServletConfig接口的对象，使得servlet能够从web application中获取初始化参数。

2. servlet初始化收，每接收一个请求，就会调用service()方法。每个请求的处理都在独立的线程中进行。Web服务器对每个请求都会调用一次service()方法。service()方法判断请求的类型，并把它转发给相应的方法进行处理。

3. 当需要销毁servlet对象时，就要调用destroy()方法。该方法释放被占用的资源。销毁动作一般在应用结束时执行。

和所有的Java程序一样，servlet运行在JVM中。引入servlet容器是为了处理复杂的HTTP请求。Servlet容器负责servlet的创建、执行和销毁。

### 二、请求被处理的过程
1. Web服务器接收到HTTP请求；

2. Web服务器将请求转发给servlet容器；

3. 如果对应的url匹配但是serlvet容器中不存在该实例，则使用反射创建其实例；

4. 容器调用servlet的init()方法对servlet进行初始化（该方法只会在servlet第一次被载入时调用）；

5. 容器调用servlet的service()方法来处理HTTP请求，即，读取请求中的数据，创建一个响应。servlet会被保留在容器的地址空间中，继续处理其他的HTTP请求；

6. Web服务器将动态生成的结果返回； 

使用servlet，就要允许JVM为处理每个请求分配独立的Java线程，这也是Servlet容器主要的优势之一。每一个servlet都是一个拥有能响应HTTP请求的特定元素的Java类。Servlet容器的主要作用是将请求转发给相应的servlet进行处理，并在JVM处理完请求后，将动态生成的结果返回至正确的地址。在大多数情况下，servlet容器运行在独立的JVM中，但如果容器需要多个JVM，也有相应的解决方案。

### 三、与Servlet相关的对象
与Servlet关联的有四个对象servletContext,servletConfig,servletRequest,servletResponse。 
Servlet的运行模式是典型的“握手型交互式”运行模式：servletContext提供交互场景模式，而交互场景的初始化由servletConfig来描述的。servletRequest和servletResponse是交互的具体对象。

### 四、JSP

	/*
	 * JSP  java Server Page 服务器端页面，由服务器处理之后（翻译成html）发送给客户端  ，其实本身就是一个Servlet
	 * 	JSP页面由HTML代码和嵌入其中的Java代码所组成。
	 * 	服务器在页面被客户端所请求以后对这些Java代码进行处理，
	 * 	然后将生成的HTML页面返回给客户端的浏览器。 
	 * 
	 * 优点：
	 * 	1、jsp既可以写html代码，也可以写java代码（<% java代码 %>，其实编写的代码都在sercvice()里面）
	 * 	2、
	 * 
	 * JSP原理
	 * 	一个jsp页面在第一次访问的时候，会被翻译成java文件，该java类继承了HttpServlet
	 * 	然后把java文件编译成.class
	 * 	然后创建该类的对象
	 * 	最后调用Service()方法
	 * 	第二次请求同一jsp时，直接调用service()方法
	 * 
	 * 
	 * 
	 * Jsp访问标签的过程
		前提：tomcat启动过程中，会加载所有WEB-INF目录下的tld文件

			1）访问jsp页面：   01.showip.jsp		（翻译和编译）
			2）执行到导入标签库： 
				<%@taglib uri="http://www.itcast.cn" prefix="itcast"%>
			
				注意：uri一定要和tld的一致！！！prefix不一定跟short-name一致！
			3）执行到指定的标签： <itcast:showip></itcast:showip>
			4）在tld文件中搜索showip名称的一个<tag>标签	
				注意：引用的标签一定在tld文件中有声明过！！
			5）找到<tag>标签，查找对应的<tag-class>: gz.itcast.tags.ShowIPTag

	 * 
	 * 
	 * JSP和Servlet分工
	 * 	jsp：
	 * 		作为请求发起页面，例如显示表单、超链接
	 * 		作为请求结束页面，例如显示数据
	 *	Servlet：
	 *		作为请求中处理数据的环节 
	 * 	
	 * jsp组成：
	 * 		html+java脚本+jsp标签（指令）
	 * 
	 * jsp模板代码：
	 * 		就是指jsp中的html代码ii
	 * 		
	 * jsp注释：
	 * 		<!-- html注释--> 两者都可以 <%--  jsp注释 --%>(在编译成java时会忽略，不会在浏览器页面显示，建议使用)		
	 * 		
	 * jsp脚本代码片段：
	 *		作用：如果需要在jsp页面上编写java代码，就会用上脚本代码片段 
	 *		格式：<% code %>
	 *		注意：
	 *			1、脚本代码片段的代码都在_JSPService方法的内部	
	 * 			2、一个jsp中的代码都在同一个方法里			
	 * 
	 * jsp脚本表达式：（可使用el表达式取代）
	 * 		作用：向页面输出数据
	 * 		格式：
	 * 		<%= 数据 %>		（相当于out.write()）  不用加';'
	 * 
	 * jsp声明：
	 * 		可以把定义的变量或者方法都写在成员位置上
	 * 		格式:<%! 声明  %>
	 * 		注意：
	 * 			1、jsp声明不能直接使用out等内置对象
	 * 			2、编写方法时不要与其中已经存在的方法同名
	 * 
	 * jsp的处理指令：
	 * 		格式：<%@ 指令名称 %>
	 * 		主要有三个：
	 * 			page		
	 * 					language 		目前jsp使用的语言，其实只有java		
	 * 					import  		导包		
	 * 					pageEncoding 	页面保存是的编码码表		
	 * 					buffer 			输出流缓存（默认8kb）		注意：使用out对象输出时，会先写在缓存里，等缓存写满或者jsp页面结束时才写出到页面
	 * 					isElIgnore		是否忽略EL表达式，默认false
	 *					errorPage		url，出现错误时的跳转页面，并且可以将出现的异常给跳转的页面 
	 *					isErrorPage		默认false，仅当true时才接收传递的异常（缺陷：配置的jsp页面只能用于当前的jsp页面，一般使用全局的jsp页面）
	 * 						e.g.	<%@ page	language="java"  import="java.util.*,java.io.*"  pageEncoidng="UTF-8" buffer="8kb" %>
	 * 
	 *			    配置全局错误页面	 
	 *				1、可以根据响应码配置
	 *				e.g.
	 *				<error-page>
	 *					<error-code>500</error-code>	//响应码		
	 *					<location>message.jsp</location>//跳转的页面
	 *				</error-page>
	 *				2、根据异常的类型配置
	 *				<error-page>
	 *					<exception-type>java.lang.NullPionterException</exception-type>
	 *					<location>message.jsp</location>//跳转的页面
	 *				</error-page>
	 *				
	 *
	 * 
	 * 			taglib		引入外部的标签， 让jsp可以使用更多的标签,可以和自定义标签一起使用
	 * 							
	 * 						e.g.	<%@ taglib	uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
	 * 				
	 * 
	 *			include		 包含指定的页面（静态包含）	,其实就是在翻译成java文件时就已经把两jsp合并起来了，只会产生一个java文件，成员变量可以共享使用
	 *					注意：如果一个jsp页面包含了另外一个jsp页面，应该把相同的信息删除，处理page指令以外
	 *
	 *						e.g.	<%@ include file="/head.jsp"%>
	 * 
	 * jsp内置对象：
	 * 		request
	 * 		response
	 * 		out
	 * 		session
	 * 		application(ServletContext)
	 * 		config
	 * 		exception(errorPage才有)
	 * 		page(this)	
	 * 		pageContext()当前页面的运行环境
	 * 						1、可以获取当前页面所有的内置对象
	 * 						getRequest()  getSession()  getOut()	getResponse()	getServletConfig()	getServletContext()	getXXX()
	 * 							
	 * 						2、也是一个域对象，其域的范围就是当前页面
	 * 						pageContext.setAttribute("name","alice");//默认存储在pageContext中
	 * 						pageContext.setAttribute("name","alice",pageContext.REQUEST_SCOPE);//存储在request域中
	 *											
	 *						pageContext.getAttribute("name")
	 *						pageContext.getAttribute("name",pageContext.REQUEST_SCOPE);//取数据
	 * 		
	 *						pageContext.findAttribute(String attr)//会从四大域对象中找搜索顺序是pageContext、request、session、ServletContext
	 * 
	 * jsp映射：
	 * 		<servlet>	
	 * 			<servlet-name>xx</servlet-name>
	 * 			<jsp-file>x.jsp</jsp-file>//在WEB-INF根目录下：/WebRoot/WEb-INF/index.jsp?
	 * 		</servlet>	
	 * 		<servlet-mapping>
	 * 			<servlet-name>xx</servlet-name>
	 * 			<url-pattern>/xxx.html</url-pattern>//将jsp映射到其他资源路径
	 * 		</servlet-mapping>
	 * 		
	 * 	
	 * jsp内置标签：（动作标签）
	 * 		<jsp:include page="/a.jsp">动态包含（翻译的时候各自翻译，运行的时候在加入java文件,多个java文件），成员变量不可以共享使用
	 * 		<jsp:forward page="/1.jsp">	请求转发
	 * 		<jsp:param value="username" name="name">	定义变量
	 * 		<jsp:useBean id="user" class="com.hq.test.user" >		创建或查找javabean类
	 * 		<jsp:useBean id="user" class="com.hq.test.user" scope="request">		在request域创建或查找javabean类
	 * 		<jsp:setProperty property="username" name="user" value="admin">			给javabean设置属性值
	 * 		<jsp:setProperty property="password" name="user" value="123">
	 * 		<jsp:setProperty property="password" name="user">						获取javabean的属性值
	 * 
	 * 
	 * 
	 * 
	 * jsp传递数据给页面：代码或El表达式
	 * 页面数据传递给jsp：使用域对象获取(request.getParameter("but")<input type="text" name="but">)
	 * */

### 五、关于Servlet3.0+
Servlet3.0中的新特性：

* 可插拔的Web框架

	几乎所有基于Java的web框架都建立在servlet之上。现今大多数web框架要么通过servlet、要么通过Web.xml插入。利用标注（Annotation）来定义servlet、listener、filter将使之（可插拔）成为可能。程序访问web.xml和动态改变web应用配置是所期望的特性。该JSR将致力于提供把不同web框架无缝地插入到web应用的能力。
* EOD

	标注——利用标注来作为编程的声明风格。
	web应用零配置是EoD努力方向之一。部署描述符将被用来覆盖配置。
	范型（generic）——在API中尽可能利用范型。
	使用其它语言增强可能需要改善API可用性的地方。
* 支持异步和Comet

	非阻塞输入——从客户端接收数据，即使数据到达缓慢也不会发生阻塞。
	非阻塞输出——发送数据到客户端，即使客户端或网络很慢也不会发生阻塞。
	延迟请求处理——Ajax web应用的Comet风格，可以要求一个请求处理被延迟，直到超时或一个事件发生。延迟请求处理对以下情况也很有用：如果远程的/迟缓的资源必须在为该请求服务之前被获得；或者如果访问一个特殊资源，其需要扼杀一些请求以防止太多的并发访问。
	延迟响应关闭——Ajax web应用的Comet风格，可以要求响应保持打开，以允许当异步事件产生时发送额外的数据。
	阻塞/非阻塞通知——通知阻塞或非阻塞事件。
	频道概念——订阅一个频道，以及从该频道获取异步事件的能力。这意味着可以创建、订阅、退订，以及应用一些诸如谁能加入、谁不能加入的安全限制。
* 安全

	login/logout能力。
	自注册。
* 结合

	结合/需求，来自REST JST JSR（JSR 311 ）。
	结合/需求，来自JSF 2.0 JSR（JSR 134 ）。
* 其它

	支持更好的欢迎文件（welcome file）。
	ServletContextListener排序。
	容器范围内定义init参数。
	文件上载——过程侦听——存储中间或最终文件。
	澄清线程安全问题。
