---
title: Java基础知识（十四）File、Filter
date: 2015-04-13 9:37:12
tags: 
	- Java基础知识
---

### 一、File类
File是从JDk1.0开始就已经存在的文件操作相关的类

	/*File类（路径）
	 * 路径：绝对路径和相对路径
	 * 
	 *构造方法：
	 * 	File(String pathname)						根据路径得到File对象(注意不是创建是得到)
	 * 	File(String parent,String child)			根据一个目录和一个子文件/目录得到对象
	 * 	File(File parent,String child)				根据一个父File对象和一个子文件/目录得到File对象
	 * 
	 * */
示例1

	System.out.println("-----------------");
	File f = new File("D:\\demo\\test.bmp");
	System.out.println(f.exists());
	String parent = "D:\\demo";
	String child = "test.bmp";
	File f1 = new File(parent, child);
	System.out.println(f1.exists());
常用方法

	/*	创建：
	 * public 	boolean		createNewFile()			创建新文件，如果存在就不创建了		 
	 * public 	boolean		mkdir()					创建新文件夹，如果存在就不创建了（文件夹也可以有后缀，但是不推荐）		 
	 * public 	boolean		mkdirs()				创建新文件夹，如如果父文件夹不存在，会帮你创建出来	 
	 * 
	 * 	重命名：
	 * public 	boolean 	renameTo(File dest)		把文件重命名为指定的文件路径(如果路径相同)
	 * 												如果路径不同就是改名并剪切
	 * 	删除：									
	 * public	boolean		delete()				删除文件或文件夹，不走回收站
	 * 												注意要删除的文件夹中不能包含文件或文件夹
	 * public	boolean		deleteOnExit()			在退出时删除文件或文件夹
	 * 
	 * 	判断：
	 * public	boolean		setReadable(boolean b)	windows中都是可读的，设置为false也是可读的	
	 * public	boolean		setwritable(boolean b)	设置为false不可写	
	 * public	boolean		canRead()
	 * public	boolean		canWrite()
	 * public	boolean		isHidden()
	 * public	boolean		isFile()		
	 * public	boolean		isDirectory()
	 * public 	boolean 	exists()
	 * 
	 * 	获取：
	 * public	String		getAbsolutePath()		获取绝对路径
	 * public 	String		getPath()				获取相对路径	
	 * public 	String		getName()				获取名称
	 * public 	long		length()				获取长度，字节数
	 * public 	long		lastModified()			获取最后一次修改时间，毫秒
	 * public 	String[]	list()					获取指定目录下的所有文件或者文件夹的名称数组
	 * public 	File[]		listFiles()				获取指定目录下的所有文件或者文件夹的File数组,如果目录存在但是没有内容，会返回一个长度为0的数组
	 * static 	File[]		listRoots()				列出可用的文件系统根（C:\\,D:\\   .etc）
	 * static	long		getFreeSpace()			返回此抽象路径名指定的分区空闲的空间
	 * 						getTotalSpace()			返回此抽象路径名指定的分区大小
	 * 						getUsableSpace()		返回此抽象路径名指定的分区上可用于虚拟机的字节数
	 * 	
	 * */
示例2

	File f2 = new File("asd");
	System.out.println(f2.mkdirs());
	System.out.println("---------");
	File dir = new File("D:\\Backups\\Apps");//获取目录下的所有文件或文件夹对象
	File[] subFiles = dir.listFiles();
	for (File subFile : subFiles) {
		if (subFile.isFile() && subFile.getName().endsWith(".jpg")) {
			System.out.print(subFile);
		}
	}
常用的静态域

	/*File类：
	 * static char 		separatorChar  	路径分隔符
	 * static String	separator		目录分隔符
	 * */
	
### 二、Filter接口
使用FileNameFilter和FileFilter对文件搜索进行过滤

	public class Demo_02 {
		public static void main(String[] args) throws IOException{
			/*
			 * Interface FileNameFilter
			 *	过滤器	
			 *	只有一个accept()方法 
			 * 
			 * 	boolean		accept(File dir ,String name)		当且仅当该名称应该包含在文件列表中时返回true
			 * 
			 * */
			String[] subFiles=new File("D:\\HQ_JAVA\\demo\\src\\test12").list(new FilterByJava());
			for (String string : subFiles) {
					System.out.println(string);
			}
			/*
			 * Interface FileFilter 
			 * 	文件过滤器
			 *	只有一个accept(File pathName)方法 
			 *
			 *	用法和FilenameFilter一样 
			 *
			 * */
			
			Files.delete(Paths.get("a"));
			
		}
		public static void getFiles(File dir,FilenameFilter filter,List<File> list){
			
			File[] files = dir.listFiles();
			
			for(File file : files){
				if(file.isDirectory()){
					getFiles(file,filter,list);
				}else{
					//对遍历到的文件进行过滤器的过滤。将符合条件File对象，存储到List集合中。 
					if(filter.accept(dir, file.getName())){
						list.add(file);
					}
				}
			}
			
		}
	
	}
上面Filter的实现

	class FilterByJava implements FilenameFilter{
		public boolean accept(File dir,String name){
			return name.endsWith(".java");
		}
	}
	class FilterBySuffix implements FilenameFilter{
		private String suffix;
		
		public FilterBySuffix(String suffix){
			super();
			this.suffix=suffix;
		}
		public boolean accept(File dir,String name){
			
			return name.endsWith(suffix);
		}
	}
注意：FileNameFilter的性能一般比FileFilter好，但是FileFilter的通用性更强一些。