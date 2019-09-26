1. Kafka的用途有哪些？使用场景如何？
  - 作为消息系统：系统解耦，流量削峰，异步通信，冗余存储等，kafka同时还能对消息进行回溯和保证分区有序
  - 作为存储系统：kafka会把消息持久化到磁盘，降低了消息丢失的风险，也正因为会持久化到磁盘，所以可以当做存储系统使用
  - 流式处理平台：kafka提供了完整的流式处理类库
  
2. Kafka中的ISR、AR又代表什么？ISR的伸缩又指什么
  - AR：分区中所有的副本称为AR
  - ISR：与leader副本保持一定程度同步的副本集合为ISR。对于producer配置的acks=all/-1时，需要ISR集合都同步完之后才认为已提交，之后会更新分区的HW，
  消费者才可以消费到这条消息
  - 最开始的时候，AR=ISR，在follower与leader不断同步的过程中，由于诸如follower宕机、follower消费缓慢等原因，
  造成follower副本与leader不再保持一定程度的同步，则需要将落后的follower副本从ISR中剔除；又随着时间的推移，follower副本重新上线并且与leader
  最终保持一定程度的同步，则有需要将follower副本添加到ISR集合中
  - 一定程度的同步，当前时间戳与副本的lastCaughtUpTimeMs比对，超过一定阀值(replica.lag.time.max.ms)，则副本变成失效副本。还有一个不再使用的消息数
  大小(replica.lag.max.messages)
  - ISR的伸缩，kafka是通过两个定时任务实现的：isr-expiration 和 isr-change-propagation
  - isr-expiration会周期性地检查每个分区是否需要缩减其ISR集合，这个周期与replica.lag.time.max.ms有关，大小是其值的一半，默认为5000ms。当检测到ISR集合
  中有失效副本时，就会收缩ISR集合。
  - ISR集合发生变更时，会将变更后的记录缓存到isrChangeSet中，isr-change-propagation会周期性的检查isrChangeSet，如果发现ISR有变更记录，则会在zookeeper
  中创建顺序节点，并添加Watcher监听，也就是发生ISR变更时，会通知控制器更新相关元数据并向它管理的broker节点发送更新元数据的通知
  
3. Kafka中的HW、LEO、LSO、LW等分别代表什么？
  - LEO：是指分区中最后一条消息的下一个位置
  - HW：是指分区对应的ISR集合中，最小的LEO，俗称高水位
  - LSO：LSO与事务有关，对于未完成的事务，LSO即事务中第一条消息的offset；对于已完成的事务而言，LSO=HW。LSO<=HW<=LEO
  - LW：是指AR中最小的logStartOffset，俗称低水位
  
4. Kafka中是怎么体现消息顺序性的？
  - Kafka支持单分区级别的消息顺序
  - 消息在被追加到分区的日志文件时，会分配一个特定的偏移量，这个offset是消息在日志文件中的唯一标识，kafka通过这个offset来保证分区有序性
  - 但存在这样的场景，消息A和B先后发送到同一个分区，A由于网络抖动导致发送失败，B发送成功，然后重试发送A，那么此时消息就乱序了
  - 对于消息需要严格有序的场景而言，设置 max.in.flight.requests.per.connection=1，代表broker端在响应之前，client端不能再向同一个broker发送请求，
  但这会降低kafka的吞吐量；producer端，使用异步回调发送 producer.send(record, callback) 
  
5. Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？
  - producer端的消息会依次经过：拦截器、序列化器、分区器的处理后，发送到broker
  - consumer端收到消息后，也会依次经过反序列化器、拦截器的处理
  - 序列化器：网络传输的是字节流，所谓序列化即将数据序列化成字节数组的过程，不同的序列化器能对数据进行不同程度的压缩以及不同的序列化性能。需要注意的是，
  producer端和consumer端使用的序列化器需要一一对应
  - 分区器：分区器是用于确定消息需要发送到哪个分区，如果构建的消息ProducerRecord指定了分区属性，那么就不需要再经过分区器的处理了。如果没有，
  则根据key计算分区：如果key不为null，则对key进行hash(采用MurmurHash2算法)，根据hash值与topic分区数求余计算分区号；如果key为null，则消息以
  轮询的方式发送到topic的各个分区。需要注意，如果key不为null，则计算得到的分区号是所有分区中的任意一个，而如果key等于null，则是可用分区中的任意
  一个。Kafka默认分区器代码如下：
  
      public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
          List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
          int numPartitions = partitions.size();
          if (keyBytes == null) {
              int nextValue = nextValue(topic);
              List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
              if (availablePartitions.size() > 0) {
                  int part = Utils.toPositive(nextValue) % availablePartitions.size();
                  return availablePartitions.get(part).partition();
              } else {
                  return Utils.toPositive(nextValue) % numPartitions;
              }
          } else {
              return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
          }
      }
    
  - 拦截器：可以用来在消息发送前进行一些处理，比如过滤掉不符合要求的消息，修改消息内容等，也可以用来在发送回调逻辑前做一些定制化的需求。KafkaProducer
  会在消息被应答之前或者消息发送失败时调用生产者拦截器的onKnowledgement() 方法，优先于用于设定的callback
    
5. Kafka生产者客户端的整体结构是什么样子的？Kafka生产者客户端中使用了几个线程来处理？分别是什么？
  - producer客户端由两个线程协调运行，主线程用于创建消息，并经过拦截器、序列化器、分区器的处理后，将消息缓存到消息累加器(RecordAccumulator)；Send线程用于从RecordAccumulator获取消息并发送到kafka中
  - RecordAccumulator 主要用于缓存消息以便Sender线程可以批量发送，进而减少网络传输的资源消耗以提升性能
  - 请求从Sender线程发往kafka之前，还会保存到InFlightRequests中，其作用是缓存应发出去但还没有收到响应的请求

6. Kafka的旧版Scala的消费者客户端的设计有什么缺陷？

7. 消费组中的消费者个数如果超过topic的分区，那么就会有消费者消费不到数据”这句话是否正确？如果正确，那么有没有什么hack的手段？
  - 默认情况下是正确的，每一个分区只能被一个消费组中的一个消费者所消费，所以如果消费者个数大于topic对应的分区数，那么就会有消费者没有分配的分区而无法消费消息
  - KafkaConsumer的partionsFor方法能获取到每个topic所对应的分区信息，通过assign，seek等方法，能自行决定消费的分区和位移，可以打破原有的消费线程的个数不能超过分区数的限制
  
8. 消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1?
  - 消费者位移提交的是 offset+1，即下一条需要拉取消息的位置
  
9. 有哪些情形会造成重复消费？
  - 消费者拉取到消息后，等待所有的消息都处理完后再提交消费位移，但是消息处理过程中出现了异常。比如，某次拉取到的消息的偏移量为 [x,x+5]，消费逻辑为等到x+5处理完再提交消费位移，但当处理到x+3时，消费者出现了异常，所以本次消费位移没有提交。故障恢复后，重新拉取消息，还是会从x拉取，那么 x 到 x+2 之间的消息就会重复消费
  - 发生再均衡时，消费者C1拉取到分区P的消息进行处理，处理过程中发生了再均衡操作，但消费位移尚未提交，之后分区P分配给了消费者C2进行消费，则之前已经被消费完的消息又会重新消费一遍

10. 那些情景下会造成消息漏消费（消息丢失）
  - 消费者拉取到消息后，立刻提交消费位移。比如，某次拉取到的消息的偏移量为 [x,x+5]，拉取到消息后，提交了消费位移 x+5，然后消费者进程出现异常，则 x到x+5之间尚未处理的消费就丢失了。因为下一次拉取消息时，会从 x+6 开始拉取
  - 多线程消费实现方式三，各个线程处理提交的消费位移不一样，也会导致消息丢失

11. KafkaConsumer是非线程安全的，那么怎么样实现多线程消费？
  - KafkaConsumer中的每个公共方法(wakeup除外)都会调用acquire方法，判断是否发生了并发操作，来保证KafkaConsumer只能被一个线程使用。
  - 方式一：多线程消费，每个线程实例化一个KafkaConsumer，其实现基本等价于多个消费者进程，这种实现，每个消费线程都需要维护一个独立的TCP连接
  - 方式二：多个消费线程消费同一个分区，通过assign，seek实现，但这种方式对于位移提交和顺序控制很复杂，不推荐使用
  - 方式三：单线程拉取消息，多线程处理，一般而言，poll拉取消息非常快。这种方式，如果要保证消息的顺序会比较复杂，对于位移的提交，可以使用共享的offset来保存消费位移，在poll方法执行时提交位移，注意对offsets的加锁处理
  - 方式四：滑动窗口。本地预先定义一个大小固定的缓存队列，消费者拉取到的消息，先缓存到队列，然后多线程消费队列的消息。队列头部的消息处理完后，提交其对应的消费位移，删除原来startOffset位置的消息，拉取最新的进入窗口，即窗口向前滑动。

12. 简述消费者与消费组之间的关系
  - 消费者负责订阅kafka的主题并拉取消息
  - 消费组是一个逻辑概念，是对消费者的一个归类，其名称可以通过 group.id 配置，消费者不是逻辑概念，其对应一个消费线程或者进行
  - 每个消费者都有一个对应的消费组，每条消息只能被投递给每个消费组中的一个消费者
  - 如果所有的消费者都属于同一个消费组，那么所有的消息会均衡的投递给每一个消费者，即每条消息只会被一个消费者消费，P2P模式
  - 如果每个消费者都属于不同的消费组，那么所有的消息都会广播给所有的消费者，pub sub模式
  
13. 当你使用kafka-topics.sh创建（删除）了一个topic之后，Kafka背后会执行什么逻辑？
  - kafka-topics.sh实际是调用了 kafka.admin.TopicCommand 类执行topic的管理工作
  - 创建主题需要执行分区分配

14. topic的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？
  - 可以，通过 alter 指令修改
  - ```bin/kafka-topic.sh --zookeeper zk地址 --alter --topic topicName --partitions partitionNumber```

15. topic的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？
  - 不可以，分区已有的数据不好处理，删除分区这个功能收益低

16. 创建topic时如何选择合适的分区数？
  - 需要根据实际的业务场景，软硬件条件等综合考量，最好经过性能测试工具(kafka-producer-perf-test.sh)
  - 并不是分区数越多，吞吐量一定越大，经测试工具测试，随着分区数的增加，达到一定的阀值后，吞吐量反而下降，这个阀值跟实际条件有关
  - 分区数的上线与操作系统的文件描述符有关系(ulimit 可以查看)

17. Kafka目前有那些内部topic，它们都有什么特征？各自的作用又是什么？
  - __consumer_offset：消费者消费位移提交

18. 优先副本是什么？它有什么特殊的作用？
  - 优先副本是AR集合中的第一个副本
  - 创建主题的时候，副本的分配会根据副本分配策略尽量均衡地分配到各个broker，所以一开始的情况是基本负载均衡的，理想情况下，优先副本即leader副本。当有broker出现问题时，kafka会从其他副本选举leader副本，这时副本负载就不再均衡。优先副本的引入就是为了有效地治理负载失衡的情况。
  - 优先副本的选举就是通过一定的方式促使优先副本选举为leader副本

19. Kafka有哪几处地方有分区分配的概念？简述大致的过程及原理
  - producer发送消息时，需要将消息发送到某一个分区
  - consumer消费消息时，分区分配是指定可以消费的分区
  - 创建主题时，也有分区分配
    - 如果指定了replica-assignment，则按照指定的分区规则创建分区副本；如果没指定，则按照内部逻辑，区分是否配置了机架(broker.rack)，使用不同的策略
    - 未指定机架的逻辑大致是，轮询所有分区，将分区的副本分配到不同的broker上

20. 简述Kafka的日志目录结构
  - 一个主题对应一个或多个分区，每个分区对应一个或多个副本，主题和分区都是逻辑概念
  - 不考虑多副本的情况，一个分区对应一个日志(LOG)，而日志文件又会切分为多个LogSegment，相当于一个巨型文件被平均分配为多个相对较小的文件，便于维护和清理
  - Log在物理上以文件夹形式存储，每个LogSegment包括一个日志文件和两个索引文件(偏移量索引文件和时间戳索引文件)，以及其他可能的文件
  - Log的命名为 <topic_name>-<partition_number>
  - 只有最后一个，称之为 activeSegment 的日志分段才能被写入
  - LogSegment的命名为 当前分段的基准偏移量(20位数字)+.log后缀，比如 00000000000000000000.log
  - kafka第一次启动时，还会在日志根目录创建一些检查点文件
 
21. Kafka中有那些索引文件？
  - 偏移量索引(.index)：建立消息偏移量与物理地址之间的映射关系
  - 时间戳索引(.timeindex)：根据指定的时间戳查找偏移量信息
  - 事务索引(.txnindex)
  - 索引文件以稀疏索引的方式构造消息的索引，用二分法查找，每当消息写入一定量(log.index.inteval.bytes,默认4KB)时，就增加两个索引文件的索引项

22. 如果我指定了一个offset，Kafka怎么查找到对应的消息？
  - 有一个offset的索引文件

23. 如果我指定了一个timestamp，Kafka怎么查找到对应的消息？
  - 有一个timestamp的索引文件

24. 聊一聊你对Kafka的Log Retention和Log Compaction的理解
  - 为了控制磁盘占用空间不断增加，需要对消息做一定的清理操作，每个Log是分段的，也便于清理
  - 日志删除：Log Retention，按照一定的策略直接删除不符合条件的日志分段
  - 日志压缩：Log compaction，针对每个消息的Key进行整合，对于有相同key，不同value的消息，只保留最后一个版本
  - 可以通过 log.cleanup.policy 设置清理策略，默认为 delete，也可以设置为 compact，或二者同时设置
  - 日志删除是有定时任务周期性检测和删除不符合条件的日志分段，定时任务的周期由 log.retention.check.interval.ms 控制，默认5分钟
    - 基于时间的删除，默认保持7天的数据，根据日志分段中最大时间戳计算
    - 基于日志大小，根据日志文件的总大小，由 log.retention.bytes 控制，默认-1，表示无穷大
    - 基于起始偏移量，判断依据是，某日志分段的下一个日志分段的起始偏移量 baseOffset 是否等于 logStartOffset，若是，则可以删除
  - Log Compaction 会生成新的日志分段文件

25. 聊一聊你对Kafka底层存储的理解（页缓存、内核层、块层、设备层）
  - 当一个进程读取磁盘文件时，操作系统首先会看待读取的数据所在的页是否在页缓存中，如果是，则直接返回，否则从磁盘读取并存入页缓存，写入也一样。
  - kafka大量使用了页缓存，这是kafka实现高吞吐的重要因素之一，消息先被写入页缓存，然后由操作系统负责同步到磁盘
  - 零拷贝，数据直接从磁盘文件复制到网卡设备。非零拷贝从磁盘到网卡需要经过2次上下文切换，4次数据赋值
  - kafka性能：消息顺序追加，页缓存，零拷贝

26. 聊一聊Kafka的延时操作的原理
  - 时间轮，嵌套的时间轮

27. 聊一聊Kafka控制器的作用
  - 在kafka集群中，会有一个或多个broker，其中一个broker会被选为控制器，它负责管理整个集群所有分区和副本的状态，比如选举leader副本，通知更新元数据，分区重分配等。
  - 控制器的选举依赖于zookeeper，成功竞选的broker会在zookeeper的 ```/controller``` 节点下创建临时节点，某个时刻，有且仅有一个控制器
  - 控制器的作用
    - 监听分区的变化。处理分区重分配；ISR集合变更；优先副本的选举
    - 监听主题的变化。处理主题增减；删除主题
    - 监听broker相关变化。
    - 从zookeeper中获取当前所有的主题，分区，及broker有关的信息，并进行管理，监听主题中分区分配的变化
    - 启动并管理分区状态机和副本状态机
    - 更新集群的元数据信息
    - 如果 auto.leader.rebalance.enable=true，则还会开启一个定时任务维护优先副本的均衡

28. 消费再均衡的原理是什么？（提示：消费者协调器和消费组协调器）
  - 

29. Kafka中的幂等是怎么实现的
  - 幂等开启，enable.idempotence=true
  - 引入producer id和序列号。producerId，由kafka生成，<producerId,分区> 对应的序列号每次消息发送会加1，broker端对比序列号，比broker端维护的序列号大1才会接受
  - kafka保证的是单生产者单分区的消息发送的幂等
  
30. Kafka中的事务是怎么实现的（这题我去面试6加被问4次，照着答案念也要念十几分钟，面试官简直凑不要脸）
  - 幂等不能跨分区，事务可以弥补这个缺陷
  - 客户端指定transactionId，transaction.id=********
  - 生产者需要开启幂等
  - transactionId 与 produerId 一一对应，区别是，transactionId由用户指定，producerID由kafka生成
  - 为了保证相同transactionId对应的旧的生产者立即失效，通过transactionId获取PID时，还会获取一个单调递增的producer epoch
  - 对消费者而言，通过 isolation.level 控制事务隔离级别，对于 read_uncommited ，生产者发送的尚未提交事务的消息对消费者是可见的。对于 read_committed，则不能看到尚未提交的事务，kafkaconsumer内部会缓存，等事务提交了，就会推送给消费端，如果事务abort了，则这部分消息会丢弃。

31. Kafka中有那些地方需要选举？这些地方的选举策略又有哪些？
  - 控制器
  - leader副本

32. 失效副本是指什么？有那些应对措施？
  - ISR集合之外的副本就是失效副本，也就是与ISR不能保持一定程度同步的副本。

33. 多副本下，各个副本中的HW和LEO的演变过程

34. 为什么Kafka不支持读写分离？
  - 代码复杂度更高，复杂度越高，出现问题的可能性越大
  - 数据一致性，主从同步需要时间，有一定的滞后性
  - 延时，kafka的消息是保存在磁盘，主从复制时间开销大
  - 一般读写分离是分摊节点压力，而kafka的分区可以实现负载

35. Kafka在可靠性方面做了哪些改进？（HW, LeaderEpoch）
  - kafka只能消费到HW之前的消息，从producer角度看，对于acks=-1的，必须是ISR集合都确认收到，也就是此时消息在HW范围内，消息就不会丢失(清理策略清理掉的除外)，
  - leader epoch 代表leader的纪元信息，初始值为0，每次leader变更一次，epoch的值就+1。follower重启收到leader的response后，比对epoch，发现一致，则不需要截断

36. Kafka中怎么实现死信队列和重试队列？
  - 发送到其他队列

Kafka中的延迟队列怎么实现（这题被问的比事务那题还要多！！！听说你会Kafka，那你说说延迟队列怎么实现？）
  - 消费者拦截器，根据时间，只消费达到时间的。但是这种方式，会有
  - 发送到内部 delay_topic_? ，内部通过DelayQueue缓存数据，另起单独的线程拉取消息，专门的消息发送线程发送到真实主题（生产者将消息发送到一个地方，然后通过另一个服务拉取并转发到真是主题）
  - 缓存在broker段，结合时间轮

Kafka中怎么做消息审计？

Kafka中怎么做消息轨迹？

Kafka中有那些配置参数比较有意思？聊一聊你的看法

Kafka中有那些命名比较有意思？聊一聊你的看法

Kafka有哪些指标需要着重关注？

怎么计算Lag？(注意read_uncommitted和read_committed状态下的不同)

Kafka的那些设计让它有如此高的性能？

Kafka有什么优缺点？

还用过什么同质类的其它产品，与Kafka相比有什么优缺点？

为什么选择Kafka?

在使用Kafka的过程中遇到过什么困难？怎么解决的？

怎么样才能确保Kafka极大程度上的可靠性？

聊一聊你对Kafka生态的理解

