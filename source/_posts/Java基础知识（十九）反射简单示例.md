---
title: Java基础知识（十九）反射简单示例
date: 2015-04-15 8:40:34
tags: 
	- Java基础知识
---
示例


	import java.lang.reflect.Constructor;
	import java.lang.reflect.Field;
	import java.lang.reflect.InvocationTargetException;
	import java.lang.reflect.Method;
	import java.util.ArrayList;
	
	public class Demo_01_Reflective {
		public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, SecurityException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException, NoSuchFieldException{
			/*
			 * 工具构造
			 * Reflective
			 * 		java 反射机制是在运行状态中，对于任意一个类，都能知道这个类的所有属性和方法
			 * 		对于任意一个对象，都能够调用它的任意一个方法和属性
			 * 		这种动态获取的信息以及动态调用对象的方法的功能称为java的反射机制
			 * 
			 * 	反射:能够分析类能力的程序
			 * 		可以使用在接口上
			 * 		提高了扩展性
			 * 
			 * 反射可以：
			 * 		在运行中分析类的能力
			 * 		在运行中查看对象，例如，编写一个toString方法供所有类使用
			 * 		实现通用的操作
			 * 		利用Method对象，这个对象和C++的指针相似
			 */
			/*
			 * class  Class
			 * 	运行时的类型标识，跟踪每一个对象所属的类
			 * 	任何数据类型都可以被class描述
			 * 
			 * 
			 * 
			 * e.g.
			 * 
			 * 源文件阶段：
			 * 	Class clazz=Class.forName(全类名);
			 * 
			 * 字节码阶段：
			 * 	Class clazz=全类名.class;
			 * 
			 * 创建对象阶段：
			 * 	Class clazz=对象.getClass();
			 * 
			 * */
	//		demo_01();
	//		demo_02();
	//		demo_03();
	//		demo_04();
	//		demo_05();
			
			
		}
demo5

		/**
		 * @throws NoSuchFieldException
		 * @throws IllegalAccessException
		 */
		public static void demo_05() throws NoSuchFieldException, IllegalAccessException {
			test p=new test("alice",12);
			setProperty(p, "id", "allen");//使用反射设置对象的域
			System.out.println(p);
		}
		public static void setProperty(Object obj,String name,Object value) throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException{
			Class clazz=obj.getClass();
			Field f=clazz.getDeclaredField(name);//获取本类中的所有方法，所有！
			f.setAccessible(true);
			f.set(obj, value);
		}
demo4

		/**
		 * @throws NoSuchMethodException
		 * @throws IllegalAccessException
		 * @throws InvocationTargetException
		 */
		public static void demo_04() throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
			ArrayList<Integer> al=new ArrayList<>();
			al.add(1);
			al.add(2);
			al.add(3);
			// AccessibleObject  subclass：  Field  	Method 	 Constructor
			//提供了将反射对象标记为在使用时取消默认java语言访问控制检查的能力  暴力反射
			//使用反射越过泛型的类型检查
			Method m=java.util.ArrayList.class.getMethod("add",Object.class);
			m.invoke(al, "abc");
			System.out.println(al);
		}
	
		/**
		 * @throws NoSuchMethodException
		 * @throws IllegalAccessException
		 * @throws InvocationTargetException
		 */
demo3

		public static void demo_03() throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
			test p=new test();
			Method m=p.getClass().getMethod("print", int.class);//获取print方法
			m.invoke(p, 2);						//让p调用m方法，参数为2
		}
	
		/**
		 * @throws NoSuchMethodException
		 * @throws InstantiationException
		 * @throws IllegalAccessException
		 * @throws InvocationTargetException
		 * @throws NoSuchFieldException
		 */
demo2

		public static void demo_02() throws NoSuchMethodException, InstantiationException, IllegalAccessException,
				InvocationTargetException, NoSuchFieldException {
			//获取Class对象
			Class clazz=j2se_26_reflective.test.class;
			
			//获取构造
			Constructor c=clazz.getConstructor(String.class,int.class);//Constructor 有泛型
			
			test p=(test)c.newInstance("12",2);//使用构造创建对象
			System.out.println(p);
			System.out.println("---------");
	//		Field f=clazz.getField("id");		获取一个域
	//		f.set(p,"123");						设置域
			//getDeclaredField()				获取一个Field的数组
			Field f=clazz.getDeclaredField("id");	//暴力反射获取私有域
			f.setAccessible(true);					//去除私有权限
			f.set(p, "123");						
			System.out.println(p);
		}
	
demo1

		/**
		 * @throws ClassNotFoundException
		 */
		public static Class demo_01() throws ClassNotFoundException {
			Class clazz1=Class.forName("demo_26.Demo_GUI");		
			System.out.println(clazz1);
			Class clazz2=j2se_gui.Demo_GUI.class;
			System.out.println(clazz2);
			System.out.println(clazz1==clazz2);
			return clazz1;
		}
	}
test类

	class test{
		private String id;
		private int num;
		public void print(int a){
			System.out.println(a+"---------");
		}
		@Override
		public int hashCode() {
			final int prime = 31;
			int result = 1;
			result = prime * result + ((id == null) ? 0 : id.hashCode());
			result = prime * result + num;
			return result;
		}
		@Override
		public boolean equals(Object obj) {
			if (this == obj)
				return true;
			if (obj == null)
				return false;
			if (getClass() != obj.getClass())
				return false;
			test other = (test) obj;
			if (id == null) {
				if (other.id != null)
					return false;
			} else if (!id.equals(other.id))
				return false;
			if (num != other.num)
				return false;
			return true;
		}
		@Override
		public String toString() {
			return "ppp [id=" + id + ", num=" + num + "]";
		}
		/**
		 * 
		 */
		public test() {
		}
		/**
		 * @param id
		 * @param num
		 */
		public test(String id, int num) {
			super();
			this.id = id;
			this.num = num;
		}
	}



