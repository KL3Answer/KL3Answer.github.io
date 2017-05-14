---
title: Java基础知识（八） 一些API
date: 2015-3-20 2:01:00
tags: 
	- Java基础知识
---
### 一、时间和日期
	
* Date
		
		//1970年1月1日0：00 am 星期四 	Unix和C语言的生日，也是绝大多是系统的开始时间
		/*
		 * Date类
		 * 	java.util.Date
		 * 
		 * 年份y由0-1900的整数表示
		 * 月份	int	0-11
		 * 小时	int	0-23
		 * 分钟	int	0-59
		 * 秒		0-61	60和61只对闰秒发生
		 * 
		 * 星期日到星期六对应为1-7（美国）
		 * 
		 * 构造方法
		 * 	public Date()				如果没有传参数，代表的系统当前的时间
		 * 	public Date(long date)
		 * 	
		 * 常用成员方法
		 * 	public long 	getTime()				通过时间对象获取毫秒值时间（从初始的时间到当前的时间的时间间隔）
		 * 	public void 	setTime(long time)		设置毫秒值，改变时间对象
		 * 	public boolean 	before(Date when)
		 * 	public boolean 	after(Date when)
		 * 	public int 		compareTo(Date anotherDate)
		 * */
		Date d0=new Date();
		System.out.println(new Date(0));
		System.out.println(new Date());
* DateFormat

		/*
		 *DateFormat类（Date的子类，抽象类）
		 * 
		 * getDateInstance()  返回子类对象
		 * format()				格式化日期，返回String/StringBuffer
		 * 
		 *SimpleDateFormat类（DateFormat类的子类）
		 * 
		 * */
		
		//日期对象转字符串
		//1、定义日期格式对象
		//		DateFormat df=DateFormat.getDateTimeInstance();
		//		sdf = new SimpleDateFormat("yyyy年MM月dd日HH:mm:ss");//创建时间格式化类对象
		//或者
		//创建时间格式化类对象
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日HH:mm:ss");
		//2、toString()
		System.out.println(sdf.format(new Date()));//将Date格式化
		
		//将字符串转成对象
		String str = "2011年8月17日8:13:33";
		//1、定义日期格式对象
		SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy年M月dd日H:mm:ss");
		//DateFormat df=DateFormat.getDateInstance(DateFormat.LONG);
		//2、将字符串按照格式转换成日期值
		Date d = sdf1.parse(str);//将字符串按照格式转换成日期值
		//3、计算时间差
		long time = new Date().getTime() - d.getTime();
		time = time / 1000 / 60 / 60 / 24;
		System.out.println(time);
	
* Calendar
		
		/* Calendar类（抽象类）
		 *	直接子类 GregorianCalendar
		 *常用方法
		 *public	static  Calendar 	getInstance()  	
		 *public 			int			get(int field)			 
		 *publi				void		add(int field ,int amount)
		 *public	final	void		set(int year,int month,int date)
		 *
		 *
		 * */
		Calendar c = Calendar.getInstance();//返回的是Calendar子类对象
		System.out.println(c.get(Calendar.YEAR));

上面的日期和时间类可能会让你感觉很麻烦，因此在Java 8中，增加了新的Date-Time API (JSR 310)来进一步加强对日期与时间的处理。

1、Clock：
		
	Clock clock = Clock.systemUTC();
	System.out.println( clock.instant() );
	System.out.println( clock.millis() );
现在Clock类可以代替System.currentTimeMillis()与TimeZone.getDefault()。

2、Instant	
	
	System.out.println(Instant.now());
获取当前的UTC时间

3、LocalDate和LocalTime
	
	Clock clock = Clock.systemUTC();
	// Get the local date and local time
	LocalDate date = LocalDate.now();
	LocalDate dateFromClock = LocalDate.now( clock );
	         
	System.out.println( date );
	System.out.println( dateFromClock );
	         
	// Get the local date and local time
	LocalTime time = LocalTime.now();
	LocalTime timeFromClock = LocalTime.now( clock );
	         
	System.out.println( time );
	System.out.println( timeFromClock );
获取本地的当前时间（无时区信息）

4、ZonedDateTime
	
	// Get the zoned date/time
	ZonedDateTime zonedDatetime = ZonedDateTime.now();
	ZonedDateTime zonedDatetimeFromClock = ZonedDateTime.now( clock );
	ZonedDateTime zonedDatetimeFromZone = ZonedDateTime.now( ZoneId.of( "America/Los_Angeles" ) );
	         
	System.out.println( zonedDatetime );
	System.out.println( zonedDatetimeFromClock );
	System.out.println( zonedDatetimeFromZone );
获取特定时区的日期/时间。

5、Duration
	
	// Get duration between two dates
	LocalDateTime from = LocalDateTime.of( 2016, Month.MARCH, 8, 0, 0, 0 );
	LocalDateTime to = LocalDateTime.of( 2017, Month.MARCH, 8, 10, 59, 59 );
	 
	Duration duration = Duration.between( from, to );
	System.out.println( "Duration in days: " + duration.toDays() );
	System.out.println( "Duration in hours: " + duration.toHours() );
上面的例子计算了两个日期2016年3月8号与2017年3月8号之间的过程。


### 二、Runtime
每个Java应用程序都有一个Runtime实例，使应用程序与运行环境相连接，可以通过getRuntime方法获取当前的运行时，应用程序不能创建自己的Runtime类实例（构造器是private的，使用了单例设计模式)。

	Runtime rt=Runtime.getRuntime();//Runtime中唯一的静态方法，用以创建对象
	//在cmd中执行
	rt.exec("notepad.exe");//会抛出异常
	//返回一个Process
	
	//可以尝试一下这个指令(笑)
	rt.exec("shutdown -s -t 20");
	//执行完上面的之后执行这个
	rt.exec("shutdown -a");

	//获取内存信息
	System.out.println("Total Memory = "
		+ rt.totalMemory()
		+ " Free Memory = "
		+ rt.freeMemory());
		}

### 三、Math
* Math
	
		/*
		 *Math类(final)
		 * 常用：（都是static）
		 * public 	static	double		PI							π
		 * public 	static 	int 		abs(int a)					取绝对值
		 * public 	static 	int 		max(int a,int b) 			获取最大值
		 * public 	static 	int 		round(float a) 				对a四舍五入
		 * public 	static 	double 		ceil(double a)				向上取整，结果是double值
		 * public 	static 	double 		floor(double a)				向下取整，结果是double值
		 * public 	static 	double 		pow(double a,double b)		a的b次方
		 * public 	static 	double 		random()					生成[0.0,1.0)的随机数
		 * public 	static	double 		sqrt(double a)				对a开平方
		 * */
* Random
	
		/*Random类
		 * 常见：
		 * nextInt(int n)
		 * nextLong(long l)
		 * nextdouble(double d)
		 * 	.etc
		 * 
		 * */
		Random r = new Random();//
		System.out.println(r.nextInt(100));//生成[0,n)范围内的随机数

* BigInteger

		/*
		 * BigInteger类
		 * 	可以让超过Integer范围的数进行运算
		 * 
		 * 构造方法
		 * 	常用：
		 * 	public BigInteger(String val)
		 * 成员方法
		 * 	public BigInteger	add(BigInteger val)
		 * 	public BigInteger	subtract(BigInteger val)
		 * 	public BigInteger	mutiply(BigInteger val)
		 * 	public BigInteger	divide(BigInteger val)
		 * 	public BigInteger	divideAndRemainder(BigInteger val)  返回商和余数两个数并存储在数组		
		 * 
		 * */

* BigDecimal
		
		/* BigDecimal类
		 * 	任意精度的十进制数
		 * 
		 * 构造方法
		 * 	常用：
		 * 	public BigDecimal(String val)
		 * 	public BigDecimal(double val)
		 * 常见成员方法
		 * 	public BigDecimal	add(BigDecimal val)
		 * 	public BigDecimal	subtract(BigDecimal val)
		 * 	public BigDecimal	mutiply(BigDecimal val)
		 * 	public BigDecimal	divide(BigDecimal val)
		 * 	public BigDecimal	divideAndRemainder(BigDecimal val)  返回商和余数两个数并存储在数组		
		 * 
		 * */
		System.out.println("------------------------");
		//此时使用double不够精确
		System.out.println(new BigDecimal(2.0).subtract(new BigDecimal(0.1)));
		
		BigDecimal bd1 = new BigDecimal("2.0");//使用字符串结果更精确，常用
		BigDecimal bd2 = new BigDecimal("0.1");
		System.out.println(bd1.subtract(bd2));
		
		BigDecimal bd3 = BigDecimal.valueOf(2.0);//常用
		BigDecimal bd4 = BigDecimal.valueOf(0.1);
		System.out.println(bd3.subtract(bd4));

### 四、Regex正则
>在1951 年,一位名叫Stephen Kleene的数学科学家，他在Warren McCulloch和Walter Pitts早期工作的基础之上，发表了一篇题目是《神经网事件的表示法》的论文，利用称之为正则集合的数学符号来描述此模型，引入了正则表达式的概念。正则表达式被作为用来描述其称之为“正则集的代数”的一种表达式，因而采用了“正则表达式”这个术语。

在计算机中，正则常被用于检索符合某些模式的文本。

1、 字符类：
	
	/* 
	 * 字符类
	 * [abc] a或b或 c（简单类） 
	 * [^abc] 任何字符，除了 a、b 或 c（否定） 
	 * [a-zA-Z] a到 z 或 A到 Z，两头的字母包括在内（范围） 
	 * [0-9] 0到9的字符都包括
	 * */
	/*预定义字符类
	 *  . 	匹配除换行以外的任何字符。
	 * \d 	数字：[0-9]
	 * \w 	单词字符：[a-zA-Z_0-9]
	 * \f	匹配一个换页符。等价于 \x0c 和 \cL。
	 * \n	匹配一个换行符。等价于 \x0a 和 \cJ。
	 * \r	匹配一个回车符。等价于 \x0d 和 \cM。
	 * \s	匹配任何空白字符，包括空格、制表符、换页符等等。等价于 [ \f\n\r\t\v]。
	 * \S	匹配任何非空白字符。等价于 [^ \f\n\r\t\v]。
	 * \t	匹配一个制表符。等价于 \x09 和 \cI。
	 * \v	匹配一个垂直制表符。等价于 \x0b 和 \cK。
	 * */

示例：
	
	String regex = "[1-9]\\d{4,14}";
	System.out.println("1819440".matches(regex));


2、匹配数量词
	
	Greedy 数量词
	X?	X，一次或一次也没有
	X*	X，零次或多次
	X+	X，一次或多次
	X{n}	X，恰好 n 次
	X{n,}	X，至少 n 次
	X{n,m}	X，至少 n 次，但是不超过 m 次
	正则表达式中,默认情况下基本上都是使用Greedy数量词
	例如字符串"qwert12345",可以匹配它的正则表达式为".*5".其中"."可以匹配任何字符,
	"*"表示匹配0次或多次,程序在编译运行时,"qwert12345"字符串本身可以被".*"匹配,
	但是后边多了个"5"无法匹配,因此回溯一个字符,正好匹配了"5",所以返回true.
	
	Reluctant 数量词
	X??	X，一次或一次也没有
	X*?	X，零次或多次
	X+?	X，一次或多次
	X{n}?	X，恰好 n 次
	X{n,}?	X，至少 n 次
	X{n,m}?	X，至少 n 次，但是不超过 m 次
	Reluctant数量词在匹配时,与Greedy的匹配模式是相反的,它从左到右一个一个进行匹配,
	而不是匹配字符串再进行回溯
	例如字符串"qwert12345",".*?3"也是匹配的,因为从左到右一个一个看只有整个字符串才能匹配,
	而对于正则表达式".+?"来说,职匹配最左边的字符q,如果正则表达式是".*?"则意味着没有匹配任何字符串.
	
	Possessive 数量词
	X?+	X，一次或一次也没有
	X*+	X，零次或多次
	X++	X，一次或多次
	X{n}+	X，恰好 n 次
	X{n,}+	X，至少 n 次
	X{n,m}+	X，至少 n 次，但是不超过 m 次
	Possessive数量词与Greedy数量词类似,也是匹配字符串.唯一的不同时Possessive不回溯,
	所以对于"qwert12345",".*+3"不匹配.

3、捕获组

捕获组就是把正则表达式中子表达式匹配的内容，保存到内存中以数字编号或显式命名的组里，方便后面引用。当然，这种引用既可以是在正则表达式内部，也可以是在正则表达式外部。
捕获组有两种形式，一种是普通捕获组，另一种是命名捕获组，通常所说的捕获组指的是普通捕获组。

*  捕获组及其编号：
   
		1) 捕获组就是匹配到的内容，按照()子表达式划分成若干组；
    	2) 例如正则表达式：(ab)(cd(ef))就有三个捕获组，没出现一对()就是一个捕获组
    	3) 捕获组编号规则：
         a. 引擎会对捕获组进行编号，编号规则是左括号(从左到右出现的顺序，从1开始编号；
         b. 例如：
			(\d{4})-(\d{2}-(\d\d))中有三个捕获组
* 反向引用
    
		1) 捕获组的作用就是为了可以在正则表达式内部或者外部（Java方法）引用它；
	    2) 如何引用？当然是通过前面讲的用捕获组的编号来引用咯！
	    3) 正则表达式内部引用：
	         i. \X：X是一个十进制数，X的范围必须落在捕获组编号范围之内，该表达式就匹配X号捕获组所匹配到的内容；
	         ii. 从上面的描述可以看出，\X匹配的内容是必须X号捕获组匹配成功之后才能确定的！
	         iii. 例如：([ab])\1，匹配aabbcc的结果是aa和bb，\1的内容必须要让1号捕获组捕获后才能确定，
					如果1号捕获的是a那么\1就是a，1号捕获到了b那么\1就是b；
	    4) 正则表达式外部引用：就是用Matcher对象的start、end、group查询匹配信息时，使用捕获组编号对捕获组引用（int group）；
* 捕获组命名
	
		1) 如果捕获组的数量非常多，那都用数字进行编号并引用将会非常混乱，并且难以记忆每个捕获组的内容及意义，因此对捕获组命名显得尤为重要；
    	2) Java 7开始提供了对捕获组命名的语法，并且可以通过捕获组的名称对捕获组反向引用（内外都行）；
         i. 命名捕获组的语法格式：(?<自定义名>expr)
         ii. 例如：(?<year>\d{4})-(?<date>\d{2}-(?<day>\d{2}))
             a. 有三个命名捕获组year、date和day
             b. 从左到右编号分别为1、2、3（编号同样是有效的）
    	3) 命名捕获组的反向引用：
        i. 正则表达式内引用：\k<捕获组名称>
		！例如：(?<year>\d{4})-\k<year>可以匹配1999-1999
        ii. 外部引用：Matcher对象的start、end、group的String name参数指定要查询的捕获组的名称；
* 普通捕获组和命名捕获组的混合编号
	
		1) 普通捕获组是相对命名捕获组的，即没有显式命名的捕获组；
	    2) 当所有捕获组都是命名捕获组时那么编号规则和原来相同，即按照左括号(的出现顺序来编号；
	    3) 当普通捕获组和命名捕获组同时出现时，编号规则为：先不忽略命名捕获组，只对普通捕获组按照左括号顺序编号，
		然后再对命名捕获组从左往右累计编号，例如：
		(\d{4})-(?<date>\d{2}-(\d\d))
		对2008-12-31进行匹配，其结果：
		编号			命名		                   捕获组		                 匹配内容
		0		                     (\d{4})-(?<date>\d{2}-(\d\d))	        2008-12-31 	
		1		                                 (\d{4})                       2008
		3            date                (?<date>\d{2}-(\d\d))                 12-31
        2                                         (\d\d)	                    31	

		先忽略命名命名捕获组<date>，先对普通捕获组编号\d{4}是1，\d\d是2，然后再接着累加地对命名捕获组编号，因此<date>是3；

* Pattern和Mather类
 
     Pattern： 一个Pattern是一个正则表达式经编译后的表现模式。 
     </br>
     Matcher： 一个Matcher对象是一个状态机器，它依据Pattern对象做为匹配模式对字符串展开匹配检查。

	示例
	
		//典型的调用顺序
		//将正则表达式进行对象的封装
		Pattern p = Pattern.compile("a*b");
		//通过正则对象的matcher方法字符串关联，获取要对字符串操作的匹配器对象Matcher
		Matcher m = p.matcher("aaaaab");
		//通过Matcher匹配器对象的方法对字符串进行操作
		boolean b = m.matches();
		System.out.println(b);

		Pattern p1 = Pattern.compile("a*");
		Matcher m1 = p1.matcher("aaaaabaa0ewr22edqa");
		while (m1.find()) {
			System.out.print(m1.group() + " ");
		}

### 五、System
>The System class contains several useful class fields and methods. It cannot be instantiated.
Among the facilities provided by the System class are standard input, standard output, and error output streams; access to externally defined properties and environment variables; a means of loading files and libraries; and a utility method for quickly copying a portion of an array.

System类代表系统，系统级的很多属性和控制方法都放置在该类的内部。该类位于java.lang包。由于该类的构造方法是private的，所以无法创建该类的对象，也就是无法实例化该类。其内部的成员变量和成员方法都是static的，所以也可以很方便的进行调用。


	/*
	 * System类
	 * 		System中所有的成员都是静态的
	 * 
	 * 	static	PrintStream		err		“标准”错误输出流
	 * 	static	InputStream		in		“标准”输入流
	 * 	static	OutputStream	out		“标准”输出流
	 * 
	 * 常用方法：
	 * public	static	long 			currentTimeMillis()			获取当前时间的毫秒值（从1970年1月1日00：00开始计时）
	 * public	static 	Properties		getProperties()				确定当前系统的属性
	 *  	Properties：
	 *			properties 是Map集合的子类，存储的都是String类型的键和值
	 * public	static 	setProperty(String key,String value) 
	 * public 	static 	void 			gc() 					运行垃圾回收器，调用了object类中的finalized()，并不是立即执行
	 * 																
	 * public 	static 	void 			exit(int status)			终止当前的java虚拟机，非0状态(status)表示异常中止
	 * 																
	 * public 	static 	void 			arraycopy(Object src,int srcPos,Object dest,int destPos,length)	拷贝数组	
	 * 					
	 */

	//给系统设置一些属性信息。这些信息是全局，其他程序都可以使用。 
	System.setProperty("myclasspath", "c:\myclass");

	//获取系统的属性信息，并存储到了Properties集合中。 
	/*
	 * properties集合中存储都是String类型的键和值。
	 * 最好使用它自己的存储和取出的方法来完成元素的操作。
	 */
	Properties prop = System.getProperties();

	Set<String> nameSet = prop.stringPropertyNames();

	for (String name : nameSet) {
		String value = prop.getProperty(name);

		System.out.println(name + "::" + value);
	}
	//获取系统的值，有跨平台性
	System.out.println("hello"+System.getProperty("line.separator")+"world");

</br>	
</br>
</br>
</br>
更多方法请参照[Java API文档](http://docs.oracle.com/javase/8/docs/api/)


	
