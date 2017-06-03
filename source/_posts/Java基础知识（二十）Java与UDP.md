---
title: Java基础知识（二十）Java与UDP
date: 2015-04-15 10:13:01
tags: 
	- Java基础知识
---
### UDP简单示例

* 关于UDP的简单介绍

		/*协议：
		 * 为计算机网络中进行数据交换而建立的规则、标准或约定的集合
		 * 
		 * UDP（数据报文协议）：
		 * 	将数据和源以及目的封装成数据包
		 * 	不需要建立连接
		 * 	每个数据包大小控制在64内
		 * 	面向无连接，数据不安全，速度快，不分客户端和服务端
		 * 
		 * TCP（传输控制协议）：
		 * 	面向连接（三次握手），数据安全，速度略低，区分客户端和服务端
		 * 	需要建立连接
		 * 	三次握手：客户端先向服务端发送请求，服务端响应请求，传输数据
		 * 
		 * 
		 * 
		 * */
		/*
		 * 端口号：
		 * 	用于标识进程的逻辑地址
		 * 	(区分不同的网络服务)
		 * 	
		 * 	int数（0-65535）
		 *	客户端又称为临时端口号，因为它只在用户运行该客户程序时才存在，而服务器则只要主机开着，其服务就运行 
		 * 
		 * 	大多数TCP/IP是实现给临时端口分配1024-5000之间的端口号，
		 * 	大于5000一般给其他服务器预留（Internet上不常用的服务）
		 * 	尽量使用1024以上
		 * 	256-1023之间的通常由Unix系统占用
		 * 	IANA管理1-1023之间的所有端口号
		 * 
		 * 	防火墙原理：禁用相关程序的端口
		 * 
		 * e.g. 192.168.1.1:2425
		 * 
		 * mysql	3306
		 * oracle	1521
		 * web		80
		 * tomcat	8080
		 * qq		4000
		 * */

* 示例


		import java.io.IOException;
		import java.net.DatagramPacket;
		import java.net.DatagramSocket;
		import java.net.Inet4Address;
		import java.net.SocketException;
		import java.net.UnknownHostException;
		import java.util.Scanner;
		
		public class Demo_UDP {
			public static void main(String[] args) throws IOException, InterruptedException {
				/*
				 * 
				 * UDP传输
				 * 	DatagramPacket	表示数据报包，不对包的投递做出保证
				 * 	DatagramSocket	标识用来发送和接收数据报包的套接字
				 * 
				 * 
				 * 1、发送
				 * 	创建DatagramSocket，随机端口号
				 * 	创建DatagramPacket，指定数据，长度，地址，端口
				 * 	使用DatagramSocket发送DataProgramPacket
				 * 	关闭DatagramSocket
				 * 
				 * */
				/*
				 * 2、接收
				 *	创建DatagramSocket，随机端口号
				 * 	创建DatapgramPacket，指定数组，长度
				 * 	使用DatagramSocket发送DataProgramPacket
				 * 	关闭DatagramSocket
				 * 	从DatagramPacket中获取数据
				 *	 
				 * */
				//		demo_01();
				demo_02();
			}
		
			/**
			 * @throws SocketException
			 * @throws UnknownHostException
			 * @throws IOException
			 */
			public static void demo_01() throws SocketException, UnknownHostException, IOException {
				String s = "hello there";
				DatagramSocket dgs = new DatagramSocket();
				DatagramPacket dgp = new DatagramPacket(s.getBytes(), s.getBytes().length,
						Inet4Address.getByName("192.168.197.1"), 9999);//IP可以设置为广播，群聊，255是网段的广播，这里用的是本机的IP地址
				dgs.send(dgp);
				dgs.close();
			}
			public static void demo_02() throws InterruptedException{
				Thread t0=new Send();
				Thread t1=new Receive();
				t0.start();
				t1.start();
			}
		}
		
	Send类

		class Send extends Thread {
			public void run() {
				System.out.println("sender ready");
				try {
					Scanner in = new Scanner(System.in);
					DatagramSocket dgs = new DatagramSocket();
					while (true) {
						String line = in.nextLine();
						if (!"exit".equals(line)) {
							DatagramPacket dgp = new DatagramPacket(line.getBytes(), line.getBytes().length,
									Inet4Address.getByName("192.168.197.1"), 4999);//创建用于发送的包,包含目的IP地址
							dgs.send(dgp);
						}else{
							break;
						}
		
					}
					dgs.close();
				} catch (Exception e) {
					System.out.println(e);
				}
			}
			
		}

	Recieve类

		class Receive extends Thread {
			public void run() {
				System.out.println("receiver ready");
				try {
					DatagramSocket dgs = new DatagramSocket(4999);
					DatagramPacket dgp = new DatagramPacket(new byte[8192], 8192);//创建用于接收的包
					while (true) {
						dgs.receive(dgp);
						byte[] arr = dgp.getData();
						int len = dgp.getLength();
						String text=new String(arr, 0, len);
						String ip = dgp.getAddress().getHostAddress();
						int port = dgp.getPort();
						System.out.println(ip+":"+port+":"+text);
						if(text.equals("exit")){
						break;	
						}
					}
					dgs.close();
				} catch (Exception e) {
					System.out.println(e);
				}
			}
		}



