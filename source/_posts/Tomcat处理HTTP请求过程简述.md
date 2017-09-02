---
title: Tomcat处理HTTP请求过程简述
date: 2014-10-12 17:02:45
tags: 
	- 笔记
---
Tomcat是一个JSP/Servlet容器(JSP本质上也是基于Servlet的)。其作为Servlet容器，有三种工作模式：独立的Servlet容器、进程内的Servlet容器和进程外的Servlet容器。
### 一、Tomcat组件结构
Tomcat是一个基于组件的服务器，它的构成组件都是可配置的，其中最外层的是Catalina servlet容器，其他组件按照一定的格式要求配置在这个顶层容器中。 
Tomcat的各种组件都是在Tomcat安装目录下的/conf/server.xml文件中配置的。
一个典型的server.xml内容如下：

	<?xml version='1.0' encoding='utf-8'?>
	<Server port="8005" shutdown="SHUTDOWN">
		<Listener className="org.apache.catalina.startup.VersionLoggerListener" />
		<!-- Security listener. Documentation at /docs/config/listeners.html
		<Listener className="org.apache.catalina.security.SecurityListener" />
		-->
		<!--APR library loader. Documentation at /docs/apr.html -->
		<Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
		<!--Initialize Jasper prior to webapps are loaded. Documentation at /docs/jasper-howto.html -->
		<Listener className="org.apache.catalina.core.JasperListener" />
		<!-- Prevent memory leaks due to use of particular java/javax APIs-->
		<Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
		<Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
		<Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
		<!-- Global JNDI resources
			Documentation at /docs/jndi-resources-howto.html
		-->
		<GlobalNamingResources>
			<!-- Editable user database that can also be used by
				UserDatabaseRealm to authenticate users
			-->
			<Resource name="UserDatabase" auth="Container"
					type="org.apache.catalina.UserDatabase"
					description="User database that can be updated and saved"
					factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
					pathname="conf/tomcat-users.xml" />
		</GlobalNamingResources>
		<!-- A "Service" is a collection of one or more "Connectors" that share
			a single "Container" Note:  A "Service" is not itself a "Container",
			so you may not define subcomponents such as "Valves" at this level.
			Documentation at /docs/config/service.html
		-->
		<Service name="Catalina">
			<!--The connectors can use a shared executor, you can define one or more named thread pools-->
			<!--
			<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
				maxThreads="150" minSpareThreads="4"/>
			-->
			<!-- A "Connector" represents an endpoint by which requests are received
				and responses are returned. Documentation at :
				Java HTTP Connector: /docs/config/http.html (blocking & non-blocking)
				Java AJP  Connector: /docs/config/ajp.html
				APR (HTTP/AJP) Connector: /docs/apr.html
				Define a non-SSL HTTP/1.1 Connector on port 8080
			-->
			<Connector port="8080" protocol="HTTP/1.1"
					connectionTimeout="20000"
					redirectPort="8443" />
			<!-- A "Connector" using the shared thread pool-->
			<!--
			<Connector executor="tomcatThreadPool"
					port="8080" protocol="HTTP/1.1"
					connectionTimeout="20000"
					redirectPort="8443" />
			-->
			<!-- Define a SSL HTTP/1.1 Connector on port 8443
				This connector uses the BIO implementation that requires the JSSE
				style configuration. When using the APR/native implementation, the
				OpenSSL style configuration is required as described in the APR/native
				documentation -->
			<!--
			<Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
					maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
					clientAuth="false" sslProtocol="TLS" />
			-->
			<!-- Define an AJP 1.3 Connector on port 8009 -->
			<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
			<!-- An Engine represents the entry point (within Catalina) that processes
				every request.  The Engine implementation for Tomcat stand alone
				analyzes the HTTP headers included with the request, and passes them
				on to the appropriate Host (virtual host).
				Documentation at /docs/config/engine.html -->
			<!-- You should set jvmRoute to support load-balancing via AJP ie :
			<Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">
			-->
			<Engine name="Catalina" defaultHost="localhost">
			<!--For clustering, please take a look at documentation at:
				/docs/cluster-howto.html  (simple how to)
				/docs/config/cluster.html (reference documentation) -->
			<!--
			<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
			-->
			<!-- Use the LockOutRealm to prevent attempts to guess user passwords
				via a brute-force attack -->
			<Realm className="org.apache.catalina.realm.LockOutRealm">
				<!-- This Realm uses the UserDatabase configured in the global JNDI
					resources under the key "UserDatabase".  Any edits
					that are performed against this UserDatabase are immediately
					available for use by the Realm.  -->
				<Realm className="org.apache.catalina.realm.UserDatabaseRealm"
					resourceName="UserDatabase"/>
			</Realm>
			<Host name="localhost"  appBase="webapps"
					unpackWARs="true" autoDeploy="true">
				<!-- SingleSignOn valve, share authentication between web applications
					Documentation at: /docs/config/valve.html -->
				<!--
				<Valve className="org.apache.catalina.authenticator.SingleSignOn" />
				-->
				<!-- Access log processes all example.
					Documentation at: /docs/config/valve.html
					Note: The pattern used is equivalent to using pattern="common" -->
				<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
					prefix="localhost_access_log." suffix=".txt"
					pattern="%h %l %u %t "%r" %s %b" />
			</Host>
			</Engine>
		</Service>
	</Server>

其中的主要结构为：

	<Server>
		<Service>
			<Connector>
					<Engine>
							<Host>
									<Context>
									</Context>
							</Host>
					</Engine>
			</Connector>
		</Service>
	</Server>

### 二、Tomcat启动和初始化简述
借用一张图来说明：
![](http://ou5y7q62a.bkt.clouddn.com/20160315185952700.jpg)

### 三、HTTP请求处理流程
关于上面所提到的Container:
>Container是容器的父接口，该容器的设计用的是典型的责任链的设计模式，它由四个自容器组件构成，分别是Engine、Host、Context、Wrapper。这四个组件是负责关系，存在包含关系。通常一个Servlet class对应一个Wrapper，如果有多个Servlet定义多个Wrapper，如果有多个Wrapper就要定义一个更高的Container，如Context。 Context 还可以定义在父容器 Host 中，Host 不是必须的，但是要运行 war 程序，就必须要 Host，因为 war 中必有 web.xml 文件，这个文件的解析就需要 Host 了，如果要有多个 Host 就要定义一个 top 容器 Engine 了。而 Engine 没有父容器了，一个 Engine 代表一个完整的 Servlet 引擎。

请求处理的过程简述：

1. 用户发起请求，请求被发送到本机端口8080，被在那里监听的Coyote HTTP/1.1 Connector获得。 

2. Connector把该请求交给它所在的Service的Engine来处理，并等待Engine的回应。 

3. Engine获得请求localhost/test/index.jsp，匹配所有的虚拟主机Host。 

4. Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机），名为localhost的Host获得请求/test/index.jsp，匹配它所拥有的所有的Context。Host匹配到路径为/test的Context（如果匹配不到就把该请求交给路径名为“ ”的Context去处理）。 

5. path=“/test”的Context获得请求/index.jsp，在它的mapping table中寻找出对应的Servlet。Context匹配到URL PATTERN为*.jsp的Servlet,对应于JspServlet类。 

6. 构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet（）或doPost（）.执行业务逻辑、数据存储等程序。 

7. Context把执行完之后的HttpServletResponse对象返回给Host。 

8. Host把HttpServletResponse对象返回给Engine。 

9. Engine把HttpServletResponse对象返回Connector。 

10. Connector把HttpServletResponse对象返回给客户Browser。	