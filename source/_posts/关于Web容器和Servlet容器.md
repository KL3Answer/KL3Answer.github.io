---
title: 关于Web容器和Servlet容器
date: 2014-10-13 23:23:11
tags: 
	- 笔记
---
### 一、关于Servlet
Servlet是一些遵从Java Servlet API的Java类，这些Java类可以响应请求。尽管Servlet可以响应任意类型的请求(对应GenericServlet)，但是它们使用最广泛的是响应web方面的请求(对应HttpServlet)。 

Servlet的运行，需要容器的支撑（用于提供一些底层的功能），而我们常使用的Tomcat与Jetty等web容器就可以提供这样的环境，在tomcat中Container下面的context就是我们的servlet容器。

### 二、Servlet初始化的过程
那么，servlet的初始化的过程是怎么样呢，以Tomcat为例，我们来简单捋一下：
1. 首先，来看一下Tomcat的目录结构：

	   tomcat
		  |--bin		可执行文件的存放目录
							e.g.	
							  startup.bat 启动
								启动时候会加载bootstrap.jar
							  shutdown.bat	关闭
		  |--conf		存放服务器自身配置的目录
							e.g.
							  Web.xml  项目部署描述配置文件, 描述项目的
							  server.xml  服务器端口配置、部署项目的维护都在此文件修改
							  其他配置文件
		  |--lib 		存放服务器运行依赖的jar
							e.g.	
							  servlet-api.jar
		  |--logs		日志
		  |--webapps	存放所有项目的目录
		  |--work		jsp的工作目录
		  |--temp		一些临时文件	
	可以知道，我们的写的项目运行是都是放在webapps里面的，那么一个标准的项目结构又是怎么样的呢：

		工程名 
		  |----静态的web资源			html、jsp、css、js文件等
		  |----WEB-INF 				该目录下的文件外界无法直接访问，由web服务器负责调用(可以存放一些需要权限的资源)（只能使用转发的方式访问）
			   |-----classes 	存放工程的class文件(整个包) 
			   |-----lib 		该工程使用的jar包
			   |-----web.xml	   	项目的描述文件
			   |-----其他资源文件等
2. 	当我们启动tomcat的时候，都发生了什么？
如果要说清楚tomcat的运行原理，可能要另起一篇文章了，所以这里只讲在tomcat context启动之后的东西。
当启动web容器加载web应用：

* 容器读取web.xml，加载两个节点: `<listener></listener>` 和 `
<context-param></context-param>`；

* 容器创建一个ServletContext,该实例有许多可以让servlet从servlet容器获取信息的方法；

* 容器将`<context-param></context-param>`转化为键值对,并交给ServletContext；

* 容器创建`<listener></listener>`中的类实例,即创建监听，在监听中会有contextInitialized(ServletContextEvent args)初始化方法；

* 得到这个context-param的值之后,你就可以做一些操作了.注意,这个时候你的WEB项目还没有完全启动完成.这个动作  会比所有的Servlet都要早。换句话说,这个时候,你对`<context-param>`中的键值做的操作,将在你的WEB项目完全启动之前被执行；

总结一下，web.xml中各个节点的加载顺序为（同个类型之间的顺序是根据对应的 mapping 的顺序进行调用）:
		
	ServletContext > context-param > Listener > Filter > Servlet



### 三、web.xml中的一些配置

	<display-name>	
		WEB应用的名字

	<description>	
		WEB应用的描述信息

	<context-param>	
		应用范围内的初始化参数
	
	<filter>	
		过滤器元素将一个名字与一个实现javax.servlet.Filter接口的类相关联
	
	<filter-mapping>	
		一旦命名了一个过滤器，就要利用filter-mapping元素把它与一个或多个servlet或JSP页面相关联
	
	<listener>	
		监听器
	
	<servlet>		
		在向servlet或JSP页面制定初始化参数或定制URL时，必须首先命名servlet或JSP页面。Servlet元素就是用来完成此项任务的

	<servlet-mapping> 
		缺省时对应的servlet的访问URL：http://host/webAppPrefix/servlet/ServletName
		注意：在一个项目中一定不能使用缺省路径，一旦使用，整个项目的静态资源会无法访问
		原因：请求静态资源的时候，并不是直接请求静态资源，tomcat服务器会创建一个DefaultHandlerservlet对象，
		然后再使用该Servlet对象进行资源的跳转，而这个对象就使用了default路径
		
	<session-config> 
		设置session过期时间  
	
	<welcome-file-list> 
		欢迎页
	
	<error-page>	
		返回特定HTTP状态代码时，或者特定类型的异常被抛出时，能够制定将要显示的页面。 
	
	<resource-ref>	
		声明一个资源工厂使用的外部资源
	