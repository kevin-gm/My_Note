LinkedList类定义

	public class LinkedList<E> 
		extends AbstractSequentialList<E>
    	implements List<E>, Deque<E>, Cloneable, java.io.Serializable

双向列表**LinkedList**继承自**AbstractSequentialList**，实现了**List、Deque、Cloneable和Serializable**接口

1. AbstractSequentialList继承自<strong>AbstractList</strong>，也就是说，AbstractSequentialList提供了有序访问的List的最小的骨干实现，LinkedList是在有基本List特性的基础上，添加了链表的特性。
2. Deque是一个线性collection，支持两端插入和移除元素，定义了双端队列的操作


LinkedList是基于双向链表实现的，JDK1.8版本的LinkedList的关键属性为：

	//size表示当前列表的大小
	transient int size = 0;

    //指向列表的第一个元素
    transient Node<E> first;

    //指向列表的最后一个元素
    transient Node<E> last;

Node是一个LinkedList的内部类，定义了LinkedList的节点结构

	//LinkedList列表实际存储的元素
	E item;
	//当前元素节点指向的前一个节点
    Node<E> next;
	//当前元素节点指向的后一个节点
    Node<E> prev;

> LinkedList正是由于每个节点都会指向前后节点，所以不需要通过数组来保证元素有序。
> 
> 所以LinkedList操作时，删除和添加元素效率是非常高的，因为只需要修改相关元素的前后的指向关系。
> 
但是随机访问效率低，因为必须遍历列表。具体看后续方法说明。
>
LinkedList不是线程安全的，操作未提供任何安全保证。
>
LinkedList本质上还是一个List，也就是元素是有序的，依靠内部节点指向的前后节点维持顺序

LinkedList提供无参构造方法以及集合参数的构造方法

	//构造一个空列表
    public LinkedList() {}

	//构造一个包含指定集合元素的LinkedList列表，元素的顺序为集合本身的迭代顺序
    public LinkedList(Collection<? extends E> c) {
		//首先调用自身的无参构造方法，构造一个空列表
        this();
		//然后将集合得元素添加到当前列表
        addAll(c);
    }

##### linkFirst(E e)
将元素e设置为首元素，即设置为链表表头节点，实际就是插入元素

	private void linkFirst(E e) {
		//获取当前表头节点f
        final Node<E> f = first;
		//构造一个新的节点newNode，关联的前节点为null，后节点为当前表头f，元素内容为e
        final Node<E> newNode = new Node<>(null, e, f);
		//设置表头节点first是新节点newNode
        first = newNode;
		//调整之前节点的关联关系。
		//如果之前的表头节点f是null，代表之前链表实际没有元素，则将链表的表尾节点也设置为新节点newNode；
        if (f == null)
            last = newNode;
		//否则将之前的表头节点f的前节点关联为newNode，也就是现在newNode是f的前一个节点
        else
            f.prev = newNode;
		//链表大小size累加        
		size++;
		//列表变化modCount累加
        modCount++;
    }

##### linkLast(E e)
将元素e设置为链表的尾节点，即设置为链表的表尾节点，实际时插入元素

	void linkLast(E e) {
		//获取当前表尾节点l
        final Node<E> l = last;
		//构造一个新节点newNode，关联的前节点为l，尾节点为null，元素内容为e
        final Node<E> newNode = new Node<>(l, e, null);
		//设置表尾节点last为新节点newNode
        last = newNode;
		//调整之前节点的关联关系。
		//如果之前的表尾节点是null，代表之前列表实际没有元素，则设置链表的表头节点也为newNode；
        if (l == null)
            first = newNode;
		//否则设置之前表尾节点l的后一个关联节点为newNode。
        else
            l.next = newNode;
		//链表大小累加
        size++;
		//列表变化modCount累加
        modCount++;
    }

##### linkBefore(E e, Node<E> succ)
将元素e插入到非空节点succ之前，实际是插入元素

	void linkBefore(E e, Node<E> succ) {
		//判断节点succ是否为空。（jdk代码中将这个判断注释掉了）
        // assert succ != null;
		//获取节点succ之前的节点pred。
        final Node<E> pred = succ.prev;
		//构造一个新节点newNode，关联的前节点为pred，尾节点为succ，元素内容为e
        final Node<E> newNode = new Node<>(pred, e, succ);
		//设置succ的前节点为新节点newNode
        succ.prev = newNode;
		//调整节点的关联关系。
		//如果pred是null，代表之前succ是头结点，则要讲新节点newNode设置为表头节点；
        if (pred == null)
            first = newNode;
		//否则设置pred的后节点为新节点newNode
        else
            pred.next = newNode;
		//链表大小累加
        size++;
		//列表变化modCount累加
        modCount++;
    }

##### unlinkFirst(Node<E> f)
取消非空节点f为首节点，并返回f对应的元素，实际是删除节点

	private E unlinkFirst(Node<E> f) {
		// 判断节点是否是首节点并且不为空（JDK代码将其注释掉了）
        // assert f == first && f != null;
		// 获取当前节点的元素
        final E element = f.item;
		// 获取节点f的下一个节点next
        final Node<E> next = f.next;
		// 将节点f的元素设置为null
        f.item = null;
		// 将节点f的下一个节点设置为null，这样就没有对象引用f，便于GC
        f.next = null; // help GC
		// 设置首节点为之前f的下一个节点next
        first = next;
		// 如果之前f的下一个节点next为null，代表链表只有一个节点，所以现在需要把为节点设置为null
        if (next == null)
            last = null;
		// 否则将之前f的前一个节点next设置为null，也就是现在next是首节点了
        else
            next.prev = null;
		// 链表大小 -1
        size--;
		// 链表变化次数 +1
        modCount++;
		// 返回变更节点的元素内容
        return element;
    }

##### unlinkLast(Node<E> l)
取消非空节点l为链表尾节点，并返回变更节点的元素内容，实际是删除节点

	private E unlinkLast(Node<E> l) {
		// 判断节点l是否为尾节点，并且非空（JDK代码注释了）
        // assert l == last && l != null;
		// 获取节点l的元素内容
        final E element = l.item;
		// 获取节点l的前一个节点prev
        final Node<E> prev = l.prev;
		// 设置l的元素内容为null
        l.item = null;
		// 设置l的前一个节点为null，这样就没有对象引用l，便于GC
        l.prev = null; // help GC
		// 设置链表的尾节点为prev
        last = prev;
		// 如果之前l的前一个节点prev为null，代表链表之前只有一个元素，所以需要把首节点设置为null
        if (prev == null)
            first = null;
		// 否则设置之前l的下一个节点prev的下一个节点为null，因为现在prev为尾节点
        else
            prev.next = null;
		// 链表大小 -1
        size--;
		// 链表变化次数 +1
        modCount++;
		// 返回变更节点的元素内容
        return element;
    }

##### unlink(Node<E> x)
取消某个节点的关联关系，并返回变更节点的元素内容，实际是删除节点

	E unlink(Node<E> x) {
		// 判断节点是否为空（JDK代码注释了）
        // assert x != null;
		// 获取节点x的元素内容
        final E element = x.item;
		// 获取节点x的下一个节点next
        final Node<E> next = x.next;
		// 获取节点x的上一个节点prev
        final Node<E> prev = x.prev;
		// 如果上一个结点prev为null，代表节点x为首节点，所以需要将x的下一个节点next设置为链表的首节点first
        if (prev == null) {
            first = next;
        } else {
			//否则设置prev的下一个节点为next，也就是x被删除，它之前的前后关系建立链接关系
            prev.next = next;
			// 设置x的前一个节点为null，去掉对象间的引用
            x.prev = null;
        }
		// 如果x的下一个节点为null，代表x为尾节点，所以需要设置x的上一个节点prev为链表的尾节点last
        if (next == null) {
            last = prev;
        } else {
			// 否则，设置next的上一个节点为prev，同样是建立x之前的前后节点的关联关系
            next.prev = prev;
			// 设置x的下一个节点为null，去掉对象间的引用
            x.next = null;
        }
		// 设置x的元素为null
        x.item = null;
		// 链表大小 -1
        size--;
		// 链表变化次数 +1
        modCount++;
		// 返回变更节点的元素内容
        return element;
    }

##### E getFirst()
返回列表的第一个元素，实际就是首节点的元素内容。如果首节点为null，则抛出NoSuchElementException异常。

##### E getLast()
返回列表的最后一个元素，实际就是尾节点的元素内容。如果尾节点为null，则抛出NoSuchElementException异常。

##### E removeFirst()
删除并返回列表的第一个元素，实际调用的是unlinkFirst方法。如果首节点为null，则抛出NoSuchElementException异常

##### E removeLast()
删除并返回列表的最后一个元素，实际调用的是unlinkLast方法。如果尾节点为null，则抛出NoSuchElementException异常

##### void addFirst(E e)
在列表的头部添加元素，实际调用的是linkFirst方法

##### void addLast(E e)
在列表的尾部添加元素，实际调用的是linkLast方法。

这个方法等价于add(E e)

##### boolean contains(Object o)
返回列表是否包含某个元素，实际调用的是indexOf方法

##### int size()
返回列表的大小，即元素个数的数量

##### boolean add(E e)
添加元素，成功返回true，实际调用的是linkLast方法。

也就是说，默认添加元素，是将元素添加到列表的尾部。

##### boolean remove(Object o)
删除元素，删除匹配到的**第一个**元素，并返回是否成功。

遍历列表，根据o是否为null进行比对，实际调用unlink方法

##### boolean addAll(Collection<? extends E> c)
将指定集合得元素插入到列表，实际调用addAll(size, c)

##### boolean addAll(int index, Collection<? extends E> c)
将指定集合插入到列表指定的位置，需要移动指定index的前后元素。(实际对于链表而言并没有需要移动元素，因为节点的有序关系是通过节点的前后指向维系的，不过从索引位置来看，确实发生了移动)

插入元素的顺序为原集合得迭代顺序

	public boolean addAll(int index, Collection<? extends E> c) {
		//检查指定位置索引的有效性，只有当index>=0并且<=size才是有效的
		//如果检查不通过，则抛出索引越界IndexOutOfBoundsException异常
        checkPositionIndex(index);
		// 将指定的集合转换为对象数组
        Object[] a = c.toArray();
		// 获取指定集合得长度numNew，即待插入列表的元素个数
        int numNew = a.length;
		// 如果numNew等于0，代表没有元素需要插入。直接返回false
        if (numNew == 0)
            return false;
		// 定义两个节点，succ代表index索引位置的当前节点，pred代表当前节点的前一个节点
        Node<E> pred, succ;
		// 如果索引值index等于原列表的长度，则设置当前节点succ为null，前一个节点pred为原链表的尾节点
		// 正常对于列表而言，索引的最大值应该是比列表长度小1的，因为索引是从0开始的
		// 所以当index=size时，代表正好这个位置是原列表最后一个元素的下一个位置
		// 我们建立链接关系时，肯定当前节点是null并且其上一个节点就是原列表的最后一个节点。
        if (index == size) {
            succ = null;
            pred = last;
        } else {
			// 否则，因为索引值前面检查过，肯定能找到对应的某一个节点
			// 查找并设置当前节点succ
            succ = node(index);
			// 设置pred为当前节点的前一个节点
            pred = succ.prev;
        }
		// 遍历待插入的元素数组
        for (Object o : a) {
			// 此处有一个类型强转
            @SuppressWarnings("unchecked") 
			E e = (E) o;
			// 构造一个新节点newNode，对应的前一个节点为前面设置的pred，后一个节点为null，元素为对应的数组元素
			// 因为是在指定索引位置插入，就是要把之前节点的前一个节点与新节点的前一个节点对应，但是当前没有执行后一个节点，所以后一个节点为null
            Node<E> newNode = new Node<>(pred, e, null);
			// 如果pred为null，代表索引指向的原始链表节点为首节点，所以需要把当前节点设置为首节点
            if (pred == null)
                first = newNode;
			// 否则，设置前一个节点的下一个节点为当前节点，到这里，新节点newNode与前一个节点pred，双方都建立了指向关系
            else
                pred.next = newNode;
			// 重新设置前一个节点为当前新的节点，因为是在循环里面，后续节点需要用到并建立关系
            pred = newNode;
        }
		// 循环结束后，处理原始索引位置的节点
		// 如果原始索引位置的节点succ为null，代表是从列表的最后开始插入元素的
		// 所以设置链表的尾节点为最新的pred，也就是集合最后一个元素生成的新节点。
        if (succ == null) {
            last = pred;
        } else {
			// 否则设置循环后的最后一个节点的后一个节点为原始索引位置的节点
            pred.next = succ;
			// 并设置原始索引位置节点的前一个节点为循环后的最后一个节点，建立双向关系
            succ.prev = pred;
        }
		// 列表大小=原大小+数组长度
        size += numNew;
		// 链表变化次数 +1
        modCount++;
		// 返回插入成功
        return true;
    }

##### void clear()
删除列表的所有元素，成功后，列表是个空列表

遍历列表元素，通过节点指向的后一个节点进行遍历，设置节点元素，前一个节点的指向，后一个节点的指向为null。

遍历后，首尾节点设置为null，size设置为0，链表变化次数 +1

##### E get(int index)
获取指定索引位置的元素

* 检查索引有效性，此处索引必须大于等于0，并且小于size，否则抛IndexOutOfBoundsException（因为索引从0开始）
* 调用node(index)获取节点并返回改节点的元素值

##### E set(int index, E element)
修改指定索引位置的元素值，并返回变更前的值

* 检查索引有效性
* 获取索引对应的节点
* 赋值并返回老值

##### void add(int index, E element)
在指定位置插入新的元素

* 检查索引有效性，index可以等于列表大小size
* 如果index=size，则调用linkLast方法插入
* 否则调用linkBefore方法插入元素

##### E remove(int index)
删除指定索引位置的元素。检查索引有效性后，调用unlink方法进行删除

##### Node<E> node(int index)
返回指定索引位置对应的非空节点

采用的折半查找的算法

##### int indexOf(Object o)
返回对象在列表中第一次出现的索引位置，可以用来判断列表是否包含对象，如果找不到返回-1

实际就是遍历列表，根据对象o是否为null进行比对，如果匹配则返回对应的索引值。

##### int lastIndexOf(Object o)
返回对象在列表中最后一次出现的索引位置。实现方式基本同indexOf，只不过是从列表的最末尾开始遍历。

##### E peek()
获取，但不删除列表的第一个元素。

**如果找不到，则返回null**

此处实际是返回列表首节点的元素值。

##### E element()
获取，但不删除列表的第一个元素

**如果找不到，将会抛出NoSuchElementException异常**

实际调用的是getFirst方法

##### E poll()
获取，并删除列表的第一个元素

**如果找不到，则返回null**

如果找到了，则调用unlinkFirst方法将元素进行删除并返回

##### E remove()
* 删除列表的首节点元素
* 如果找不到，抛NoSuchElementException异常
* 实际调用removeFirst方法，也就是默认删除首节点元素

##### boolean offer(E e)
末尾添加元素，实际调用的是add方法

##### boolean offerFirst(E e)
在列表的最前面添加元素，双向链表的特性。实际调用的是addFirst方法

##### boolean offerLast(E e)
在列表的末尾添加元素，实际调用addLast方法

##### E peekFirst()
同peek方法，获取但不删除列表的第一个元素

##### E peekLast()
获取但不删除列表的最后一个元素，如果找不到则返回null

##### E pollFirst()
同poll方法，获取并删除第一个元素，找不到则返回null

##### E pollLast()
获取并删除最后一个元素，找不到则返回null

##### void push(E e)
入栈，即在列表的头部插入元素，无返回值，实际调用addFirst方法

##### E pop()
出栈，即在列表的头部获取元素，返回元素值，实际调用removeFirst方法，也就是出栈会将列表元素删除

##### boolean removeFirstOccurrence(Object o)
删除列表中第一个匹配指定对象的元素

##### boolean removeLastOccurrence(Object o)
删除列表中最后一个匹配指定对象的元素


### 获取元素，包括队列逻辑上的
* getFirst：获取第一个元素，找不到抛NoSuchElementException异常
* getLast：获取最后一个元素，找不到抛NoSuchElementException异常
* get(index)：获取指定索引位置的元素
* peek：队列操作。获取但不删除第一个元素，找不到返回null
	* peekFirst
	* peekLast
* element：队列操作。获取但不删除第一个元素，找不到抛NoSuchElementException异常
* poll：队列操作。获取并删除第一个元素，找不到返回null
	* pollFirst
	* pollLast
* remove：队列操作。获取并删除第一个元素，找不到抛NoSuchElementException异常
* pop：栈操作。获取并删除第一个元素，找不到抛NoSuchElementException异常

### 添加元素
* addFirst：在首位添加
* addLast：在末尾添加
* add：在末尾或者指定索引位置添加
* addAll：在指定位置添加集合元素
* offer：队列操作。在末尾添加
* offerFirst：队列操作。在首部添加
* offerLast：队列操作。在末尾添加
* push：栈操作，首部添加