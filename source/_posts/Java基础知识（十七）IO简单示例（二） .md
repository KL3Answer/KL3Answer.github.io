---
title: Java基础知识（十七）IO流简单示例（二）
date: 2015-04-14 14:50:46
tags: 
	- Java基础知识
---

### 六、ByteArrayOutputStream、ByteArrayInputStream
内存输出流，数据被写入一个可增长的byte数组，初始容量为32。

	private static void test06() throws IOException, FileNotFoundException {
		File f=new File("D:/a.txt");
		FileInputStream fis=new FileInputStream(f);
		ByteArrayOutputStream baos=new ByteArrayOutputStream();//write into EMS memory
		byte[] b=new byte[5];
		int len;
		while((len=fis.read(b))!=-1){
			baos.write(b, 0, len);
		}
		fis.close();
		//byte[] newArr = baos.toByteArray();	//将内存缓冲区中所有的字节存储在newArr中
		//System.out.println(new String(newArr))
		System.out.println(baos);
	}

### 七、ObjectInputStream、ObjectOutpurStream
对象操作流：流可以将对象写出，或者读取一个对象到程序中，也就是执行了序列化和反序列化的操作可以实现对象的持久化，如果流是网络套接字流，则可以在另一台主机上重构对象。
被操作的对象要实现Serializable接口（可以添加序列号属性，是一个标识接口，没有待实现方法）

示例7.1
	public static void test_07() throws IOException, FileNotFoundException, ClassNotFoundException {
		new File("D:\\d2.txt").createNewFile();
		//序列化
		ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream("D:\\d2.txt"));
		Person p1=new Person("alice",20);
		oos.writeObject(p1);//一次读写一个对象
		oos.close();
		
		//反序列化
		ObjectInputStream ois=new ObjectInputStream(new FileInputStream("D:\\d2.txt"));
		//可以将对象封装在集合，对集合进行序列化和反序列化中以加快速度
		Person p=(Person)ois.readObject();
		ois.close();
		System.out.println(p);
	} 

Person类

	//实现Seriablizable
	class Person implements Serializable/*标记接口*/{
	/* 序列号用于验证序列化对象的发送者和接收者是否对该对象加载了与序列化兼容的类，
	 * 如果接收者加载的该对象的类的serialVersionUID的字段与对应的发送者的类的版本号不相同，则反序列化会导致一个InvalidClassException
	 * 序列号根据类的特征以及类的签名算出
	 * */
		private static final long serialVersionUID = 1L;
		private String name;
		private transient String created;//静态和瞬态的数据不被写入
		private int age;
		public Person(String name, int age) {
			super();
			this.name = name;
			this.age = age;
		}
		public Person() {
			super();
			
		}
		@Override
		public int hashCode() {
			final int prime = 31;
			int result = 1;
			result = prime * result + age;
			result = prime * result + ((name == null) ? 0 : name.hashCode());
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
			Person other = (Person) obj;
			if (age != other.age)
				return false;
			if (name == null) {
				if (other.name != null)
					return false;
			} else if (!name.equals(other.name))
				return false;
			return true;
		}
		@Override
		public String toString() {
			return "Person [name=" + name + ", age=" + age + "]";
		}
		public String getName() {
			return name;
		}
		public int getAge() {
			return age;
		}
	}
当然，也可以将对象放入集合中批量写入。

### 八、DataInputStream、DataOutputStream
用操作基本数据类型值，其他的类似，没什么好说的

示例8.1

	DataInputStream dis=new DataInputStream(new FileInputStream("D:\\demo.txt"));
	dis.readInt();
	//dis.readByte();
	//dis.readShort();
	//dis.readInt();
	//dis.readLong();
	//dis.readBoolean();
	//dis.readFloat();
	//dis.readDouble();
	//dis.readChar();
	//dis.read();
### 九、PipedInputStream、PipedOutputStream
管道流，输出输入流可以进行连接，通过结合线程使用，不建议单线程使用，易死锁。
其构造方法：

	public PipedInputStream(PipedOutputStream src)
	public PipedInputStream(PipedOutputStream src, int pipeSize)
	public PipedOutputStream(PipedInputStream snk)
	public PipedOutputStream()
示例9.1 
	
	PipedInputStream pis = new PipedInputStream();
	PipedOutputStream pos = new PipedOutputStream();
	pis.connect(pos);//连接管道流
	new Thread(new Input(pis)).start();
	new Thread(new Output(pos)).start();
Input：

	class Input implements Runnable {
		private PipedInputStream in;
	
		Input(PipedInputStream in) {
			this.in = in;
		}
	
		public void run() {
			try {
				byte[] buf = new byte[8192];
				int len;
				String s = null;
				len = in.read(buf);
				s = new String(buf, 0, len);
				in.close();
				System.out.println("s=" + s);
			} catch (Exception e) {
	
			}
		}
	}
Output:

	class Output implements Runnable {
		private PipedOutputStream out;
	
		Output(PipedOutputStream out) {
			this.out = out;
		}
	
		public void run() {
			try {
				out.write("piped ".getBytes());
			} catch (Exception e) {
	
			}
		}
	}
### 十、RandomAccessFile
RandomAccessFile不属于InputStream和OutputStream类系的。实际上，除了实现DataInput和DataOutput接口之外(DataInputStream和DataOutputStream也实现了这两个接口)，它和这两个类系毫不相干，甚至不使用InputStream和OutputStream类中已经存在的任何功能；它是一个完全独立的类，所有方法(绝大多数都只属于它自己)都是从零开始写的。这可能是因为RandomAccessFile能在文件里面前后移动，所以它的行为与其它的I/O类有些根本性的不同。总而言之，它是一个直接继承Object的、独立的类。

随机访问流，主要是结合线程使用。

	/*
	 * RandomAccessFile	extends Object
	 * 不是IO体系中的子类
	 * 
	 * A:随机访问流概述
	 * 	RandomAccessFile类不属于流，是Object类的子类。但它融合了InputStream和OutputStream的功能。
	 * 	支持对随机访问文件的读取和写入。
	 * B：该对象内部维护一个大型的byte数组，并通过指针可以操作数组中的元素
	 * C：可以通过getFilePointer获取指针的位置，并通过seek()方法设置
	 * D：其实就是对输入输出流进行了封装
	 * 
	 * 该对象的源和目的只能是文件
	 * 
	 * :read()：读取,write()：写入,seek()：将指针设定到指定位置
	 * 
	 * 可以用于多线程下载，因为可以指定位置写入与读取
	 * 
	 * 
	 * 构造方法：
	 * RandomAccessFile(File file,String mode)
	 * RandomAccessFile(String name, String mode)
	 * 
	 * String mode:
	 * 		r
	 * 		rw		读&写
	 * 		rwd		
	 * 		rws
	 * 
	 * */
示例10.1
	
	public static void main(String[] args) throws Exception {  
        // 预分配文件所占的磁盘空间，磁盘中会创建一个指定大小的文件  
        RandomAccessFile raf = new RandomAccessFile("D://abc.txt", "rw");  
        raf.setLength(1024*1024); // 预分配 1M 的文件空间  
        raf.close();  
          
        // 所要写入的文件内容  
        String s1 = "s1";  
        String s2 = "s2";  
        String s3 = "s3";  
        String s4 = "s4";  
        String s5 = "s5";  
          
        // 利用多线程同时写入一个文件  
        new FileWriteThread(1024*1,s1.getBytes()).start(); // 从文件的1024字节之后开始写入数据  
        new FileWriteThread(1024*2,s2.getBytes()).start(); // 从文件的2048字节之后开始写入数据  
        new FileWriteThread(1024*3,s3.getBytes()).start(); // 从文件的3072字节之后开始写入数据  
        new FileWriteThread(1024*4,s4.getBytes()).start(); // 从文件的4096字节之后开始写入数据  
        new FileWriteThread(1024*5,s5.getBytes()).start(); // 从文件的5120字节之后开始写入数据  
    }  
      
    // 利用线程在文件的指定位置写入指定数据  
    static class FileWriteThread extends Thread{  
        private int skip;  
        private byte[] content;  
          
        public FileWriteThread(int skip,byte[] content){  
            this.skip = skip;  
            this.content = content;  
        }  
          
        public void run(){  
            RandomAccessFile raf = null;  
            try {  
                raf = new RandomAccessFile("D://abc.txt", "rw");  
                raf.seek(skip);  
                raf.write(content);  
            } catch (FileNotFoundException e) {  
                e.printStackTrace();  
            } catch (IOException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            } finally {  
                try {  
                    raf.close();  
                } catch (Exception e) {  
                }  
            }  
        }  
    }  
据说，RandomAccessFile的绝大多数功能，已经被JDK 1.4的nio的"内存映射文件(memory-mapped files)"取代了。

示例10.2 使用内存映射文件读取大文件

	
	public void test10(){
		int length = 0x8000000; // 128 Mb 
		// 为了以可读可写的方式打开文件，这里使用RandomAccessFile来创建文件。  
	    FileChannel fc = new RandomAccessFile("test.dat", "rw").getChannel();  
	    //注意，文件通道的可读可写要建立在文件流本身可读写的基础之上  
	    MappedByteBuffer out = fc.map(FileChannel.MapMode.READ_WRITE, 0, length);  
	    //写128M的内容  
	    for (int i = 0; i < length; i++) {  
	        out.put((byte) 'x');  
	    }  
	    System.out.println("Finished writing");  
	    //读取文件中间6个字节内容  
	    for (int i = length / 2; i < length / 2 + 6; i++) {  
	        System.out.print((char) out.get(i));  
	    }  
	    fc.close();
	} 
MappedByteBuffer是ByteBuffer的子类，并增加了force()将缓冲区的内容强制刷新到存储设备中去、load()将存储设备中的数据加载到内存中、isLoaded()位置内存中的数据是否与存储设置上同步。这里只简单地演示了一下put()和get()方法，除此之外，你还可以使用asCharBuffer( )之类的方法得到相应基本类型数据的缓冲视图后，可以方便的读写基本类型数据。