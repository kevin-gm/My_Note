事务，主要是为了保证数据的一致性。spring事务的本质其实是对数据库事务的支持，没有数据库事务的支持，spring是无法提供事务保证的。

### Spring事务的作用：
将若干的数据库操作作为一个整体控制,一起成功或一起失败。

  1. 原子性：指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
  2. 一致性：指事务前后数据的完整性必须保持一致。
  3. 隔离性：指多个用户并发访问数据库时，一个用户的事务不能被其他用户的事务所干扰，多个并发事务之间数据要相互隔离。
  4. 持久性：指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，即时数据库发生故障也不应该对其有任何影响。
  
# 常见实现方式

## 编程式事务管理
使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。

对于编程式事务管理，spring推荐使用TransactionTemplate，实际最终也是调用的PlatformTransactionManager对应方法

###使用PlatformTransactionManager的实现，如DataSourceTransactionManager

	JdbcTemplate template = new JdbcTemplate(datasource);
	//定义事务管理器，因为使用的是JdbcTemplate，对应的是DataSourceTransactionManager
	PlatformTransactionManager txManager = new DataSourceTransactionManager(datasource);
	//事务定义类，可以设置传播属性、隔离级别
	DefaultTransactionDefinition def = new DefaultTransactionDefinition();
	def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
	//事务状态，包含回滚点等内容
	TransactionStatus status = tran.getTransaction(def);
	try {
		template.update("Insert into userinfo(username,password) values('aaaaa','bbbbb')");
		template.update("Insert into userinfo(username,password) values('cccc','ddd')");
		tran.commit(status);
	} catch (Exception ex) {
		tran.rollback(status);
	}

### 使用TransactionTemplate
	
	JdbcTemplate template = new JdbcTemplate(datasource);
	DataSourceTransactionManager tran = new DataSourceTransactionManager(datasource);
	TransactionTemplate trantemplate = new TransactionTemplate(tran);
	trantemplate.execute(new TransactionCallback() {
		public Object doInTransaction(TransactionStatus status) {
		int i = 0;
		try {
			template.update("Insert into userinfo(username,password) values('jjj','kkk')");
			template.update("Insert into userinfo(username,password) values('llll','mmm')");
			i = 1;
		} catch (Exception ex) {
			ex.printStackTrace();
			status.setRollbackOnly();
			i = 0;
		}
		return new Integer(i);
	}

## 声明式事务管理

### 基于切面（xml配置）
#### 配置事务管理器

	<!-- 配置事务管理器 -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
	   <property name="sessionFactory">
	       <ref bean="mySessionFactory"/>
	   </property>
	</bean>
#### 配置切面

	<!--  配置事务传播特性 -->
	<tx:advice id="TestAdvice" transaction-manager="transactionManager">
	    <tx:attributes>
	      <tx:method name="save*" propagation="REQUIRED"/>
	      <tx:method name="del*" propagation="REQUIRED"/>
	      <tx:method name="update*" propagation="REQUIRED"/>
	      <tx:method name="add*" propagation="REQUIRED"/>
	      <tx:method name="find*" propagation="REQUIRED"/>
	      <tx:method name="get*" propagation="REQUIRED"/>
	      <tx:method name="apply*" propagation="REQUIRED"/>
	    </tx:attributes>
	</tx:advice>
	<!--  配置参与事务的类 -->
	<aop:config>
		<aop:pointcut id="allTestServiceMethod" expression="execution(* com.test.testAda.test.model.service.*.*(..))"/>
		<aop:advisor pointcut-ref="allTestServiceMethod" advice-ref="TestAdvice" />
	</aop:config>

### 基于注解
#### 配置事务管理器

	<!-- 配置事务管理器 -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
	   <property name="sessionFactory">
	       <ref bean="mySessionFactory"/>
	   </property>
	</bean>
### 配置注解驱动
	
	<tx:annotation-driven transaction-manager="txManager">

### 之后可以在需要事务的类以及方法上使用@transactional注解，进行事务管理
建议只在具体的类或者具体类的方法上进行@Transactional注解，而不建议在接口或者接口接口方法上进行注解，虽然是可以的。但是如果注解在接口或者接口方法上，它仅仅只会在基于接口的代理上起作用。一个事实是，Java注解不继承接口，意味着，如果使用基于类的代理（proxy-target-class="true"）或者织入的代理（mode="aspectj"），那么事务的设置将不会被代理或者织入接口识别，对象将不被事务代理包裹

在默认的代理模式中，只有外部方法的调用才会被代理截获。意味着，自我调用，即目标类的一个方法调用目标类的另一个方法，是不会导致运行时事务的，即使方法被标注了@Transactional注解。（**嵌套事务需要单独成类**）

并且，代理必须完全初始化后才能提供预期行为，所以不能在初始化方法中依赖事务特性。比如@PostConstruct

如果希望自我调用也被事务包裹，可以考虑使用AspectJ模式。在这种情况下，首先不会有代理，换而是，目标类将会被织入，为了将任何方法上的@Transactional转换为运行时行为（目标类的字节码将会被修改）

# 事务相关概念
## 事务传播属性
事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为

传播属性定义在类：TransactionDefinition

1. PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这是默认值。
2. PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
3. PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
4. PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
5. PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
6. PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
7. PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

## 事务的隔离级别
隔离级别是指若干个并发的事务之间的隔离程度。

TransactionDefinition 接口中定义了五个表示隔离级别的常量

1. ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
2. ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读，不可重复读和幻读，因此很少使用该隔离级别。比如PostgreSQL实际上并没有此级别。
3. ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
4. ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。该级别可以防止脏读和不可重复读。
5. ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

<strong>脏读</strong>：
脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。

<strong>不可重复读</strong>：
是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。（即不能读到相同的数据内容）
例如，一个编辑人员两次读取同一文档，但在两次读取之间，作者重写了该文档。当编辑人员第二次读取文档时，文档已更改。原始读取不可重复。如果只有在作者全部完成编写后编辑人员才可以读取文档，则可以避免该问题。

<strong>幻读</strong>:
是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象
发生了幻觉一样。
例如，一个编辑人员更改作者提交的文档，但当生产部门将其更改内容合并到该文档的主复本时，发现作者已将未编辑的新材料添加到该文档中。如果在编辑人员和生产部门完成对原始文档的处理之前，任何人都不能将新材料添加到文档中，则可以避免该问题。

## 事务的只读属性（readonly=true）
“只读”并不是一个强制选项，它只是一个“暗示”，提示数据库驱动程序和数据库系统，这个事务并不包含更改数据的操作，那么JDBC驱动程序和数据库就有可能根据这种情况对该事务进行一些特定的优化，比方说不安排相应的数据库锁，以减轻事务对数据库的压力，毕竟事务也是要消耗数据库的资源的。 

但是你非要在“只读事务”里面修改数据，也并非不可以，只不过对于数据一致性的保护不像“读写事务”那样保险而已。 

因此，“只读事务”仅仅是一个性能优化的推荐配置而已，并非强制你要这样做不可

## 事务的回滚
spring事务管理器回滚一个事务的推荐方法是在当前事务的上下文内抛出异常。spring事务管理器会捕捉任何未处理的异常，然后依据规则决定是否回滚抛出异常的事务。

默认配置下，spring只有在抛出的异常为运行时unchecked异常时才回滚该事务，也就是抛出的异常为RuntimeException的子类(Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。可以明确的配置在抛出那些异常时回滚事务，包括checked异常。也可以明确定义那些异常抛出时不回滚事务。还可以编程性的通过setRollbackOnly()方法来指示一个事务必须回滚，在调用完setRollbackOnly()后你所能执行的唯一操作就是回滚。

## 事务处理器的底层实现
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
	
		int PROPAGATION_REQUIRED = 0;
		int PROPAGATION_SUPPORTS = 1;
		int PROPAGATION_MANDATORY = 2;
		int PROPAGATION_REQUIRES_NEW = 3;
		int PROPAGATION_NOT_SUPPORTED = 4;
		int PROPAGATION_NEVER = 5;
		int PROPAGATION_NESTED = 6;
	
		int ISOLATION_DEFAULT = -1;
		int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;
		int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;
		int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;
		int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;
		int TIMEOUT_DEFAULT = -1;
	
		int getPropagationBehavior();
	
		int getIsolationLevel();
	
		int getTimeout();
	
		boolean isReadOnly();
	
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

### DataSourceTransactionManager实现
> http://blog.csdn.net/chjttony/article/details/6538452

	public class DataSourceTransactionManager extends AbstractPlatformTransactionManager  
        implements ResourceTransactionManager, InitializingBean { 
	    //注入数据源  
	    private DataSource dataSource;  
		//数据源事务处理器默认构造方法，创建一个数据源事务处理器实例，并设置允许嵌套事务  
	    public DataSourceTransactionManager() {  
	        setNestedTransactionAllowed(true);  
	    }  
	    //根据给定数据源，创建一个数据源事务处理器实例  
	    public DataSourceTransactionManager(DataSource dataSource) {  
	        this();  
	        setDataSource(dataSource);  
	        afterPropertiesSet();  
	    }  
	    //设置数据源  
	    public void setDataSource(DataSource dataSource) {  
	        if (dataSource instanceof TransactionAwareDataSourceProxy) {  
	            //如果数据源是一个事务包装数据源代理，则获取事务包装代理的目标数据源   
	            this.dataSource = ((TransactionAwareDataSourceProxy) dataSource).getTargetDataSource();  
	        }  
	        else {  
	            this.dataSource = dataSource;  
	        }  
	    }  
	    //获取数据源  
	    public DataSource getDataSource() {  
	        return this.dataSource;  
	    }  
	    //数据源事务处理器对象构造方法的回调函数  
	    public void afterPropertiesSet() {  
	        if (getDataSource() == null) {  
	            throw new IllegalArgumentException("Property 'dataSource' is required");  
	        }  
	    }  
	public Object getResourceFactory() {  
	        return getDataSource();  
	    }  
	//创建事务，对数据库而言，是由Connection来完成事务工作的。该方法把数据库的//Connection对象放到一个ConnectionHolder对象中，然后封装到一个  
	//DataSourceTransactionObject对象中  
	    protected Object doGetTransaction() {  
	        //创建数据源事务对象  
	        DataSourceTransactionObject txObject = new DataSourceTransactionObject();  
	        //设置数据源事务对象对嵌套事务使用保存点  
	        txObject.setSavepointAllowed(isNestedTransactionAllowed());  
	        //从事务管理容器中获取存放数据库Connection的对象  
	        ConnectionHolder conHolder =  
	            (ConnectionHolder) TransactionSynchronizationManager.getResource(this.dataSource);  
	        txObject.setConnectionHolder(conHolder, false);  
	        return txObject;  
	    }  
	    //判断是否已经存在事务  
	    protected boolean isExistingTransaction(Object transaction) {  
	        DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;  
	    //根据存放数据库连接的ConnectionHolder的isTransactionActive属性来判断  
	        return (txObject.getConnectionHolder() != null && txObject.getConnectionHolder().isTransactionActive());  
	    }  
	    //处理事务开始的方法  
	    protected void doBegin(Object transaction, TransactionDefinition definition) {  
	        DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;  
	        Connection con = null;  
	        try {  
	            //如果数据源事务对象的ConnectionHolder为null或者是事务同步的  
	            if (txObject.getConnectionHolder() == null ||  
	        txObject.getConnectionHolder().isSynchronizedWithTransaction()) {  
	                //获取当前数据源的数据库连接  
	                Connection newCon = this.dataSource.getConnection();  
	                if (logger.isDebugEnabled()) {  
	                    logger.debug("Acquired Connection [" + newCon + "] for JDBC transaction");  
	                }  
	                //为数据源事务对象设置ConnectionHolder  
	                txObject.setConnectionHolder(new ConnectionHolder(newCon), true);  
	            }  
	    //设置数据源事务对象的事务同步    txObject.getConnectionHolder().setSynchronizedWithTransaction(true);  
	            //获取数据源事务对象的数据库连接  
	            con = txObject.getConnectionHolder().getConnection();  
	            //根据数据连接和事务属性，获取数据库连接的事务隔离级别  
	            Integer previousIsolationLevel = DataSourceUtils.prepareConnectionForTransaction(con, definition);  
	    //为数据源事务对象设置事务隔离级别  
	    txObject.setPreviousIsolationLevel(previousIsolationLevel);  
	            //如果数据库连接设置了自动事务提交属性，则关闭自动提交  
	            if (con.getAutoCommit()) {  
	                //保存数据库连接设置的自动连接到数据源事务对象中  
	                txObject.setMustRestoreAutoCommit(true);  
	                if (logger.isDebugEnabled()) {  
	                    logger.debug("Switching JDBC Connection [" + con + "] to manual commit");  
	                }  
	                //设置数据库连接自动事务提交属性为false，即禁止自动事务提交  
	                con.setAutoCommit(false);  
	            }  
	            //激活当前数据源事务对象的事务配置  
	            txObject.getConnectionHolder().setTransactionActive(true);  
	            //获取事务配置的超时时长  
	int timeout = determineTimeout(definition);  
	//如果事务配置的超时时长不等于事务的默认超时时长  
	            if (timeout != TransactionDefinition.TIMEOUT_DEFAULT) {  
	        //数据源事务对象设置超时时长  
	        txObject.getConnectionHolder().setTimeoutInSeconds(timeout);  
	            }  
	            //把当前数据库Connection和线程绑定  
	            if (txObject.isNewConnectionHolder()) {  
	        TransactionSynchronizationManager.bindResource(getDataSource(), txObject.getConnectionHolder());  
	            }  
	        }  
	        catch (Exception ex) {  
	            DataSourceUtils.releaseConnection(con, this.dataSource);  
	            throw new CannotCreateTransactionException("Could not open JDBC Connection for transaction", ex);  
	        }  
	    }  
	    //事务挂起  
	    protected Object doSuspend(Object transaction) {  
	        //获取事务对象  
	        DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;  
	        //将事务对象中的ConnectionHolders设置为null  
	        txObject.setConnectionHolder(null);  
	        ConnectionHolder conHolder = (ConnectionHolder)  
	        //解除事务对象和当前线程的绑定    TransactionSynchronizationManager.unbindResource(this.dataSource);  
	        return conHolder;  
	    }  
	    //事务恢复  
	    protected void doResume(Object transaction, Object suspendedResources) {  
	        //获取已暂停事务的ConnectionHolder  
	        ConnectionHolder conHolder = (ConnectionHolder) suspendedResources;  
	        //重新将事务对象和当前线程绑定  
	        TransactionSynchronizationManager.bindResource(this.dataSource, conHolder);  
	    }  
	    //事务提交  
	    protected void doCommit(DefaultTransactionStatus status) {  
	        //获取事务对象  
	        DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();  
	        //通过事务对象获取数据库连接  
	        Connection con = txObject.getConnectionHolder().getConnection();  
	        if (status.isDebug()) {  
	            logger.debug("Committing JDBC transaction on Connection [" + con + "]");  
	        }  
	        try {  
	            //使用数据库连接手动进行事务提交  
	            con.commit();  
	        }  
	        catch (SQLException ex) {  
	            throw new TransactionSystemException("Could not commit JDBC transaction", ex);  
	        }  
	    }  
	    //事务回滚  
	    protected void doRollback(DefaultTransactionStatus status) {  
	        //获取事务对象  
	        DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();  
	        //通过事务对象获取数据库连接  
	        Connection con = txObject.getConnectionHolder().getConnection();  
	        if (status.isDebug()) {  
	            logger.debug("Rolling back JDBC transaction on Connection [" + con + "]");  
	        }  
	        try {  
	            //通过调用数据库连接的回滚方法完成事务回滚操作  
	            con.rollback();  
	        }  
	        catch (SQLException ex) {  
	            throw new TransactionSystemException("Could not roll back JDBC transaction", ex);  
	        }  
	    }  
	    //设置回滚  
	    protected void doSetRollbackOnly(DefaultTransactionStatus status) {  
	        DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();  
	        if (status.isDebug()) {  
	            logger.debug("Setting JDBC transaction [" + txObject.getConnectionHolder().getConnection() +  
	                    "] rollback-only");  
	        }  
	        txObject.setRollbackOnly();  
	    }  
	    //操作完成之后清除操作  
	    protected void doCleanupAfterCompletion(Object transaction) {  
	        DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;  
	        //移除当前线程绑定的ConnectionHolder  
	        if (txObject.isNewConnectionHolder()) {  
	    TransactionSynchronizationManager.unbindResource(this.dataSource);  
	        }  
	        Connection con = txObject.getConnectionHolder().getConnection();  
	        try {  
	        //如果事务对象保存了自动事务提交属性，则设置数据库连接的自动事务提交属性  
	            if (txObject.isMustRestoreAutoCommit()) {  
	                con.setAutoCommit(true);  
	            }  
	            //事务结束后重置数据库连接  
	            DataSourceUtils.resetConnectionAfterTransaction(con, txObject.getPreviousIsolationLevel());  
	        }  
	        catch (Throwable ex) {  
	            logger.debug("Could not reset JDBC Connection after transaction", ex);  
	        }  
	        //如果事务对象中有新的ConnectionHolder   
	        if (txObject.isNewConnectionHolder()) {  
	            if (logger.isDebugEnabled()) {  
	                logger.debug("Releasing JDBC Connection [" + con + "] after transaction");  
	            }  
	            //释放数据库连接  
	            DataSourceUtils.releaseConnection(con, this.dataSource);  
	        }  
	        //清除事务对象的ConnectionHolder  
	        txObject.getConnectionHolder().clear();  
	    }  
		//数据源事务对象，内部类  
	    private static class DataSourceTransactionObject extends JdbcTransactionObjectSupport {  
	        //是否有新的ConnectionHolder  
	        private boolean newConnectionHolder;  
	        //是否保存自动提交  
	        private boolean mustRestoreAutoCommit;  
	        //设置ConnectionHolder  
	        public void setConnectionHolder(ConnectionHolder connectionHolder, boolean newConnectionHolder) {  
	            //为父类JdbcTransactionObjectSupport设置ConnectionHolder  
	            super.setConnectionHolder(connectionHolder);  
	            this.newConnectionHolder = newConnectionHolder;  
	        }  
	        public boolean isNewConnectionHolder() {  
	            return this.newConnectionHolder;  
	        }  
	        //调用父类JdbcTransactionObjectSupport的相关方法，查询收费存在事务  
	        public boolean hasTransaction() {  
	            return (getConnectionHolder() != null && getConnectionHolder().isTransactionActive());  
	        }  
	        //设置是否保存自动提交  
	        public void setMustRestoreAutoCommit(boolean mustRestoreAutoCommit) {  
	            this.mustRestoreAutoCommit = mustRestoreAutoCommit;  
	        }  
	        public boolean isMustRestoreAutoCommit() {  
	            return this.mustRestoreAutoCommit;  
	        }  
	        //设置数据库连接在操作失败时，是否只回滚处理  
	        public void setRollbackOnly() {  
	            getConnectionHolder().setRollbackOnly();  
	        }  
	        public boolean isRollbackOnly() {  
	            return getConnectionHolder().isRollbackOnly();  
	        }  
	    }  
	}

## 事务原理

## springboot 开始事务支持
@EnableTransactionManagement