---
title: Java基础知识（十八）多线程（四）多线程同步
date: 2015-04-14 23:40:01
tags: 
	- Java基础知识
---
### 一、使用synchronized


	public class Demo_01_synch {
		static int i=0;
		public static void main(String[] args) {
			/*
			 * 同步方法（同步代码块的简写形式）
			 *   使用 synchronized关键字修饰方法
			 * e.g.  public synchronized void show（）{}
			 * 	
			 * */
			/*同步代码块
			 * 使用同步锁（保护数据）
			 * 		锁对象可以使任意的对象（不能使用匿名对象，因为无法保证锁的唯一性）
			 * synchronized(Object obj){
			 * 		code;
			 * }
			 * 
			 * */
			/*同步方法和同步代码块的区别：
			 * 	非静态同步方法的锁是固定的this，同步代码块是任意的对象
			 * 	静态同步方法的锁是当前所属的class文件所属的对象（该类的字节码对象）
			 * 
			 * 建议使用同步代码块
			 * */
			//demo_01();
			demo_02();
			//Collections.synchronizedxxx() 将xxx集合变为同步的
		}
		
		/**
		 * 
		 */
		public static void demo_02() {
			Printer p1=new Printer();
			Thread t0=new Thread(){
				public void run(){
					while(true){
						p1.print1();
					}
				}
			};
			Thread t1=new Thread(){
				public void run(){
					while(true){
						p1.print2();
					}
				}
			};
			t0.start();
			t1.start();
		}
		
		public static void demo_01() {
			final Object obj = new Object();
			Thread t1 = new Thread() {
				Thread t = Thread.currentThread();
				public void run() {
					while (true) {
					synchronized (obj) {
						t.setName("01");
							System.out.println(t.getName() + "------" + i++);
						}
					}
				}
			};
			Thread t2 = new Thread() {
				Thread t = Thread.currentThread();
				public void run() {
					while (true) {
					synchronized (obj) {
						t.setName("02");
							System.out.println(t.getName() + "------" + i++);
						}
					}
				}
			};
			t1.start();
			t2.start();
		}
	}
Printer类

	class Printer{
		public synchronized void print1(){
			System.out.print("1");
			System.out.print("2");
			System.out.print("3");
			System.out.print("4");
			System.out.println("5");
		
		}
		Object obj=new Object();
		public void print2(){
			synchronized(obj){
				System.out.print("a");
				System.out.print("b");
				System.out.print("c");
				System.out.print("d");
				System.out.println("e");
			}
		}
	}

### 二、多线程卖票示例
Ticket类示例：

	class Ticket implements Runnable {//使用接口创建多线程
		private int num = 100;
		private Object object = new Object();
		
		public void sale() {
			/*线程安全问题的原因：
			 * 1、多个线程操作共享的数据
			 * 2、操作共享数据的线程代码有多条
			 * 
			 * 当一个线程在执行共享数据的多条代码的过程中，其他线程参与了运算，
			 * 就会导致线程安全问题的产生
			 */
			while (true) {
				/*解决思路：将多条操作共享数据的线程代码封装起来，当有线程在执行这些代码的时候，
				 * 其他线程是不可以参与运算的，必须要当前线程把这些代码都执行完毕后，其他代码才可以参与运算
				 * 
				 * 在java中使用同步代码块
				 * 	关键字synchronized(对象)  
				 * 		这里的对象是同步锁，标识当前参与运算的线程
				 * 
				 *	同步的好处：解决了线程的安全问题
				 *	同步的弊端:相对降低效率，因为同步外的线程都会判断同步锁
				 * 
				 * 	同步的前提：必须有多个线程并使用同一个锁 
				 * */
				//同步代码块,带有特性的封装，对锁的操作是隐式的
				synchronized (object) {
					if (num > 0) {//可能的线程安全问题
		
						System.out.println(Thread.currentThread().getName() + "..t3.." + num--);
					} else {
						break;
					}
				}
			}
		}
		
		public void run() {
			sale();
		}
		
	}
		
运行示例


	public class Demo_02_Tickets {
	
		public static void main(String[] args) {
			Ticket t = new Ticket();
			Thread t0 = new Thread(t);
			Thread t1 = new Thread(t);
			Thread t2 = new Thread(t);
			Thread t3 = new Thread(t);
			t0.start();
			t1.start();
			t2.start();
			t3.start();
		}
	
	}
	
上面示例中的Ticket类

	class Ticket implements Runnable {
		private int tickets = 10000;
		Object object = new Object();
	
		@Override
		public void run() {
			sale();
		}
	
		/**
		 * 
		 */
		public void sale() {
			while (true) {
				synchronized (object) {
					if (tickets > 0) {
						try {
							Thread.sleep(1);
						} catch (InterruptedException e) {
						}
	
						System.out.println(Thread.currentThread().getName() + ".....sale...." + tickets--);
					}
				}
			}
		}
	
	}	

### 三、死锁示例
常见情景之一:同步的嵌套
	

	class DeadLock implements Runnable {
		private int num = 100;
		Object object = new Object();
	
		public void sale() {
			while (num > 0) {
				synchronized (object) {
					run();
				}
			}
		}
	
		public synchronized void run() {
			synchronized (object) {
				if (num > 0) {
					System.out.println(Thread.currentThread().getName() + "..tt.." + num--);
				}
			}
		}
	}

示例

	public class Demo_03_DeadLock {
		public static void main(String[] args) {
			Dead d1 = new Dead(true);
			Dead d2 = new Dead(false);
			Thread t1 = new Thread(d1);
			Thread t2 = new Thread(d2);
			t1.start();
			t2.start();
	
		}
	}
	
	
Dead 实现了Runnable

	class Dead implements Runnable {//死锁
		private boolean flag;
	
		Dead(boolean flag) {
			this.flag = flag;
		}
	
		public void run() {
			if (flag) {
				synchronized (Lock.lockA) {
					System.out.println("if-----lock a");
					synchronized (Lock.lockB) {
						System.out.println("if-----lock b");
					}
				}
			} else {
				synchronized (Lock.lockB) {
					System.out.println("else-----lock b");
					synchronized (Lock.lockA) {
						System.out.println("else-----lock a");
					}
				}
			}
		}
	}
	
锁

	class Lock {
		public static final Object lockA = new Object();
		public static final Object lockB = new Object();
	}


### 四、等待唤醒机制示例

	public class Demo_03_Wait_Nofity {
		public static void main(String[] args) {
			/*线程间通信：多个线程在处理同一资源，但是任务却不同 
			 * 
			 * */
			/*等待/唤醒机制
			 * 
			 * 涉及的方法：（这些都是Object中的方法）
			 * 	1、wait()		让线程处于冻结状态，被wait的状态会被存储在线程池中（释放锁）
			 *		wait(long timeout)		到时间后停止冻结
			 * 	2、notify()		唤醒线程池中的一个线程（任意的）
			 * 	3、notityAll()	唤醒线程池中的所有线程//低效
			 * 	 
			 * 
			 * 这些方法必须定义在同步中
			 * 因为这些方法是用于操作线程状态的方法
			 * 必须要明确到底操作的是哪个锁上的线程
			 *  * */
			demo_01();
		}
	
		/**
		 * 
		 */
		public static void demo_01() {
			Printer p = new Printer();
			Thread t1 = new Thread() {
				public void run() {
					p.print1();
				}
			};
			Thread t2 = new Thread() {
				public void run() {
					p.print2();
				}
			};
			Thread t3 = new Thread() {
				public void run() {
					p.print3();
				}
			};
			t1.start();
			t2.start();
			t3.start();
		}
	}
Printer类	

	class Printer {
		private int flag = 1;
	
		public void print1() {
			while (true) {
				//
				synchronized (this) {
					while(flag != 1) {
						try {
							this.wait();
						} catch (InterruptedException e) {
	
							e.printStackTrace();
						}
					}
					System.out.print("*-----Emily--------");
					System.out.println("female");
					flag = 2;
					//唤醒所有线程
					this.notifyAll();//锁对象调用notifyAll方法（native），对线程进行操作
				}
			}
		}
	
		public void print2() {
			while (true) {
				synchronized (this) {
					while(flag != 2) {
						try {
							this.wait();
						} catch (InterruptedException e) {
	
							e.printStackTrace();
						}
					}
					System.out.print("**-----小明--------");
					System.out.println("男");
					flag = 3;
					this.notifyAll();
				}
			}
		}
		public void print3() {
			while (true) {
				synchronized (this) {
					while (flag != 3) {
						try {
							this.wait();
						} catch (InterruptedException e) {
	
							e.printStackTrace();
						}
					}
					System.out.print("***-----????--------");
					System.out.println("Alien");
					flag = 1;
					this.notifyAll();
				}
			}
		}
	}

### 五、使用ReentrantLock
	
	public class Demo_04_Lock {
		public static void main(String[] args) {
			/*interface Lock
			 * 一个可以重入的互斥锁Lock，具有和synchronized相同的功能，但是更强大
			 * jdk1.5以后将同步和锁封装成了对象，并将操作锁的隐式动作变成了对象
			 * 
			 * Lock lock=new ReentrantLock()
			 * void show(){
			 * 	lock.lock();//获取锁
			 * 	try{
			 * 		代码....		//如果发生异常，就会出问题，所以释放锁的动作一般放在finally中
			 * 	}
			 * 	finally{
			 * 		lock.unlock();//释放锁
			 * 	}
			 * }
			 * 
			 * */
			/*
			 * Lock接口：
			 * 		替代同步代码块或同步方法，将同步的隐式锁操作变成了显示锁操作
			 * 		同时更灵活，可以一个锁上加多个监视器
			 * 	lock(): 获取锁
			 * 	unlock():释放锁，通常需要定义在finally代码块中
			 * Condition接口：
			 * 		替代了Object中的wait notify notifyAll,将这些监视器单独封装，变成了condition监视器对象
			 * 		可以任意锁组合
			 * 	await()
			 * 	signal()
			 *  singalAll()
			 *  
			 * */
	
			
			demo_01();
	
		}
	
		
		/**
		 * 
		 */
		public static void demo_01() {
			MyPrinter p = new MyPrinter();
			Thread t1 = new Thread() {
				public void run() {
					p.print1();
				}
			};
			Thread t2 = new Thread() {
				public void run() {
					p.print2();
				}
			};
			Thread t3 = new Thread() {
				public void run() {
					p.print3();
				}
			};
			Thread t4 = new Thread() {
				public void run() {
					p.print4();
				}
			};
			t1.start();
			t2.start();
			t3.start();
			t4.start();
		}
	}
MyPrinter类
	
	class MyPrinter implements Runnable{
		private int flag = 1;
		private Lock lock = new ReentrantLock();
		private Condition c1 = lock.newCondition();
		private Condition c2 = lock.newCondition();
		private Condition c3 = lock.newCondition();
		private Condition c4 = lock.newCondition();
	
		public void print1() {
			while (true) {
				lock.lock();
				if (flag != 1) {
					try {
						c1.await();
					} catch (InterruptedException e) {
	
						e.printStackTrace();
					}
				}
				System.out.print("*");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.println("-");
				flag = 2;
				c2.signal();
				lock.unlock();
			}
		}
	
		public void print2() {
			while (true) {
				lock.lock();
				if (flag != 2) {
					try {
						c2.await();
					} catch (InterruptedException e) {
	
						e.printStackTrace();
					}
				}
				System.out.print("*");
				System.out.print("*");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.println("-");
				flag = 3;
				c3.signal();
				lock.unlock();
			}
		}
	
		public void print3() {
			while (true) {
				lock.lock();
				if (flag != 3) {
					try {
						c3.await();
					} catch (InterruptedException e) {
	
						e.printStackTrace();
					}
				}
				System.out.print("*");
				System.out.print("*");
				System.out.print("*");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.println("-");
				flag = 4;
				c4.signal();
				lock.unlock();
			}
		}
	
		public void print4() {
			while (true) {
				lock.lock();
				if (flag != 4) {
					try {
						c4.await();
					} catch (InterruptedException e) {
	
						e.printStackTrace();
					}
				}
				System.out.print("*");
				System.out.print("*");
				System.out.print("*");
				System.out.print("*");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.print("-");
				System.out.println("-");
				flag = 1;
				c1.signal();
				lock.unlock();
			}
		}
	
		@Override
		public void run() {
			int count=0;
			while(count<400){
				System.out.println(Thread.currentThread().getName()+":"+Thread.getAllStackTraces());
				count++;
			}
		}
	}

### 六、多个生产者消费者问题示例

	public class LockDemo {
		public static void main(String[] args) {
			NewResource nr = new NewResource();
	
			NewProducer p = new NewProducer(nr);
	
			NewConsumer c = new NewConsumer(nr);
	
			Thread t0 = new Thread(p);
			Thread t1 = new Thread(p);
			Thread t2 = new Thread(c);
			Thread t3 = new Thread(c);
			t0.start();
			t2.start();
			t1.start();
			t3.start();
		}
	}
		
NewResource类

	class NewResource {
		private String name;
		private int count = 1;
		private boolean flag = false;
		//创建锁对象
		Lock lock = new ReentrantLock();//ReentrantLock可重入锁
		//通过已有的锁获取该锁上的监视器对象
		//	Condition con=lock.newCondition();
		//通过锁获取两组监视器，一组监视生产者，一组监视消费者
		Condition pro_con = lock.newCondition();
		Condition con_con = lock.newCondition();
	
	
		public void set(String name) {
			lock.lock();
			try {
				while (flag) { //while判断标记解决了线程获取执行权后是否要执行
					try {
						pro_con.await();
					} catch (InterruptedException e) {
					}
				}
				this.name = name + count;
				count++;
				System.out.println(Thread.currentThread().getName() + "----生产者----" + this.name);
				flag = true;
				con_con.signal();
			} finally {
				lock.unlock();
			}
		}
	
		public void out() {
			lock.lock();
			try {
				while (!flag) {
					try {
						con_con.await();
					} catch (InterruptedException e) {
	
					}
				}
				System.out.println(Thread.currentThread().getName() + "---------消费者------------" + this.name);
				flag = false;
				pro_con.signalAll();
			} finally {
				lock.unlock();
			}
		}
	}
	
NewProdsucer和NewConsumer类

	class NewProducer implements Runnable {
		private NewResource r;
	
		NewProducer(NewResource r) {
			this.r = r;
		}
	
		public void run() {
			while (true) {
				r.set("commodity");
			}
		}
	}
	
	class NewConsumer implements Runnable {
		private NewResource r;
	
		NewConsumer(NewResource r) {
			this.r = r;
		}
	
		public void run() {
			while (true) {
				r.out();
			}
		}
	}

### 七、单例模式和多线程

单例设计模式(饿汉式)

	/* 空间换时间
	 * 
	 * 不会出现创建多个对象的问题
	 * */
	class Single1 {
		//final防止对象被修改
		private static /*final*/ Single1 s = new Single1();
		//私有构造方法
		private Single1() {
		};
		//使用本类的方法创建对象
		public static Single1 getInstance() {
			return s;
		}
	
	}
	
延迟加载单例设计设计模式(懒汉式)

	/* 时间换空间
	 * */
	class Single2 {
		private static Single2 s = null;
	
		private Single2() {
		};
	
		public static Single2 getInstance() {
			//此时多线程有安全隐患
			/*if(s==null){
				s=new Single1();
			}*/
			//改写成同步方法会降低效率，多一步判断锁的过程
			if (s == null) {//双重判断，提高效率
				synchronized (Single2.class) {//使用同步代码块
					if (s == null) {
						s = new Single2();//对数据使用同步以防止多线程出问题
					}
				}
			}
			return s;
		}
	}