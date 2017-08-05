---
title: 关于Spring Boot使用默认连接池配置连接数据库的问题
date: 2017-7-24 21:40:00
tags: 
	- 错误记录
---
### 一、发现的问题

1. 问题概要
    
    微信端前端显示获取openid失败(微信公众号项目，前后端分离),后端连接不上数据库，项目可以访问。

2. 详细描述

    这几天接手了一个微信公众号的项目，说实话，我也没想到这个项目竟然是我的职业生涯（截止到2017.07.14）遇到的第一个也是最大的挑战。

    当我第一次看见这个代码的时候，严重怀疑这是小学生写的,各种类名变量名文件名全都是拼音。。。当然，这不是我猜测代码原作者是小学生的唯一原因，因为这些代码的逻辑也相当混乱，经常出现一大堆非常啰嗦而且明显可以用更加简单的方式写然而确并没有这么做的代码。

    但是没有办法，接手的代码只能硬着头皮上啊，然后我就上了。。。。

    几天之后项目上线，而且上线后也运行正常，然后我就没在意了。

    但是，我居然被小学生的代码坑了。。。周末玩了两天，回来一看，发现有上面的问题，也就是数据库连接失败的问题。

### 二、详细错误信息

发现问题后，我花了两分钟时间喝了一口水顺便玩了一会儿手机，然后看了一下服务器上的日志，其中的异常信息如下：

* localhost日志中的信息错误信息

      java.net.SocketException: Software caused connection abort: socket write error
      at java.net.SocketOutputStream.socketWrite0(Native Method)
      at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:109)
      at java.net.SocketOutputStream.write(SocketOutputStream.java:153)
      at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
      at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
      at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:3634)
      at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2460)
      at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2625)
      at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2551)
      at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:1861)
      at com.mysql.jdbc.PreparedStatement.executeQuery(PreparedStatement.java:1962)
      at org.hibernate.engine.jdbc.internal.ResultSetReturnImpl.extract(ResultSetReturnImpl.java:82)
      at org.hibernate.loader.Loader.getResultSet(Loader.java:2066)
      at org.hibernate.loader.Loader.executeQueryStatement(Loader.java:1863)
      at org.hibernate.loader.Loader.executeQueryStatement(Loader.java:1839)
      at org.hibernate.loader.Loader.doQuery(Loader.java:910)
      at org.hibernate.loader.Loader.doQueryAndInitializeNonLazyCollections(Loader.java:355)
      at org.hibernate.loader.Loader.doList(Loader.java:2554)
      at org.hibernate.loader.Loader.doList(Loader.java:2540)
      at org.hibernate.loader.Loader.listIgnoreQueryCache(Loader.java:2370)
      at org.hibernate.loader.Loader.list(Loader.java:2365)
      at org.hibernate.loader.hql.QueryLoader.list(QueryLoader.java:497)
      at org.hibernate.hql.internal.ast.QueryTranslatorImpl.list(QueryTranslatorImpl.java:387)
      at org.hibernate.engine.query.spi.HQLQueryPlan.performList(HQLQueryPlan.java:236)
      at org.hibernate.internal.SessionImpl.list(SessionImpl.java:1300)
      at org.hibernate.internal.QueryImpl.list(QueryImpl.java:103)
      at org.hibernate.jpa.internal.QueryImpl.list(QueryImpl.java:573)
      at org.hibernate.jpa.internal.QueryImpl.getSingleResult(QueryImpl.java:495)
      at org.hibernate.jpa.criteria.compile.CriteriaQueryTypeQueryAdapter.getSingleResult(CriteriaQueryTypeQueryAdapter.java:71)
      at sun.reflect.GeneratedMethodAccessor125.invoke(Unknown Source)
      at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
      at java.lang.reflect.Method.invoke(Method.java:498)
      at org.springframework.orm.jpa.SharedEntityManagerCreator$DeferredQueryInvocationHandler.invoke(SharedEntityManagerCreator.java:368)
      at com.sun.proxy.$Proxy120.getSingleResult(Unknown Source)
      at org.springframework.data.jpa.repository.query.JpaQueryExecution$SingleEntityExecution.doExecute(JpaQueryExecution.java:206)
      at org.springframework.data.jpa.repository.query.JpaQueryExecution.execute(JpaQueryExecution.java:78)
      at org.springframework.data.jpa.repository.query.AbstractJpaQuery.doExecute(AbstractJpaQuery.java:100)
      at org.springframework.data.jpa.repository.query.AbstractJpaQuery.execute(AbstractJpaQuery.java:91)
      at org.springframework.data.repository.core.support.RepositoryFactorySupport$QueryExecutorMethodInterceptor.doInvoke(RepositoryFactorySupport.java:462)
      at org.springframework.data.repository.core.support.RepositoryFactorySupport$QueryExecutorMethodInterceptor.invoke(RepositoryFactorySupport.java:440)
      at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
      at org.springframework.data.projection.DefaultMethodInvokingMethodInterceptor.invoke(DefaultMethodInvokingMethodInterceptor.java:61)
      at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
      at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:99)
      at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:281)
      at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:96)
      at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
      at org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:136)
      at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
      at org.springframework.data.jpa.repository.support.CrudMethodMetadataPostProcessor$CrudMethodMetadataPopulatingMethodInterceptor.invoke(CrudMethodMetadataPostProcessor.java:131)
      at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
      at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
      at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
      at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:208)
      at com.sun.proxy.$Proxy92.findById(Unknown Source)  

正如上面所说的，socket通信上的问题，我这边的socket 写出的时候出错，除此以外没有其他的异常信息，发现具体原因后，我陷入了深思。
    

###  三、分析过程
1. 起初，我推断可能是防火墙的问题导致无法连接数据库，但是我发现同一台机器上的其他应用都是好的，而且当我重启服务之后，程序又正常运行，没有产生上面的异常。所以防火墙的问题排除。

2. 同理数据库服务器应该也是正常的，获取数据库连接的基本逻辑应该也是没有问题的。

3. 在我仔细查看了日志之后，发现这些异常产生的时候都是在早上，而且大部分是在周一的早上，并且可以从日志中看出，在抛出异常之前，服务很长一段时间没有人访问，而且在抛出异常的访问的上一次访问，也就是出问题之间前的那一个访问，距离出问题的时间的间隔都非常的长，基本都在九个小时左右，难道是服务器被攻击了？

4. 我又去查看了nginx和tomcat的access的日志，发现并没有任何异常，至少可以证明不是由那几次访问造成的异常，此时我又突然想起了不知道在哪里看到的关于MySQL空闲连接8小时失效的东西，发现这个问题和MySQL8小时空闲连接有一点类似，但是我又有一个新的问题，难道连接池不会判断连接是否可用吗。带着疑问，我来到了StackOverflow，发现还真的是这样的。。。spring boot的默认来连接池中虽然有这个配置，但是在默认情况下是不开启的


### 四、解决方式
在确定了问题可能的原因之后，解决问题就非常简单了，因为这个项目使用的是spring boot默认的连接池，所以我在application.properties中添加了几条配置：

    spring.datasource.test-on-borrow=true
    spring.datasource.timeBetweenEvictionRunsMillis=60000  
    spring.datasource.removeAbandoned=true  
    spring.datasource.removeAbandonedTimeout=1800

这几条配置见名知意，大家看一眼就知道是干嘛用的了。    

在添加了配置之后，我并不确定是否能真正解决这个问题，但是从服务的日志上来看，8小时连接的问题应该不会再出现了。

在这个项目安然运行十几天之后，也一直没出现这个问题。
至此，应该可以确定MySQL的8小时策略和spring boot傻逼的默认连接池设置就是这个问题的罪魁祸首。
