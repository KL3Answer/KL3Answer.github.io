---
title: Java基础知识（十八）多线程（二）Thread的一些方法
date: 2015-04-14 22:11:34
tags: 
	- Java基础知识
---
### 一、构造方法
	
	//target为实现Runnable接口后的类
	Thread t=new Thread(target,name);
	Thread t=new Thread(target);
	Thread t=new Thread();
