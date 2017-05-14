---
title: Java基础知识（九）Collection
date: 2015-3-20 14:50:03
tags: 
	- Java基础知识
---
### 一、Collection接口简介

早期的Java中为数据结构只提供很少的类：Vector、Stack、Hashtable、BitSet和Enumeration接口，但是现在已经大不相同了。Java关于数据结构的类基本能够满足一些常用的需求。

这个是	Collection接口及其子类与子接口

	Collection
	├List
	│├LinkedList
	│├ArrayList
	│└Vector
	│　└Stack
	└Set

1. Collection接口

	Collection是集合的顶层接口，Java SDK提供的类都是继承自Collection的子接口如List和Set。
　

	所有实现Collection接口的类都必须提供两个标准的构造函数：无参数的构造函数用于创建一个空的Collection，有一个Collection参数的构造函数用于创建一个新的Collection，这个新的Collection与传入的Collection有相同的元素。后一个构造函数允许用户复制一个Collection。

		//遍历Collection可以使用iterator
		Iterator it = collection.iterator(); // 获得一个迭代子
		while(it.hasNext()) {
		　　　Object obj = it.next(); // 得到下一个元素
		}
	

*  List接口

	List是有序的Collection，使用此接口能够精确的控制每个元素插入的位置。用户能够使用索引（元素在List中的位置，类似于数组下标）来访问List中的元素，这类似于Java的数组。
	和下面要提到的Set不同，List允许有相同的元素。
　　除了具有Collection接口必备的iterator()方法外，List还提供一个listIterator()方法，返回一个ListIterator接口，和标准的Iterator接口相比，ListIterator多了一些add()之类的方法，允许添加，删除，设定元素，还能向前或向后遍历。
　　实现List接口的常用类有LinkedList，ArrayList，Vector和Stack。

* LinkedList类

	LinkedList实现了List接口，允许null元素。此外LinkedList提供额外的get，remove，insert方法在LinkedList的首部或尾部。这些操作使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。

	注意LinkedList没有同步方法。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List：
　　　

		List list = Collections.synchronizedList(new LinkedList(...));

* ArrayList类

	ArrayList实现了可变大小的数组(其增长策略是newCapacity = (oldCapacity * 3)/2 + 1，和C++通常的Vector是翻倍的策略不同)。它允许所有元素，包括null。ArrayList没有同步。
	size，isEmpty，get，set方法运行时间为常数。但是add方法开销为分摊的常数，添加n个元素需要O(n)的时间。其他的方法运行时间为线性。
	每个ArrayList实例都有一个容量（Capacity），即用于存储元素的数组的大小。这个容量可随着不断添加新元素而自动增加，但是增长算法并没有定义。当需要插入大量元素时，在插入前可以调用ensureCapacity方法来增加ArrayList的容量以提高插入效率。
	和LinkedList一样，ArrayList也是非同步的（unsynchronized）。

* Vector类
　	
	
	Vector非常类似ArrayList，但是Vector是同步的。由Vector创建的Iterator，虽然和ArrayList创建的Iterator是同一接口，但是，因为Vector是同步的，当一个Iterator被创建而且正在被使用，另一个线程改变了Vector的状态（例如，添加或删除了一些元素），这时调用Iterator的方法时将抛出ConcurrentModificationException，因此必须捕获该异常。

* Stack 类
　　Stack继承自Vector，实现一个后进先出的堆栈。Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop方法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置。Stack刚创建后是空栈。

* Set接口

	Set是一种不包含重复的元素的Collection（相当于单列的Map），即任意的两个元素e1和e2都有e1.equals(e2)=false，Set最多有一个null元素。
	很明显，Set的构造函数有一个约束条件，传入的Collection参数不能包含重复的元素。
	注意：必须小心操作可变对象（Mutable Object）。如果一个Set中的可变元素改变了自身状态导致Object.equals(Object)=true将导致一些问题。

	HashSet实现排序：

		/*HashSet
		 * 	使用哈希算法，对元素进行运算并获取其位置（不需要遍历，性能稳定）
		 * 	当哈希值相同时，使用equals方法判断元素是否为同一个
		 *  因此一般实现自己的排序时会重写要排序对象的hashcode和equals方法
		 * */

	TreeSet实现排序：
	
		/* a、自然排序（Comparable）	
		 *  被排序的对象实现Comparable接口并覆盖compareTo方法
		 * 	使用compareTo()方法进行排序（被称为自然比较方法）
		 * 		当compareTo返回的正数时，都存在根元素右边
		 * 		当compareTo返回的负数时，都存在根左边
		 * 		当compareTo返回0时，不存储，集合中只有一个元素
		 * 顺序从小到大
		 * 
		 * b、比较器排序（Comparator）
		 * 	创建TreeSet对象时使用匿名内部类Comparator传入
		 *  调用的对象是compare方法的第一个参数，比较的对象时compare方法的笫二个参数
		 * */
	

###二、总结

如果涉及到堆栈，队列等操作，应该考虑用List，对于需要快速插入，删除元素，应该使用LinkedList，如果需要快速随机访问元素，应该使用ArrayList。
　　
如果程序在单线程环境中，或者访问仅仅在一个线程中进行，考虑非同步的类，其效率较高，如果多个线程可能同时操作一个类，应该使用同步的类。

要特别注意对哈希表的操作，作为key的对象要正确复写equals和hashCode方法。
尽量返回接口而非实际的类型，如返回List而非ArrayList，这样如果以后需要将ArrayList换成LinkedList时，客户端代码不用改变。这就是针对抽象编程。

	