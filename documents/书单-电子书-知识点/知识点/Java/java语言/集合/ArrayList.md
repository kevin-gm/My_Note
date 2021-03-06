## ArrayList
1. ArrayList，底层使用数组实现的，默认容量大小为10，可以在初始化时指定容量大小。
	1. 不是线程安全的
	2. 支持null元素
	3. 元素可以重复
	4. 随机访问速度快，插入和移除性能较差(数组的特点)
2. 初始化时，可以传入另一个集合对象进行构造
	1. 首先将传入的集合对象转换为数组(collection接口方法toArray())
	2. 如果数组大小不为0，则将数组内容调用Arrays.copy方法赋值给ArrayList。
	3. 如果数组大小为0，则初始化一个空数组ArrayList。
3. trimToSize()：ArrayList的元素个数可能远小于本身容量大小，该方法可以清理空余容量，只保留元素个数的容量。
	1. modCount累加，该属性记录集合被修改的次数，用于判断集合是否被修改
	2. 如果元素个数(size)小于数组容量大小(elementData.length)，调用Arrays.copy将元素重新拷贝
4. ensureCapacity(int minCapacity)：增加当前ArrayList实例的容量。如有必要，需保证能容纳由最少元素参数(minCapacity)指定的元素
	1. 首先判断最小的扩展容量，如果当前不是空数组，则最小扩展容量为0，否则为默认容量10
	2. 如果目标最小容量 大于 最小扩展容量，则调用ensureExplicitCapacity(minCapacity)进行容量扩展
5. ensureExplicitCapacity(minCapacity)：
	1. modCount累加，该属性记录集合被修改的次数，用于判断集合是否被修改
	2. 如果目标最小容量 大于 当前元素数组长度，则调用grow(minCapacity)进行扩展
6. grow(minCapacity)：每次扩容50%
	1. 设置oldCapacity为当前元素数组长度
	2. 设置newCapacity为oldCapacity+oldCapacity的一半（为什么是一半）
	3. 如果newCapacity 小于 minCapacity，则调整newCapacity=minCapacity
	4. 如果newCapacity 大于 允许的最大数组大小，判断设置minCapacity为最大整数值或者允许的最大数组大小
	5. 调用Arrays.copy拷贝数组元素
7. contains(o)：列表是否包含某元素，实际调用indexOf方法进行判断
8. indexOf(o)：返回元素在列表中首次出现的位置索引，如果包含，返回位置索引，如果不包含返回-1.
	> 如果待判断元素o是null，则遍历列表，比对是否有元素为null，如果有直接返回当前索引。
	> 
	> 如果o不是null，则遍历列表，调用对象的equals方法判断是否有元素相等，如果有直接返回当前索引。
9. lastIndexOf(o)：返回元素在列表中最后出现的位置索引，方法同indexOf，只不过列表的遍历从末尾开始
10. clone()：克隆当前列表。实际时浅克隆，即元素本身是没有克隆的，只是克隆了元素的引用，调用object的clone方法生成ArrayList对象；Arrays.copyOf拷贝元素；modCount设置为0.
11. toArray()：返回包含列表所有的数组元素内容。这个方法实际调用的是Arrays.copyOf方法，也就是返回的是一个新生成的数组，可以安全的使用返回的数组。
12. get(index)：返回索引处的元素
13. set(index, element)：修改索引处的元素
14. add(e)：添加列表元素。首先会处理判断容量，如果容量不够，会先扩容。判断容量时，会累加modCount，因为对于列表而言，添加元素是改变了列表本身的。
15. add(index, element)：指定位置添加指定元素。
	1. 判断索引有效性。index 大于 列表长度，或者小于0，抛异常
	2. 处理容量，扩容为当前列表长度+1
	3. 复制并移动现有列表元素，也就是将index索引位置的元素以及之后的所有元素后移一位
	4. 设置index处元素为新元素。
16. remove(index)：删除指定索引位置的元素，并返回被删除元素
	1. 索引检查
	2. modCount累加
	3. index之后的元素往前移一位
	4. 设置数组最后一位为null，不再持有对象引用，方便GC发现并清理
17. remove(o)：删除列表第一个指定元素。遍历比对元素，如果删除成功返回true，否则false
18. clear()：删除所有元素。modCount累加，并设置所有索引值为null
19. addAll(Collection<? extends E> c)：将指定的集合内容添加到列表后面
20. addAll(int index, Collection<? extends E> c)：在指定索引位置插入指定集合，涉及到数组的移动。
21. removeAll(Collection<?> c)：从当前列表中删除所有指定集合中的元素。实际调用私有方法batchRemove(Collection<?> c, boolean complement)
22. retainAll(Collection<?> c)：从当前列表中删除，非指定集合中的元素，也就是只保留指定集合中的元素。实际调用私有方法batchRemove(Collection<?> c, boolean complement)
23. batchRemove(Collection<?> c, boolean complement)：私有方法。
	1. 新建一个数组对象，内容为当前数组元素
	2. 遍历，如果传入的集合包含列表元素，根据complement对数组重新赋值
	3. 如果最终的索引值不等于数组大小，可能意味着执行过程中有异常，则将未处理的数组元素往前移动。可能最后的结果是由于异常，结果不一定准确（什么时候会议常？手动抛异常测试，会导致）
	4. 如果不是所有元素都进行了重新赋值操作，将未变动的元素索引值设置为null，也就是往前移动了多少个元素，最后会有多少个元素设置为null
>
	private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
				//这一步没看懂
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }