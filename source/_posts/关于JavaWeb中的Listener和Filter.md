---
title: 关于JavaWeb中的Listener和Filter
date: 2014-10-14 16:27:12
tags: 
	- 笔记
---
### 一、关于Listener
Listener是JavaWeb三大组件之一（另外两个分别是Servlet和Filter），主要用于监听域对象的创建与销毁，或监听域对象属性的添加删除修改行为。

通常用于监听三个域对象：ServletContext、request、session，分别使用以下几个Listener：

	ServletContextListener 	ServletContextAtttributeListener
	ServletRequestListener	ServletRequestAttributeListener
	HttpSessionListener		HttpSessionAttributeListener  	HttpSessionActivationListener

### 二、Listener简单示例：

1. ServletContextListener

	常见应用场景：
	* 预加载数据
	* 准备项目的运行环境
	* 清理环境

	示例：

			public class MyServletContext implements ServletContextListener{
				public void contextInitialized(ServletContextEvent sce) {
					MyDao.prepareDB();
					//other codes... 准备缓存等
					System.out.println("========环境准备完毕=======");
				}

				public void contextDestroyed(ServletContextEvent sce) {
					MyDao.cleanDB();
					//other codes...
					System.out.println("======清理环境成功========");
				}
			}
2.  ServletContextAttributeListener
	
	用于监听ServletContext对象的属性操作（增加属性，修改属性，删除属性）

	示例：
		
		public class MyContextAttribute implements ServletContextAttributeListener {

			public void attributeAdded(ServletContextAttributeEvent scab) {
				System.out.println("获取ServletContext对象："+ scab.getServletContext());
				System.out.println("ServletContext添加了属性："+ scab.getName());
				System.out.println("添加的属性值："+scab.getValue());
			}

			public void attributeRemoved(ServletContextAttributeEvent scab) {
				System.out.println("ServletContext删除的属性："+ scab.getName());
				System.out.println("删除的的属性值："+scab.getValue());
			}

			public void attributeReplaced(ServletContextAttributeEvent scab) {
				System.out.println("ServletContext被替换的属性名："+ scab.getName());
				System.out.println("替代的原属性值："+scab.getValue());
				System.out.println("替代的目前属性值："+scab.getServletContext().getAttribute(scab.getName()));
			}
		}

3. ServletRequestListener
	ServletRequestListener用于监听ServletRequest对象的创建和销毁。
	ServletRequest对象用于封装请求信息，每次发出请求时创建，请求完毕销毁。

	示例：获取用户的IP地址
		略

4. HttpSessionListener
	HttpSessionListener用于监听HttpSession对象的创建和销毁
	
		HttpSession对象:
				创建：调用request.getSession()方法
				销毁：
					  1）默认情况下，30分钟服务器自动回收
					  2）设置有效时长: setMaxActiveInterval(秒);
					  3）web.xml配置全局的过期时间
							<session-config>
								<session-timeout>分钟</session-timeout>
							</session-config>	
					  4）手动销毁： invalidate()方法

	示例： 粗略统计在线访客人数

		public class OnLineNumlistener implements HttpSessionListener {

			private int onlineCount=  0; 

			public void sessionCreated(HttpSessionEvent se) {
				HttpSession session = se.getSession();
				System.out.println("人数+1");
				onlineCount++;
				session.getServletContext().setAttribute("count",onlineCount);
			}

			public void sessionDestroyed(HttpSessionEvent se) {
				HttpSession session = se.getSession();
				System.out.println("已经离线一位");
				onlineCount--;
				session.getServletContext().setAttribute("count",onlineCount);
			}

		}

5. 其他对象的使用大同小异，这里就不一一列举了		

### 三、Filter
直接贴上来吧：

	/*
	 * JavaWeb三大组件：
	 * 		Servlet：	接收请求参数和给浏览器响应信息
	 * 		Filter：		过滤请求的资源或者响应的信息
	 * 		Listener：	监听所有域对象的创建销毁或者添加删除修改属性的行为
	 * 
	 * Filter  过滤器
	 *	服务器启动时创建Filter：
	 *	一般服务器关闭时销毁 
	 * 	应用场景：
	 * 		1、解决全站的乱码问题
	 * 		2、权限登录检查
	 * 		3、页面静态化（一般用于不常更新的资源：如果对应的html文件存在直接转到该文件，
	 否则生成html文件，即把该资源的输出流转到html文件，使用装饰者设计模式掉包其输出方法）
	 * 
	 * 方法：
	 * 	Init():		创建Filter时调用Init()方法(可以准备执行过滤任务代码的准备工作)
	 * 	doFilter():	执行过滤任务时调用doFilter()方法
	 * 	destroy():	销毁时调用destroy()方法（可以删除临时文件）
	 *		 		
	 * 
	 * 
	 * 在web.xml下配置过滤器：
	 * 	<filter>
	 * 		<filter-name>xxx</filter-name>
	 * 		<filter-class>xxx.xxx.xxx</filter-class>
	 * 	</filter>
	 * 	<init-param>//初始化参数，也可以不设置
	 * 		<param-name>xx</param-name>
	 * 		<param-value>xxx</param-value>
	 * 	</init-param>
	 *	<filter-mapping> 
	 * 		<filter-name>xxx</filter-name>
	 *		<url-pattern>xxxxx</url-pattern>//配置过滤路径，访问这个路径时才会访问过滤器，设置为"/*"过滤项目下的所有
	 *	</filter-mapping> 
	 * 
	 *			
	 *	<filter-mapping> 
	 * 		<filter-name>xxx</filter-name>
	 *		<url-pattern>xxxxx</url-pattern>//配置过滤路径，访问这个路径时才会访问过滤器，设置为"/*"过滤项目下的所有
	 *		<dispatcher>FORWARD</dispatcher>//拦截请求转发
	 *		<dispatcher>REQEUST</dispatcher>//拦截在url地址上的访问（默认，如重定向和直接访问）
	 *		<dispatcher>INCLUDE</dispatcher>//拦截动态包含
	 *		<dispatcher>ERROR</dispatcher>//拦截全局错误页面的跳转（直接访问时不会拦截）
	 *	</filter-mapping> 
	 * 
	 * 实现过滤器的步骤：
	 * 	1、自定义类实现Filter接口
	 * 	2、把过滤任务代码编写在doFilter()中
	 * 		放行：chain.doFilter(request, response);
	 * 			传递给后面的Filter，若是最后一个就放行（invoke the resource）
	 * 
	 * */

	/*FilterConfig对象：配置Filter
	 * 方法和ServletConfig类似
	 * 	获取：
	 * 	getInitParameter(name)	 	获取对应的参数值
	 * 	getInitParameterNames()   	获取所有参数名字，返回以一个枚举
	 * 
	 * */

	/*过滤路径问题
	 * 	精准过滤：
	 * 		e.g.  /index.jsp		
	 * 
	 * 	模糊过滤：
	 * 	     路径可以使用通配符'*',但是只有两种形式（'/'开头的路径优先级高于'*'开头的优先级）
	 * 		1、路径以"/"开头以'*'结尾  
	 * 			e.g.  
	 *  			<url-pattern>/aa/bb/*</url-pattern> 			
	 * 				 如果放在中间就代表普通字符'*'
	 * 
	 * 		2、如果需要匹配的是路径的后缀名，必须以'*'开头
	 * 			e.g.
	 *  		   <url-pattern>*.ad</url-pattern> 
	 * 
	 * 过滤流程：
	 * 		访问该地址时，先执行该Filter的doFilter()，由chain.doFilter()放行,然后执行下一个Filter的doFilter()，
	 * 		当为最后一个Filter时chain.doFilter()放行后执行Servlet（默认请求转发不会被拦截）
	 * 
	 * 
	 * */
	
	/*过滤器链：多个过滤器进行过滤
	 * 多个过滤器指向同一个路径时，会按照配置文件中的顺序依次执行
	 * 
	 * 
	 * 
	 * */
	
	
	/*
	 * 使用Filter解决全站的乱码问题：
	 * 	对post请求直接setCharacterEncoding(xxx)
	 * 	对get请求的request使用装饰者设计模式（或者直接对所有参数转码），修改getParameter()(创建一个类继承HttpServletRequestWrapper)
	 * 
	 * 
	 * 
	 * */