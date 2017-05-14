---
title: Java基础知识（十二）Collection、Map总结
date: 2014-04-12 11:23:45
tags: 
	- Java基础知识
---

### 一、接口

Collection

* 子接口：

		BlockingDeque<E>, BlockingQueue<E>, Deque<E>, List<E>, NavigableSet<E>, Queue<E>, Set<E>, SortedSet<E> 

* 实现类：

		ArrayBlockingQueue, ArrayDeque, ArrayList,  ConcurrentLinkedQueue, ConcurrentSkipListSet, 
		CopyOnWriteArrayList, CopyOnWriteArraySet, DelayQueue, EnumSet, HashSet, LinkedBlockingDeque, 
		LinkedBlockingQueue, LinkedHashSet, LinkedList, PriorityBlockingQueue, PriorityQueue, 
		Stack, SynchronousQueue, TreeSet, Vector 

1、  List<E>

子接口：
	
	无
 
实现类：

	ArrayList, CopyOnWriteArrayList, LinkedList,Stack, Vector 


2、Queue<E>

子接口：

	BlockingDeque<E>, BlockingQueue<E>, Deque<E> 

实现类：

	ArrayBlockingQueue, ArrayDeque, ConcurrentLinkedQueue, DelayQueue, 
	LinkedBlockingDeque, LinkedBlockingQueue, LinkedList, PriorityBlockingQueue,
	PriorityQueue, SynchronousQueue 

 

3、Set<E>

子接口：

	NavigableSet<E>, SortedSet<E> 

实现类：

	ConcurrentSkipListSet, CopyOnWriteArraySet, EnumSet, HashSet, LinkedHashSet, TreeSet 

 

4、Map<K,V>

子接口：

	ConcurrentMap<K,V>, ConcurrentNavigableMap<K,V>,  SortedMap<K,V> 

实现类：

	ConcurrentHashMap, ConcurrentSkipListMap, EnumMap, HashMap, Hashtable, IdentityHashMap, LinkedHashMap, TreeMap, WeakHashMap 

 

 

并发与线程安全等

通常含有Concurrent，CopyOnWrite，Blocking的是线程安全的，但是这些线程安全通常是有条件的，所以在使用前一定要仔细阅读文档。

 

 

### 二、具体实现

1、 List<E>系列：

	ArrayList<E>，如其名，但是其容量增长计划是newCapacity = (oldCapacity * 3)/2 + 1，和C++通常的Vector是翻倍的策略不同。

	CopyOnWriteArrayList<E>，里面有一个ReentrantLock，每当add时，都锁住，把所有的元素都复制到一个新的数组上。

	只保证历遍操作是线程安全的，get操作并不保证，也就是说如果先得到size，再调用get(size-1)，有可能会失效

	那么CopyOnWriteArrayList是如何实现线程安全的迭代操作？

	在迭代器中保存原数组。

* LinkedList<E>，标准双向链表

* Vector<E>，过时，多数方法上加上了synchronized

* Stack<E>，继承自Vector，过时，优先应使用 Deque<Integer> stack = new ArrayDeque<Integer>();

 

2、Queue<E>系列：

	LinkedList<E>，见List<E>系列

	ArrayDeque<E>，内部用一个数组保存元素，有int类型head和tail的。

	PriorityQueue<E>，内部用一个数组来保存元素，但数组是以堆的形式来组织的，因此是有序的。

	PriorityBlockingQueue<E>，包装了一个PriorityQueue<E>，一个ReentrantLock，一个Condition，TODO

	ArrayBlockingQueue<E>，TODO

	ConcurrentLinkedQueue<E>，TODO

	DelayQueue<E>，TODO

	LinkedBlockingDeque<E>，TODO

	LinkedBlockingQueue<E>，TODO

	SynchronousQueue<E>，TODO

 

3、Deque<E>（双端队列）系列：

	ArrayDeque<E>，见Queue系列

	LinkedList<E>，见List系列

	LinkedBlockingDeque<E>，TODO

 

 

4、Set系列：

	HashSet，包装了一个HashMap：
	
	    public HashSet() {
	
	        map = new HashMap<E,Object>();
	
	    }
	
	TreeSet，包装了一个TreeMap，参考HashSet
	
	LinkedHashSet，包装了LinkedHashMap，参考HashSet
	
	EnumSet，TODO
	
	CopyOnWriteArraySet，简单包装了CopyOnWriteArrayList，注意这个Set的get的时间复杂度。
	
	ConcurrentSkipListSet，包装了一个ConcurrentSkipListMap，参考HashSet。



5、Map系列：

	HashMap<K,V>，标准链地址法实现（全域哈希）
	
	TreeMap<K,V>，红黑二叉树
	
	LinkedHashMap<K,V>，在Entry中增加before和after指针把HashMap中的元素串联起来，这样在迭代时，就可以按插入顺序历遍。
	
	EnumMap，TODO
	
	ConcurrentHashMap，参考之前的文章
	
	ConcurrentSkipListMap，TODO，log(n)的时间复杂度，有点像多级链表保存的，貌似有点像Redis中的SortedSet的实现
	
	Hashtable，过时
	
	IdentityHashMap，正常的HashMap中比较是用equals方法，这个用的是“==”比较符
	
	WeakHashMap<K,V>，弱引用的HashMap，正常的HashMap是强引用，即里面的value不会被GC回收，在WeakHashMap<K,V>中，
	V中最好是WeakReference类型的，用像这样的代码：m.put(key, new WeakReference(value))