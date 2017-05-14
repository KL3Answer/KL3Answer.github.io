---
title: Java基础知识（七） String类
date: 2015-3-20 1:11:00
tags: 
	- Java基础知识
---

### 一、String
>从概念上讲，Java字符串即使Unicode字符序列/例如，串“Java\u2122”由5个Unicode字符组成。Java没有内置的字符串类型，而是在标准的Java类库中提供了一个预定义类，很自然地叫做String。每一个用双引号括起来的字符串都是String1类的一个实例。

从上面这段话中，我们知道Java中字符串是由字符序列组成（Character sequence），此外String类中并没有提供修改字符串的方法（不可变字符串）。实际上，Java的字符串是作为常量存储在常量池中，这样使得字符串可以共享。所以当使用初始化String类时，实际上他引用了一个常量。
	
	String str="abc";//在常量池中创建该常量并引用

String中一些常用的方法

	/* 返回值类型		方法名		                 说明
	 * int 		length()						获取字符串的长度
	 * char    	charAt(int index)				获取指定索引位置的字符
	 * public 	String(byte[] bytes) 			将字节数组转成字符串
	 * booolean equalsIgnoreCase(String str)  	判断字符串，不区分大小写
	 * boolean 	contains(String str)			判断字符串是否包含小字符串
	 * boolean	startsWith(String str)			判断字符串是否以小字符串开头
	 * boolean	endWith(String str)				判断字符串是否以小字符串结尾
	 * boolean	isEmpty(String str)				判断字符串是否为空
	 * int 		indexOf(int ch)					返回指定字符在此字符串中第一次出现的索引(传递char类型的会自动提升)
	 * int 		indexOf(int ch,int index)		返回指定字符在此字符串中在指定位置后第一次出现的索引(传递char类型的会自动提升)
	 * int		indexOf(String str)	
	 * int		indexOf(String str,from index)	
	 * 			lastIndexOf						从后向前找第一次出现的字符,参数列表同上
	 * String 	substring(int start )			从指定位置开始截取字符串，默认到末尾
	 * String 	substring(int start ,int end)	end位不接收	
	 * 
	 * String类转化
	 * String	toUpperCase()					把字符串转化成大写	
	 * String	toLowerCase()					把字符串转换成小写
	 * byte[]	getBytes()						把字符串转换成字节数组（gbk码表中一个汉字两个字节）
	 * char[]   toCharArray()					把字符串转换成字符数组
	 * String	concat(String str)				拼接字符串
	 * String	valueOf(params)					使用String的构造方法，转换成字符
	 * String 	replace(char old,char new)		替换字符
	 * String 	replace(String old,String new)	替换字符串
	 * String 	trim()							去除字符串两端的空白
	 * int   	compareTo(String str)			按照字典顺序比较			
	 * int		compareToInoreCase(String str)						
	 * */
	

### 二、StringBuilder
StringBuilder是在JDK1.5中提供的线程不安全的用于操作String的API。如果需要使用多线程的方式操作字符，可以使用StringBuilder的前身StringBuffer，两者的API是一样的，但是后者的效率稍低。
	
	/*
	 * String和StringBuffer：
	 * 	String是不可变的字符序列
	 * 	StringBuffer初始化长度为16
	 * */
	StringBuffer sb=new StringBuffer();
	System.out.println(sb.length());//获取缓冲区的字符串实际长度
	System.out.println(sb.capacity());//缓冲区的初始容量
	StringBuffer sb1=new StringBuffer("hello");
	System.out.println(sb1.length());//
	System.out.println(sb1.capacity());//字符串长度+length()
	
	/*	public 		StringBuffer 	append(String str) 
	 * 在StringBuffer后面添加元素，并返回本身
	 * StringBuffer是字符缓冲区，new的时候在堆内存中创建了一个对象，
	 * 底层是一个长度为16的字符数组，当调用方法时，不会重新创建对象，
	 * 而是在原缓冲区添加新的字符
	 * 	public		StringBuffer 	insert(int offset,String str)
	 * */
	StringBuffer sb2=sb1.append(true);
	StringBuffer sb3=sb1.append(1000);
	sb2.insert(2, "haha");//将元素添加到第几个索引
	sb3.append("world");
	System.out.println(sb3);
	/*	public 		StringBuffer 	deleteChatAt(int index)
	 * 删除index索引的字符，并返回本身
	 *	public 		StringBuffer 	deleteChatAt(int start,int end)
	 * 删除时包头不包尾	
	 *	public 		StringBuffer 	replace(int start,int end,String str)
	 * 使用str替换从start开始到end结束，
	 * 
	 * public 		StringBuffer 	reverse();
	 * 反转
	 * public 		String			subString(int start)	
	 * public 		String 			subString(int start ,int end)
	 * 		注意返回值是String不是StringBuffer
	 * 
	 * */
	/*String----StringBuffer
	 * a:通过StringBuffer构造方法
	 * b:通过append()
	 * 
	 *StringBuffer--String 
	 * a:通过String构造方法
	 * b:通过toString()
	 * c:通过subString(0,length())
	 * 
	 * */
	System.out.println(sb3.toString());
	System.out.println(sb3);
	System.out.println(sb3.append(""));
	System.out.println(sb3.capacity());
	System.out.println(sb3.length());
	System.out.println(sb3.substring(0, sb3.length()));
注意，对String使用'+'实际上重载了append()方法，所以当你需要频繁拼接字符串是，应该考虑使用StringBuilder（StringBuffer）。

更多方法请参照[Java API文档](http://docs.oracle.com/javase/8/docs/api/)


	
