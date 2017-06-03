---
title: Java基础知识（十八）多线程（三）Thread简单示例
date: 2015-04-14 22:11:34
tags: 
	- Java基础知识
---
### 一、Thread的方法
	
* 构造方法

		//target为实现Runnable接口后的类
		Thread t=new Thread(target,name);
		Thread t=new Thread(target);
		Thread t=new Thread();

* 设置为守护线程

		/*
		 * Daemon守护线程：
		 * 		和一般线程一样，只是在没有非守护线程运行时守护线程会自动结束
		 * 		
		 * setDaemon(boolean b)		设置为守护线程
		 * 
		 * 守护线程不应该去访问固有资源，因为它可能随时中断
		 * */
		public static void demo_01() {
		Thread t1=new Thread(){
			public void run(){
				Thread t=Thread.currentThread();
				t.setName("非守护线程");
				int i=0;
				while(i<100){
					System.out.println(t.getName()+"------"+i);
					i++;
				}
			}
		};		
		Thread t2=new Thread(){
			public void run(){
				Thread t=Thread.currentThread();
				t.setName("Daemon");
				int i=0;
				while(true){
					System.out.println(t.getName()+"-------"+i);
					i++;
				}
			}
		};
		t2.setDaemon(true);
		t1.start();
		t2.start();
	}

* 加入线程

		public class Demo_04_join {
			static int i;
			public static void main(String[] args){
				/*join()加入线程
				 * 当前线程暂停，等待指定的线程执行结束后，当前线程再继续
				 * 
				 * join(int time) 插队执行time毫秒，time之后交替执行
				 * 
				 * 
				 * 
				 * */
				final Thread t1=new Thread(){
					public void run(){
						Thread t=Thread.currentThread();
						t.setName("t1");
						while(true){
							System.out.println(t.getName()+"-------"+i++);
						}
					}
				};
				Thread t2=new Thread(){
					public void run(){
						Thread t=Thread.currentThread();
						t.setName("t2");
						while(true){
							if(i%3==0){
								try {
									t1.join(10);
								} catch (InterruptedException e) {
									
									e.printStackTrace();
								}
							}
							System.out.println(t.getName()+"--------"+i++);
						}
					}
				};
				t1.start();
				t2.start();
			}
		}


* yield和priority

	没有示例

		/*
		 * static void yield()
		 * 使当前线程处于让步状态 
		 * 使其他优先级不高于它的线程被调度
		 * 	并不一定立即让出线程
		 * 
		 * */	

		/*void	setPriority(int newPriority)
		 * 设置优先级，默认是5
		 * static	int	MIN_PRIORITY		1	
		 * static	int MAX_PRIORITY		10
		 * 优先级越高cpu分配的处理时间越多
		 * 
		 * 
		 * 两个相邻的数并没有太大的区别
		 * 
		 * */

* 线程组


		public class Demo_05_ThreadGroup {
			public static void main(String[] args){
					/*ThreadGroup
					 * 
					 * 线程组：
					 * 	将线程编组
					 * */
					/* Java中使用ThreadGroup来表示线程组，它可以对一批线程进行分类管理，Java允许程序直接对线程组进行控制。
					* 默认情况下，所有的线程都属于主线程组。
					* public final ThreadGroup getThreadGroup()//通过线程对象获取他所属于的组
					* public final String getName()//通过线程组对象获取他组的名字
					* 我们也可以给线程设置分组
					* 1,ThreadGroup(String name) 创建线程组对象并给其赋值名字
					* 2,创建线程对象
					* 3,Thread(ThreadGroup?group, Runnable?target, String?name) 
					* 4,设置整组的优先级或者守护线程*/
					
					// 默认情况下，所有的线程都属于同一个组
					System.out.println(Thread.currentThread().getThreadGroup().getName());

					
					demo_02();
				}
			

			public static void demo_02() {
				ThreadGroup tg=new ThreadGroup("A_Demo_ThreadGroup");//创建新的线程组
				Printer p=new Printer();						    //创建一个实现了Runnable接口的类
				Thread t0=new Thread(tg,p,"thread_01");				//将线程一放在组中
				Thread t1=new Thread(tg,p,"thread_02");
				System.out.println(tg.getName());					
				System.out.println(t0.getThreadGroup().getName());
				System.out.println(t1.getThreadGroup().getName());
				System.out.println(t0.getName());
				System.out.println(t1.getName());
				tg.setDaemon(true);									//将tg设置为守护线程
			}
			
		}

* 其他

		/*
		 * 停止线程：
		 * 	1、stop()方法 	     	已过时
		 * 	2、run()方法结束时			没有唤醒时会出问题
		 * 		怎么控制程序的结束
		 * 		任务中都会有循环结构，只要控制循环，就可以结束任务
		 * 
		 * 		控制循环通常用定义标记来实现
		 * 	3、interrupt()			中断线程的冻结状态，将线程强制恢复到运行状态，
		 * 							让线程具备cpu的执行资格
		 * 							但是会抛出InterruptedException，记得要在catch中处理
		 * 
		 * 
		 * */
注意：停止线程推荐使用一个标志变量来终止