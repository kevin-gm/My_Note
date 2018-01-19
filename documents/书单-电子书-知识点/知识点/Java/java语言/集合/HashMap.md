
Map是一种键值映射的数据结构，一个Map不能包含重复的键，每一个键最多映射一个值。

## HashTable
HashTable是一种实现了键值映射关系的哈希表，任何**非null**的对象都可以作为键或者值。

HashTable是**线程安全**的，因为其数据变更的操作都有synchronized进行同步保证。

要成功并正确的在HashTable中存储和获取对象，这些对象必须实现**hashCode**和**equals**方法。

一个HashTable的实例有两个主要的影响性能的因素：初始化容量(**capacity**)和负载因子(**load factor**)。容量指的是HashTable中桶(<strong>buckets</strong>)的数量，初始化容量就是HashTable创建时的容量。负载因子是衡量在容量达到什么程度是触发自动扩容。
>每一个键值对就是一个条目(Entry)，hash值相同的对象置于一个链表中，称之为桶(bucket)，桶中的每一个元素也还是一个条目。

> 在HashTable中，当发生Hash碰撞时，具有相同hash值的对象存储在同一个桶的不同条目中，也就是一个桶有多个条目，搜索时，必须按照顺序查找。

> 默认的负载因子是0.75，在时间和空间开销方面提供了一个比较好的权衡。更高的值会降低空间开销，但是当查找条目时会增加时间开销

> 初始化容量时空间开销和耗费时间的rehash操作之间的权衡。当初始化容量大于HashTable将会包含的最大数量除以负载因子时，就会发生rehash操作。设置过大的初始化容量会造成空间浪费。如果有很多条目会存储到HashTable，设置一个充分大的初始化容量会比让它自动扩容要好。

HashTable的迭代时快速失败(fail fast)的迭代。迭代创建后的任何时刻进行结构性的修改，除了迭代器(iterator)自己的remove方法，都会抛出ConcurrentModificationException异常。

### HashTable主要属性
    private transient Entry<?,?>[] table;
Entry[]数组类型，HashTable实际存储的数据，entry代表了条目，每一个条目都是一个键值对。

	private transient int count;
HashTable中存储的条目数量

	private int threshold;
Hashtable的阈值，当大小超过这个阈值后就会发生rehash操作。threshold=(int)capacity*loadFactor

	private float loadFactor;
负载因子

	private transient int modCount = 0;
HashTable被结构化修改的次数。所谓结构化修改是指，改变HashTable中条目的数量，或者改变其内部结构(比如rehash)。这个字段是用于集合迭代时的快速失败的。

	private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
HashTable中数组的最大长度。有些JVM会在数组中存储一些额外的头信息，分配更大的数组大小可能会引发OutOfMemoryError异常

### HashTable构造方法

	1.指定初始化容量和负载因子
	public Hashtable(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);

        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        table = new Entry<?,?>[initialCapacity];
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    }

	2.指定初始化容量，负载因子为磨人的0.75
    public Hashtable(int initialCapacity) {
        this(initialCapacity, 0.75f);
    }

	3.使用默认，capacity为11，负载因子为0.75
    public Hashtable() {
        this(11, 0.75f);
    }

	4.传入一个Map集合
	public Hashtable(Map<? extends K, ? extends V> t) {
        this(Math.max(2*t.size(), 11), 0.75f);
        putAll(t);
    }

### HashTable主要常用方法

	// 返回HashTable中entry对象的数量
    public synchronized int size() {
        return count;
    }

     // 判断是否有键值对
    public synchronized boolean isEmpty() {
        return count == 0;
    }

     // 判断HashTable中是否有指定value值的key存在，如果指定的value为null，将会抛出空指针异常
     // 此方法比containsKey代价更大。
    public synchronized boolean contains(Object value) {
        if (value == null) {
            throw new NullPointerException();
        }
        // 定义局部变量，内容为HashTable的所有条目内容
        Entry<?,?> tab[] = table;
        // 遍历HashTable
        for (int i = tab.length ; i-- > 0 ;) {
        	// 此处是遍历每一个条目的桶(buckets)对应的内容，前面说过，桶实际就是entry链表
            for (Entry<?,?> e = tab[i] ; e != null ; e = e.next) {
            	// 通过value的equals方法进行匹配，找到就返回true
                if (e.value.equals(value)) {
                    return true;
                }
            }
        }
        return false;
    }

     // 判断是否包含指定的value值，实际调用的是contains方法
    public boolean containsValue(Object value) {
        return contains(value);
    }

     // 判断是否包含指定的key
     // 前面说过，contains方法比containsKey方法代价大，因为contains需要整个遍历，而containsKey只需要遍历指定的链表
    public synchronized boolean containsKey(Object key) {
    	// HashTable所有元素
        Entry<?,?> tab[] = table;
        // 获取指定key的hash值
        int hash = key.hashCode();
        // 计算指定的key值在HashTable中的位置索引
        int index = (hash & 0x7FFFFFFF) % tab.length;
        // 遍历指定索引位置的桶(bucket)，此时找到的是唯一的链表
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
        	// 匹配条件是，key的hash值相等，并且key的equals方法返回true
            if ((e.hash == hash) && e.key.equals(key)) {
                return true;
            }
        }
        return false;
    }

     // 获取指定key 的value值，如果没有，则返回null
    @SuppressWarnings("unchecked")
    public synchronized V get(Object key) {
        Entry<?,?> tab[] = table;
        // 获取指定key的hash值
        int hash = key.hashCode();
        // 计算指定key在数组中的索引位置
        int index = (hash & 0x7FFFFFFF) % tab.length;
        // 遍历指定索引位置的链表，获取指定key对应的value值。
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
        	// 匹配条件是键的hash值和equals匹配。
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }

     // 为了更有效的容纳和访问条目，而增加容量和进行内部调整。
     // 这个方法会在超过HashTable的容量和负载因子的计算值后自动触发。
    @SuppressWarnings("unchecked")
    protected void rehash() {
    	// 当前HashTable数组的长度
        int oldCapacity = table.length;
        // 当前HashTable的所有键值对
        Entry<?,?>[] oldMap = table;

        // 扩容的目标容量时原容量的2倍，为什么要+1？
        int newCapacity = (oldCapacity << 1) + 1;
        // 如果目标容量大于最大的数组大小(MAX_ARRAY_SIZE)，则设置为最大数组大小
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
            if (oldCapacity == MAX_ARRAY_SIZE)
                // Keep running with MAX_ARRAY_SIZE buckets
                return;
            newCapacity = MAX_ARRAY_SIZE;
        }
        // 以新容量定义新的条目数组buckets
        Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];
        // 维护的被修改次数累加
        modCount++;
        // 重新计算负载的阈值
        threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        table = newMap;
        // 从数组的尾端开始遍历数组
        for (int i = oldCapacity ; i-- > 0 ;) {
        	// 在对数组中的每一个entry链表进行遍历
            for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            	// 代表循环中当前的entry对象
                Entry<K,V> e = old;
                // 设置遍历的指针为当前entry对象指向的下一个entry对象，实现遍历
                old = old.next;
                // 重新计算键在数组中的索引位置
                int index = (e.hash & 0x7FFFFFFF) % newCapacity;
                // 设置当前entry对象的下一个节点为新数组本来节点对象
                // 新数组其实最开始可能是个空数组，这个地方意味着，每次添加元素时，新添加的元素都会在链表的前面。
                // 设置新数组当前索引位置的元素为之前的entry对象。
                e.next = (Entry<K,V>)newMap[index];
                newMap[index] = e;
            }
        }
    }

    // 添加entry对象
    private void addEntry(int hash, K key, V value, int index) {
        modCount++;

        Entry<?,?> tab[] = table;
        // 如果当前entry对象的大小，大于阈值，则需要进行扩容，重新hash
        if (count >= threshold) {
            // Rehash the table if the threshold is exceeded
            rehash();

            tab = table;
            hash = key.hashCode();
            index = (hash & 0x7FFFFFFF) % tab.length;
        }

        // Creates the new entry.
        @SuppressWarnings("unchecked")
        // 重新扩容后指定索引位置的entry对象
        Entry<K,V> e = (Entry<K,V>) tab[index];
        // 新建一个entry对象，其下一个entry节点指向原entry对象，道理同rehash中对应的。
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }

     // 添加键值映射关系，键和值都 不可以 为null
    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        // 遍历链表
        for(; entry != null ; entry = entry.next) {
        	// 如果存在匹配的key，则设置key的value为指定的value，并返回被更改的value值。
        	// 也就是添加时，如果key相同，则会将value进行覆盖。
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }
        // 否则调用addEntry建立关系
        addEntry(hash, key, value, index);
        return null;
    }

     // 从HashTable中删除指定的key，如果没有，则不做任何处理
    public synchronized V remove(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        // 根据计算的索引，获取数组指定元素，也就是对应的桶
        Entry<K,V> e = (Entry<K,V>)tab[index];
        // 遍历桶的链表
        for(Entry<K,V> prev = null ; e != null ; prev = e, e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                modCount++;
                if (prev != null) {
                    prev.next = e.next;
                } else {
                    tab[index] = e.next;
                }
                count--;
                V oldValue = e.value;
                e.value = null;
                return oldValue;
            }
        }
        return null;
    }

    // 将Map集合维护到当前HashTable中
    public synchronized void putAll(Map<? extends K, ? extends V> t) {
        for (Map.Entry<? extends K, ? extends V> e : t.entrySet())
            put(e.getKey(), e.getValue());
    }

    // 清空HashTable，即数组的每一个元素都设置为null
    public synchronized void clear() {
        Entry<?,?> tab[] = table;
        modCount++;
        for (int index = tab.length; --index >= 0; )
            tab[index] = null;
        count = 0;
    }

     // HashTable克隆，是浅克隆，只克隆结构，不克隆键和值
     // 这是一个代价相当大的操作
    public synchronized Object clone() {
        try {
            Hashtable<?,?> t = (Hashtable<?,?>)super.clone();
            t.table = new Entry<?,?>[table.length];
            for (int i = table.length ; i-- > 0 ; ) {
                t.table[i] = (table[i] != null)
                    ? (Entry<?,?>) table[i].clone() : null;
            }
            t.keySet = null;
            t.entrySet = null;
            t.values = null;
            t.modCount = 0;
            return t;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }

    // 重写tostring方法，输出键值
    public synchronized String toString() {
        int max = size() - 1;
        if (max == -1)
            return "{}";

        StringBuilder sb = new StringBuilder();
        Iterator<Map.Entry<K,V>> it = entrySet().iterator();

        sb.append('{');
        for (int i = 0; ; i++) {
            Map.Entry<K,V> e = it.next();
            K key = e.getKey();
            V value = e.getValue();
            sb.append(key   == this ? "(this Map)" : key.toString());
            sb.append('=');
            sb.append(value == this ? "(this Map)" : value.toString());

            if (i == max)
                return sb.append('}').toString();
            sb.append(", ");
        }
    }

	// 返回key的集合
	public Set<K> keySet() {
        if (keySet == null)
            keySet = Collections.synchronizedSet(new KeySet(), this);
        return keySet;
    }

	// equals方法，每个key值都要相等
	public synchronized boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map))
            return false;
        Map<?,?> t = (Map<?,?>) o;
        if (t.size() != size())
            return false;

        try {
            Iterator<Map.Entry<K,V>> i = entrySet().iterator();
            while (i.hasNext()) {
                Map.Entry<K,V> e = i.next();
                K key = e.getKey();
                V value = e.getValue();
                if (value == null) {
                    if (!(t.get(key)==null && t.containsKey(key)))
                        return false;
                } else {
                    if (!value.equals(t.get(key)))
                        return false;
                }
            }
        } catch (ClassCastException unused)   {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }

        return true;
    }

## HashMap
+ HashMap是依托于Hash表实现的Map，拥有所有的map实现，允许null的键和null的值。
+ HashMap可以大致的等同于HashTable，除了不是线程安全的并允许null。
+ Hashmap不保证数据有序性，特别的，也不保证顺序随时间变化而维持不变。
+ HashMap的基本操作get和put方法有恒定的性能，假设散列函数(hash)在桶(buckets)之间正确的分散元素。
+ HashMap迭代的时间与容量(capacity)成正比。这个容量是指HashMap实例(buckets数量)加上其大小(key-value键值映射数量)。所以当迭代性能要求很重要的情况下，不要将初始容量设置太高，或者负载因子设置太低。
+ 有两个影响HashMap实例性能的参数：容量(capacity)和负载因子(load factor)。capacity是指hash表(hash table)中桶(buckets)的数量，负载因子是散列表在其容量自动增加前所允许的已有元素满度的度量。当散列表中条目(entries)的数量超过负载因子和当前容量的乘积时，散列表将会触发rehash操作(内部数据结构将会重建)，来获得大约两倍的桶数量。
+ 一般来说，默认的负载因子(0.75)提供了时间和空间开销的一个良好折中。更大的值会减少空间开销，但是会增加遍历的开销。(减少空间开销体现在，本来有12个元素就需要rehash，如果是16个元素再rehash，减少了rehash的次数，也就减少了使用不到的空间出现的可能)。在设置初始容量时，应当考虑map中条目的数量和负载因子，以便最小化rehash操作的次数。如果初始容量大于最大条目数除以负载因子，则不会发生rehash。
+ 如果map中会存储非常多的映射，创建时指定一个充分大的容量，会比使用默认值，并自动rehash更有效率。(因为会减少rehash的次数)。注意，如果有很多key具有相同的散列码(hashcode)，这将会很明确的减慢散列表的性能。为了改善这个影响，当key是Comparable实现时，可以通过key之间的比较顺序来断开连接。
+ HashMap不是线程安全的，如果有多个线程并发访问hashmap，并且至少有一个线程会结构化的修改map，则必须进行额外的同步操作
+ HashMap的迭代时快速失败的迭代。
+ 默认的初始化容量大小为16；最大容量为2^30(1<<30)；默认的负载因子为0.75；默认的链表转树的阈值为8