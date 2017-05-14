---
title: Java基础知识（十一）泛型
date: 2015-04-11 23:54:03
tags: 
	- Java基础知识
---
### 一、关于泛型的一些简单笔记

	/*generic泛型
	 * 泛型技术时用于编译时期的，确保类型的安全，
	 * 运行时会将泛型去掉，生成的的class文件不带泛型，这个称为泛型的擦除
	 * (擦除类型变量，替换为限定类型，无限定时使用Object)
	 * 擦除是为了兼容运行时的类加载器
	 * 
	 * 泛型的补偿：再运行时，通过获取元素的类型进行转换动作，不用使用者进行强制转换了
	 * 
	 * 泛型的好处：
	 * 		将运行时期的ClassCastException转到了编译时期，提高了安全性
	 * 		减少强制类型转换的麻烦
	 * 
	 * 泛型方法：
	 * 	泛型类的静态方法必须在声明上加泛型类型,否则不能使用泛型
	 * public <T> T a(){//泛型方法
	 *		T t=null;
	 *		ArrayList<T> a;
	 *		//code
	 *		return t;
	 *	}
	 * 泛型类：
	 * 	class b<T>{//泛型类
	 *		private T t;
	 *	}
	 * 
	 * 泛型接口：
	 * 		子类实现时要指定泛型类型,或将子类也指定相同的泛型(此时由子类在创建对象时指定泛型)
	 * 
	 * interface Inter<T>{//泛型接口
	 *		public void show(T t);
	 *	}
	 *	class a implements Inter<String>{
	 *			@Override
	 *		public void show(String t) {
	 *		}
	 *		
	 *	}
	 *		
	 * 泛型通配符<?>
	 * 		任意类型，如果没有确定，就是Object以及任意的java类
	 * 		
	 * 
	 * ? extends E
	 * 		向上限定，E及其子类，在存储时用的比较多
	 * 
	 * ? super E
	 * 		向下限定，E及其父类，在对集合的元素进行取出操作时使用
	 * */

### 二、泛型的约束和局限
1. 不能用基本数据类型实例化类型参数
	
	只能能使用基本数据类型的包装类，这样做是为了与Java中的基本数据类型的独立状态保持一致。
		
		没有List<int>只有List<Integer>)
2. 运行时类型查询只食用与原始类型。
	
		if(a instanceof List<String>)//error
		//只能测试a是否是任意类型的List，下面这个也是
		if(a instanceof List<T>)//error
		
3. 不能创建参数化类型的数组(还有Java数组协变这个坑爹属性)

	不能实例化参数类型的数组，如：
	
		List<String> table=new ArrayList<String>[10];//error
	如果这样写，擦除之后，table的类型是ArrayList[],数组会记住它的类型元素，比如你无法存储一个String在table中，但是如果你这样做：
	
		table[0]=new ArrayList<Integer>();
	
	这样可以通过数组存储检查，但是仍然会导致类型错误。因此，不允许创建参数化类型的数组。
	请注意，只是不允许创建这些数组，二声明类型为ArrayList<String>[]的变量是合法的，但是不能用ArrayList<String>[10]初始化这个变量。
	
4. Varargs警告
	
	看下面的这个示例：
		
		public static <T> void addAll(Collection<T> coll,T... ts){
			for(t:ts){
				coll.add(t);		
			}
		} 
	
	可变参ts实际上是一个数组，如果现在使用一下调用：
	
		Collection<ArrayList<String>> table=...;
		ArrayList<String> list1=...;
		ArrayList<String> list2=...;
		addAll(table,list1,list2);

	为了调用这个方法，JVM必须建立一个ArrayList<String>数组，这就违反了上一条，不过，对于这种情况规则有所放松，你只会得到一个警告。

5. 不能实例化类型变量
	
	不能使用像new T(...),new T[...]或T.class这样的表达式中的类型变量。
		
		a=new T();//error
		a=T.class.newInstance();//error
	但是可以使用下面的方式（如果有继承关系的话可以使用反射获取父类或接口的泛型类型）：
		
		public	static <T> Foo<T> createFoo(Class<T> cl){
			try{
				return new Foo<>(cl.newInstance());	
			}catch(Exception e){
				return null;
			}
		}
		
		//调用
		Foo foo=Foo.createFoo(String.class);
	注意，Class类本身是泛型，如String.class是一个Class<String>的实例（也是唯一的实例），因此createFoo可以推断出foo的类型。

6. 泛型类的静态上下文中类型变量无效
	
	不能在静态域或方法中引用类型变量，例如下面这个：
		
		public class SingleTon<T>{
			private static T singleInstance;//error
			public static T getSingleInstace(){
				if(singleInstance==null){
					return singleInstacne;
				}
			}
		}
在类型擦除之后只剩下Singleton类，它只包含一个singleInstance域，因此禁止使用带有类型变量的静态与和方法。

7. 不能抛出或捕获泛型类的实例
	
泛型类继承Throwable类不合法，如
	
	public class Problem<T> extends Exception {...} //ERROR 不能通过编译

catch子句不能使用类型变量

	public static <T extends Throwable> void doWork(Class<T> t){
	    try{
	            do work
	        }catch (T e){ // ERROR
	            Logger.global.info(...)
	        }
	}

不过，在异常规范中使用类型变量是允许的：　　

	public static <T extends Throwable> void doWork(T t) throws T { //此时可以throws T
    try{
            do work
        }catch (Throwable realCause){ //捕获到具体实例
            t.initCause(realCause); 
            throw t; //这时候抛具体实例，所以throw t 和 throws T 是可以的!
        }
	}

此特性作用：可以利用泛型类、类型擦除、SuppressWarnings标注，来消除对已检查(checked)异常的检查，

* unchecked和checked异常: Java语言规范将派生于Error类或RuntimeException的所有异常称为未检查(unchecked)异常，其他的是已检查(checked)异常
* Java异常处理原则：必须为所有已检查(checked)异常提供一个处理器，即一对一个，多对多个
  
		//SuppressWarning标注很关键，使得编译器认为T是unchecked异常从而不强迫为每一个异常提供处理器
		@SuppressWarnings("unchecked")
		public static <T extends Throwable> void throwAs(Throwable e) throws T{  
		//因为泛型和类型擦除，可以传递任意checked异常，例如RuntimeException类异常
		    throw (T) e;
		}
假设该方法放在类Block中，如果调用 Block.<RuntimeException>throwAs(t); 编译器就会认为t是一个未检查的异常

		public abstract class Block{
		    public abstract void body() throws Exception;
		    public Thread toThread(){
		        return new Thread(){
		                        public void run(){
		                            try{
		                                 body();
		                            }catch(Throwable t){
		                                 Block.<RuntimeException>throwAs(t);
		                            }
		                        }
		                    };
		    }
		
		    @SuppressWarnings("unchecked")
		    public static <T extends Throwable> void throwAs(Throwable e) throws T{
		    	throw (T) e ;
		    }
		}

	测试类

		public class Test{
		    public static void main(String[] args){
		        new Block(){
		            public void body() throws Exception{
		                //不存在ixenos文件将产生IOException，checked异常！
		                Scanner in = new Scanner(new File("ixenos"));
		                while(in.hasNext())
		                    System.out.println(in.next());
		            }
		        }.toThread().start();
		    }
		}    

	启动线程后，throwAs方法将捕获线程run方法所有checked异常，“处理”成unchecked Exception(其实只是骗了编译器)后抛出；

	有什么意义？正常情况下，因为run()方法声明为不抛出任何checked异常，所以必须捕获所有checked异常并“包装”到未检查的异常中；意义：而我们这样处理后，就不必去捕获所有并包装到unchecked异常中，我们只是抛出异常并“哄骗”了编译器而已

<br/>
8. 注意擦除后的冲突

Java泛型规范有个原则：“要想支持擦除的转换，就需要强行限制一个泛型类或类型变量T不能同时成为两个接口类型的子类，而这两个接口是统一接口的不同参数化”
注意：非泛型类可以同时实现同一接口，毕竟没有泛型，很好处理
	
	class Calender implements Comparable<Calender>{...}
	
	class GGCalender extends Calender implements Comparable<GGCalender>{...} //ERROR
在这里GGCalender类会同时实现Comparable<Calender> 和 Comparable<GGCalender>，这是同一接口的不同参数化


### 三、泛型类型的继承规则

看看下面的例子：
	
	//Bar是Foo的子类
	List<Foo> list=new ArrayList<Foo>;//it's ok 
	ArrayList<Bar> arrayList=new ArrayList<Foo>();//error
或许你会感觉很奇怪，为什么无法编译成功。在Java中泛型类型的检查如此严格，因为这对类型安全来说非常重要。假设允许ArrayList<Bar> 转换成ArrayList<Foo>，考虑下面的代码：
	
	ArrayList<Bar> barList=new ArrayList<>(bar1,bar2);
	ArrayList<Foo> fooList=barList;//假设可以这么写
	fooList.add(foo1);
显然最后一句是合法的，但是fooList和barList引用了同一个对象。

注：注意泛型和数组的区别，可以将Foo[]数组赋给类型为Bar[]的变量。

当然你可以把参数化类型转换成一个原始类型，但是依然会产生类型错误（当你传递一个其他类型给泛型时）。
