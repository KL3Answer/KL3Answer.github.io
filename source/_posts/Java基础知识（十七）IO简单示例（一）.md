---
title: Java基础知识（十七）IO流简单示例（一）
date: 2015-04-14 14:33:19
tags: 
	- Java基础知识
---
### 注：这些流对象的关系不是成对的，你可以使用任意一个输入流与任意一个输出流进行操作，这里只是为了方便示例的书写


### 一、FileInputStream、FileOutputStream
FileInputStream

	/*read方法
		 int	read()
		 int	read(byte[] b)
		 int	read(byte[] b, int off, int len)
	*/
示例1.1

	//创建字节输入流对象
	//用读取流关联一个文件，文件必须存在
	FileInputStream fis=new FileInputStream("D:\\test.txt");

	//读取一个字节,读取完后指针后移
	int x=fis.read();

	//将byte存储为int是为了防止读取到-1的时候出错
	System.out.println(x);
	//读一个字节,读到流的结束标记时返回-1
	x=fis.read();
	System.out.println(x);

	//常用的做法，一直读取直到末尾
	while(x!=-1){
		x=fis.read();
		System.out.print(Character.toChars(x));
	}
	fis.close();
FileOutpuStream
	
	/*write方法
		void	write(byte[] b)
		void	write(byte[] b, int off, int len)
		void	write(int b)
	*/
示例1.2

	//创建字节输出流对象，没有就会自动创建一个文件
	//如果有这个文件的话会清空，重新写这个文件
	FileOutputStream fos=new FileOutputStream("D:\\old.txt");
	fos.write(97);
	fos.write(98);
	fos.write(99);

	//在文件后面继续添加，不清空
	FileOutputStream fos1=new FileOutputStream("D:\\old.txt",true);
	fos1.write(102);
	fos1.write(133);
	FileInputStream fis1=new FileInputStream("D:\\old.txt");
	int y=0;
	while((y=fis1.read())!=-1){
		try{
			System.out.println((char)y);
		}catch(Exception e){
			e.printStackTrace();
		}
	}
	fis1.close();//无缓冲
	
使用数组进行缓冲
	
	private static void demo_03() throws FileNotFoundException, IOException {
		System.out.println("-------");
		FileInputStream fis4=new FileInputStream("D:\\Backups\\Data\\Pictures\\old.bmp");
		FileOutputStream fos4=new FileOutputStream("D:\\Backups\\Data\\Pictures\\new.bmp");
		byte[] arr1=new byte[1024*8];//使用小数组
		int len;
		//将文件的字节流读取到byte数组中，返回读取的有效字节个数,如果读到流的结尾则返回-1
		while((len=fis4.read(arr1))!=-1){
			//arr将从0到len（不含）的元素写出
			fos4.write(arr1,0,len);
			//不指定长度写出arr中所有元素，当未存满的时候可能会出现被覆盖的风险
			//fos3.write(arr);
		}
		//关闭并将缓存区中剩余的刷出，一定要将缓存区中的刷出！
		fis4.close();
		fos4.close();
	}

		
### 二、BufferedInputStream、BufferedOutputStream
1. 示例2.1

		private static void demo_01() throws FileNotFoundException, IOException {
		BufferedInputStream bis = null;
		BufferedOutputStream bos = null;
		//try finally 用于关流，不catch,不隐藏Exception	
		try {
			//创建缓冲区对象，对输入流进行包装
			bis = new BufferedInputStream(new FileInputStream(
					"D:\\old.bmp"));
			//使用了数组
			bos = new BufferedOutputStream(new FileOutputStream("D:\\new.bmp"));
			int len;
			while ((len = bis.read()) != -1) {
				//write len个缓冲区的字节
				//只有在缓冲区写满的时候调用write才会flush刷新写出，因此在流末尾的时候（即读取的最后一次）并没有写出
				bos.write(len);
			}
		} finally {
			//设置关流
			try {//嵌套try finally ，防止关流时的异常，能多关一个就多关一个
				if (bis != null)
					bis.close();//关闭BufferedInputStream,此时其实就是关闭的FIS
			} finally {
				if (bos != null)
					//必须关流
					bos.close();
			}
		}
		/*
		 * flush()	刷新缓冲区	具备刷新的功能，但是不关闭流
		 * 	
		 * close()	关闭流		关闭流，在关闭流之前会先刷新一次缓冲区，
		 * 						将缓冲区的字节都刷新到文件上再关闭（因为最后一次没写进文件，或存取可能没存满）
		 * */
		/*
		 * 字节流读取中文
		 * 		因为中文是两个字符表示，所以读取的时候可能会出问题，只读半个codepointer出现乱码
		 * 字节流写出中文
		 * 		字节流直接操作的字节,所以写出中文必须将字符串转换成字节数组 
		 * 		写出回车换行 write("\r\n".getBytes());
		 * */
		}
这里其实可以使用JDK1.7的新特新ARM来管理流资源，示例这么写是为了逻辑更加清晰。

2. 示例2.2

		private static void demo_02() throws IOException, FileNotFoundException {
			//简单的加密
			try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("D:\\new.jpg"));
					BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("D:\\copy.jpg"));) {
				int b;
				while ((b = bis.read()) != -1) {
					bos.write(b ^ 123);//对字节使用异或运算
				}
			}
		}

### 三、	InputStreamReader、OutputStreamWriter
转换流

	/*转换流：
	 * 字节流与字符流相转换（可以转换不同码表的字符）
	 * 
	 * InputStreamReader	extends	Reader	解码
	 * 	可以被BufferedReader包装
	 * 	可以包装Stream
	 * OutputStreamwriter	extends	Writer	编码
	 * 	BufferedWriter
	 * 
	 * 
	 * 	InputStreamReader(InputStream in)
	 *		Creates an InputStreamReader that uses the default charset.
	 *
	 *	InputStreamReader(InputStream in, Charset cs)
	 *		Creates an InputStreamReader that uses the given charset.
	 *
	 *	InputStreamReader(InputStream in, CharsetDecoder dec)
	 *		Creates an InputStreamReader that uses the given charset decoder.
	 *
	 *	InputStreamReader(InputStream in, String charsetName)
	 *		Creates an InputStreamReader that uses the named charset.
	 * */	

	/*BufferedReader常用方法：
	 * readLine()	读一行
	 * 
	 *Bufferedwriter方法：
	 * newLine()	写一个行分隔符
	 * 
	 * */
	 
示例3.1

	public void test01() throws IOException{
		InputStream in=System.in;
		InputStreamReader isr=new 
		//转换流，将读取的字节使用码表转换成字符
		InputStreamReader(in);
		BufferedReader br=new 
		//对字符流进行装饰
		BufferedReader(isr);
		String line=null;
		while((line=br.readLine())!=null){
			if("over".equals(line)){
				break;
			}
			System.out.println(line.toUpperCase());
		}
		
		//OutputStreamWriter  字符流转换成字节流，类似  
		
	}	
	
其子类：FileWriter

示例3.2
	
	FileWriter fw0 = null;//创建写入流对象
		try {
			//字符流FileWriter
			fw0 = new FileWriter("D:\\demo.txt");
			//此时把数据写入到了临时缓冲区,在这抛出异常有可能是写入时外存出现问题
			fw0.write("asdsdsa");
			//注意：在windows中换行"\r\n"
			//在unix中换行"\n"
			fw0.flush();//进行刷新，将数据直接写入目的地
		} finally {
			try {
				if (fw0 != null) {
					fw0.close();
				}
			} catch (IOException e) {
				throw new RuntimeException("关闭失败");
			}
		}

示例3.3 倒序拷贝

	//倒序拷贝
	try (
		BufferedReader br = new BufferedReader(new FileReader("D:\\d1.txt"));
		BufferedWriter bw = new BufferedWriter(new FileWriter("D:\\mow.txt"));
	) {
		ArrayList<String> al = new ArrayList<>();
		String s;
		while ((s = br.readLine()) != null) {
			al.add(s);
		}
		for (int i = al.size() - 1; i >= 0; i--) {
			bw.write(al.get(i));
			bw.newLine();
		}
	}

	
LineNumberReader(extends BufferedReader)

使用了装饰者模式
 
	/*带行号的BufferedReader，行号从0开始
	 *	
	 *getLineNumber():
	 *setLineNumber(int number):设置为0则第一行行号为1
	 */

### 四、标准输入输出流

	/*System
	 * 	static	PrintStream		err		“标准”错误输出流
	 * 	static	InputStream		in		“标准”输入流
	 * 	static	OutputStream	out		“标准”输出流
	 * /
示例4.1 从外部设备录入数据

	private static void demo() throws IOException {
		System.out.println("start:");
		InputStream in=System.in;//字节流
		//从外部设备录入数据
		int ch=in.read();//在输入数据可用、检测到流末尾或者抛出异常前，此方法为阻塞
		//	in.close();
		//	InputStream in2=System.in;
		//	in2.close();系统流对象只有一个，关闭之后就不能使用了，不需要关，随系统的关闭自动关闭
		StringBuilder sb=new StringBuilder();
		int ch1=0;
		System.out.println("please enter anything,"exit" to quit:");
		while((ch1=in.read())!=-1){//读一行数据
			if(ch1=='\r'){
				continue;
			}
			if(ch1=='\n'){
				String temp=sb.toString();
				if("exit".equals(temp)){
					System.out.println("quit now");
					break;
				}
				System.out.println(temp);
				sb.delete(0,sb.length());
			}else{
				sb.append((char)ch1);
			}
		}
	}
示例4.2 使用System中的流拷贝文件

	private static void copy() throws FileNotFoundException, IOException {
		System.setIn(new FileInputStream("D:\\c.jpg"));//改变标准输入流
		System.setOut(new PrintStream("D:\\d1.jpg"));
		InputStream is=System.in;//改变后指向了一个文件
		PrintStream ps=System.out;
		byte[] b=new byte[8192];
		int len;
		while((len=is.read(b))!=-1){
			ps.write(b,0,len);
		}
		is.close();
		ps.close();
	}
### 五、SequenceInputStream
序列流，使多个输入流有顺序地输入

示例5.1 合并两个文件

	private static void test_01() throws FileNotFoundException, IOException {
		FileInputStream fis0 = new FileInputStream("D:/a.txt");
		FileInputStream fis1 = new FileInputStream("D:/b.txt");
		//输出到一个文件中
		SequenceInputStream sis = new SequenceInputStream(fis0, fis1);
		FileOutputStream fos = new FileOutputStream("D:/c.txt");
		int b;
		while ((b = sis.read()) != -1) {
			fos.write(b);
		}
		fis1.close();
		//SequenceInputStream在关闭对象的时候会将构造方法中的流对象都关闭
		sis.close();
		fos.close();
	}
示例5.2 	切个文件并合并

切割
		
	//切割文件
	public static void splitFile(File file) throws IOException {
		FileInputStream fis = new FileInputStream(file);
		byte[] buf = new byte[size];
		FileOutputStream fos = null;
		int len = 0;
		int count = 1;
		File dir = new File("d:\\");
		while ((len = fis.read(buf)) != -1) {
			fos = new FileOutputStream(new File(dir, count++ + "_" + ".part"));
			fos.write(len);
			fos.close();
		}
		
		//将被切割的文件信息保存到prop集合中
		fos = new FileOutputStream(new File(dir, count + ".Properties"));
		Properties prop = new Properties();
		prop.setProperty("part", count + "");
		prop.setProperty("filename", file.getName());
		prop.store(fos, "save");
		fos.close();
		fis.close();
	}
合并

	//合并文件
	public static void mergeFile(File dir) throws IOException{
		//获取配置信息
		File[] file=dir.listFiles(new SuffixFilter(".properties"));
		if(file.length!=1){
			throw new RuntimeException(".properties does not exists or is spoiled");
		}
		File configFile=file[0];
		Properties prop=new Properties();
		String filename=prop.getProperty("filename");
		int count=Integer.parseInt(prop.getProperty("count"));
		prop.load(new FileInputStream(configFile));
		
		File[] partFiles=dir.listFiles(new SuffixFilter(".part"));
		if(partFiles.length!=count-1){
			throw new RuntimeException("the file is not complete or is spoiled");
		}
		
		ArrayList<FileInputStream> al=new ArrayList<>();
		for(int x=1;x<=count;x++){
			al.add(new FileInputStream(dir+""+x+"_"+".part"));
		}
		Enumeration<FileInputStream> e=Collections.enumeration(al);
		SequenceInputStream sis=new SequenceInputStream(e);
		FileOutputStream fos=new FileOutputStream(new File(dir,filename));
		byte[] buf=new byte[size];
		int len=0;
		while((len=sis.read(buf))!=-1){
			fos.write(buf,0,len);
			fos.close();
		}
		sis.close();
	}