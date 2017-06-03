---
title: Java基础知识（二十二）Java连接数据库
date: 2015-04-15 16:07:45
tags: 
	- Java基础知识
---
### 一、关于数据库
	
	自己看书去

## 二、示例代码

	/**
	 * @throws ClassNotFoundException
	 * @throws SQLException
	 */
	public static void demo_01() throws ClassNotFoundException, SQLException {
		/*面向接口开发
		 * 
		 * Java Data Base Connectivity java数据库连接
		 * 执行SQL语句的API
		 * 
		 * 驱动也是类库
		 * 其实现类一般由写数据库的公司写
		 * 
		 * */
		
		/*	1、注册驱动
		 * 		告知JVM使用的是哪一个数据库的驱动
		 */
		//java.sql.DriverManager 类的静态方法registerDriver(Driver driver)
		//Driver接口，MySQL驱动程序的实现类,
		//使用构造会二次注册，浪费资源
		//一般使用反射Class.forName()
		//常用
		Class.forName("com.mysql.jdbc.Driver");
		/*
		 * 此时会执行Driver的静态代码块创建一个实例，然后调用DriverManager.registerDriver()注册
		 * 
		 * */
		//注册驱动的另外两种方式
		//System.setProperty("jdbc.driver", "com.mysql.jdbc.Driver");
		//new com.mysql.jdbc.Driver();
		
		
		
		/*
		 * 	2、获得链接
		 * 		使用JDBC中的类，完成对MySQL的数据库连接
		 */
		//static Connection getConnection(String url.String user,String password)
		//返回值是 Connection 的实现类，在mysql驱动程序
		//url数据库连接地址 jdbc:mysql://连接主机IP:端口号//数据库名字
		String url="jdbc:mysql://localhost:3306/homework";
		String username="root";
		String password="root";
		Connection con=DriverManager.getConnection(url,username,password);
		System.out.println(con);
		
		
		/*
		 * 	3、获得语句执行平台
		 * Statement
		 * 		通过连接对象获取对SQL语句的执行者对象
		 */
		// con对象调用方法，Statement createStatement() 获取	Statement对象，将SQL语句发送到数据库
		//  返回值是Statement接口的实现类，在mysql驱动对象
		//Statement statement = con.createStatement();//容易被注入攻击，不推荐使用
		
		
		
		/*
		 * 	4、执行sql语句
		 * 		使用执行者对象，向数据库执行SQl语句
		 * 		获取到数据库执行后的结果
		 */
		//int executeUpdate(String sql) 执行数据库的sql语句
		//返回值int，操作成功数据的行数
		//String sql="insert into sort(sname,sprice,sdesc) values('药',1000,'biotech')";
		//int row=statement.executeUpdate(sql);
		//TestSystem.out.println(row+" executed");
		
		//String sql1=in.nextLine();//输入' or true '  实现注入攻击
		String sql1="select * from sort where sid=?";
		//调用Connection的preparedStatement()，实现参数绑定
		PreparedStatement ps=con.prepareStatement(sql1);
		ps.setObject(1, 5);//使用set设置？的参数
		
	
		
		/*
		 * 	5、处理结果
		 * 		
		 */
		//ResultSet executeQuery(Strin sql) 执行sql语句中的select查询
		//返回值ResultSet接口的实现类对象，实现类在mysql中
		//ResultSet rs=statement.executeQuery(sql1);
		ResultSet rs=ps.executeQuery();//使用preparedstatement不需要传参数
		while(rs.next()){//next() 将光标下移一行，返回值boolean,没有结果返回false
			//行数从1开始
			//getInt() getString() getObejct() .etc
			System.out.println(rs.getInt("sid")+"\t"+rs.getString("sname")+"\t\t"+rs.getDouble("sprice")+"\t\t"+rs.getString("sdesc"));
		}
		
		
		/*
		 * 	6、释放资源
		 * 		调用close()关闭资源
		 * */
		rs.close();
		ps.close();
		con.close();
	}