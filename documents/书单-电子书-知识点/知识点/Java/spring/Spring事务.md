## 基本概念
### 什么是事务

事务，是对数据读或写的一个操作序列。是在一个逻辑单元中执行一系列操作，这些操作要么一起成功，要么一起失败。主要是为了保证数据的一致性。

### 为什么需要事务
一个简单的例子，银行转账：

- 假设数据发生在同一个数据库，账户A余额100，账户B余额50。
- 现在A给B转账20元，预期的结果是，A账户余额80元，B账户余额70元。
- 这个逻辑单只考虑金额的变化有两步操作，A账户余额减去20元，B账户余额增加20元。
- 没有事务保证时，如果A账户减少20元后，由于各种原因，B账户余额没有增加，那么结果是A账户的钱凭空少了20元
- 如果是这种结果，那么系统完全玩不下去，所以必须要有事务的保证。

### 事务的特性：
将若干的数据库操作作为一个整体控制,一起成功或一起失败。

  1. 原子性：指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。一旦某一个过程发生问题，就需要执行回滚操作。前面转账的例子中，如果B账户余额增加失败，则回滚后，A账户的钱不会减少，数据会回到事务执行之前的状态。
  2. 一致性：指事务前后数据的完整性必须保持一致。前面的例子中，转账前后，A账户和B账户的余额总额必须保持不变。
  3. 隔离性：指多个用户并发访问数据库时，一个用户的事务不能被其他用户的事务所干扰，多个并发事务之间数据要相互隔离。示例中，如果A给B转账的过程中，C也在给B转账，那么当两个事务都结束后，结果是B账户的余额时原有的余额+A转账金额+C转账金额
  4. 持久性：指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，即时数据库发生故障也不应该对其有任何影响。

### 保存点(savepoint)
在关系型数据库中，有一个保存点的概念(savepoint)，也叫回滚点。

维基百科对其的定义为：
>
A savepoint is a way of implementing subtransactions (also known as nested transactions) within a relational database management system by indicating a point within a transaction that can be "rolled back to" without affecting any work done in the transaction before the savepoint was created. 
>
Multiple savepoints can exist within a single transaction. 
>
Savepoints are useful for implementing complex error recovery in database applications. If an error occurs in the midst of a multiple-statement transaction, the application may be able to recover from the error (by rolling back to a savepoint) without needing to abort the entire transaction.

大致的意思是，保存点是一种在关系型数据库中实现子事务的方式，可以在事务开始前指定一个点，如果事务发生异常，可以回滚到指定的savepoint而不影响其他工作，一个事务中可以有多个保存点。

### Spring事务传播属性
事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。

个人的理解是，Spring事务的传播特性就是对数据库中savepoint的抽象。

传播属性定义在类：TransactionDefinition

1. PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这是默认值。
2. PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
3. PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
4. PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
5. PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
6. PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
7. PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

### 事务的隔离性
隔离级别是指若干个并发的事务之间的隔离程度。

有并发经验的都知道，对数据的访问方面，共享数据是最容易出现问题的，而数据库就是一个大的共享数据中心。一般会存在以下问题：

<strong>脏读</strong>：
脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。

<strong>不可重复读</strong>：
是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。（即不能读到相同的数据内容）

不可重复读和脏读的区别是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据

<strong>幻读</strong>:
是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。

幻读和不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。

#### Spring事务的隔离级别
Spring事务的隔离级别与关系型数据库中的隔离级别是对应的。

TransactionDefinition 接口中定义了五个表示隔离级别的常量

1. ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
2. ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读，不可重复读和幻读，因此很少使用该隔离级别。比如PostgreSQL实际上并没有此级别。
3. ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
4. ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。该级别可以防止脏读和不可重复读。
5. ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。
  
## Spring事务
Spring事务的本质是对数据库事务的支持，它是对数据库事务的封装，没有数据库事务的支持，spring是无法提供事务保证的。

### 编程式事务管理
使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。

对于编程式事务管理，spring推荐使用TransactionTemplate，其实际是对事务管理的过程的封装，编码时不必关注事务的开启提交回滚。

<strong>1. 使用PlatformTransactionManager的实现，如DataSourceTransactionManager</strong>

	JdbcTemplate template = new JdbcTemplate(datasource);
	//定义事务管理器，因为使用的是JdbcTemplate，对应的是DataSourceTransactionManager
	PlatformTransactionManager txManager = new DataSourceTransactionManager(datasource);
	//事务定义类，可以设置传播属性、隔离级别
	DefaultTransactionDefinition def = new DefaultTransactionDefinition();
	def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
	//事务状态，包含回滚点等内容
	TransactionStatus status = txManager.getTransaction(def);
	try {
		template.update("Insert into userinfo(username,password) values('aaaaa','bbbbb')");
		template.update("Insert into userinfo(username,password) values('cccc','ddd')");
		// 正常提交事务
		txManager.commit(status);
	} catch (Exception ex) {
		// 异常回滚事务
		txManager.rollback(status);
	}

<strong>2. 使用TransactionTemplate</strong>
	
	JdbcTemplate template = new JdbcTemplate(datasource);
	DataSourceTransactionManager tran = new DataSourceTransactionManager(datasource);
	TransactionTemplate trantemplate = new TransactionTemplate(tran);
	trantemplate.execute(status -> {
		template.update("Insert into userinfo(username,password) values('jjj','kkk')");
		template.update("Insert into userinfo(username,password) values('llll','mmm')");
		return null;
	});

### 声明式事务管理

<strong>1. 基于切面（xml配置）</strong>

	<!-- 配置事务管理器 -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>

	<!--  配置事务传播特性 -->
	<tx:advice id="transAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="trans*" propagation="REQUIRED"/>
		</tx:attributes>
	</tx:advice>

	<!--  配置事务切面和建言 -->
	<aop:config>
		<aop:pointcut id="transServiceConfig" expression="execution(* site.coloured.trans.service.*.*(..))"/>
		<aop:advisor advice-ref="transAdvice" pointcut-ref="transServiceConfig"/>
	</aop:config>

<strong>2. 基于注解</strong>

	<!-- 配置事务管理器 -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>
	
	<tx:annotation-driven transaction-manager="transactionManager"/>

	<!--之后可以在需要事务的类以及方法上使用@transactional注解，进行事务管理-->
	@Transactional
	public void transfer(String from, String to, Long money) {
		nonTransaction(from, to, money);
	}

<strong>Notice：</strong>

- 建议只在具体的类或者具体类的方法上进行```@Transactional```注解，而不建议在接口或者接口的方法上进行注解
	> 虽然是可以在接口上进行注解，但是如果注解在接口或者接口方法上，它仅仅只会在基于接口的代理上起作用。如果使用基于类的代理（proxy-target-class="true"）或者织入的代理（mode="aspectj"），那么事务的设置将不会被代理或者织入接口识别，对象将不被事务代理包裹
- 在默认的代理模式中，只有外部方法的调用才会被代理截获。意味着，自我调用，即目标类的一个方法调用目标类的另一个方法，是不会导致运行时事务的，即使方法被标注了@Transactional注解。（**嵌套事务需要单独成类**）
- 代理必须完全初始化后才能提供预期行为，所以不能在初始化方法中依赖事务特性。比如   ```@PostConstruct```中是无法提供基于代理的事务的。
- 如果希望自我调用也被事务包裹，可以考虑使用AspectJ模式。在这种情况下，首先不会有代理，换而是，目标类将会被织入，为了将任何方法上的```@Transactional```转换为运行时行为（目标类的字节码将会被修改）

### 事务的只读属性（readonly=true）
“只读”并不是一个强制选项，它只是一个“暗示”，提示数据库驱动程序和数据库系统，这个事务并不包含更改数据的操作，那么JDBC驱动程序和数据库就有可能根据这种情况对该事务进行一些特定的优化，比方说不安排相应的数据库锁，以减轻事务对数据库的压力，毕竟事务也是要消耗数据库的资源的。 

但是你非要在“只读事务”里面修改数据，也并非不可以，只不过对于数据一致性的保护不像“读写事务”那样保险而已。 

因此，“只读事务”仅仅是一个性能优化的推荐配置而已，并非强制你要这样做不可

### 事务的回滚
spring事务管理器回滚一个事务的推荐方法是在当前事务的上下文内抛出异常。spring事务管理器会捕捉任何未处理的异常，然后依据规则决定是否回滚抛出异常的事务。

默认配置下，spring只有在抛出的异常为运行时unchecked异常时才回滚该事务，也就是抛出的异常为RuntimeException的子类(Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。可以明确的配置在抛出那些异常时回滚事务，包括checked异常。也可以明确定义那些异常抛出时不回滚事务。还可以编程性的通过setRollbackOnly()方法来指示一个事务必须回滚，在调用完setRollbackOnly()后你所能执行的唯一操作就是回滚。

## Spring事务原理
### PlatformTransactionManager
PlatformTransactionManager是spring事务管理的顶层接口。

一般而言，通用的事务处理是由实现了PlatformTransactionManager接口的抽象类AbstractPlatformTransactionManager来提供的。如DataSourceTransactionManager 、JtaTransactionManager和 HibernateTransactionManager等

	package org.springframework.transaction;

	public interface PlatformTransactionManager {
	
		//根据指定的传播行为，返回当前激活的事务或者新建一个
		TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
		//提交事务
		void commit(TransactionStatus status) throws TransactionException;
		//回滚事务
		void rollback(TransactionStatus status) throws TransactionException;
	}

### TransactionDefinition
事务属性配置类，包括传播行为、隔离级别的常量定义，返回超时时间，是否只读，以及事务名称

	package org.springframework.transaction;
	import java.sql.Connection;
	public interface TransactionDefinition {
	
		// 事务传播属性
		int PROPAGATION_REQUIRED = 0;
		int PROPAGATION_SUPPORTS = 1;
		int PROPAGATION_MANDATORY = 2;
		int PROPAGATION_REQUIRES_NEW = 3;
		int PROPAGATION_NOT_SUPPORTED = 4;
		int PROPAGATION_NEVER = 5;
		int PROPAGATION_NESTED = 6;
		// 事务隔离级别
		int ISOLATION_DEFAULT = -1;
		int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;
		int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;
		int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;
		int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;
		// 超时时间
		int TIMEOUT_DEFAULT = -1;
		int getPropagationBehavior();
		int getIsolationLevel();
		int getTimeout();
		// 是否只读
		boolean isReadOnly();
		// 事务名称
		String getName();
	}

### TransactionStatus
代表当前事务的状态

	package org.springframework.transaction;
	import java.io.Flushable;
	public interface TransactionStatus extends SavepointManager, Flushable {
	
		//返回当前事务是否新事务
		boolean isNewTransaction();
		//返回事务是否包含基于嵌套事务创建的回滚点
		boolean hasSavepoint();
		//设置事务的结果为仅仅回滚
		void setRollbackOnly();
		//返回事务是否被标记为回滚
		boolean isRollbackOnly();
		//将当前回话的数据同步到数据库
		@Override
		void flush();
		//返回当前事务是否已完成，即已提交或者已回滚
		boolean isCompleted();
	}

### spring事务逻辑，以基于注解的为例
基本原理就是通过spring aop的实现，在实际方法调用前后基于PlatformTransactionManager进行事务处理。

1. xml配合文件解析，根据不同的命名空间通过不同的解析器解析配置。```DefaultBeanDefinitionDocumentReader.parseBeanDefinitions```

		// 解析xml文件的方法
		protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
			// 省略其余代码,除默认命名空间的配置外，其余会调用parseCustomElement方法
			delegate.parseCustomElement(ele);
		}
	
		public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
			// 根据节点名称，获取对应的命名空间地址
			String namespaceUri = getNamespaceURI(ele);
			// ...
			// 根据命名空间地址，查找对应的处理器，与spring.handlers文件有关
			NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
			// 省略...
			// 通过处理器解析对应的配置
			return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
		}

2. spring加载拥有的命名空间处理器
	> NamespaceHandler：spring的命名空间处理器，在进行基于xml的spring配置时，必须要引入对应标签的命名空间，比如aop，tx等。

	>spring.handlers：一个文件，该文件定义的是各种处理器，每个标签都与不同的处理器，tx标签的处理器为TxNamespaceHandler。

	>

		// TxNamespaceHandler初始化方法，注册需要的解析器
		// spring在解析xml配置文件时，遇到对应的标签，就会使用对应解析器的parse方法进行解析。
		public void init() {
			registerBeanDefinitionParser("advice", new TxAdviceBeanDefinitionParser());
			registerBeanDefinitionParser("annotation-driven", new AnnotationDrivenBeanDefinitionParser());
			registerBeanDefinitionParser("jta-transaction-manager", new JtaTransactionManagerBeanDefinitionParser());
		}

3. 注册tx命名空间所需要的解析器
	>AnnotationDrivenBeanDefinitionParser：这个是基于注解驱动的事务管理配置解析器

4. ```AnnotationDrivenBeanDefinitionParser``` 解析配置
	> 默认使用Proxy模式，除非显示指定为aspectj

		public BeanDefinition parse(Element element, ParserContext parserContext) {
			registerTransactionalEventListenerFactory(parserContext);
			String mode = element.getAttribute("mode");
			if ("aspectj".equals(mode)) {
				// mode="aspectj"
				registerTransactionAspect(element, parserContext);
			}
			else {
				// mode="proxy"
				AopAutoProxyConfigurer.configureAutoProxyCreator(element, parserContext);
			}
			return null;
		}

5. Proxy模式下，根据配置自动创建代理

		public static void configureAutoProxyCreator(Element element, ParserContext parserContext) {
			// 关键代码，见名知意，如果必要，创建并注册需要的代理类
			AopNamespaceUtils.registerAutoProxyCreatorIfNecessary(parserContext, element);
			// 省略...
		}

		public static void registerAutoProxyCreatorIfNecessary(
				ParserContext parserContext, Element sourceElement) {
	
			// 生成InfrastructureAdvisorAutoProxyCreator的Bean定义
			BeanDefinition beanDefinition = AopConfigUtils.registerAutoProxyCreatorIfNecessary(
					parserContext.getRegistry(), parserContext.extractSource(sourceElement));
			// 省略...
		}

6. 注册 ```InfrastructureAdvisorAutoProxyCreator```。
	> 查看这个类的继承以及实现的接口，可以知道，这个类最终实现了 ```BeanPostProcessor```接口。熟悉spring加载的都知道，这个是Bean的后置处理器，也就是在每个bean都初始化前后，做的一些额外的操作。

		public static BeanDefinition registerAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry,
			@Nullable Object source) {

			return registerOrEscalateApcAsRequired(InfrastructureAdvisorAutoProxyCreator.class, registry, source);
		}

7. 重点看下继承结构中 ```AbstractAutoProxyCreator``` 的 ```postProcessAfterInitialization``` 方法
	> postProcessAfterInitialization 方法，是在Bean初始化完成后才执行的
		
		public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) throws BeansException {
			if (bean != null) {
				// 根据规则，生成缓存key
				Object cacheKey = getCacheKey(bean.getClass(), beanName);
				// 判断当前的key是否已经生成了对应的代理类，如果没有，则进行生成代理的逻辑。
				if (!this.earlyProxyReferences.contains(cacheKey)) {
					return wrapIfNecessary(bean, beanName, cacheKey);
				}
			}
			return bean;
		}

8. 对Bean进行代理处理

		protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
			// 省略...
			// 获取需要进行代理的Bean
			Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
			if (specificInterceptors != DO_NOT_PROXY) {
				this.advisedBeans.put(cacheKey, Boolean.TRUE);
				// 生成代理
				Object proxy = createProxy(
						bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
				this.proxyTypes.put(cacheKey, proxy.getClass());
				return proxy;
			}
	
			this.advisedBeans.put(cacheKey, Boolean.FALSE);
			return bean;
		}

9. 代理生成逻辑

		protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
			@Nullable Object[] specificInterceptors, TargetSource targetSource) {

			// 省略...			

			return proxyFactory.getProxy(getProxyClassLoader());
		}

		public Object getProxy(@Nullable ClassLoader classLoader) {
			return createAopProxy().getProxy(classLoader);
		}

10. 首先获取对应的AOP代理，可以看到，提供了两种代理实现，JDK以及Cglib

		public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
			if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
				Class<?> targetClass = config.getTargetClass();
				if (targetClass == null) {
					throw new AopConfigException("TargetSource cannot determine target class: " +
							"Either an interface or a target is required for proxy creation.");
				}
				if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
					return new JdkDynamicAopProxy(config);
				}
				return new ObjenesisCglibAopProxy(config);
			}
			else {
				return new JdkDynamicAopProxy(config);
			}
		}

11. 看下```JdkDynamicAopProxy```
	> 看到 ```Proxy.newProxyInstance``` 这个基本心理就有数了，就是用的JDK的动态代理生成对应你Bean的代理类。Cglib同理。

		public Object getProxy(@Nullable ClassLoader classLoader) {
			if (logger.isDebugEnabled()) {
				logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
			}
			Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
			findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
			return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
		}

12. 回到 ```AopAutoProxyConfigurer.configureAutoProxyCreatorspring``` aop实现，定义```TransactionInterceptor```
	
		public static void configureAutoProxyCreator(Element element, ParserContext parserContext) {
			// 省略...

				// 创建事务拦截器
				RootBeanDefinition interceptorDef = new RootBeanDefinition(TransactionInterceptor.class);
				interceptorDef.setSource(eleSource);
				interceptorDef.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
				// 注册事务管理器
				registerTransactionManager(element, interceptorDef);
				interceptorDef.getPropertyValues().add("transactionAttributeSource", new RuntimeBeanReference(sourceName));
				String interceptorName = parserContext.getReaderContext().registerWithGeneratedName(interceptorDef);
			}
		}

13. ```TransactionInterceptor```就是事务的代理类
	
		protected Object invokeWithinTransaction(Method method, Class<?> targetClass, final InvocationCallback invocation)
			throws Throwable {

		// 获取事务的定义，TransactionAttribute接口继承了TransactionDefinition接口
		final TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);
		// 获取事务管理器，不同的实现有不同的事务管理器
		final PlatformTransactionManager tm = determineTransactionManager(txAttr);
		final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);

		if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
			// TransactionInfo包含TransactionStatus
			TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
			Object retVal = null;
			try {
				// 方法的实际执行
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) {
				// 包含事务管理器的回滚处理
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			}
			finally {
				cleanupTransactionInfo(txInfo);
			}
			commitTransactionAfterReturning(txInfo);
			return retVal;
		}
	}


### 分布式事务

JTA

TCC