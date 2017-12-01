	## 刷新时，关闭安全验证
	management.security.enabled=false
	## 开启消息跟踪
	spring.cloud.bus.trace.enabled=true

### spring cloud bus集成在微服务实例
多个实例通过消息总线相连，每个实例都会订阅配置更新事件，当其中一个微服务节点/bus/refresh端点被请求时，该实例就会向消息总线发起一个配置更新的请求，其他实例获得该事件后也会更新配置。

#### 大致步骤：
1. 添加依赖
2. 配置RabbitMQ
3. 添加@RefreshScope注解
4. 启动Eureka server
5. 启动Config server
6. 启动Config client
7. 修改端口启动另一个Config client实例（如果启动失败，可以参考“IDEA同一项目启动多个实例”）
8. 访问接口，查看配置
9. 修改配置，并提交到仓库
10. 查看配置，此时返回结果应该不变。
11. POST调用某一个实例的/bus/refresh端口，
12. 再次查看配置，配置已经更新

##### 添加依赖
使用AMQP标准的消息同步组件来订阅配置更新，默认为RabbitMQ。

如果想使用kafka，则可以将spring-cloud-starter-bus-amqp依赖换成spring-cloud-starter-bus-kafka即可。

	<!-- 引入消息服务依赖 -->
	<dependency>
    	<groupId>org.springframework.cloud</groupId>
    	<artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>
	<!-- 监控需要的依赖，可以开始/refresh端点 -->
	<dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

添加spring-cloud-starter-bus-amqp依赖后，将会开启新的端点“/bus/refresh”。

这个端点的作用：

1. 获取config server上最新的配置信息，并且更新注解了@RefreshScope的配置
2. 发送消息到AMQP服务上，通知刷新事件
3. 所有订阅了相同节点的服务将刷新自己的配置

##### 在application.yml中添加RabbitMQ的配置

	spring:
	  rabbitmq:
	    host: localhost
	    port: 5672 # 默认端口
	    username: guest # 默认用户名
	    password: guest # 默认密码
	management:
	  security:
	    enabled: false  # 关闭安全验证，否则调用/refresh等敏感端点时会没有权限

##### 引用配置的类，即需要使用会发生变更属性的类上，配置@RefreshScope注解
如果不加这个注解，配置刷新不生效

	@RestController
	@RefreshScope
	public class OrderController {
	
	    @Value("${test.bus.value}")
	    private String busValue;
	
	    @GetMapping("/testBR")
	    public String testBusRefresh() {
	        System.out.println("test bus refresh");
	        return busValue;
	    }
	}

##### 根据步骤提示启动，修改配置查看结果。任一个实例调用刷新端点后会同步到其他实例。

##### 局部刷新
某些情况下，可能我们只想刷新部分微服务配置，此时可以通过“/bus/refresh”端点的destination参数来定位要刷新的应用程序



### spring cloud bus集成到config server
前面集成到某个微服务上的/bus/refresh端点的方式来实现配置刷新，这样违反了SRP原则（单一职责原则），微服务是业务模块，却承担了配置刷新的职责

我们可以将消息总线（bus）添加到Config server中，并使用Config server的/bus/refresh端点实现配置刷新

实现方式跟集成到业务服务一致，不再赘述。

注意：
> config server和config client都需要配置消息总线依赖，要进行配置同步。