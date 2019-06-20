一、概述(源码doc翻译)

CurrentHashMap是一个完全支持并发读，并具有高可预期并发性能更新的 Hash 表。这个类遵循与 HashTable相同的功能规范，并且包含有与HashTable每个方法相对应的方法版本。 然而，即使每个方法都是线程安全的，检索(获取)操作并不需要锁定，并且也不会锁定整个 table 而阻止其他访问。 在依赖于线程安全而不是同步细节的程序中，这个类与HashTable具有互操作性(也就是说，HashTable 与 CurrentHashMap都是线程安全的，只不过 CurrentHashMap 具有更高的并发性能)

检索(获取)操作(包括get)通常不会阻塞，所以可以与修改操作(包括put和remove)交替并发执行。检索操作反映了最近完成的更新操作的结果，更具意义的说法就是，给定key的更新操作，先行发生于(happens-before)任何获取到更新值的检索操作。 对于聚合操作(比如putAll和clear)，并发检索可能得到的是插入获取删除的某些entry。 同样，Iterators, Spliterators and Enumerations 返回的是在 iterator/enumeration 创建之后某个时刻的状态对应的元素。 不会抛出ConcurrentModificationException。 然而，iterators被设计为一次只能有一个线程使用。 请记住，聚合状态方法的结果(包括size，isEmpty，containsValue)，通常只有在没有其他线程并发更新时才有用，否则这些结果只能反映数据的瞬态，可以用于一个监控或者预估的目的，但不适合用于程序控制

当存在很多碰撞时(比如虽然 keys 具有不同的 hash码，但是计算索引后，落在同一个 slot 中)，Table 会动态扩容，每个映射维持大致的两个 bins 具有可预期的平均影响(对应于 resize 时0.75 这个负载因子)。 随着映射的添加和删除，这个平均值可能存在很大的差异，但是总的来说，这个维持了hash表在时间和空间上普遍接受的权衡。 然而，调整hash表大小的操作相对而言都是比较慢的，可能的话，在构造方法传入 预估的 大小(initialCapacity) 是一个好主意。一个可选的构造方法参数loadFactor，提供了另一种自定义初始化table容量的方式，通过指定用于为给定数量的元素计算需要分配的总空间的table密度。 此外，为了兼容此类以前的版本，构造方法可以选择指定预期的并发度(concurrencyLevel)作为内部大小调整的提示。 请注意，使用大量hashCode完全相同的key，对任何hash table而言都会减慢其性能。 为了改善影响，当 key 是 comparable时(实现了comparable接口)，此类可以使用 key 的顺序来打破这种关系。

ConcurrentHashMap的 Set 投影，可以通过 newKeySet() 或者 newKeySet(int) 创建，或者 当只有 keys 是感兴趣的，映射值(也许是瞬态的)没有被使用或采用相同的映射值，也可以使用 keySet(Object) 创建。

一个ConcurrentHashMap能够通过LongAdder作为value，并通过computeIfAbsent初始化后，被用于可伸缩频率映射(直方图或多集的形式)。比如 添加一个计数到 ```ConcurrentHashMap<String,LongAdder> freqs```， 就可以使用 ```freqs.computeIfAbsent(k -> new LongAdder()).increment();```

此类以及其视图和迭代器，实现了 Map 和 Iterator 的所有可选方法

类似于 Hashtable ，但不同于 HashMap，ConcurrentHashMap 不允许 key和value 为null

ConcurrentHashMap 支持一组顺序和并行批量操作，与大多数流式方法(Stream)不同，他们被设计为安全的且通常合理的使用，即使被其他线程同时并发更新也是如此。 比如，在共享注册表中计算只的快照摘要时。 有三种不同的操作，每种操作有四种形式，接受 Keys， values， Entries, 以及 (key,value) 参数，返回或不返回 values。 由于 ConcurrentHashMap 没有任何特定的方式来保证元素的顺序，在不同的并行执行中也可能表现出不同的顺序，所提供功能的正确性不应依赖于任何顺序，也不应取决于计算过程中可能瞬时变化的其他对象和值。 除了 forEach 动作外，理想情况下应该是无副作用的。 java.util.Map.Entry 对象上的批量操作不支持 setValue 方法。

- forEach： 每个元素上执行给定的操作。 一个在执行操作前对每个元素应用给定的转换方法的变种。
- search： 在每个元素上应用给定的方法，并返回第一个可用的非空(non-null)结果。 在找到结果后，跳过后续搜索。 也就是说找到符合的结果就返回。
- reduce： 累积每个元素。 提供的 reduction 功能不能依赖于顺序(更正式的说，它应该是联想(associative)的和可交换(commutative)的)。有五种变体
	- Plain reductions： 因为没有相应的返回类型，没有 (key, value) 这种方法参数形式
	- 累积应用于每个元素的方法结果的映射缩减(mapped reduction)
	- 使用给定的基值减少标量的 double，long，int


批量操作接受一个并行阀值(parallelismThreshold)的参数。 如果当前map的size预计会小于给定的阀值，则方法会顺序执行，使用 Long.MAX_VALUE 会抑制所有的并行性。