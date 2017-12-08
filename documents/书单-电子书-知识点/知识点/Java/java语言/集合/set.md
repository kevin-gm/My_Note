Set是一种**不**包含**重复**元素的集合。换言之，set集合中的任意两个元素(x、y)，不允许x.equals(y) 返回true。set集合中最多允许存在一个null元素。这个接口建立了跟数学模型上的集合得抽象。

**注意：**必须特别注意易变对象作为set的元素。如果一个对象set的一个元素，但是对象的值发生影响equals比较结果的变化时，set没有对应指定的行为来进行保证。这种禁令的一个特例是，一个set集合不允许将自身作为元素包含。

## HashSet
HashSet是依托于hash表实现的set，它不保证集合中元素的顺序，特别的，也不保证顺序随时间变化而保持不变。这个集合对象允许null元素。

HashSet 不是 线程安全的，多线程环境下，必须额外进行同步保证。可以使用 ```Collections.synchronizedSet``` 来进行线程安全的set操作。

HashSet的迭代(iterator)是一种快速失败(fail-fast)的迭代。如果set在迭代开始后的任何时刻被修改，通常而言，不同步地多线程修改时，是没有提供任何硬性保证正确性的。快速失败的迭代将会抛出 ```ConcurrentModificationException``` 异常

因此，编程时依靠这个异常来保证正确性是错误的，迭代时的快速失败行为应当被用来检测BUG。

HashSet内部是通过维护一个HashMap来实现的，它的操作基本都是基于HashMap。

* HashMap的key为set的元素值
* 因为HashMap的key可以为null，所以HashSet可以为null。
* 也因为元素值是作为HashMap的key，所以不会出现重复元素，因为当元素值相同时，HashMap会进行值覆盖。
* HashMap的value是一个静态变量对象，是一个虚拟值，没有任何含义。

	```private static final Object PRESENT = new Object();```
* HashSet的无参构造方法，即构造一个默认的无参HashMap，默认的容量大小是16，默认的负载因子是0.75
	
		public HashSet() {
	        map = new HashMap<>();
	    }
* HashSet支持传入一个集合构造方法的参数，此时容量的大小会是默认的16与集合参数大小扩容结果的最大值
	
		public HashSet(Collection<? extends E> c) {
	        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
	        addAll(c);
	    }

* HashSet支持指定初始化容量，甚至负载因子。

		public HashSet(int initialCapacity, float loadFactor) {
        	map = new HashMap<>(initialCapacity, loadFactor);
    	}

		public HashSet(int initialCapacity) {
	        map = new HashMap<>(initialCapacity);
	    }

## TreeSet
TreeSet是依据TreeMap实现的set集合。TreeSet的元素是有序的，根据构造方法不同，提供基于自然序以及通过Comparator比较结果的排序。

TreeSet操作的时间复杂度是log(n)

TreeSet，由Set维护的有序性(不管是不是提供了明确的Comparator进行比较)，其结果必须跟equals保持一致。

同样，TreeSet不是线程同步的。可以通过```Collections.synchronizedSortedSet(new TreeSet(...));```实现同步

同HashSet，TreeSet的迭代时快速失败的。