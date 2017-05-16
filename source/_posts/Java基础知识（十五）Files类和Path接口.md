---
title: Java基础知识（十三）Java数据结构总结
date: 2015-04-13 10:53:45
tags: 
	- Java基础知识
---

Files类和Path接口是JDK1.7添加的文件操作的类，可以是我们对文件的操作更加方便。

### 一、Path接口
Path表示的是一个目录名序列，其后可以跟一个文件名。路径中的第一个可以是绝对路径和相对路径。如在类Unix系统中，可以这样：

	Path absolute=Paths.get("/home","demo");
	Path relative=Paths.get("test","conf","a.properties");
这里使用了Paths类，这个类非常简洁：

	public final class Paths {
	    private Paths() { }
	    public static Path get(String first, String... more) {
	        return FileSystems.getDefault().getPath(first, more);
	    }
	    public static Path get(URI uri) {/*具体实现*/}
	}	
	
我一般使用Paths来获取Path对象，Path对象的常用方法有：

	boolean isAbsolute();
	Path getFileName();
	Path getParent();
	int getNameCount();//根据路径分隔符来统计
	Path getName(int index);//由上面的统计数确定index
	Path subpath(int beginIndex, int endIndex);
	Path resolve(Path other);//组合路径
	Path resolveSibling(Path other);
	Path relativize(Path other);
	Path toAbsolutePath();
	File toFile();	

### 二、Files
Files类是对文件的操作变得快捷。

1. 常用读写方法：

		public static InputStream newInputStream(Path path, OpenOption... options)
	
		public static OutputStream newOutputStream(Path path, OpenOption... options)
		
		public static BufferedReader newBufferedReader(Path path, Charset cs)
		
		public static BufferedReader newBufferedReader(Path path)
		
		public static BufferedWriter newBufferedWriter
		
		public static byte[] readAllBytes(Path path)
		
		public static List<String> readAllLines(Path path)
		
		public static Path write(Path path, byte[] bytes, OpenOption... options)
		
		public static long copy(InputStream in, Path target, CopyOption... options)
		
		public static long copy(Path source, OutputStream out)
其中copy方法有多个重载：

		private static long copy(InputStream source, OutputStream sink)
	
		public static long copy(InputStream in, Path target, CopyOption... options)
		
		public static long copy(Path source, OutputStream out)
		
		public static Path copy(Path source, Path target, CopyOption... options)	
copy示例：

		Path source = Paths.get("a.txt");
		Path destination = Paths.get("b.txt");
		Files.copy(p,q);
同样的还有move和delete方法，在此就不再写示例了。		

2. 文件创建、获取的方法

		public static Path createFile(Path path, FileAttribute<?>... attrs)
	
		public static Path createDirectory(Path dir, FileAttribute<?>... attrs)
		//创建多级目录
		public static Path createDirectories(Path dir, FileAttribute<?>... attrs)
		
		public static Path createTempFile(Path path, FileAttribute<?>... attrs)
3. 文件信息获取方法

		public static boolean isSameFile(Path path, Path path2)
		public static boolean isHidden(Path path)
		public static String probeContentType(Path path)
		public static boolean isDirectory(Path path, LinkOption... options)
		public static boolean isRegularFile(Path path, LinkOption... options)
		public static long size(Path path)
		public static boolean exists(Path path, LinkOption... options)
		public static boolean isReadable(Path path)
		public static boolean isWritable(Path path)
4. 文件的迭代

	使用目录流迭代可以显著提高效率	

		public static DirectoryStream<Path> newDirectoryStream(Path dir)

		public static DirectoryStream<Path> newDirectoryStream(Path dir, String glob)
		
		public static DirectoryStream<Path> newDirectoryStream(Path dir,DirectoryStream.Filter<? super Path> filter)
	目录流迭代示例：
	
	a、
		
		DirectoryStream<Path> d = Files.newDirectoryStream(Paths.get("D://demo"));
		for (Path p : d){
		  System.out.println(p.getFileName());
		}
		//输出结果：
		a
		b.txt
		c
	b、
		
		DirectoryStream<Path> d = Files.newDirectoryStream(Paths.get("D://demo"),"*.txt");
		for (Path p : d){
		   System.out.println(p.getFileName());
		}
		//输出结果：
		b.txt
	c、

		DirectoryStream<Path> d = Files.newDirectoryStream(Paths.get("D://demo"),new DirectoryStream.Filter<Path>(){
		    @Override
		    public boolean accept(Path entry)  {
		       return Files.isDirectory(entry)?true:false;
		    }
	    });
	    for (Path p : d){
	       System.out.println(p.getFileName());
	    }
	    //输出结果：
	  	a
	    c

4. 使用walkFileTree方法访问目录中的子孙成员

	以下是书上的一个示例：
	
		Files.walkFileTree(dir,new SimpleFileVisiotr<Path>{
			public FileVisitResult(Path path ,BasicFileAttributes attrs) throws IOException{
				if(attrs.isDirectory()){
					System.out.println(path);
					return FileVisitResult.CONTINUE;
				}
			}
			public FileVisitResult visitFileFailed(Path path,IOException exc) throws IOException
				return FileVisit.CONTINUE;
			}
		});
	
	注意，需要覆盖visitFileFailed方法。
	
5. 使用文件系统操作ZIP文件

	FileSystems操作ZIP文件比ZipInputStream更简单
		
		FileSystem fs=FileSystem.newFileSystem(Paths.get(zipname),null);
		//将zip中的指定文件拷贝出来
		Files.copy(fs.getPath(sourceName),targetPath);
