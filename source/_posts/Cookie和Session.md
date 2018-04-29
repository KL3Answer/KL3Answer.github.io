---
title: Cookie和Session
date: 2014-10-14 23:34:57
tags: 
	- 笔记
---
### 一、Cookie

	/*
	* Cookie	Cookie是由HTTP协议制定的，一个Cookie的大小不能超过4kb，一个服务器最多向一个浏览器保存20个cookie，
				一个浏览器最多可以保存300个cookie
	* 	服务器使用cookie追踪客户端状态，用于存储信息的文件（用户名密码等）,
	* 	cookie技术也称为会话跟踪技术（默认浏览器打开到关闭的过程称为会话），
	* 不能跨浏览器
	* 
	*	浏览器归还Cookie的请求头：Cookie:aaa=AAA;bbb=BB
	*			response头：Set-Cookie:aaa=AAA;Set-Cookie:bbb=BBB
	* 
	* 因为浏览器竞争很激烈，所以很多浏览器会在一定范围违反http协议
	* 
	* 服务器 ：
	* 	1、校验信息
	* 	2、保存信息在cookie上
	* 	3、把cookie发送到浏览器（浏览器可以选择保存cookie）
	* 	4、下一次登录时浏览器会发送cookie
	* 
	*Cookie属性
	*	name value	maxAge Path
	*
	*
	* Cookie使用：
	* 	Cookie		new Cookie(name,value)		创建一个Cookie对象
	*  Cookie[]	request.getCookies()		获取若request中的所有cookie文件		
	* 
	* Cookied的方法
	* 			cookie对象.setMaxAge(int expiry)		 在一个硬盘上的存活时间(秒值)(不设置值或负值只在内存中，不写入硬盘)
	*  String	cookie对象.getName()		
	*  				  getValue()
	*  				  setValue()			
	*					  setPath(String uri)	
						设置cookie有效路径(与客户端保存路径无关！！关联servlet路径？)（此时只有在访问该路径时，浏览器才会发送这个cookie）
	*					如果cookie没有设置有效路径，那么默认的有效路径就是当前工程（访问路径包含Cookie路径）
	*	e.g.
	*		a在/a/下，b在/a/b/下，c在/a/b/c/下
	*		aCookie.path=/a/;bCookie.path=/a/b/;cCookie.path=/a/b/c/;
	*		则访问a时，返回aCookie
	*		访问b时，返回aCookie、bCookie
	*		访问c时，返回aCookie、bCookie、cCookie
	*	
	* 	response.addCookie(cookie)  把cookie对象发送给浏览器，
	* 		如果一个cookie没有设置一个有效(存活)时间，是不会在产生cookie文件的,
	* 		但是在浏览器关闭之前可以获取到（一个会话之间）
	* 
	* 	删除cookie
	* 		setMaxAge(0)
	* 		其实是创建一个同名的cookie，设置有效时间为0（0时浏览器会马上删除这个Cookie），覆盖掉源文件
	* 	注意：如果要删除一个cookie，其有效时间要为0，而且path路径必须一致（信息一致才能覆盖） 
	*  
	*  
	*  
	*  
	*  Cookie的domain
	*  	domain用于指定Cookie的域名，当多个二级域中共享Cookie中才使用
	*  e.g.
	*  	www.news.baidu.com和www.zhidao.baidu.com和www.tieba.baidu.com之间共享Cookie时可以使用domain
	*  设置domain为：cookie.setDomain("baidu.com")
	*  设置path为:cookie.setPath("/")
	*  
	*  
	* */

### 二、Session

	/*
	 * HttpSession是由javaweb提供的，用于会话跟踪的类，是服务器端对象，保存在服务器里
	 * HttpSession是javaweb三大域对象之一（request、session、application（SerlvetContext））
	 * 底层依赖于cookie，或是URL重写 
	 * 
	 * HttpSession的作用范围，是从用户访问服务器开始到该用户关闭浏览器结束
	 * 会话：一个用户对服务器的多次连贯性请求
	 * 服务器会为每一个客户端创建一个session对象，sessoion对象就好比客户在服务器的账户，
	 * 它们被服务器保存在一个Map中，这个map被称为session缓存
	 * 
	 * 在Servlet中获取session request.getSession()
	 * 在jsp中获取session：session是jsp的内置对象，不用创建就可以使用
	 * 
	 * session的默认最大不活动时间是30分钟（从最后一次访问时间计算，若超过改时间没有被使用，服务器会将其从session池中移除），
	   可以在tomcat/config/web.xml中配置的（当然也可以自己手动配置），
	 * 若要改变项目的最大不活动时间，可以在项目下的web.xml中配置
	 * 
	 * 	<sesion-config>
	 * 		<session-timeout>30</session-timeout>//单位：分钟
	 * 	</sesion-config>
	 * 
	 * 获取session：
	 * 	request.getSession()	向服务器获取session，如果已经存在session为该浏览器服务，就不会创建新的对象的
	 * 							创建后把session的id保存到cookie（这个cookie的有效时间为-1，只在内存中存在）中，
	 							  然后发送给浏览器（JSESSIONID）
	 * 							如果浏览器带有session的id，服务器就会从所有的session中寻找该session为浏览器服务
	 * 		
	 * session常见方法：
	 * 	String	getId()  	获取JSESSONID的值
	 *	int 	getMaxInactiveInterval()	获取最大不活动时间 
	 * 	void	invalidate()	让sesion失效
	 * 	boolean	isNew()		查看session是否为新
	 * 
	 * 
	 * */

	/*Session的钝化与激活
	 * 钝化：服务器在关闭的时候，会把session保存的数据存在tomcat/work/catalina/localhost目录下的一个文件上（从内存写到硬盘上）
	 * 激活：服务器在启动时会自动把硬盘上的session文件读取到内存中，还原session数据
	 * 
	 * 注意：session的数据如果要实现钝化，其该数据所属的类要实现Serializable接口
	 * 
	 * 在web.xml配置session
	 * <Manager></Manager>等（自己查）
	 * 
	 * 
	 * 
	 *  //以保证后面可以根据sessionid获取到session  
	    getServletContext().setAttribute(session.getId(), session);
	 * */

	/*当用户禁用了Cookie,session功能无法使用时
	 * 可以URL重写，其实就是把session的id放在url里面发送给服务器
	 * String url=request.getContextPath()+"/跳转的资源";
	 * url=response.encodeURL(url);//该方法会对url进行重写；当请求中没有归还session这个cookie时会重写url，否则不重写，url必须时指向本站的url
	 * response.sendRedirect(url);
	 * 
	 * 或者也可以自己在url后面添加";jsessionid=它的id"
	 * 
	 * */
	
	/*Cookie和Session的区别
	 * 	Cookie是客户端的技术，Session是属于服务器端的技术
	 *	Session依赖与Cookie
	 * 	
	 * 
	 * */	