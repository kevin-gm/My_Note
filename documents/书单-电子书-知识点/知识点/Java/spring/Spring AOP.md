## AspectJ（简单了解）
AspectJ是一套独立的面向切面编程的解决方案，是一个java实现的AOP框架，它能够对java代码进行AOP编译（一般在编译期进行），让java代码具有AspectJ的AOP功能（当然需要特殊的编译器）
> 示例来源：http://blog.csdn.net/javazejian/article/details/56267036
> 安装步骤：http://www.jianshu.com/p/fe8d1e8bd63e

##### 安装aspectj
1. 下载AspectJ jar包，下载地址（http://www.eclipse.org/aspectj/downloads.php）
2. 双击安装，安装好的目录结构为：
	- bin：存放了 aj、aj5、ajc、ajdoc、ajbrowser 等命令，其中 ajc 命令最常用，它的作用类似于 javac
	- doc：存放了 AspectJ 的使用说明、参考手册、API 文档等文档
	- lib：该路径下的 4 个 JAR 文件是 AspectJ 的核心类库

##### 客户端，业务方法

	public class HelloWord {

	    public void sayHello(){
	        System.out.println("hello world !");
	    }
	    public static void main(String args[]){
	        HelloWord helloWord =new HelloWord();
	        helloWord.sayHello();
	    }
	}

##### AspectJ类
注意关键字为aspect(MyAspectJDemo.aj,其中aj为AspectJ的后缀)，含义与class相同，即定义一个AspectJ的类

	public aspect MyAspectJDemo {
	    /**
	     * 定义切点,日志记录切点
	     */
	    pointcut recordLog():call(* HelloWord.sayHello(..));
	
	    /**
	     * 定义切点,权限验证(实际开发中日志和权限一般会放在不同的切面中,这里仅为方便演示)
	     */
	    pointcut authCheck():call(* HelloWord.sayHello(..));
	
	    /**
	     * 定义前置通知!
	     */
	    before():authCheck(){
	        System.out.println("sayHello方法执行前验证权限");
	    }
	
	    /**
	     * 定义后置通知
	     */
	    after():recordLog(){
	        System.out.println("sayHello方法执行后记录日志");
	    }
	}

##### AspectJ的织入方式及其原理概要
> http://blog.csdn.net/javazejian/article/details/56267036

AspectJ应用到java代码的过程（这个过程称为织入），对于织入这个概念，可以简单理解为aspect(切面)应用到目标函数(类)的过程

一般分为**动态织入**和**静态织入**:

- 动态织入的方式是在运行时动态将要增强的代码织入到目标类中，这样往往是通过动态代理技术完成的，如Java JDK的动态代理(Proxy，底层通过反射实现)或者CGLIB的动态代理(底层通过继承实现)，Spring AOP采用的就是基于运行时增强的代理技术
- ApectJ主要采用静态织入的方式,即编译期织入，在这个期间使用AspectJ的acj编译器(类似javac)把aspect类编译成class字节码后，在java目标类编译时织入，即先编译aspect类再编译目标类

ajc编译器，是一种能够识别aspect语法的编译器，它是采用java语言编写的，由于javac并不能识别aspect语法，便有了ajc编译器，注意ajc编译器也可编译java文件

# Spring AOP
aspect-oriented programming，面向切面编程。

作为面向对象编程的一种补充，广泛应用于处理一些具有横切性质的系统级服务，如事务管理、安全检查、权限校验等。

Spring AOP 与ApectJ 的目的一致，都是为了统一处理横切业务，但与AspectJ不同的是，Spring AOP 并不尝试提供完整的AOP功能(即使它完全可以实现)，Spring AOP 更注重的是与Spring IOC容器的结合，并结合该优势来解决横切业务的问题

Spring注意到AspectJ在AOP的实现方式上依赖于特殊编译器(ajc编译器)，因此Spring很机智回避了这点，转向采用动态代理技术的实现原理来构建Spring AOP的内部机制（动态织入），这是与AspectJ（静态织入）最根本的区别。

Spring缺省使用J2SE 动态代理（dynamic proxies）来作为AOP的代理。 这样任何接口（或者接口集）都可以被代理。
Spring也可以使用CGLIB代理. 对于需要代理类而不是代理接口的时候CGLIB代理是很有必要的。如果一个业务对象并没有实现一个接口，默认就会使用CGLIB

@AspectJ使用了Java 5的注解，可以将切面声明为普通的Java类。@AspectJ样式在AspectJ 5发布的AspectJ project部分中被引入。Spring 2.0使用了和AspectJ 5一样的注解，并使用AspectJ来做切入点解析和匹配。但是，AOP在运行时仍旧是纯的Spring AOP，并不依赖于AspectJ的编译器或者织入器（weaver）

### Spring AOP实现示例
Spring组件默认为基于注解驱动的配置
##### 1. spring配置文件，启用@AspectJ支持和spring bean的注解扫描

	<context:component-scan base-package="site.coloured.aop"/>

    <aop:aspectj-autoproxy/>

##### 2. 定义切面，切点和建言

	@Aspect // 声明当前类为切面类
	@Component // 声明切面类为Spring管理
	public class LogAspect {
	
		// 切面配置，当前为具有LogAnnotation注解的将会进入AOP代理
	    @Pointcut(value = "@annotation(site.coloured.aop.annotation.LogAnnatation)")
	    public void pointCut(){}
	
		// 前置通知
	    @Before(value = "pointCut()")
	    public void before() {
	        System.out.println("before method invoke...");
	    }
	
		// 后置通知
	    @After(value = "pointCut()")
	    public void after() {
	        System.out.println("after method invoke...");
	    }
	
		// 环绕通知
		// JoinPoint是连接点
	    @Around(value = "pointCut()")
	    public Object around(ProceedingJoinPoint point) {
	        System.out.println("before around");
	        Object o = null;
	        try {
				// 获取方法的参数
	            Object[] params = point.getArgs();
	            if(params != null) {
	                for (Object obj : params) {
	                    System.out.println("params : " + obj);
	                }
	            }
	            o = point.proceed();
				// 打印返回结果
				System.out.println(o);
	        } catch (Throwable throwable) {
	            throwable.printStackTrace();
	        }
	        System.out.println("after around");
	        return o;
	    }
	}

##### 3. 定义需要的注解

	@Documented
	@Inherited
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)
	public @interface LogAnnatation {
	}

##### 4. 业务方法

	@Service
	public class BizService {
	
	    @LogAnnatation
	    public String testAop(String userName) {
	        System.out.println("userName = " + userName);
	        return userName + "000";
	    }
	
	    @LogAnnatation
	    public void testAop() {
	        System.out.println("none of params and return type");
	    }
	}

##### 5. 客户端调用

	public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-config.xml");
        BizService service = (BizService)context.getBean("bizService");
        service.testAop();
        System.out.println("------------------------");
        service.testAop("lisi");
    }

##### 6. 结果打印

	before around
	before method invoke...
	none of params and return type
	null
	after around
	after method invoke...
	------------------------
	before around
	params : lisi
	before method invoke...
	userName = lisi
	lisi000
	after around
	after method invoke...


## AOP概念

* 切面（Aspect）：一个关注点的模块化，这个关注点可能会横切多个对象。事务管理是J2EE应用中一个关于横切关注点的很好的例子。在Spring AOP中，切面可以使用基于Schema的AOP支持或者基于@Aspect注解的方式来实现。
* 连接点（Joinpoint）：在程序执行过程中某个特定的点，比如某方法调用的时候或者处理异常的时候。在Spring AOP中，一个连接点总是表示一个方法的执行。
* 通知（Advice）：在切面的某个特定的连接点上执行的动作。其中包括了“around”、“before”和“after”等不同类型的通知（通知的类型将在后面部分进行讨论）。许多AOP框架（包括Spring）都是以拦截器做通知模型，并维护一个以连接点为中心的拦截器链。
* 切入点（Pointcut）：匹配连接点的断言。通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行（例如，当执行某个特定名称的方法时）。切入点表达式如何和连接点匹配是AOP的核心：Spring缺省使用AspectJ切入点语法。
* 引入（Introduction）：用来给一个类型声明额外的方法或属性（也被称为连接类型声明（inter-type declaration））。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用引入来使一个bean实现IsModified接口，以便简化缓存机制。
* 目标对象（Target Object）： 被一个或者多个切面所通知的对象。也被称做被通知（advised）对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个被代理（proxied）对象。
* AOP代理（AOP Proxy）：AOP框架创建的对象，用来实现切面契约（例如通知方法执行等等）。在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。
* 织入（Weaving）：把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。

实现 ```Introduction``` 的一般方式为使用 ```@DeclareParents``` 注解，下面用一个例子说明。
> http://www.blogjava.net/jackfrued/archive/2010/02/27/314060.html

假设有一个用户保存的逻辑，现在需要对用户数据进行校验，合理的才允许保存。基础用户服务类

	@Service
	public class UserService {
	
	    public void save(User user) {
	        System.out.println("save user : " + user.getUserName());
	    }
	}

现在利用引入增强实现这个功能

定义校验接口

	public interface Verifier {

	    boolean validate(User user);
	}

校验接口的基本实现类

	public class BasicVerifier implements Verifier {

	    @Override
	    public boolean validate(User user) {
	        if(user != null && "kevin".equals(user.getUserName())) {
	            return true;
	        }
	        return false;
	    }
	}

引入增强的切面配置，利用 DeclareParents 注解

	@Aspect
	@Component
	public class VerifyAspect {
	
	    @DeclareParents(value = "site.coloured.aop.UserService", defaultImpl = site.coloured.aop.BasicVerifier.class)
	    public Verifier verifier;
	}

客户端调用

	public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-config.xml");
        User user = new User();
        user.setUserName("lisi");
        UserService service = (UserService)context.getBean("userService");
        Verifier v = (Verifier)service;
        if(v.validate(user)) {
            System.out.println("验证成功");
            service.save(user);
        }
    }

根据打印结果可知，只有在 userName=kevin时，才会有打印信息。这就是引入增强。
	

**通知类型**：

* 前置通知（Before advice）：在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。
* 后置通知（After returning advice）：在某连接点正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。
* 异常通知（After throwing advice）：在方法抛出异常退出时执行的通知。
* 最终通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
* 环绕通知（Around Advice）：包围一个连接点的通知，如方法调用。这是最强大的一种通知类型。环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它自己的返回值或抛出异常来结束执行。

# Spring AOP原理初探
从上面的示例可知，Spring AOP开始于配置 ```<aop:aspectj-autoproxy/>```。又由spring加载和初始化的知识可知，在spring对应的组件中，对应有aop这个自定义标签的处理解析器，查看spring-aop这个组件， ```META-INF``` 文件夹下的 ```spring.handlers``` 可以看到，对应有 ```AopNamespaceHandler``` 的配置，这个就是aop自定义标签的处理器。

	public class AopNamespaceHandler extends NamespaceHandlerSupport {

		@Override
		public void init() {
			registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
			registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
			registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());
			registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
		}
	
	}

我们的切入点是基于注解驱动的，所以只看 ```AspectJAutoProxyBeanDefinitionParser``` 。所有解析器，都是对 ```BeanDefinitionParser``` 接口的实现，所以解析都是从parse方法开始的

	public BeanDefinition parse(Element element, ParserContext parserContext) {
		AopNamespaceUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(parserContext, element);
		extendBeanDefinition(element, parserContext);
		return null;
	}

```registerAspectJAnnotationAutoProxyCreatorIfNecessary``` 是关键逻辑

	public static void registerAspectJAnnotationAutoProxyCreatorIfNecessary(
			ParserContext parserContext, Element sourceElement) {

		// 注册或者升级 AnnotationAwareAspectJAutoProxyCreator
		BeanDefinition beanDefinition = AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(
				parserContext.getRegistry(), parserContext.extractSource(sourceElement));
		// 对 proxy-target-class 以及 expose-Proxy属性的处理
		// proxy-target-class ： 为true时，强制使用cglib代理，默认为false
		// expose-Proxy：使内部方法调用可以得到增强，但需要从aop上下文获取代理进行调用，比如((AService)AopContext.currentProxy()).b();
		useClassProxyingIfNecessary(parserContext.getRegistry(), sourceElement);
		// 注册组件并通知，便于监听器进一步处理
		// 此时，beanDefinition的BeanName 为 AnnotationAwareAspectJAutoProxyCreator
		registerComponentIfNecessary(beanDefinition, parserContext);
	}

注册或者升级 AnnotationAwareAspectJAutoProxyCreator

	public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry,
			@Nullable Object source) {
		return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
	}

	private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry,
			@Nullable Object source) {
		// 如果已经存在了自动代理创建器，并且存在的自动代理创建器与现在的不一致，则需要根据优先级判断使用哪个
		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
			BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
				int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
				int requiredPriority = findPriorityForClass(cls);
				if (currentPriority < requiredPriority) {
					apcDefinition.setBeanClassName(cls.getName());
				}
			}
			// 如果已经存在并且一致，则无需创建
			return null;
		}
		// 定义Bean并进行注册
		RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
		beanDefinition.setSource(source);
		beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
		beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
		return beanDefinition;
	}

##### AnnotationAwareAspectJAutoProxyCreator
查看该类的层次结构，可以发现，其实际上实现了BeanPostProcessor接口，意味着Spring的Bean在初始化后，会调用其 ```postProcessAfterInitialization``` 方法

	public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) throws BeansException {
		if (bean != null) {
			// 构建缓存key
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (!this.earlyProxyReferences.contains(cacheKey)) {
				// 如果当前Bean需要被代理，则需要对这个Bean进行包装。
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
	}

	protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
		// 如果已经处理过，则不继续处理
		if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
			return bean;
		}
		// 如果已经进行了增强，也不继续
		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
			return bean;
		}
		// 如果Bean是基础设施类，比如实现了pointcut接口，则不需要进行自动代理
		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
			this.advisedBeans.put(cacheKey, Boolean.FALSE);
			return bean;
		}

		// 获取需要创建代理的增强
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
		if (specificInterceptors != DO_NOT_PROXY) {
			this.advisedBeans.put(cacheKey, Boolean.TRUE);
			// 创建代理
			Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
			return proxy;
		}

		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
	}

由此可见，创建代理的逻辑主要包含两个步骤：

- 获取增强
- 根据获取的增强创建代理

接下来看下获取增强方法的逻辑

	protected Object[] getAdvicesAndAdvisorsForBean(Class<?> beanClass, String beanName, @Nullable TargetSource targetSource) {
		// 查找符合条件的增强
		List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
		if (advisors.isEmpty()) {
			return DO_NOT_PROXY;
		}
		return advisors.toArray();
	}
	
	protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
		// 首先获取候选的增强，也就是目前所有的增强
		List<Advisor> candidateAdvisors = findCandidateAdvisors();
		// 然后查找跟当前Bean匹配的
		List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
		extendAdvisors(eligibleAdvisors);
		if (!eligibleAdvisors.isEmpty()) {
			// 排序
			eligibleAdvisors = sortAdvisors(eligibleAdvisors);
		}
		return eligibleAdvisors;
	}

由于分析的是基于注解的AOP，所以 ```findCandidateAdvisors``` 的实现是由 ```AnnotationAwareAspectJAutoProxyCreator``` 提供的。

	protected List<Advisor> findCandidateAdvisors() {
		// 调用父类方法加载啊配置文件中配置的AOP声明
		List<Advisor> advisors = super.findCandidateAdvisors();
		if (this.aspectJAdvisorsBuilder != null) {
			// 获取基于注解增强的Bean
			advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
		}
		return advisors;
	}

可以猜测一下，获取基于注解的增强，需要做的是查找所有在Spring容器中注册的Bean，然后判断其是否有 ```@aspectj``` 的注解，然后对有该注解的进行处理。这就是 ```buildAspectJAdvisors``` 的内容

	public List<Advisor> buildAspectJAdvisors() {
		List<String> aspectNames = this.aspectBeanNames;

		if (aspectNames == null) {
			synchronized (this) {
				aspectNames = this.aspectBeanNames;
				if (aspectNames == null) {
					List<Advisor> advisors = new LinkedList<>();
					aspectNames = new LinkedList<>();
					// 从Spring容器中获取所有的BeanName
					String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
							this.beanFactory, Object.class, true, false);
					// 遍历，找出增强方法
					for (String beanName : beanNames) {
						// 不合法的过滤掉，该方法可以由子类扩展，默认都是合法的
						if (!isEligibleBean(beanName)) {
							continue;
						}
						// 获取对应的Bean类型
						Class<?> beanType = this.beanFactory.getType(beanName);
						if (beanType == null) {
							continue;
						}
						// 如果存在Aspect注解
						// 这个判断实际包括两部分，有Aspect注解，并且该类没有经过ajc编译器进行编译。因为Spring aop使用的是动态织入而不是静态织入。
						if (this.advisorFactory.isAspect(beanType)) {
							aspectNames.add(beanName);
							// 构建Aspect元数据，虽然Spring AOP没有使用AspectJ的静态织入，但是使用AspectJ来做切入点解析和匹配
							AspectMetadata amd = new AspectMetadata(beanType, beanName);
							if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
								MetadataAwareAspectInstanceFactory factory =
										new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
								// 解析标记AspectJ注解中的增强方法
								List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
								if (this.beanFactory.isSingleton(beanName)) {
									this.advisorsCache.put(beanName, classAdvisors);
								}
								else {
									this.aspectFactoryCache.put(beanName, factory);
								}
								advisors.addAll(classAdvisors);
							}
							else {
								// Per target or per this.
								if (this.beanFactory.isSingleton(beanName)) {
									throw new IllegalArgumentException("Bean with name '" + beanName +
											"' is a singleton, but aspect instantiation model is not singleton");
								}
								MetadataAwareAspectInstanceFactory factory =
										new PrototypeAspectInstanceFactory(this.beanFactory, beanName);
								this.aspectFactoryCache.put(beanName, factory);
								advisors.addAll(this.advisorFactory.getAdvisors(factory));
							}
						}
					}
					this.aspectBeanNames = aspectNames;
					return advisors;
				}
			}
		}

		if (aspectNames.isEmpty()) {
			return Collections.emptyList();
		}
		// 结果缓存
		List<Advisor> advisors = new LinkedList<>();
		for (String aspectName : aspectNames) {
			List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
			if (cachedAdvisors != null) {
				advisors.addAll(cachedAdvisors);
			}
			else {
				MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
				advisors.addAll(this.advisorFactory.getAdvisors(factory));
			}
		}
		return advisors;
	}
 
进一步，增强的获取是委托给 ```this.advisorFactory.getAdvisors(factory)``` 进行的。

	public List<Advisor> getAdvisors(MetadataAwareAspectInstanceFactory aspectInstanceFactory) {
		// 获取标记为AspectJ的类，前面说过解析还是通过AspectJ解析的，在构建元数据阶段，会将类标记为AspectJ
		Class<?> aspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
		// 获取对应的name名称
		String aspectName = aspectInstanceFactory.getAspectMetadata().getAspectName();
		// 验证
		validate(aspectClass);

		MetadataAwareAspectInstanceFactory lazySingletonAspectInstanceFactory =
				new LazySingletonAspectInstanceFactoryDecorator(aspectInstanceFactory);

		List<Advisor> advisors = new LinkedList<>();
		// 获取当前类中所有需要增强的方法
		for (Method method : getAdvisorMethods(aspectClass)) {
			// 每一个方法构建一个增强器
			Advisor advisor = getAdvisor(method, lazySingletonAspectInstanceFactory, advisors.size(), aspectName);
			if (advisor != null) {
				advisors.add(advisor);
			}
		}

		// 如果寻找的增强不为空，并且配置了增强延迟初始化，则需要在首位加入同步实例化增强器
		if (!advisors.isEmpty() && lazySingletonAspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
			Advisor instantiationAdvisor = new SyntheticInstantiationAdvisor(lazySingletonAspectInstanceFactory);
			advisors.add(0, instantiationAdvisor);
		}

		// 获取并处理 DeclareParents 注解，这个是处理引入增强的。
		for (Field field : aspectClass.getDeclaredFields()) {
			Advisor advisor = getDeclareParentsAdvisor(field);
			if (advisor != null) {
				advisors.add(advisor);
			}
		}

		return advisors;
	}

	// 获取需要增强的方法
	private List<Method> getAdvisorMethods(Class<?> aspectClass) {
		final List<Method> methods = new LinkedList<>();
		ReflectionUtils.doWithMethods(aspectClass, method -> {
			// 当前类中，有pointcut注解的方法不处理
			if (AnnotationUtils.getAnnotation(method, Pointcut.class) == null) {
				methods.add(method);
			}
		});
		// 方法进行排序
		methods.sort(METHOD_COMPARATOR);
		return methods;
	}

进一步跟踪增强器构建的逻辑

	public Advisor getAdvisor(Method candidateAdviceMethod, MetadataAwareAspectInstanceFactory aspectInstanceFactory,
			int declarationOrderInAspect, String aspectName) {
		// 校验
		validate(aspectInstanceFactory.getAspectMetadata().getAspectClass());
		// 获取切点信息
		AspectJExpressionPointcut expressionPointcut = getPointcut(
				candidateAdviceMethod, aspectInstanceFactory.getAspectMetadata().getAspectClass());
		if (expressionPointcut == null) {
			return null;
		}
		// 根据切点信息，生成增强器
		return new InstantiationModelAwarePointcutAdvisorImpl(expressionPointcut, candidateAdviceMethod,
				this, aspectInstanceFactory, declarationOrderInAspect, aspectName);
	}

切点信息的获取就是对注解表达式的解析，比如@Before

	private AspectJExpressionPointcut getPointcut(Method candidateAdviceMethod, Class<?> candidateAspectClass) {
		// 获取方法上的注解
		AspectJAnnotation<?> aspectJAnnotation =
				AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
		if (aspectJAnnotation == null) {
			return null;
		}
		// 使用 AspectJExpressionPointcut 封装获取到的信息
		AspectJExpressionPointcut ajexp =
				new AspectJExpressionPointcut(candidateAspectClass, new String[0], new Class<?>[0]);
		// 提取注解中的表达式信息。不如@pointcut("execution(* a.b.*(..))") 中的 execution(* a.b.*(..))
		ajexp.setExpression(aspectJAnnotation.getPointcutExpression());
		if (this.beanFactory != null) {
			ajexp.setBeanFactory(this.beanFactory);
		}
		return ajexp;
	}

	protected static AspectJAnnotation<?> findAspectJAnnotationOnMethod(Method method) {
		// 需要匹配的注解
		Class<?>[] classesToLookFor = new Class<?>[] {
				Before.class, Around.class, After.class, AfterReturning.class, AfterThrowing.class, Pointcut.class};
		for (Class<?> c : classesToLookFor) {
			AspectJAnnotation<?> foundAnnotation = findAnnotation(method, (Class<Annotation>) c);
			if (foundAnnotation != null) {
				return foundAnnotation;
			}
		}
		return null;
	}

对增强器信息进行封装，所有的增强器信息都有Advisor接口的实现类 ```InstantiationModelAwarePointcutAdvisorImpl``` 统一封装。

	public InstantiationModelAwarePointcutAdvisorImpl(AspectJExpressionPointcut declaredPointcut,
			Method aspectJAdviceMethod, AspectJAdvisorFactory aspectJAdvisorFactory,
			MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {

		this.declaredPointcut = declaredPointcut;
		this.declaringClass = aspectJAdviceMethod.getDeclaringClass();
		this.methodName = aspectJAdviceMethod.getName();
		this.parameterTypes = aspectJAdviceMethod.getParameterTypes();
		this.aspectJAdviceMethod = aspectJAdviceMethod;
		this.aspectJAdvisorFactory = aspectJAdvisorFactory;
		this.aspectInstanceFactory = aspectInstanceFactory;
		this.declarationOrder = declarationOrder;
		this.aspectName = aspectName;

		if (aspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
			Pointcut preInstantiationPointcut = Pointcuts.union(
					aspectInstanceFactory.getAspectMetadata().getPerClausePointcut(), this.declaredPointcut);
			this.pointcut = new PerTargetInstantiationModelPointcut(
					this.declaredPointcut, preInstantiationPointcut, aspectInstanceFactory);
			this.lazy = true;
		}
		else {
			this.pointcut = this.declaredPointcut;
			this.lazy = false;
			this.instantiatedAdvice = instantiateAdvice(this.declaredPointcut);
		}
	}

封装的过程，大部分都是属性的设置，在实例初始化的过程中，还完成了对增强器的初始化。不同的增强注解有不同的增强器。

	private Advice instantiateAdvice(AspectJExpressionPointcut pointcut) {
		Advice advice = this.aspectJAdvisorFactory.getAdvice(this.aspectJAdviceMethod, pointcut,
				this.aspectInstanceFactory, this.declarationOrder, this.aspectName);
		return (advice != null ? advice : EMPTY_ADVICE);
	}

	public Advice getAdvice(Method candidateAdviceMethod, AspectJExpressionPointcut expressionPointcut,
			MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {
		// 获取方法所在类的class类型
		Class<?> candidateAspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
		validate(candidateAspectClass);
		// 获取方法上的注解
		AspectJAnnotation<?> aspectJAnnotation =
				AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
		if (aspectJAnnotation == null) {
			return null;
		}
		// 逻辑到此处，肯定有AspectJ的方法，再次判断其是否有Aspect注解
		if (!isAspect(candidateAspectClass)) {
			throw new AopConfigException("Advice must be declared inside an aspect type: " +
					"Offending method '" + candidateAdviceMethod + "' in class [" +
					candidateAspectClass.getName() + "]");
		}
		AbstractAspectJAdvice springAdvice;
		// 根据不同的注解类型封装不同的增强器
		switch (aspectJAnnotation.getAnnotationType()) {
			case AtBefore:
				springAdvice = new AspectJMethodBeforeAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				break;
			case AtAfter:
				springAdvice = new AspectJAfterAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				break;
			case AtAfterReturning:
				springAdvice = new AspectJAfterReturningAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				AfterReturning afterReturningAnnotation = (AfterReturning) aspectJAnnotation.getAnnotation();
				if (StringUtils.hasText(afterReturningAnnotation.returning())) {
					springAdvice.setReturningName(afterReturningAnnotation.returning());
				}
				break;
			case AtAfterThrowing:
				springAdvice = new AspectJAfterThrowingAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				AfterThrowing afterThrowingAnnotation = (AfterThrowing) aspectJAnnotation.getAnnotation();
				if (StringUtils.hasText(afterThrowingAnnotation.throwing())) {
					springAdvice.setThrowingName(afterThrowingAnnotation.throwing());
				}
				break;
			case AtAround:
				springAdvice = new AspectJAroundAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
				break;
			case AtPointcut:
				if (logger.isDebugEnabled()) {
					logger.debug("Processing pointcut '" + candidateAdviceMethod.getName() + "'");
				}
				return null;
			default:
				throw new UnsupportedOperationException(
						"Unsupported advice type on method: " + candidateAdviceMethod);
		}

		// Now to configure the advice...
		springAdvice.setAspectName(aspectName);
		springAdvice.setDeclarationOrder(declarationOrder);
		String[] argNames = this.parameterNameDiscoverer.getParameterNames(candidateAdviceMethod);
		if (argNames != null) {
			springAdvice.setArgumentNamesFromStringArray(argNames);
		}
		springAdvice.calculateArgumentBindings();

		return springAdvice;
	}

比如前置通知 ```AtBefore``` 对应的增强器为 ```AspectJMethodBeforeAdvice```，在其中完成了增强方法的逻辑。

方法的调用栈很深，是否还记得我们的目的，前面说到，我们需要的是，获取所有的增强，然后匹配获取符合条件的增强，最后创建代理。前面的内容都是获取所有的增强，我们查找spring的bean，查找有Aspect注解的bean，然后根据其对应的增强注解，将其解析成Advisor的实例，注册在spring的容器中。那么接下来，需要的就是匹配出符合条件的增强。

虽然可能有很多增强器，但不一定适用于当前Bean，必须要满足通配符的增强器才行。
> - 之所以是但当前Bean，是自定义标签解析的时候注册了一个实现了BeanPostProcessor的Bean，所以Spring在创建每个Bean的时候都会调用其 ```postProcessAfterInitialization``` 方法。
> - 满足通配符是指，当前Bean需要满足增强器配置的切点内容：pointcut

跟踪匹配的代码，最终调用的canApply逻辑如下，大致就是根据切点信息解析出来的匹配规则对方法进行匹配，有符合匹配规则的就算当前类需要被代理。

	public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
		Assert.notNull(pc, "Pointcut must not be null");
		if (!pc.getClassFilter().matches(targetClass)) {
			return false;
		}

		MethodMatcher methodMatcher = pc.getMethodMatcher();
		if (methodMatcher == MethodMatcher.TRUE) {
			// No need to iterate the methods if we're matching any method anyway...
			return true;
		}

		IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
		if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
			introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
		}

		Set<Class<?>> classes = new LinkedHashSet<>(ClassUtils.getAllInterfacesForClassAsSet(targetClass));
		classes.add(targetClass);
		for (Class<?> clazz : classes) {
			Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
			for (Method method : methods) {
				if ((introductionAwareMethodMatcher != null &&
						introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions)) ||
						methodMatcher.matches(method, targetClass)) {
					return true;
				}
			}
		}

		return false;
	}

获取了符合条件的增强后，最后就是创建代理了。还记得哪里获取增强然后创建代理的逻辑么。 在 ```AbstractAutoProxyCreator#wrapIfNecessary``` 方法中。 ```createProxy``` 就是创建代理。

	protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
			@Nullable Object[] specificInterceptors, TargetSource targetSource) {

		if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
			AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
		}

		ProxyFactory proxyFactory = new ProxyFactory();
		// 赋值当前类中的属性
		proxyFactory.copyFrom(this);
		// 决定对于给定的Bean是否使用targetClass，而不是他的代理接口
		// 检查proxyTargetClass设置，并设置代理接口
		if (!proxyFactory.isProxyTargetClass()) {
			if (shouldProxyTargetClass(beanClass, beanName)) {
				proxyFactory.setProxyTargetClass(true);
			}
			else {
				evaluateProxyInterfaces(beanClass, proxyFactory);
			}
		}

		// 构建增强器，包括特定的和通用的，全都封装成Advisor类型。
		// 具体逻辑不表，其中会将拦截器也封装成Advisor类型。
		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
		// 将增强器添加到代理工厂上下文
		proxyFactory.addAdvisors(advisors);
		// 设置要代理的类
		proxyFactory.setTargetSource(targetSource);
		// 定制代理，空方法，留给子类扩展
		customizeProxyFactory(proxyFactory);
		// 代理工厂冻结属性设置，用来控制代理工厂在配置后，是否还允许修改。默认false
		proxyFactory.setFrozen(this.freezeProxy);
		if (advisorsPreFiltered()) {
			proxyFactory.setPreFiltered(true);
		}
		// 通过代理工厂创建代理
		return proxyFactory.getProxy(getProxyClassLoader());
	}

以上逻辑大致分为几个步骤：

- 获取当前类中的属性
- 添加代理接口
- 封装Advisor并加入到ProxyFactory
- 设置要代理的类
- 提供可定制的扩展
- 进行代理操作。

接下来就是代理的创建了，查看代理工厂的方法

	public Object getProxy(@Nullable ClassLoader classLoader) {
		return createAopProxy().getProxy(classLoader);
	}

首先是要创建对应的AOP代理对象，前面也有提到，AOP使用的动态代理包括JDK 和 CGLIB，此处就是要选择对应的代理。查看 ```createAopProxy``` 方法

	protected final synchronized AopProxy createAopProxy() {
		if (!this.active) {
			activate();
		}
		return getAopProxyFactory().createAopProxy(this);
	}

	// 创建AOP代理对象的逻辑
	// config对象代表着AOP的配置内容
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
		// 首先有三个属性影响判断。optimize、proxyTargetClass、hasNoUserSuppliedProxyInterfaces
		// 如果属性条件成立，则进一步判断
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
			// 使用cglib代理
			return new ObjenesisCglibAopProxy(config);
		}
		// 否则使用JDK代理
		else {
			return new JdkDynamicAopProxy(config);
		}
	}

上面代理里面，首先的判断涉及到三个方面：

- optimize ： 用来控制通过CGLIB创建的代理是否使用激进的优化策略，目前仅适用于CGLIB代理，但不建议一般用户进行设置。
- proxyTargetClass ： 如果为true，则目标类本身将被代理，而不是目标接口，此时CGLIB代理将被创建
- hasNoUserSuppliedProxyInterfaces ： 是否存在代理接口

以JDK代理为例，看下其逻辑。

如果对JDK动态代理有一定了解的话，可以知道，是通过实现 ```InvocationHandler``` 接口，重写其invoke方法，来实现对目标方法的增强，然后通过这个实现了 ```InvocationHandler``` 接口的增强类来创建代理。AOP这里也是一样。

查看 ```JdkDynamicAopProxy``` 类的层级结构可知，其实现了JDK动态代理所必须的 ```InvocationHandler``` 接口。

	final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable

接着看下其 invoke 方法

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		MethodInvocation invocation;
		Object oldProxy = null;
		boolean setProxyContext = false;

		TargetSource targetSource = this.advised.targetSource;
		Object target = null;

		try {
			// equals方法的处理
			if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
				return equals(args[0]);
			}
			// hashcode方法的处理
			else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
				return hashCode();
			}
			else if (method.getDeclaringClass() == DecoratingProxy.class) {
				return AopProxyUtils.ultimateTargetClass(this.advised);
			}
			else if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&
					method.getDeclaringClass().isAssignableFrom(Advised.class)) {
				return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
			}

			Object retVal;
			// exposeProxy属性可以实现代理类内部方法的增强，不过是内部方法的调用需要通过AOPContent.getCurrentProxy进行调用
			if (this.advised.exposeProxy) {
				oldProxy = AopContext.setCurrentProxy(proxy);
				setProxyContext = true;
			}

			target = targetSource.getTarget();
			Class<?> targetClass = (target != null ? target.getClass() : null);

			// 获取当前方法的拦截器链
			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

			if (chain.isEmpty()) {
				// 如果没有发现有拦截器链，则直接调用连接点方法(目标方法)
				Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
				retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
			}
			else {
				// 将拦截器链封装在 ReflectiveMethodInvocation 对象中
				invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
				// 执行拦截器链，获取返回结果
				retVal = invocation.proceed();
			}

			// 返回结果处理
			Class<?> returnType = method.getReturnType();
			if (retVal != null && retVal == target &&
					returnType != Object.class && returnType.isInstance(proxy) &&
					!RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
				retVal = proxy;
			}
			else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
				throw new AopInvocationException(
						"Null return value from advice does not match primitive return type for: " + method);
			}
			return retVal;
		}
		finally {
			if (target != null && !targetSource.isStatic()) {
				targetSource.releaseTarget(target);
			}
			if (setProxyContext) {
				AopContext.setCurrentProxy(oldProxy);
			}
		}
	}

先不管拦截器链的构建，先看下方法连接器链的调用。

	public Object proceed() throws Throwable {
		//	执行完所有拦截器后执行连接点方法(目标方法)
		if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
			return invokeJoinpoint();
		}
		// 获取下一个待执行的拦截器
		Object interceptorOrInterceptionAdvice =
				this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
		if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
			// 动态匹配
			InterceptorAndDynamicMethodMatcher dm =
					(InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
			if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
				return dm.interceptor.invoke(this);
			}
			// 如果不匹配，则继续调用
			else {
				return proceed();
			}
		}
		// 普通拦截器，比如 MethodBeforeAdviceInterceptor
		else {
			return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
		}
	}

可以看到，实际调用的是拦截器的invoke方法。

这里不深入拦截器链的构建逻辑，但是得说明一下，AOP中拦截器的接口是 ```MethodInterceptor``` 

	public List<Object> getInterceptorsAndDynamicInterceptionAdvice(
			Advised config, Method method, Class<?> targetClass) {
		// 省略其他逻辑，只是说明在拦截器的类型。
		MethodInterceptor[] interceptors = registry.getInterceptors(advisor);
	}

不知是否还记得，前面对于增强器的处理，增强器会封装成 ```InstantiationModelAwarePointcutAdvisorImpl```，并且在实例化的时候，会根据不同的增强注解选择不同的增强器，比如before会对应 ```AspectJMethodBeforeAdvice```，而 ```AspectJMethodBeforeAdvice``` 实现了 ```MethodBeforeAdvice``` 接口。

虽然有这么一个增强器，但是实际是怎样调用的了，这时候就与 ```MethodInterceptor``` 有关了，查看 ```MethodInterceptor``` 的实现类，可以很容易找到 ```MethodBeforeAdviceInterceptor``` 这个拦截器。

	public class MethodBeforeAdviceInterceptor implements MethodInterceptor, Serializable {

		private MethodBeforeAdvice advice;
	
		public MethodBeforeAdviceInterceptor(MethodBeforeAdvice advice) {
			Assert.notNull(advice, "Advice must not be null");
			this.advice = advice;
		}
	
		@Override
		public Object invoke(MethodInvocation mi) throws Throwable {
			this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis() );
			return mi.proceed();
		}
	
	}

可以看到 ```MethodBeforeAdviceInterceptor``` 持有 ```MethodBeforeAdvice``` 实例的引用，并且其invoke方法，会先调用Before增强器的方法，然后调用目标方法。

以上就是Spring AOP的大致内容，有些细节没有深入，自身也没有全部弄懂，但是可以基本看出AOP的实现逻辑（基于注解的）：

- 自定义标签的处理解析器，注册实现了 ```BeanPostProcessor``` 的 ```AnnotationAwareAspectJAutoProxyCreator```
- Bean初始化后，调用 ```AnnotationAwareAspectJAutoProxyCreator``` 的 ```postProcessAfterInitialization``` 方法进行后置处理。
- 后置处理的过程，会获取所有的增强器过滤匹配后，找到与当前Bean对应方法匹配的，进而创建代理，Spring容器中，管理的Bean就是其对应的代理对象。
- 过滤匹配获取适合的增强器时，会根据不同的增强配置定义成不同的增强器，当然，增强器会统一封装
- 创建代理时，会根据不同的配置，选择JDK动态代理或者CGLIB动态代理
- 客户端用spring的Bean进行调用时，调用的就是其对应的代理类，其中会获取对应的拦截器链，遍历匹配调用。
- 拦截器持有上述不同类型增强器的对象，所以代理类调用时就可以在不同的拦截器调用对应的增强器逻辑，实现各种增强。