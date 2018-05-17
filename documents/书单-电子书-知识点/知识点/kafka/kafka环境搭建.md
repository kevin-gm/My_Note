#### 下载

	[root@VM_0_14_centos kafka]# wget http://mirror.bit.edu.cn/apache/kafka/1.1.0/kafka_2.11-1.1.0.tgz

#### 解压

	[root@VM_0_14_centos kafka]# tar -zxvf kafka_2.11-1.1.0.tgz

#### 切换目录

	[root@VM_0_14_centos kafka]# cd kafka_2.11-1.1.0
	[root@VM_0_14_centos kafka_2.11-1.1.0]# 

当前目录下，config 文件夹里面是配置文件，初始需要用到的配置文件涉及:

- zookeeper.properties：zookeeper配置文件，新版的kafka内置了一个zookeeper，无需单独搭建zookeeper服务
- server.properties：kafka服务端配置文件
- producer.properties：生产者配置文件，使用命令创建producer时，指定使用
- consumer.properties：消费者配置文件，使用命令创建consumer时，指定使用

#### 启动zookeeper

	[root@VM_0_14_centos kafka_2.11-1.1.0]# bin/zookeeper-server-start.sh config/zookeeper.properties &

命令的含义是使用 config/zookeeper.properties 配置文件启动zookeeper。

最后的"&"是为了能退出命令行，也就是这种模式启动后，按ctrl+c退出，zookeeper服务仍然是正常启动状态。

#### 启动kafka

	[root@VM_0_14_centos kafka_2.11-1.1.0]# bin/kafka-server-start.sh config/server.properties &

此时服务启动，可以在服务端测试基本使用。

#### 创建topic

	[root@VM_0_14_centos kafka_2.11-1.1.0]# bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

- 使用 kafka-topics.sh 脚本创建topic
- --zookeeper指定对应的zookeeper服务
- --replication-factor：指定副本数量，数量为1，代表只有本身，没有副本。
- --partitions：指定分区数量
- --topic：指定topic的名字

#### 创建producer生产者

	[root@VM_0_14_centos kafka_2.11-1.1.0]# bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
	> This a test message
	> 

命令为，向指定端口9092的kafka服务，对应名字为 test 的topic 发送消息。

该命令回车后，后出现一个向右的箭头，此时可以输入消息内容，回车后，消息就会发送出去。

上述命令表明，发送了一条内容为 “This a test message” 到 topic上

#### 创建consumer消费者

	[root@VM_0_14_centos kafka_2.11-1.1.0]# bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
	Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper]
	This a test message

消费者需要通过zookeeper调度才知道对应的生产者，上述命令回车后，如果之前向对应的topic已经发送了消息，则会显示改消息。

此时可以，producer和consumer分两个终端打开，一个发送消息，另一个就会立即看到消息。


## Java操作。

### 直接使用

#### 引入kafka依赖

	<dependency>
		<groupId>org.apache.kafka</groupId>
		<artifactId>kafka_2.12</artifactId>
		<version>1.1.0</version>
	</dependency>
	<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.1.41</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.1.11</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.1.11</version>
        </dependency>

引入slf4j和logback，设置日志级别为debug。引入日志组件的目的是，报错的时候可以查看异常栈。

#### 创建producer发送消息

	import com.alibaba.fastjson.JSON;
	import org.apache.kafka.clients.producer.*;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import site.coloured.kafka.common.config.KafkaProperties;
	
	import java.util.Properties;
	
	public class KafKaProducerTask {
	
	    private final static Logger LOGGER = LoggerFactory.getLogger(KafKaProducerTask.class);
	
	    private static Properties props;
	
	    static {
	        props = new Properties();
	        props.put("bootstrap.servers", KafkaProperties.KAFKA_SERVER_URL);
	        props.put("producer.type", "sync");
	        props.put("request.required.acks", "1");
	        props.put("serializer.class", "kafka.serializer.DefaultEncoder");
	        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	        props.put("bak.partitioner.class", "kafka.producer.DefaultPartitioner");
	        props.put("bak.key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	        props.put("bak.value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	
	    }
	
	    public void sendMsg(String topic, String key, String value) {
	        LOGGER.info("properties:" + JSON.toJSONString(props));
	        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);
	        LOGGER.info("kafkaproducer:" + JSON.toJSONString(producer));
	        ProducerRecord<String, String> record = new ProducerRecord<String, String>(topic, key, value);
	        producer.send(record, (RecordMetadata metadata, Exception exception) -> {
	            if(exception != null) {
	                LOGGER.info("记录的offset在：" + metadata.offset());
	                LOGGER.error(exception.getMessage() + exception);
	            } else {
	                LOGGER.info("send success");
	            }
	        });
	        producer.close();
	    }
	}

#### 创建consumer消费消息

	import kafka.consumer.ConsumerConfig;
	import kafka.consumer.ConsumerIterator;
	import kafka.consumer.KafkaStream;
	import kafka.javaapi.consumer.ConsumerConnector;
	import kafka.message.MessageAndMetadata;
	import kafka.serializer.StringDecoder;
	import kafka.utils.VerifiableProperties;
	import org.apache.kafka.clients.consumer.ConsumerRecord;
	import org.apache.kafka.clients.consumer.ConsumerRecords;
	import org.apache.kafka.clients.consumer.KafkaConsumer;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import site.coloured.kafka.common.config.KafkaProperties;
	
	import java.util.*;
	
	public class KafkaConsumerTask {
	
	    private final static Logger LOGGER = LoggerFactory.getLogger(KafkaConsumerTask.class);
	
	    private static Properties props;
	
	    /**
	     * 初始化配置文件
	     */
	    static {
	        props = new Properties();
	        // 必须
	        props.put("zookeeper.connect", KafkaProperties.ZOOKEEPER_URL);
	        // 必须
	        props.put("bootstrap.servers", KafkaProperties.KAFKA_SERVER_URL);
	        props.put("group.id", KafkaProperties.GROUPID);
	        props.put("zookeeper.session.timeout.ms", "4000");
	        props.put("zookeeper.sync.time.ms", "200");
	        props.put("auto.commit.interval.ms", "1000");
	        //props.put("auto.offset.reset", "smallest");
	        props.put("serializer.class", "kafka.serializer.StringEncoder");
	        // 必须
	        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
	        // 必须
	        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
	    }
	
	    public void getMsg() {
	        useNewApi();
	        useOldApi();
	    }
	
	    public void useNewApi() {
	        // 创建consumer对象
	        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
	        // 指定订阅主体
	        consumer.subscribe(Arrays.asList(KafkaProperties.TOPIC));
	        // 循环消费消息，初次启动，会消费topic上的所有消息，客户端可以保留offset，避免此问题
	        while (true) {
	            ConsumerRecords<String, String> records = consumer.poll(100);
	            for (ConsumerRecord<String, String> record : records)
	                LOGGER.info(String.format("offset = %d, key = %s, value = %s", record.offset(), record.key(), record.value()));
	        }
	    }
	
	    /**
	     * 老版本的api，包括一些已经过期的方法调用。
	     */
	    public void useOldApi() {
	        ConsumerConfig config = new ConsumerConfig(props);
	        Map<String, Integer> topicCountMap = new HashMap<String, Integer>();
	        topicCountMap.put(KafkaProperties.TOPIC, new Integer(1));
	        StringDecoder keyDecoder = new StringDecoder(new VerifiableProperties());
	        StringDecoder valueDecoder = new StringDecoder(new VerifiableProperties());
	        ConsumerConnector consumer = kafka.consumer.Consumer.createJavaConsumerConnector(config);
	        Map<String, List<KafkaStream<String, String>>> consumerMap = consumer.createMessageStreams(topicCountMap, keyDecoder, valueDecoder);
	        KafkaStream<String, String> stream = consumerMap.get(KafkaProperties.TOPIC).get(0);
	        ConsumerIterator<String, String> it = stream.iterator();
	        while (it.hasNext()) {
	            // kafka获取到的数据
	            MessageAndMetadata<String, String> keyVlaue = it.next();
	            LOGGER.info(" kafka get message , key : " + keyVlaue.key() + " ; value : " + keyVlaue.message());
	        }
	    }
	}

#### 启动调用

	public class Client {

	    public static void main(String[] args) {
	        new KafKaProducerTask().sendMsg(KafkaProperties.TOPIC, "key", "lsn-20171024");

			System.out.println("---------------");

        	new KafkaConsumerTask().getMsg();
	    }
	}

### 使用spring-kafka

#### kafka配置
主要是对直接调用时的配置，java config化。

	import org.apache.kafka.clients.admin.AdminClientConfig;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.kafka.annotation.EnableKafka;
	import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
	import org.springframework.kafka.core.*;
	import site.coloured.kafka.consumer.task.SimpleConsumerListener;
	
	import java.util.HashMap;
	import java.util.Map;
	
	@Configuration
	@EnableKafka
	public class KafkaConfig {
	
	    /**
	     * kafka 服务器配置
	     * @return
	     */
	    @Bean
	    public KafkaAdmin admin() {
	        Map<String, Object> configs = new HashMap<>();
	        configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, KafkaProperties.KAFKA_SERVER_URL);
	        return new KafkaAdmin(configs);
	    }
	    
	    @Bean
	    public KafkaTemplate<String, String> kafkaTemplate() {
	        return new KafkaTemplate<String, String>(producerFactory());
	    }
	
	    /**
	     * producer 配置
	     * @return
	     */
	    @Bean
	    public Map<String, Object> producerConfig() {
	        Map<String, Object> props = new HashMap<>();
	        props.put("bootstrap.servers", KafkaProperties.KAFKA_SERVER_URL);
	        props.put("producer.type", "sync");
	        props.put("request.required.acks", "1");
	        props.put("serializer.class", "kafka.serializer.DefaultEncoder");
	        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	        props.put("bak.partitioner.class", "kafka.producer.DefaultPartitioner");
	        props.put("bak.key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	        props.put("bak.value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
	        return props;
	    }
	
	    /**
	     * 使用默认的producer工厂
	     * @return
	     */
	    @Bean
	    public ProducerFactory<String, String> producerFactory() {
	        return new DefaultKafkaProducerFactory<String, String>(producerConfig());
	    }
	
	    /**
	     * consumer配置
	     * @return
	     */
	    @Bean
	    public Map<String, Object> consumerConfig() {
	        Map<String, Object> props = new HashMap<>();
	        props.put("zookeeper.connect", KafkaProperties.ZOOKEEPER_URL);
	        props.put("bootstrap.servers", KafkaProperties.KAFKA_SERVER_URL);
	        props.put("group.id", KafkaProperties.GROUPID);
	        props.put("zookeeper.session.timeout.ms", "4000");
	        props.put("zookeeper.sync.time.ms", "200");
	        props.put("auto.commit.interval.ms", "1000");
	        props.put("serializer.class", "kafka.serializer.StringEncoder");
	        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
	        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
	        return props;
	    }
	
	    /**
	     * 使用默认的consumer工厂
	     * @return
	     */
	    @Bean
	    public ConsumerFactory<String, String> consumerFactory() {
	        return new DefaultKafkaConsumerFactory<String, String>(consumerConfig());
	    }
	
	    /**
	     * kafka监听器工厂配置
	     * @return
	     */
	    @Bean
	    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
	        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
	        factory.setConsumerFactory(consumerFactory());
	        return factory;
	    }
	
	    /**
	     * 将自定义的监听类配置成bean
	     * @return
	     */
	    @Bean
	    public SimpleConsumerListener simpleConsumerListener() {
	        return new SimpleConsumerListener();
	    }
	}

自定义监听器

	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.kafka.annotation.KafkaListener;
	
	public class SimpleConsumerListener {
	
	    private final static Logger LOGGER = LoggerFactory.getLogger(SimpleConsumerListener.class);
	
	    @KafkaListener(topics = "java-test")
	    public void getMsg(String msg) {
	        LOGGER.info("get msg : " + msg);
	    }
	}

启动

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;
	import org.springframework.kafka.core.KafkaTemplate;
	import org.springframework.kafka.support.SendResult;
	import org.springframework.lang.Nullable;
	import org.springframework.util.concurrent.ListenableFuture;
	import org.springframework.util.concurrent.ListenableFutureCallback;
	import site.coloured.kafka.common.config.KafkaConfig;
	import site.coloured.kafka.common.config.KafkaProperties;
	
	import java.util.concurrent.ExecutionException;
	
	public class Client {
	
	    public static void main(String[] args) throws ExecutionException, InterruptedException {
	        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(KafkaConfig.class);
	        KafkaTemplate<String, String> template = (KafkaTemplate<String, String>)context.getBean("kafkaTemplate");
	        ListenableFuture<SendResult<String, String>> future = template.send(KafkaProperties.TOPIC, "spring-key",
	                "yesyesyes");
	        future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
	            @Override
	            public void onFailure(Throwable throwable) {
	                System.out.println(throwable);
	            }
	
	            @Override
	            public void onSuccess(@Nullable SendResult<String, String> stringStringSendResult) {
	                System.out.println("success" + stringStringSendResult.getProducerRecord().value());
	            }
	        });
	        System.out.println("----------------------");
	    }
	}

## 遇到的问题。
1. kafka服务部署在腾讯云远程服务器，默认配置下，使用java调用发送消息失败。异常栈为：

		14:23:54.946 [kafka-producer-network-thread | producer-1] DEBUG org.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Error connecting to node VM_0_14_centos:9092 (id: 0 rack: null)
		java.io.IOException: Can't resolve address: VM_0_14_centos:9092
		at org.apache.kafka.common.network.Selector.doConnect(Selector.java:235)
		at org.apache.kafka.common.network.Selector.connect(Selector.java:214)
		at org.apache.kafka.clients.NetworkClient.initiateConnect(NetworkClient.java:793)
		at org.apache.kafka.clients.NetworkClient.access$700(NetworkClient.java:62)
		at org.apache.kafka.clients.NetworkClient$DefaultMetadataUpdater.maybeUpdate(NetworkClient.java:944)
		at org.apache.kafka.clients.NetworkClient$DefaultMetadataUpdater.maybeUpdate(NetworkClient.java:848)
		at org.apache.kafka.clients.NetworkClient.poll(NetworkClient.java:458)
		at org.apache.kafka.clients.producer.internals.Sender.run(Sender.java:239)
		at org.apache.kafka.clients.producer.internals.Sender.run(Sender.java:163)
		at java.lang.Thread.run(Thread.java:748)
		Caused by: java.nio.channels.UnresolvedAddressException: null
		at sun.nio.ch.Net.checkAddress(Net.java:101)
		at sun.nio.ch.SocketChannelImpl.connect(SocketChannelImpl.java:622)
		at org.apache.kafka.common.network.Selector.doConnect(Selector.java:233)
		... 9 common frames omitted

可以看到是无法连接节点【VM_0_14_centos:9092】，根据前面所说listener的配置内容可知，在没有指定hostname的情况想，默认使用的是服务器本身的hostname。

而kafka的连接原理是：

- 首先连接 ip:9092,也就是KafkaProperties.KAFKA_SERVER_URL配置的内容，这个是指定kafka服务的地址。端口9092是默认值，可以更改。
- 再连接返回的listener，而此时配置是默认的，获取的是服务器本身的hostname，VM_0_14_centos这个就是，前面的命令也可以看出这个是个人服务器的默认hostname，可以通过修改linux文件进行更改。
- 最后继续连接advertised.listener，默认没有该配置，所以实际使用的就是listener的配置。

很明显，java客户端，调用【VM_0_14_centos:9092】肯定是不能识别的。

此时，方法一，修改hosts文件，测试是，java客户端是本机Windows系统运行的

打开： C:\Windows\System32\drivers\etc 目录下的 hosts 文件，新增内容：

	ip VM_0_14_centos

ip是kafka服务器地址，也就是让【VM_0_14_centos:9092】解析到kafka服务器。

再次执行main方法，可以看到日志显示，消息发送成功，在服务器上使用命令创建一个consumer，则可以收到发送的消息。

此时再想，虽然可以修改hosts解决问题，但这种方式并不合理，因为需要额外的在本地配置hosts，仔细阅读kafka官方文档，看到一个配置，```advertised.listeners```，将其配置为云主机的外网IP，重新执行代码，成功发送消息到kafka服务器。




## 配置文件简述
对部分配置内容进行简要说明

### zookeeper.properties

	# Licensed to the Apache Software Foundation (ASF) under one or more
	# contributor license agreements.  See the NOTICE file distributed with
	# this work for additional information regarding copyright ownership.
	# The ASF licenses this file to You under the Apache License, Version 2.0
	# (the "License"); you may not use this file except in compliance with
	# the License.  You may obtain a copy of the License at
	# 
	#    http://www.apache.org/licenses/LICENSE-2.0
	# 
	# Unless required by applicable law or agreed to in writing, software
	# distributed under the License is distributed on an "AS IS" BASIS,
	# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	# See the License for the specific language governing permissions and
	# limitations under the License.
	# the directory where the snapshot is stored.
	dataDir=/data/logs/zookeeper
	# the port at which the clients will connect
	clientPort=2181
	# disable the per-ip limit on the number of connections since this is a non-production config
	maxClientCnxns=0

### server.properties
broker.id=0
> 当前kafka服务在集群中的唯一ID，数字型，可以默认，如果是集群配置，则需要修改这个值。

listeners=PLAINTEXT://:9092
> - 申明此kafka服务器需要监听的端口号，如果是在本机或者虚拟机上运行可以使用默认，如果是远程服务器，则必须配置。
> - "PLAINTEXT"表示协议，可选的值有PLAINTEXT和SSL，hostname可以指定IP地址，也可以用"0.0.0.0"表示对所有的网络接口有效，如果hostname为空表示只对默认的网络接口有效
> - 如果没有配置advertised.listeners，就使用listeners的配置通告给消息的生产者和消费者，这个过程是在生产者和消费者获取源数据(metadata)。如果都没配置，那么就使用java.net.InetAddress.getCanonicalHostName()返回的值，对于ipv4,基本就是localhost了
