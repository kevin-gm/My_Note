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
  - 
