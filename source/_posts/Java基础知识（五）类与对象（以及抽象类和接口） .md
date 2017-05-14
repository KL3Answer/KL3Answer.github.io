---
title: Java基础知识（五） 类与对象（以及抽象类和接口）
date: 2015-3-19 11:57:00
tags: 
	- Java基础知识
---

### 一、面向对象思想

Java是一门面向对象的语言，所以我们要学会面向对象程序设计（object oriented programing）才能编写高质量的Java程序。之所以Java面向对象，是因为oop可以帮助我们将大规模的问题交给我们设计（或其他人设计）的对象来完成，这与传统的结构化编程相比，更容易掌握和debug。

在Java中类是构造对象的模板，由类构造的对象的过程称为创建对象类的实例（instance）。

### 二、封装

面向对象三大特征：封装encapsulation、多态inheritation、继承polymorphism。

封装示例
	
	/*封装：将事物的属性和行为隐藏起来，仅对外提供公共访问方法
	 * 
	 * A:封装概述
	 *就是把对象的属性和实现细节封装起来，仅对外提供公共的访问方式
	 *
	 * B:封装好处
	 * 
	 * 隐藏实现细节，提供公共的访问方式
	 * 提高了代码的复用性
	 * 提高安全性。
	 * 
	 * C:封装原则
	 * 将不需要对外提供的内容都隐藏起来。
	 * 把属性隐藏，提供公共方法对其访问。
	 * 
	 * 修饰符private: 被修饰的变量、方法只能在同一个类中调用（也可修饰类）
	 * 		在其他类中用setxxx()和getxxx()来调用
	 * private只是封装的一种体现形式，不完全等于封装
	 * 
	 * Java中最小的封装就是方法
	 * */	


	public static void main(String[] args){
	 	Student s = new Student("Allen", "male", 20);//创建一个student类的对象 s
		System.out.println(s.name);//调用成员变量：对象名.成员变量名,当变量不是private时;
		s.greeting();//调用成员方法：对象名.成员方法名（参数）;	
			
		new Student().getPhone()//匿名对象的调用
	}


	class Student {//基本类
		private String name;//系统初始化其值为null
		private String gender;
		private int age;//系统初始化其值为0
		private String id;//私有化成员变量id
		private Phone phone;
		/*
		 * 三种初始化数据域的方式：
		 * 		a、在构造器中设置
		 * 		b、在声明中赋值	
		 *  	c、初始化代码块（initialization block）
		 * 
		 * */
		{//初始化代码块（initialization block），一般直接把初始化代码块放在构造器中，优先于构造器执行
			String id="2333333";
		}
		public Student(String name, String gender, int age) {//constructor构造器或构造方法
			this.name = name;
			this.gender = gender;
			this.age = age;
		}

		//这个构造器将引用同一个类的另一个构造器，这样的方式非常有用，对公共的构造器代码只用编写一次即可
		public Student() {//重载一个无参构造方法，如果没有给出构造方法，系统将自动提供一个无参的构造方法
			this("lucy","female",15);//（放在构造器第一行！）
		}
		public void setPhone() {
			this.phone = new Phone("huawei", 2000);
		}
	
		public Phone getPhone() {
			return this.phone;
		}
	
	
		public String getId() {//访问器，获取id（不要编写返回引用可变对象的访问器方法，这样会破坏封装性！）
			return id;
		}
	
		public String getName() {
			return this.name;
		}
	
		public void showName() {//显示名字
			System.out.println("name= " + this.name);
		}
	
		public void setId(String id) {//更改器，设置id
			if (id != " ") {
				this.id = id;
				/*this关键字：代表对所在方法所属对象的引用，记录对象的地址，代表隐式参数
				（即出现在方法名前面的类对象，显式参数时明显地列在方法声明中的）
				*用于区分局部变量和成员变量
				* 
				* 在不重名时系统默认加
				*/
			} else {
				System.out.println("輸入id錯誤");
			}
		}
	
		public void renew() {//重建
			this.name = "lucy";
			this.age = 12;
			this.gender = "female";
		}
	
		/*static是一个修饰符，用于修饰成员(变量与方法)
		 * static修饰的成员被所有对象(该类)所共享
		 * static优先于对象存在，因为static的成员随着类的加载就已经存在了
		 * static修饰的成员多了一种调用方式，就是可以直接被类名所调用(类名.静态成员)
		 * static修饰的成员数据是共享数据，对象中存储的是特有的数据
		 * */
		static String country = "CN";
	
		/*构造方法
		 * 1、方法名称与类名一致
		 * 2、不用定义返回值类型
		 * 3、没有具体的返回值
		 * 
		 * 作用：初始化对象 
		 * 
		 * 构造方法与一般方法的区别：
		 * 		构造方法:对象创建时就会调用与之对应的构造方法对对象进行初始化
		 * 		一般方法:对象创建后，需要方法功能时才创建
		 * 
		 * 		构造方法：对象创建时，会调用且只调用一次
		 *     (不能对已经存在的对象调用构造器来重新设置实例域)
		 * 		一般方法：对象创建后，可以被调用多次
		 * 		
		 * 		构造方法：在堆中构造对象
		 * 		一般方法：在栈中
		 * 		构造方法 前面不加 修饰符（void int 等，不同于一般方法）
		 * 
		 * 		构造方法可用this调用其他构造方法，注意只能定义在构造方法的第一行,因为初始化动作要先执行
		 * */
		
	
		public void study() {//成员方法“study”
			System.out.println("studying");
		}
	
		public void greeting() {//成员方法“greeting”
			System.out.println("Hello,my name is " + this.name + ",I'm a " + this.age + " year-old " + this.gender + ".");
		}
	}

关于static
	
	/*
	 *静态（类）static关键字特点：
	 * a、随着类的加载而存在
	 * b、优先于对象存在
	 * c、所有对象共享
	 * d、可以使用类名调用（也可用对象名调用）
	 * 
	 * 静态成员方法无法使用非静态的成员变量
	 * 
	 * 静态变量（类变量、类域） 		存储在方法区中的静态区		随着类的加载而加载
	 * 成员变量（实例变量、实例域）	存储在堆内存				随着对象的加载而加载
	 *  
	 *  如果一个类中的所有方法都是static，应该把构造器private 
	 */	
static示例

	/**
 	 * 这是一个简单的数组工具类
 	 * @author HQ
 	 * @vesion 1.0.0
 	 * */
	class ArrayTool {
		/**
		 * 私有构造方法
		 * */
		private ArrayTool() {
		}
	
		/**
		 * 这是获取数组中最大值的方法
		 * @param arr 接收一个int类型的数组
		 * @return 返回数组的最大值
		 * */
		/*a.maximum*/
		public static int getMax(int[] arr) {
			int num = 0;
			for (int i = 0; i < arr.length; i++) {
				if (num < arr[i]) {
					num = arr[i];
				}
			}
			return num;
		}
	
		/**
		 * 这是反转数组的方法
		 * @param arr 接收一个int类型的数组
		 * 
		 * */
		/* b.revese*/
		public static void reverseArray(int[] arr) {
			for (int i = 0; i < arr.length; i++) {
				int temp = 0;
				temp = arr[i];
				arr[i] = arr[arr.length - 1 - i];
				arr[arr.length - 1 - i] = temp;
			}
		}
	
		/**
		 * 这是遍历数组的方法
		 * @param arr 接收一个int类型的数组
		 * 
		 * */
		/* c.printall*/
		public static void printAll(int[] arr) {
			for (int i = 0; i < arr.length; i++) {
				System.out.print(arr[i] + " ");
			}
			System.out.println();
		}
	}	



### 三、继承

	
	/*继承 extends
	 * 让类与类之间产生关系，子父类关系.
	 * 子类subclass也称为派生类derived class或孩子类child class,
	 * 父类parent class也称为超类superclass 、基类base class)
	 * 
	 * 继承的好处：
	 * 1、提高了代码的复用性
	 * 2、让类与类之间产生了关系，给第三个特征多态提供了前提
	 * 继承的弊端：
	 * 	类与类的耦合性增强
	 * 
	 * 继承的注意事项：
	 * 	a、子类只能继承父类非私有的成员（成员方法与成员变量）
	 * 	b、不要为了部分功能使用继承
	 * 	c、子类不能继承父类的构造方法（因为构造方法与类名一致，构造方法中有this），
	 * 	但是可以使用super区访问父类的构造方法（一定会访问父类）
	 * 
	 * java中支持单继承，不直接支持多继承，但对c++的多继承机制进行了改良。
	 * 单继承：一个子类只能有一个父类
	 * 多继承：一个子类可以有多个直接父类（java中不允许），
	 * 不直接支持是因为多个父类中有相同成员时会产生调用的不确定性，在java中是使用“多实现”的方法来体现的
	 * java支持多层（多重）继承
	 * c继承b，b继承a，就会出现继承体系
	 * 
	 * 当要使用一个继承体系时
	 * 1、查看该体系中的顶层类。了解该体系的基本功能
	 * 2、创建体系中的最子类对象，完成功能的使用
	 * 
	 * 当类与类之间存在着所属关系（is a关系）的时候，就可以定义继承
	 * 当本类成员变量同名用super区分父类
	 * this和super的用法很相似。
	 * this代表一个本类对象的引用
	 * super代表一个父类空间
	 * 
	 *  在构造对象的时候，发现，访问子类构造函数时，父类也运行了。
	 * 	原因：在子类的构造函数中的第一行有一个默认的隐式语句。super（）；
	 * 
	 * 子类的实例化过程：子类的所有的构造函数都会默认访问父类中的空参数的构造函数 
	 * */
	 
	class Person_1 {		
		int age;
		String name;
		String gender;
		int a = 1;
	
		Person_1() {
			//super();//显示初始化实际上在super（）；之后运行
			this.age = 0;
			this.name = "小明";
			this.gender = "male";
			System.out.println("Person_1 run");
		}
	
		Person_1(int age, String gender, String name) {
			this.age = age;
			this.name = name;
			this.gender = gender;
			System.out.println("Student run");
		}
	
		void show() {
			System.out.println("show Person_1");
		}
		/*继承弊端：打破了封装性
		 *final 关键字：
		 *1、final是一个修饰符，可以修饰类、方法、变量
		 *2、final修饰的类不可以被继承
		 *3、final修饰的方法不可以被覆盖
		 *4、final修饰的变量时是常量，只能赋值一次（一般会与public和static一起用）
		 *5、final修饰引用数据类型时，地址值不能改变，对象的属性可以改变
		 *
		 *final修饰的class中的方法自动变成final的方法
		 *
		 *为什么要用fianl修饰变量，其实在程序中如果一个数据是固定的，那么直接使用这个数据就可以了，
		 *但是这样阅读性差，所以它该给数据起个名称，而且这个变量名称的值不能变化，所以加上final固定
		 *
		 *写法规范：常量所有字母大写，中间用下划线连接
		 */
		final void setGender(String gender){//防止被覆盖
			this.gender=gender;
		}
		/*final 修饰变量的初始化时机
		 * 		显示初始化
		 * 		在对象构造完毕前即可
		 */
		public static final String COUNTRY = "china";// 全局常量
	}


	class Student_08 extends Person_1 {//继承
		private int age;
	
		Student_08(int age, String gender, String name) {
			super();//如果不写，系统会默认加上，
			调用的就是父类中的空参数的构造函数（父类的构造函数不继承）;必须放在第一行
			/*每一个构造方法都有super(); super（） object类最顶层的父类
			 * 
			 * 如果父类中没有空参构造
			 * 		将子类有参构造的参数赋给super或者赋给this，不能同时使用super和this
			 * 		Son(String name ,int age){
			 * 			super(name,age);
			 * 		}
			 * 		Son(){
			 * 			super(name,age);//自己初始化
			 * 		}
			 * 		或者
			 * 		Son(String name ,int age){
			 * 			super(name,age);
			 * 		}
			 * 		Son(){
			 * 			this(name,age);//此时会调用子类中的有参构造
			 * 		}
			 * */
			this.age = age;
			this.name = name;
			this.gender = gender;
			System.out.println("Student run");
		}
	
		Student_08() {
			this.age = 15;
			this.gender = "female";
			this.name = "小白";
		}
	
		/*当子父类中出现成员函数一模一样的情况(函数声明)，会运行子类的函数，这种现象称为覆盖操作。
	     这时函数在子父类中的特性
		 * 方法的两个特性：
		 * 	  	重载overload		同一个类中。
		 *    	覆盖override		子类中。覆盖也称为重写，覆写
		 *    
		 * 覆盖注意事项：
		 * 	1、子类方法覆盖父类方法时，子类权限必须要大于等于父类的权限，最好是权限一致。
		 * 	2、静态只能覆盖静态，或被静态覆盖（其实并不是覆盖）
		 * 	3、私有方法不能被重写，因为私有不被继承
		 * 	4、返回值类型是一致（可以是父类方法返回值的子类）,参数类型与个数一致
		 * 
		 * 什么时候使用覆盖操作？
		 * 		当对一个子类进行子类的扩展时，子类需要保留父类的功能声明，
		 * 		但是要定义子类中该功能的特有内容时，就使用覆盖操作完成
		 *
		
		 */
		void show() {
			System.out.println("show Student_08");
		}
	
		void Study() {
			//this也可以调用从父类继承的成员
			System.out.println("studying " + this.age + " " + super.age + " " + this.a);
			
		}
	}	

关于代码块和初始化顺序
	
	/*
	 * 用{}括起来的代码就称为代码块
	 * 代码块分类：
	 * 		局部代码块   在方法内					限定变量的生命周期，提早释放内存，提高内存利用                                               
	 * 		构造代码块 	在类中方法外 				   每创建一次对象就执行一次,优先于构造方法
	 * 		静态代码块	static修饰，在类中方法外		用于给类初始化，一般用于加载驱动，优先于主方法执行
	 * 		同步代码块	synchronized			   用于多线程的同步
	 * 
	 * 初始化顺序:
		在一个类里，初始化的顺序是由变量在类内的定义顺序决定的。即使变量定义大量遍布于方法定义的中间，
		那些变量仍会在调用任何方法之前得到初始化——甚至在构建器调用之前。例如
		
	 */
示例

	class Tag {
		Tag(int marker) {
			System.out.println("Tag(" + marker + ")");
		}
	}
	class Card {
		Tag t1 = new Tag(1); // Before constructor
		Card() {
		// Indicate we're in the constructor:
			System.out.println("Card()");
			t3 = new Tag(33); // Re-initialize t3
		}
		Tag t2 = new Tag(2); // After constructor
		void f() {
			System.out.println("f()");
		}
		Tag t3 = new Tag(3); // At end
	}

	public class OrderOfInitialization {
			public static void main(String[] args) {
				Card t = new Card();
				t.f(); // Shows that construction is done
	}
	 
	/* 
	 * 它的输入结果如下：
		Tag(1)
		Tag(2)
		Tag(3)
		Card()
		Tag(33)
		f()
	 * */

### 四、多态
>在面向对象语言中，接口的多种不同的实现方式即为多态。引用Charlie Calverts对多态的描述——多态性是允许你将父对象设置成为一个或更多的他的子对象相等的技术，赋值之后，父对象就可以根据当前赋值给它的子对象的特性以不同的方式运作（摘自“Delphi4 编程技术内幕”）。简单的说，就是一句话：允许将子类类型的指针赋值给父类类型的指针。多态性在Object Pascal和C++中都是通过虚函数实现的。
	
	/*多态polymorphic
	 * 多态的好处
	 * 		提高代码的维护性（由继承保证）
	 * 		提高代码的扩展性（有多态保证）前期定义的代码可以使用后期的内容
	 * 多态的弊端
	 * 		前期定义的内容不能使用（调用）后期子类的特有内容 
	 * 
	 * 多态的前提：
	 * 	a、要有继承关系
	 * 	b、要有方法重写
	 * 	c、要有父类引用指向子类对象
	 * 
	 * 成员变量与static
	 * 	 编译看左边，运行看左边
	 * 成员方法
	 * 	动态绑定（使用method table），编译看左边，运行看右边
	 * 
	 * Foo f= new Bar();
	 * Bar b=(Bar) f;//转型 
	 * 
	 */
	
	public class Polymorphic {
		public static void main(String[] args) {
			Cat c = new Cat();
			c.eat();
			System.out.println(c.age);
			Animal a = new Cat(1);//父类引用指向子类对象
			a.eat();
			System.out.println(a.age);
			method(a);
			AppleTree appleTree =new AppleTree();
			appleTree.grow();
			appleTree.getDescription();
		}
	
		public static void method(Animal a) {
			if (a instanceof Cat) {//instanceof 关键字，判断前面的引用是否是后面的数据类型
				Cat c = (Cat) a;
				c.catchMouse();
				c.eat();
			}
		}
	}
	abstract class Animal {
		int age = 0;
	
		Animal(int age) {
			this.age = age;
		}
	
		abstract public void eat();
	}
	
	class Cat extends Animal {
		int age;
	
		Cat(int age) {
			super(age);
		}
	
		Cat() {
			super(2);
		}
	
		public void eat() {
			System.out.println("cat eats fish" + super.age);
		}
	
		public void catchMouse() {
			System.out.println("mew");
		}
	}

关于抽象（abstract）
	
	/*
	 * 抽象类： 
	 * 	关键字abstract class 类名{}
	 * 特点：
	 *		1、方法只有声明，没有实现时，该方法就是抽象方法，需要被abstract修饰。
	 *			抽象方法必须定义在抽象类中。该类必须也被abstract修饰。
	 *  	2、抽象类不可以被实例化,因为调用抽象方法没意义
	 * 		3、抽象类必须由其子类覆盖了所有的抽象方法后，该子类才可以实例化，
	 * 			否则该子类还是抽象类 。其实这也是一种多态，抽象类多态
	 * 
	 * 抽象类中有构造函数吗？ 
	 * 		有，用于给子类对象初始化
	 * 抽象类可以不定义抽象方法吗？ 
	 * 		可以，但是很少见，目的就是不让该类创建对象。AWT的适配器对象就是这种类 
	 * 
	 * 抽象类中不一定有抽象方法，有抽象方法的一定是抽象类或者接口
	 * 
	 * 通常，这个类中的方法有方法体，但是却没有内容
	 * 
	 * 抽象关键字不可以和哪些关键字共存？ 
	 * 	private不行，私有方法无法被覆盖 
	 * 	static不行，静态方法不需要对象
	 *  final不行，final方法不能被覆盖
	 * 
	 * 抽象类和一般类的异同点：
	 *  相同点：抽象类和一般类都是用来描述事物的，都在内部定义了成员
	 * 	不同点：
	 *  	1、一般类有足够的信息描述事物，抽象类描述事物的信息有可能不足
	 * 		2、一般类中不能定义抽象方法只能定义非抽象方法，抽象类中可以定义抽象方法与非抽象方法
	 * 		3、一般类可以被实例化，抽象类不可以被实例化 抽象类一定是父类吗？
	 * 			是的，抽象类需要子类覆盖其方法后子类才可以被实例化
	 * 
	 */
	abstract class FruitTree {
		private int age;
	
		FruitTree() {
			this.age = 0;
		}
	
		FruitTree(int age) {
			this.age = 0;
		}
		public int getAge(){
			return this.age;
		}
		abstract void grow();
		abstract void getDescription();
	}
	
	class AppleTree extends FruitTree {
		AppleTree(int age) {
			super(age);
		}
		AppleTree(){
			super(0);
		}
		public void grow() {
			System.out.println("grow a apple tree");
		}
		public void getDescription(){
			System.out.println("I'm a "+getAge()+" year-old"+getClass());
		}
	}
	
	class PearTree extends FruitTree {
		PearTree(int age) {
			super(age);
		}
		PearTree(){
			super(0);
		}
		public void grow() {
			System.out.println("grow a pear tree");
		}
		public void getDescription(){
			System.out.println("I'm a "+getAge()+" year-old"+getClass());
		}
	}
关于接口
	
	/*接口:提供额外的扩展功能
	 * 当一个抽象类的方法都是抽象时，这时可将该抽象类用另一种定义表示，就是接口interface
	 * 接口关键字interface
	 * interface 接口名{
	 * 
	 * }
	 * 
	 * 接口的子类可以是抽象类，也可以是具体类（重写接口中的所有抽象方法）
	 * 类与接口之间是实现关系(可以多实现),接口与接口是继承关系（可以多继承）
	 * class A implements B{     B是接口名
	 * }     
	 * 接口中常见的成员，而且这些成员都有固定的修饰符
	 * 		成员变量   	只能是全局常量：public static final(不加修饰符会默认加上)
	 * 		成员方法  	只能是抽象方法：public abstract(不加自动加)接口中成员都是公共权限,定义额外功能
	 *		构造方法	没有
	 * 
	 * 接口不可以实例化，只能由实现了接口的子类并覆盖了接口的所有抽象方法后
	 * 该子类才可以实例化，
	 * 否则这个子类就是一个抽象类
	 * 
	 * 在java中不直接支持多继承，因为会出现调用的不确定性 
	 * 所以java将多继承机制进行了改良，变成了多实现 一个类可以实现多个接口
	 * 
	 */
	interface interA{
		int ONE=1;
		public abstract  void print();
	}
	class B implements interA{
		//实现接口要覆盖接口的方法
	    @Override
		public void print(){
			System.out.println(this.ONE);	
		}
	}


### 五、一个有意思的小示例
这个示例出现在《Java编程思想》这本书中，注意看这个示例的运行结果，你会对Java中的继承有更深入的了解
	
	public class C {
		public static void main(String[] args) {
			new RoundGlyph(5);
		}
	}

	/*
	 *注意这个程序的执行结果：类的方法在对象创建之前加载？
	 * 
	 * 
	 * */
	/*初始化的实际过程是这样的：
	(1) 在采取其他任何操作之前，为对象分配的存储空间初始化成二进制零。
	(2) 就象前面叙述的那样，调用父类(supclass)构建器。
	    此时，被覆盖的draw()方法会得到调用（的确是在RoundGlyph 构建器调用之前），
		此时会发现radius 的值为0，这是由于步骤(1)造成的。
	(3) 按照原先声明的顺序调用成员初始化代码。
	(4) 调用子类(subclass)构建器的主体。*/
	
	/*因此，设计构建器时一个特别有效的规则是：
	 * 用尽可能简单的方法使对象进入就绪状态；如果可能，避免调用任何方法。
		在构建器内唯一能够安全调用的是在基础类中具有final 属性的那些方法（也适用于private方法）
		。这些方法不能被覆盖，所以不会出现上述潜在的问题。
		
	 */
	abstract class Glyph {
		abstract void draw();
	
		Glyph() {
			System.out.println("Glyph() before draw()");
			draw();
			System.out.println("Glyph() after draw()");
		}
	}
	
	class RoundGlyph extends Glyph {
		int radius = 1;
	
		RoundGlyph(int r) {
			radius = r;
			System.out.println("RoundGlyph.RoundGlyph(), radius = " + radius);
		}
	
		void draw() {
			System.out.println("RoundGlyph.draw(), radius = " + radius);
		}
	}												
