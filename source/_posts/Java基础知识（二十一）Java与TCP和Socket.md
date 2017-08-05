---
title: Java基础知识（二十一）Java与TCP和Socket
date: 2015-04-15 12:56:28
tags: 
	- Java基础知识
---
### 一、关于HTTP和URL，你需要知道的
* HTTP协议

		/*
		 * http hypertext transfer protocol
		 * 	规范browser与server数据传输的格式
		 */
		
		/*
		 * 
		 * 请求信息的组成
		 * 
		 * 	GET //hq_04servlet//servlet/Test_01 HTTP/1.1		//请求行  请求方式+资源路径+协议版本
			Host: localhost:8080								//请求头，以下到空行都是请求行
			Connection: keep-alive
			Cache-Control: max-age=0
			Upgrade-Insecure-Requests: 1
			User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.59 Safari/537.36
			Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,/*;q=0.8
			Accept-Encoding: gzip, deflate, sdch, br
			Accept-Language: zh-CN,zh;q=0.8
		 * 														//空行					
		 * 														//实体内容
		 * 
		 * 
		 * 注意：
		 * 	如果是post，所有的请求行都在实体内容里
		 * 
		 */			
		/*
		 * 
		 *  a.	post方式
				1.	提交的数据在实体内容上， 不在url地址栏上显示。
				2.	没有大小限制。
				3.	数据更加安全
				4.	提交的数据是不会产生用缓存文件（cookie）的。
						
			b.	get 方式 
				1.	提交的数据在地址栏上。
				2.	有大小限制，大小容量不能超过1kb。
				3.	敏感数据不能使用get方式。
				4.	提交的数据是会产生缓存文件（记录页面数据和服务器资源文件的最后修改时间）
				
				（1）静态资源使用缓存文件的情况
						第一次访问静态资源的时候，浏览器会对改资源生成缓存文件，
						第二次访问的时候直接使用缓存文件即可,不会向服务器请求新的资源
						
				（2）动态资源使用缓存文件的情况
						每一次访问都会请求新的资源（如果目标内容没有被修改，建议直接使用缓存文件）
		 *				因为Servlet中的getLastModified()写了返回-1
		 *				如果不想每一次都请求新的资源，可以 重写getLastModified()方法
		 */
注意：http1.0每次连接服务器只能请求一个资源，1.1可以请求多个资源

* URL是什么

		/*
		 * URL  Uniform Resource Locator
		 * 统一资源定位符
		 * 
		 * URI 
		 * 统一资源标识符  每一个URL都是URI 
		 * 
		 * URN
		 * 统一资源名称
		 * */
		URL url=new URL("http://192.168.153.1:8080/web/1.html");
		//获取url对象的url连接器，将连接封装成对象;java中内置的可以解析的具体协议的对象+socket
		URLConnection con=url.openConnection();

### 二、Socket通信的示例

Server代码

	import java.io.BufferedReader;
	import java.io.IOException;
	import java.io.InputStreamReader;
	import java.io.PrintStream;
	import java.net.ServerSocket;
	import java.net.Socket;
	
	public class TCP_IP_Server implements Runnable {
	
		public void run() {
			/*建立TCP服务端
			 * ServerSocket 
			 * 		此类实现服务器套接字。服务器套接字等待请求通过网络传入
			 * 
			 * */
			try {
				ServerSocket server = new ServerSocket(4999);
				//获取连接过来的客户端对象
				
				Socket client = server.accept();//接收客户端的请求
				
				//获取socket流
				BufferedReader br = new BufferedReader(new InputStreamReader(client.getInputStream()));
				PrintStream ps = new PrintStream(client.getOutputStream(),true);
				
				String line = null;
				System.out.println("server at service");
	
				while (true) {
					line=br.readLine();
					line = new StringBuilder(line).reverse().toString();
					ps.println(line);//阻塞式方法，添加結束標記
					
					System.out.print("get:");
					System.out.println(line);
					if (line.equals("exit")) {
						client.close();
						//break;
					}
				}
			} catch (IOException e) {
				System.out.println(e);
			}
		}
	}
Client代码

	import java.io.BufferedReader;
	import java.io.IOException;
	import java.io.InputStreamReader;
	import java.io.PrintStream;
	import java.net.Inet4Address;
	import java.net.Socket;
	import java.net.UnknownHostException;
	import java.util.Scanner;
	
	public class TCP_IP_Client implements Runnable {
		/*Socket间的通信
		 *数据在Socket之间通过IO传输
		 **/
		/*Socket：
		 * 此类实现客户端套接字。套接字是两台机器间通信的端点
		 * 		
		 * */
		public void run() {
			try {
				Scanner in = new Scanner(System.in);
				//使用Socket对象创建tcp客户端的socket服务
				
				//指定要连接的主机IP和端口号
				Socket socket = new Socket("192.168.100.1", 4999);
				/*如果连接成功就说明数据传输通道已经建立好
				 * 	该通道是socket流，是底层建立的（字节流）
				 * 	既有输出也有输入
				 * */
				//通过Socket获取输入输出流对象
				
				BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
				PrintStream ps = new PrintStream(socket.getOutputStream(),true);//設置自動刷新
				String s=null;
				while (true) {
					ps.println(in.nextLine());//将字符串写出服务器
					
					s=br.readLine();
					System.out.println("intput:");
					System.out.println(s);
					if(s.equals("exit")){
						break;
					}
				}
				socket.close();
			} catch (IOException e) {
				e.printStackTrace();
			} 
	
		}
	}

测试

	public class TCP_IP_Test {
		public static void main(String[] args){
			Thread t0=new Thread(new TCP_IP_Client());
			Thread t1=new Thread(new TCP_IP_Server());
			t0.start();
			t1.start();
		}
	}




