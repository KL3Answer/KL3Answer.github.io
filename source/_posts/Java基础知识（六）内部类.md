---
title: Java基础知识（六） 内部类
date: 2015-3-19 14:39:00
tags: 
	- Java基础知识
---

内部类是定义在类中的类，其特征有两点：

1、内部类的方法可以访问该类作用域中的数据，包括私有的数据。

2、内部类可以对同一个包中的其他类隐藏。

当想定义一个回调函数且不想编写大量代码时，可以使用匿名（anonymous）内部类。

### 一、简单示例：

* 在main中
	
		public static void main(String[] args) {

			Outer.Inner1 inner1 = new Outer().new Inner1();
			Outer.Inner2 inner2 = new Outer.Inner2();//static的内部类相当于外部类，可直接创建对象
			Outer aOuter = new Outer();
			Object object = aOuter.getInner();//得到一个private的Inner对象
			aOuter.innerMethod();//调用内部类的方法
			aOuter.show();
			Outer.Inner2.method();
			aOuter.anonymousPrint(new anonymous(){
				void print(){
					System.out.println("afk");//只是为了继承抽象类anonymous
				}
			});
			Outer.oop().print();//链式编程
		}

* 这些类的声明
	
	 	/*内部类
	 	* 特点：
		 * 	a、内部类可以直接访问外部类的所有成员(因为内部类持有外部类的引用  外部类名.this)
		 * 	b、外部类访问内部类成员必须要创建对象
		 * 
		 * 格式：
		 * 		Outer.Inner inner=new Outer().new Inner();//Inner 不是private时
				inner.method();
		 */
		class Outer {
			private int num = 10;
		
			private class Inner {//定义内部类在外部类的成员位置,可被成员修饰符修饰
				private int num = 11;
		
				public void method() {
					System.out.println("Inner run");
				}
		
				public void show() {
					System.out.println(Outer.this.num);//对外部类的引用Outer.this
					System.out.println(Inner.this.num);//类名.this调用该类的当前对象
				}
			}
		
			static class Inner2 {
				public static void method() {
					System.out.println("Inner2 run");
				}
			}
		
			public Inner getInner() {//返回一个Outer.Inner类型的对象
				return new Outer().new Inner();
			}
		
			public void innerMethod() {
				Inner inner = new Inner();
				inner.method();
				inner.show();
			}
		
			public void show() {
				class Inner3 {//局部内部类，在方法中定义的类,只能在所在的方法内访问
					//局部内部类中使用的变量必须被声明为常量，以防止对象引用已经被弹出栈的变量，
					//在jdk1.8中默认加上
					//常量被存储在方法区中的常量池
					final int num = 0;
					
				}
				System.out.println();
			}
		
			/*匿名内部类(局部内部类的一种)
			 * 
			 * new 类名或接口名{
			 * 	重写方法；
			 * }
			 * 
			 * */
			class Inner1 implements Inter{
				public void print(){
					System.out.println("Inner1 print");
				}
			}
		
			public void print_2() {
				new Inter() {//实现Inter接口，实际上new的是实现了Inter的一个子类
					public void print() {
						System.out.println("print");
					}
				}.print();
				
				Inter inter=new Inter() {//多态实现，匿名内部类不能向下转型，因为没有子类
					public void print() {
						System.out.println("print");
					}
				};
				inter.print();
			}
			public void anonymousPrint(anonymous n){
				System.out.println("anonymous");
			}
			public static Inter oop(){//返回一个Inter类型的对象
				return new Inter(){
					public void print(){
						System.out.println("apm");
					}
				};
			} 
		}
		
* inter接口和anonymous抽象类

		interface Inter {
			public abstract void print();
		}
		abstract class anonymous{//抽象类anonymous
			abstract void print();
		}

### 二、关于这些修饰符的权限
		
		//修饰符					   不能用于类				不能用于外部类
		/* 访问			public     protected       默认		private	
		 * 本类			可以			可以		可以		 可以
		 * 同一包子类  	可以		    可以		 可以
		 * 不同包子类	   可以		    可以
		 * 不同包无关类	 可以
		 * */



### 三、关于包

这段代码在Java源文件的开头
	
	package Outer_Inner;
	/*package:将字节码文件分开存放
	 * 包就是文件夹（多层目录）
	 * 包名一般使用域名倒过来  
	 * 	e.g. 
	 * 		package com.hq.bean;//此时bean.java就会在com\goole中
	 * 		package com.hq.tools;
	 * 
	 * a、package语句必须是第一条可执行的语句
	 * b、在一个.java文件中只能有一个
	 * c、如果没有package，默认无包名
	 * */
	/*带包名如何编译运行
	 * 	写全名 :java  packA.Test
	 * */
	import java.util.Scanner;//导入不同包中的类
	
	....code.....


注意：一个Java源文件只能有一个public类（JVM为了提高查找类的速度，使用import语句导入时，只会导入对应空间的文件名对应的class文件，
	而public是公共的，因此直接导入这个类对应的class文件即可）