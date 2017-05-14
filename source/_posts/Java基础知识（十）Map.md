---
title: Java基础知识（十）Map
date: 2015-4-11 14:54:03
tags: 
	- Java基础知识
---
### 一、Map

1、Map接口

Map提供key到value的映射。一个Map中不能包含相同的key，每个key只能映射一个value。Map接口提供3种集合的视图，Map的内容可以被当作一组key集合，一组value集合，或者一组key-value映射。

* Hashtable类

	Hashtable继承Map接口，实现一个key-value映射的哈希表。任何非空（non-null）的对象都可作为key或者value。

	添加数据使用put(key, value)，取出数据使用get(key)，这两个基本操作的时间开销为常数。

	Hashtable通过initial capacity和load factor（请看算法导论中关于哈希算法和装载因子的论述）两个参数调整性能。通常缺省的load factor 0.75较好地实现了时间和空间的均衡。增大load factor可以节省空间但相应的查找时间将增大，这会影响像get和put这样的操作。
	
	使用Hashtable的简单示例如下，将1，2，3放到Hashtable中，他们的key分别是”one”，”two”，”three”：
		
		Hashtable numbers = new Hashtable();
		numbers.put(“one”, new Integer(1));
		numbers.put(“two”, new Integer(2));
	要取出一个数，比如2，用相应的key：
		
		Integer n = (Integer)numbers.get(“two”);
		System.out.println(“two = ” + n);

	由于作为key的对象将通过计算其散列函数来确定与之对应的value的位置，因此任何作为key的对象都必须实现hashCode和equals方法。hashCode和equals方法继承自根类Object，如果你用自定义的类当作key的话，要相当小心，按照散列函数的定义，如果两个对象相同，即obj1.equals(obj2)=true，则它们的hashCode必须相同，但如果两个对象不同，则它们的hashCode不一定不同，如果两个不同对象的hashCode相同，这种现象称为冲突，冲突会导致操作哈希表的时间开销增大，所以尽量定义好的hashCode()方法，能加快哈希表的操作。
	
	如果相同的对象有不同的hashCode，对哈希表的操作会出现意想不到的结果（期待的get方法返回null），要避免这种问题，只需要牢记一条：要同时复写equals方法和hashCode方法，而不要只写其中一个。

* HashMap类
　　HashMap和Hashtable类似，不同之处在于HashMap是非同步的，并且允许null，即null value和null key。，但是将HashMap视为Collection时（values()方法可返回Collection），其迭代子操作时间开销和HashMap的容量成比例。因此，如果迭代操作的性能相当重要的话，不要将HashMap的初始化容量设得过高，或者load factor过低。

* WeakHashMap类
　　WeakHashMap是一种改进的HashMap，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收。

* IdentityHashMap类
  一种使用==而不是equals来比较键值的Map

### 二、关于同步

可以使用四种方法同步Map：

1、使用 synchronized 

	synchronized(anObject)  
	{  
	    value = map.get(key);  
	} 

注：JDK1.2 提供了 Collections.synchronizedMap(originMap) 方法，同步方式其实和上面这段代码相同。

2、使用 JDK1.5 提供的锁（java.util.concurrent.locks.Lock）

	lock.lock();  
    value = map.get(key);  
	lock.unlock(); 

3、实际应用中，可能多数操作都是读操作，写操作较少。针对这种情况，可以使用 JDK1.5 提供的读写锁（java.util.concurrent.locks.ReadWriteLock）

	rwlock.readLock().lock();  
	value = map.get(key);  
	rwlock.readLock().unlock(); 

这样两个读操作可以同时进行，理论上效率会比Lock高。


4、使用 JDK1.5 提供的 java.util.concurrent.ConcurrentHashMap 类。

该类将 Map 的存储空间分为若干块，每块拥有自己的锁，大大减少了多个线程争夺同一个锁的情况。

    value = map.get(key); //同步机制内置在 get 方法中 

 
5、这几种方法的效率：
	ConcurrentHashMap>ReadWriteLock>Lock>synchronized

注：Collection也可以使用类似的方法。