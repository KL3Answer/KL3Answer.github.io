---
title: Java基础知识（十六）IO流简介
date: 2015-04-14 11:23:07
tags: 
	- Java基础知识
---

### 一、顶层类和接口

	/*IO流
	 * 处理设备之间的数据传输，java对数据的操作是流的方式
	 * java用于操作流的对象都在IO包中
	 * 
	 * 流按流向
	 * 		输入流		（从外存导入到内存）
	 * 		输出流		（从内存导出到外存）
	 * 
	 * 流按操作：
	 * 		字节流：可以操作任意数据，因为计算机中的数据都是以字节（byte）为单位的二进制码的形式存储的
	 * 			拷贝非纯文本的文件推荐使用字节流，因为字符流转码浪费时间
	 * 			
	 * 		字符流：只能操作字符数据，比较方便（字节流和编码表相结合，防止乱码）
	 * 			文字操作优先考虑字符流
	 * 			无法操作非纯文本文件，因为并不是每一个字节都能对应一个存在的字符
	 * 			要将数据从内存拷贝到外存，优先考虑字符流的输出流Writer
	 * 
	 * RandomAccessFile
	 * 		随机存取文件类（可以用于多线程下载，因为可以指定位置写入与读取）
	 * 		RandomAccessFile	extends Object
	 *	 		不是IO体系中的子类
	 * 
	 * 		A:随机访问流概述
	 * 			RandomAccessFile类不属于流，是Object类的子类。但它融合了InputStream和OutputStream的功能。
	 * 			支持对随机访问文件的读取和写入。
	 * 		B：该对象内部维护一个大型的byte数组，并通过指针可以操作数组中的元素
	 * 		C：可以通过getFilePointer获取指针的位置，并通过seek()方法设置
	 * 		D：其实就是对输入输出流进行了封装
	 * 
	 * 		该对象的源和目的只能是文件
	 * 
	 * 		:read()：读取,write()：写入,seek()：将指针设定到指定位置
	 * 		
	 * 		构造方法：
	 * 			RandomAccessFile(File file,String mode)
	 * 			RandomAccessFile(String name, String mode)
	 * 
	 * 		String mode:
	 * 				r
	 * 				rw		读&写
	 * 				rwd		
	 * 				rws
	 * 
	 * 
	 * InputStream类（abstract）
	 * 		所有输入字节流的超类
	 * 		子类 如：FileInputStream		构造：FileInputStream(String name)		FileInputStream(File file)		FileInputStream(FileDescriptor fdObj)
	 * OutputStream类（abstract）
	 * 		所有输出字节流的超类
	 * 
	 * Reader类（abstract）(实际上是读取器，对流对象使用了解码)
	 * 		所有输入字符流的超类
	 * 		子类如：FileReader
	 * Writer类（abstract）
	 * 		所有输出字符流的超类
	 */

### 二、

![](https://raw.githubusercontent.com/KL3Answer/KL3Answer.github.io/hexo/source/pics/io_pic01.jpg)


	
	
	