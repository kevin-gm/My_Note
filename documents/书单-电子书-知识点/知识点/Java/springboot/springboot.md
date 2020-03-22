# springboot源码
pom.xml，这里仅引入 ```spring-boot-starter-web```

	<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>2.1.6.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

	<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

看下这种情况下，应用的实际依赖情况
![](../images/springboot-one.png)

管中窥豹，可以看出一个基于springboot的最简单的springmvc的情况

- spring-web和spring-webmvc，这是mvc应用必不可少的
- spring-boot-starter-tomcat，内置tomcat容器的依赖
- spring-boot-starter，springboot应用所需的基础依赖
- spring-boot-starter-json，springboot应用的json依赖和配置，我们最常用的mvc的messageconverter就是json的
- hibernate-validator，这个实际跟hibernate没有太大关系，如果应用不需要数据库查询，是用不到ORM的，hibernate-validator实际上是对象属性校验的一套标准。

启动应用

	public static void main(String[] args) {
        SpringApplication.run(HelloWorldApplication.class, args);
    }

而run方法实际做了两件事

	public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
		return new SpringApplication(primarySources).run(args);
	}

- 实例化SpringApplication对象
- 执行SpringApplication实例的run方法

看下SpringApplication对象的实例化过程

	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}

- 判断当前应用的类型
	- 如果classpath下 仅 有，org.springframework.web.reactive.DispatcherHandler，则应用类型为响应式的web应用
	- 如果classpath下有 javax.servlet.Servlet 以及org.springframework.web.context.ConfigurableWebApplicationContext，则应用类型为普通的servlet web应用
	- 前面都不是，则是非web应用
- 从 spring.factories 文件中获取所有的 ApplicationContextInitializer 实现，并实例化
- 从 spring.factories 文件中获取所有的 ApplicationListener 实现，并实例化
- 判断main方法所在的类，判断的方式是实例化一个RuntimeException 对象，遍历异常堆栈，找到main方法所在的类


## org.springframework.context.ApplicationContextInitializer
- spring-boot.jar
	- org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer
	- org.springframework.boot.context.ContextIdApplicationContextInitializer
	- org.springframework.boot.context.config.DelegatingApplicationContextInitializer
	- org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer
- spring-boot-aotuconfiure.jar
	- org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer
	- org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

## ApplicationListener
- spring-boot.jar
	- org.springframework.boot.ClearCachesApplicationListener
	- org.springframework.boot.builder.ParentContextCloserApplicationListener
	- org.springframework.boot.context.FileEncodingApplicationListener
	- org.springframework.boot.context.config.AnsiOutputApplicationListener
	- org.springframework.boot.context.config.ConfigFileApplicationListener
	- org.springframework.boot.context.config.DelegatingApplicationListener
	- org.springframework.boot.context.logging.ClasspathLoggingApplicationListener
	- org.springframework.boot.context.logging.LoggingApplicationListener,
	- org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
- spring-boot-aotuconfigure.jar
	- org.springframework.boot.autoconfigure.BackgroundPreinitializer


## 正式启动SpringBoot

	public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}

### SpringApplicationRunListener
- spring-boot.jar
	- org.springframework.boot.context.event.EventPublishingRunListener

### 获取应用当前的执行监听器
getRunListeners(args) 实际是从 spring.factories 文件中获取所有的 SpringApplicationRunListener 实现，并实例化。它的作用是监听Spring应用运行过程，并针对某些事件进行通知。

这里实际上默认情况下，只会有一个监听器 EventPublishingRunListener

SpringApplication 类中的 getRunListeners(args) 方法

	private SpringApplicationRunListeners getRunListeners(String[] args) {
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
		return new SpringApplicationRunListeners(logger,
				getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
	}

EventPublishingRunListener 的构造方法

	public EventPublishingRunListener(SpringApplication application, String[] args) {
		this.application = application;
		this.args = args;
		this.initialMulticaster = new SimpleApplicationEventMulticaster();
		for (ApplicationListener<?> listener : application.getListeners()) {
			this.initialMulticaster.addApplicationListener(listener);
		}
	}

结合上面两个方法，EventPublishingRunListener 实例化的时候，传入了 SpringApplication 对象，所以才能通过 application.getListeners() 获取到 SpringApplication 中的所有监听器。

而 SpringApplication 中的监听器，则是在 SpringApplication 对象实例化的时候，从 spring.factories 文件中获取到的 ApplicationListener 实现。

也就是说，可以认为 EventPublishingRunListener 实际上是 ApplicationListener 调用的一个门面类，它抽象了 springboot 应用在执行过程中的一些操作，最后实际调用 ApplicationListener 来完成这些操作。

我们还应该看到，在 EventPublishingRunListener 实例化时，还实例化了 SimpleApplicationEventMulticaster 对象，它是spring的默认事件广播器，是一个典型的观察者模式的实现，这里我们不深究其原理，需要知道的是，EventPublishingRunListener 抽象的方法，最终会通过 SimpleApplicationEventMulticaster 这个事件广播器，调用 ApplicationListener ，完成相应事件的执行，做好相关准备。

### 执行spring容器启动的监听事件
这个步骤也就是 listeners.starting(); 对应的实现

	public void starting() {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.starting();
		}
	}

前面说到，这里的 SpringApplicationRunListener 实际上只有一个实例，那就是 EventPublishingRunListener

	public void starting() {
		this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
	}

可以看到，这里与前面说的一致，通过 SimpleApplicationEventMulticaster 广播器广播事件，而这里的事件是 ApplicationStartingEvent，也就是应用启动事件，见名知意，它是应用开始启动的事件。

### starting执行的监听方法
- org.springframework.boot.context.logging.LoggingApplicationListener
- org.springframework.boot.autoconfigure.BackgroundPreinitializer
- org.springframework.boot.context.config.DelegatingApplicationListener
- org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener

#### org.springframework.boot.context.logging.LoggingApplicationListener
应用日志系统配置，如果应用上下文包含 ```logging.config``` 配置，则使用该配置作为日志打印的配置，否则使用默认的。

日志打印级别，可以通过 ```logging.level.*``` 进行自定义， 也可以通过 ```logging.group``` 配置日志组

Spring, Tomcat, Jetty 以及 Hibernate 框架调试时一般需要 debug或者trace 的日志级别，这些当应用环境变量中包含 debug或者trace 配置时，就会被启用，比如 ```java -jar myapp.jar [--debug | --trace]``` .

默认情况下，日志仅打印在控制台，如果要输出到文件，可以配置 ```logging.path``` 和 ```logging.file```

如果日志配置支持占位符(比如log4j,logback)，则可以使用系统属性进行配置。比如 LOG_FILE  和  PID

	static {
		Map<String, String> systems = new LinkedHashMap<>();
		systems.put("ch.qos.logback.core.Appender", "org.springframework.boot.logging.logback.LogbackLoggingSystem");
		systems.put("org.apache.logging.log4j.core.impl.Log4jContextFactory",
				"org.springframework.boot.logging.log4j2.Log4J2LoggingSystem");
		systems.put("java.util.logging.LogManager", "org.springframework.boot.logging.java.JavaLoggingSystem");
		SYSTEMS = Collections.unmodifiableMap(systems);
	}

SpringBoot默认支持三种日志配置，只要在应用类路径下由对应的类就会实例化该日志系统，当前类路径下找到的第一个就是当前的日志系统，如果没有，则抛出异常。

最后需要调用日志系统初始化之前的操作。

	private void onApplicationStartingEvent(ApplicationStartingEvent event) {
		this.loggingSystem = LoggingSystem.get(event.getSpringApplication().getClassLoader());
		this.loggingSystem.beforeInitialize();
	}

#### org.springframework.boot.autoconfigure.BackgroundPreinitializer
在后台通过异步多线程执行一些预初始化操作，可以通过 ```spring.backgroundpreinitializer.ignore``` 系统属性配置禁用这项功能，使其在前台执行。

	Thread thread = new Thread(new Runnable() {

				@Override
				public void run() {
					runSafely(new ConversionServiceInitializer());
					runSafely(new ValidationInitializer());
					runSafely(new MessageConverterInitializer());
					runSafely(new MBeanFactoryInitializer());
					runSafely(new JacksonInitializer());
					runSafely(new CharsetInitializer());
					preinitializationComplete.countDown();
				}

				public void runSafely(Runnable runnable) {
					try {
						runnable.run();
					}
					catch (Throwable ex) {
						// Ignore
					}
				}

			}, "background-preinit");
			thread.start();

- 实例化默认的ConversionService：DefaultFormattingConversionService
- 初始化javax.validation
- 初始化http的消息转换器：AllEncompassingFormHttpMessageConverter
- 实例化用于JMX监控的MBeanFactory
- 实例化jackson
- 实例化系统默认的编码格式：UTF-8


#### org.springframework.boot.context.config.DelegatingApplicationListener
用于代理其他非SpringBoot默认事件的监听，可以通过 ```context.listener.classes``` 环境属性指定这些额外的监听器，最终也是通过默认的广播器 ```SimpleApplicationEventMulticaster``` 进行事件传播。

这个监听器实际能识别的事件类型为 ```ApplicationEnvironmentPreparedEvent``` , 所以启动事件在这里实际并不会产生影响。

#### org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
如果应用集成了Liquibase，则使用于SpringBoot版本兼容的ServiceLocator替换到默认的

### 解析命令行参数
命令行参数解析完后，分为两类：选项参数(option arguments) 以及 非选项参数(non-option arguments)

- 选项参数是指，按照 ```--optName[=optValue]``` 语法格式指定的参数。如果指定了 optValue ，则 name和value之前仅能由=号，不能由其他任何空白字符以及其他字符。
- 非选项参数：任何其他没有按照 选项参数 给格式配置的参数都是非选项参数。

命令行参数解析逻辑如下：

	public CommandLineArgs parse(String... args) {
		CommandLineArgs commandLineArgs = new CommandLineArgs();
		for (String arg : args) {
			if (arg.startsWith("--")) {
				String optionText = arg.substring(2, arg.length());
				String optionName;
				String optionValue = null;
				if (optionText.contains("=")) {
					optionName = optionText.substring(0, optionText.indexOf('='));
					optionValue = optionText.substring(optionText.indexOf('=')+1, optionText.length());
				}
				else {
					optionName = optionText;
				}
				if (optionName.isEmpty() || (optionValue != null && optionValue.isEmpty())) {
					throw new IllegalArgumentException("Invalid argument syntax: " + arg);
				}
				commandLineArgs.addOptionArg(optionName, optionValue);
			}
			else {
				commandLineArgs.addNonOptionArg(arg);
			}
		}
		return commandLineArgs;
	}

最后需要将解析结果 以 commandLineArgs 为键

### 准备环境
spring上下文的环境属性包括，命令行参数，激活状态的profile配置，系统属性，jvm属性等。

松散绑定？

#### 获取或新建环境示例

	private ConfigurableEnvironment getOrCreateEnvironment() {
		if (this.environment != null) {
			return this.environment;
		}
		switch (this.webApplicationType) {
		case SERVLET:
			return new StandardServletEnvironment();
		case REACTIVE:
			return new StandardReactiveWebEnvironment();
		default:
			return new StandardEnvironment();
		}
	}

由于，我们这里时servlet环境，所以实例化 ```StandardServletEnvironment``` 对象。后续所有与环境变量有关的操作都基于这个对象。

查看 ```StandardServletEnvironment``` 类继承结构可知，实例化StandardServletEnvironment时，会调用其祖父类 ```AbstractEnvironment``` 的无参构造方法，然后通过模板模式回过头来执行 StandardServletEnvironment 的 customizePropertySources()

	protected void customizePropertySources(MutablePropertySources propertySources) {
		propertySources.addLast(new StubPropertySource(SERVLET_CONFIG_PROPERTY_SOURCE_NAME));
		propertySources.addLast(new StubPropertySource(SERVLET_CONTEXT_PROPERTY_SOURCE_NAME));
		if (JndiLocatorDelegate.isDefaultJndiEnvironmentAvailable()) {
			propertySources.addLast(new JndiPropertySource(JNDI_PROPERTY_SOURCE_NAME));
		}
		super.customizePropertySources(propertySources);
	}

这个方法，就是添加应用默认支持的属性配置来源

- servletConfigInitParams
- servletContextInitParams
- jndiProperties，默认环境不需要JNDI，所以实际情况下，这个可能没有加载
- systemProperties
- systemEnvironment

其中 servletConfigInitParams 和 servletContextInitParams 用到的属性源实现为 StubPropertySource ，stub可以翻译为 桩，用于后续相关环境准备好后，替换掉本来的占位配置。比如，基于 ServletContext 的属性，必须要等 ApplicationContext 中 ServletContext 创建好才能获取到。

这里如果一个配置存在于多个属性源，则上述的顺序即为属性配置的优先顺序。Apollo这种配置中心之所以优先级高，也是因为将其属性源放到了列表的前面。

还需要注意的是，在 ```StandardServletEnvironment``` 的父类```AbstractEnvironment``` 中，定义了应用的默认 profile 为 **default**

##### systemProperties
去掉多余的异常处理代码后，可以看到，systemProperties 对应的就是 JVM 的系统属性

	public Map<String, Object> getSystemProperties() {
		return (Map) System.getProperties();
	}

##### systemEnvironment
同样去掉多余的代码，systemEnvironment 对应的时与操作系统有关的环境变量，这里对于是否能获取到操作系统的环境变量配置，有一个判断条件，即 系统配置 spring.getenv.ignore 如果为true，则操作系统相关的变量是不会返回的，该值默认为false

	public Map<String, Object> getSystemEnvironment() {
		if (suppressGetenvAccess()) {
			return Collections.emptyMap();
		}
		return (Map) System.getenv();
	}

##### 合并spring父子容器属性
可以看到，这里不仅会合并父子容器的属性，也会合并父子容器的profile

	public void merge(ConfigurableEnvironment parent) {
		for (PropertySource<?> ps : parent.getPropertySources()) {
			if (!this.propertySources.contains(ps.getName())) {
				this.propertySources.addLast(ps);
			}
		}
		String[] parentActiveProfiles = parent.getActiveProfiles();
		if (!ObjectUtils.isEmpty(parentActiveProfiles)) {
			synchronized (this.activeProfiles) {
				for (String profile : parentActiveProfiles) {
					this.activeProfiles.add(profile);
				}
			}
		}
		String[] parentDefaultProfiles = parent.getDefaultProfiles();
		if (!ObjectUtils.isEmpty(parentDefaultProfiles)) {
			synchronized (this.defaultProfiles) {
				this.defaultProfiles.remove(RESERVED_DEFAULT_PROFILE_NAME);
				for (String profile : parentDefaultProfiles) {
					this.defaultProfiles.add(profile);
				}
			}
		}
	}

还需要注意的是，在 ```AbstractEnvironment``` 中对于变量的解析用到了 ```PropertySourcesPropertyResolver```

	private final ConfigurablePropertyResolver propertyResolver =
			new PropertySourcesPropertyResolver(this.propertySources);

其他关于profile的相关解析后面用到了再讨论。

### 配置环境
前面，默认的servlet web环境下，配置了5个属性配置源，接下来看看具体的配置过程

如下方法，入参中，ConfigurableEnvironment 是前面实例化的环境示例，args 是启动参数

	protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
		if (this.addConversionService) {
			ConversionService conversionService = ApplicationConversionService.getSharedInstance();
			environment.setConversionService((ConfigurableConversionService) conversionService);
		}
		configurePropertySources(environment, args);
		configureProfiles(environment, args);
	}

- 配置 ConversionService。前面提到 environment 中，属性解析用到了 ```PropertySourcesPropertyResolver``` ，而这个对象有一个 ConversionService 属性用于处理对象转换，比如，配置的是string，但是Bean实际上Integer。这里添加的实例是 ApplicationConversionService
- 配置前面加载的属性原
- 配置profile相关

#### 配置属性 configurePropertySources

	protected void configurePropertySources(ConfigurableEnvironment environment, String[] args) {
		// 此时的 sources 包含的就是创建 environment 对象是 加载的4个属性源
		MutablePropertySources sources = environment.getPropertySources();
		// 如果defaultProperties不为空，则将其添加到属性源
		if (this.defaultProperties != null && !this.defaultProperties.isEmpty()) {
			sources.addLast(new MapPropertySource("defaultProperties", this.defaultProperties));
		}
		// 如果允许添加命令行参数，并且当前传入由命令行参数，则解析并添加到属性源
		if (this.addCommandLineProperties && args.length > 0) {
			String name = CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME;
			if (sources.contains(name)) {
				PropertySource<?> source = sources.get(name);
				CompositePropertySource composite = new CompositePropertySource(name);
				composite.addPropertySource(
						new SimpleCommandLinePropertySource("springApplicationCommandLineArgs", args));
				composite.addPropertySource(source);
				sources.replace(name, composite);
			}
			else {
				sources.addFirst(new SimpleCommandLinePropertySource(args));
			}
		}
	}

对于 defaultProperties ，默认情况下，这里是 null，如果程序需要自定义一些默认的配置，可以自行实例化 SpringApplication 对象，并设置这个属性。

如果不希望程序加载命令行参数，可以设置 addCommandLineProperties 为false。

**注意：**命令行参数添加到属性源列表是，调用的是 addFirst() 方法，所以，命令行参数优先级很高。对于程序当前执行进度来说，是最高的。

这里有点每太理解的是，命令行参数解析了两次，而不是解析一次然后传递解析结果。

#### 配置profile属性 configureProfiles

	protected void configureProfiles(ConfigurableEnvironment environment, String[] args) {
		// 这行代码看起来是多余的，但实际是起到一个初始化作用
		environment.getActiveProfiles();
		// 可以自定义设置额外的profile
		Set<String> profiles = new LinkedHashSet<>(this.additionalProfiles);
		profiles.addAll(Arrays.asList(environment.getActiveProfiles()));
		// 默认情况下，如果没有配置profile，则这里此时激活状态的为空
		environment.setActiveProfiles(StringUtils.toStringArray(profiles));
	}

首先，从当前环境中获取激活的profile，最终调用的是 AbstractEnvironment 的方法。

属性名称为 spring.profiles.active，可以在前面属性源中任意一种配置这个属性，当然，通常是在启动是添加JVM参数，比如 ```java -jar -Dspring.profiles.active=dev HelloWorld.jar```，当没有配置时，其结果为null，这里返回的也是一个大小为0的集合。

	protected Set<String> doGetActiveProfiles() {
		synchronized (this.activeProfiles) {
			if (this.activeProfiles.isEmpty()) {
				String profiles = getProperty(ACTIVE_PROFILES_PROPERTY_NAME);
				if (StringUtils.hasText(profiles)) {
					setActiveProfiles(StringUtils.commaDelimitedListToStringArray(
							StringUtils.trimAllWhitespace(profiles)));
				}
			}
			return this.activeProfiles;
		}
	}

### 广播环境准备好了的事件
广播的事件为：```ApplicationEnvironmentPreparedEvent```

这里涉及到的监听器有：

- ConfigFileApplicationListener

#### ConfigFileApplicationListener的环境准备事件
- 从 spring.factories 中获取 ```EnvironmentPostProcessor``` 实现，并实例化
	- spring-boot.jar
		- org.springframework.boot.cloud.CloudFoundryVcapEnvironmentPostProcessor
		- org.springframework.boot.env.SpringApplicationJsonEnvironmentPostProcessor
		- org.springframework.boot.env.SystemEnvironmentPropertySourceEnvironmentPostProcessor
- 当前实例也是一个环境后置处理器，添加到后置处理器列表
- 对后置处理器进行排序
- 遍历执行后置处理器

代码如下：

	private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
		List<EnvironmentPostProcessor> postProcessors = loadPostProcessors();
		postProcessors.add(this);
		AnnotationAwareOrderComparator.sort(postProcessors);
		for (EnvironmentPostProcessor postProcessor : postProcessors) {
			postProcessor.postProcessEnvironment(event.getEnvironment(), event.getSpringApplication());
		}
	}

##### SystemEnvironmentPropertySourceEnvironmentPostProcessor
这个处理器时对系统属性 systemEnvironment 的属性持有对象进行替换，由 SystemEnvironmentPropertySource 替换为 OriginAwareSystemEnvironmentPropertySource，替换后能跟踪属性的变换

	public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
		String sourceName = StandardEnvironment.SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME;
		PropertySource<?> propertySource = environment.getPropertySources().get(sourceName);
		if (propertySource != null) {
			replacePropertySource(environment, sourceName, propertySource);
		}
	}

	@SuppressWarnings("unchecked")
	private void replacePropertySource(ConfigurableEnvironment environment, String sourceName,
			PropertySource<?> propertySource) {
		Map<String, Object> originalSource = (Map<String, Object>) propertySource.getSource();
		SystemEnvironmentPropertySource source = new OriginAwareSystemEnvironmentPropertySource(sourceName,
				originalSource);
		environment.getPropertySources().replace(sourceName, source);
	}

##### SpringApplicationJsonEnvironmentPostProcessor
这个处理器实现的是，解析 由属性Key为 ```spring.application.json``` 或者 ```SPRING_APPLICATION_JSON``` 使用JSON格式配置的属性，并添加到当前环境中，添加到上下文环境时，其优先级高于 systemProperties

	public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
		// 此时的propertySources包含5个
		MutablePropertySources propertySources = environment.getPropertySources();
		propertySources.stream().map(JsonPropertyValue::get).filter(Objects::nonNull).findFirst()
				.ifPresent((v) -> processJson(environment, v));
	}

	private void processJson(ConfigurableEnvironment environment, JsonPropertyValue propertyValue) {
		// 根据上下文JSON依赖情况，实例化JSON解析器
		// 根据引入的依赖情况，可选的由 jackson，gson，Snake YAML，以及BasicJsonParser
		JsonParser parser = JsonParserFactory.getJsonParser();
		Map<String, Object> map = parser.parseMap(propertyValue.getJson());
		if (!map.isEmpty()) {
			addJsonPropertySource(environment, new JsonPropertySource(propertyValue, flatten(map)));
		}
	}

	// 可以看到，这里的解析结果如果添加到环境上下文，优先级比 systemProperties高
	private void addJsonPropertySource(ConfigurableEnvironment environment, PropertySource<?> source) {
		MutablePropertySources sources = environment.getPropertySources();
		// 这里查找的是 systemProperties 或者 jndiProperties
		String name = findPropertySource(sources);
		if (sources.contains(name)) {
			sources.addBefore(name, source);
		}
		else {
			sources.addFirst(source);
		}
	}

第一步获取的当前propertySources列表，此时包含5项内容：命令行参数(commandLineAgrs), servletConfigParams, servletContextParams, systemProperties, systemEnviroment(此时的对象已经过 ```SystemEnvironmentPropertySourceEnvironmentPostProcessor``` 的替换。)

第二步中，lambda表达式的含义是：遍历propertySources，将 PropertySource 转换为 JsonPropertyValue 对象，对于转换结果不为空的第一个，进行JSON解析。

也就是说，从已有的5个属性源中，查找 key 为 ```spring.application.json``` 或者 ```SPRING_APPLICATION_JSON``` 的json配置，如果有，则只算找到的第一个，对其进行json解析。

	public static JsonPropertyValue get(PropertySource<?> propertySource) {
			for (String candidate : CANDIDATES) {
				Object value = propertySource.getProperty(candidate);
				if (value != null && value instanceof String && StringUtils.hasLength((String) value)) {
					return new JsonPropertyValue(propertySource, candidate, (String) value);
				}
			}
			return null;
		}

##### CloudFoundryVcapEnvironmentPostProcessor
这个处理器是对 云平台 的支持，需要配置属性 VCAP_APPLICATION 或者 VCAP_SERVICES

从源码来看，是从配置中获取 VCAP_APPLICATION 和 VCAP_SERVICES 的属性值，进行json解析，然后添加到环境上下文。

这里的优先级为 仅比 commandLineArgs 低

后面有时间再具体看实现细节。因为默认情况下，是不会解析这个的。

	public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
		if (CloudPlatform.CLOUD_FOUNDRY.isActive(environment)) {
			Properties properties = new Properties();
			JsonParser jsonParser = JsonParserFactory.getJsonParser();
			addWithPrefix(properties, getPropertiesFromApplication(environment, jsonParser), "vcap.application.");
			addWithPrefix(properties, getPropertiesFromServices(environment, jsonParser), "vcap.services.");
			MutablePropertySources propertySources = environment.getPropertySources();
			if (propertySources.contains(CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME)) {
				propertySources.addAfter(CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME,
						new PropertiesPropertySource("vcap", properties));
			}
			else {
				propertySources.addFirst(new PropertiesPropertySource("vcap", properties));
			}
		}
	}

##### ConfigFileApplicationListener
这个处理器由两个作用，一是生成随机值的属性源。属性名以 ```random.``` 开头的；二是根据profiles解析应用的配置文件

添加的 PropertySource 实现为 ```RandomValuePropertySource```，内置 Random 对象用于获取随机数

可以看到，随机属性的优先级为再 systemEnvironment 之后，优先级比较低

	protected void addPropertySources(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
		RandomValuePropertySource.addToEnvironment(environment);
		new Loader(environment, resourceLoader).load();
	}

	public static void addToEnvironment(ConfigurableEnvironment environment) {
		environment.getPropertySources().addAfter(StandardEnvironment.SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME,
				new RandomValuePropertySource(RANDOM_PROPERTY_SOURCE_NAME));
		logger.trace("RandomValuePropertySource add to Environment");
	}

**PS:** 到此刻，environment中已有的属性源有6个，按优先顺序为：

- commandLineArgs
- serveltConfigInitParams
- servletContextInitParams
- systemProperties
- systemEnvironment
- random


这个逻辑里面第二个步骤是实例化一个加载器去加载应用的属性配置： ```new Loader(environment, resourceLoader).load()```

	Loader(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
			this.environment = environment;
			this.placeholdersResolver = new PropertySourcesPlaceholdersResolver(this.environment);
			this.resourceLoader = (resourceLoader != null) ? resourceLoader : new DefaultResourceLoader();
			this.propertySourceLoaders = SpringFactoriesLoader.loadFactories(PropertySourceLoader.class,
					getClass().getClassLoader());
		}

- 实例化一个属性解析器
- 获取资源类加载器
- 从 spring.factories 文件中 获取并实例化 所有 PropertySourceLoader 的实现。

**获取类加载器**
首先获取当前线程的上下文类加载器，如果没有，则获取当前类的类加载器

	public static ClassLoader getDefaultClassLoader() {
		ClassLoader cl = null;
		try {
			cl = Thread.currentThread().getContextClassLoader();
		}
		catch (Throwable ex) {
		}
		if (cl == null) {
			cl = ClassUtils.class.getClassLoader();
			if (cl == null) {
				try {
					cl = ClassLoader.getSystemClassLoader();
				}
				catch (Throwable ex) {
				}
			}
		}
		return cl;
	}

**获取PropertySourceLoader实现**

- spring-boot.jar
	- org.springframework.boot.env.PropertiesPropertySourceLoader
	- org.springframework.boot.env.YamlPropertySourceLoader

所以，我们的SpringBoot应用，既能解析yml格式的配置，也能解析properties格式的配置

#### 加载应用配置
加载应用配置，即前面 ```new Loader(environment, resourceLoader).load()``` 中的 load() 方法。

	public void load() {
			this.profiles = new LinkedList<>();
			this.processedProfiles = new LinkedList<>();
			this.activatedProfiles = false;
			this.loaded = new LinkedHashMap<>();
			initializeProfiles();
			while (!this.profiles.isEmpty()) {
				Profile profile = this.profiles.poll();
				if (profile != null && !profile.isDefaultProfile()) {
					addProfileToEnvironment(profile.getName());
				}
				load(profile, this::getPositiveProfileFilter, addToLoaded(MutablePropertySources::addLast, false));
				this.processedProfiles.add(profile);
			}
			resetEnvironmentProfiles(this.processedProfiles);
			load(null, this::getNegativeProfileFilter, addToLoaded(MutablePropertySources::addFirst, true));
			addLoadedPropertySources();
		}

#### 初始化profiles

	private void initializeProfiles() {
			// 用null代表默认的profile
			this.profiles.add(null);
			Set<Profile> activatedViaProperty = getProfilesActivatedViaProperty();
			this.profiles.addAll(getOtherActiveProfiles(activatedViaProperty));
			// Any pre-existing active profiles set via property sources (e.g.
			// System properties) take precedence over those added in config files.
			addActiveProfiles(activatedViaProperty);
			if (this.profiles.size() == 1) { // only has null profile
				for (String defaultProfileName : this.environment.getDefaultProfiles()) {
					Profile defaultProfile = new Profile(defaultProfileName, true);
					this.profiles.add(defaultProfile);
				}
			}
		}

profiles 是通过一个双向队列维护的，第一步先向队列添加了一个null，用于代表默认的profile

然后，从当前环境获取 ```spring.profiles.active``` 以及 ```spring.profiles.include```  所指定的应用激活的profile

**Notice:** 此时，尚未加载配置文件，所以该配置只能是环境上下文已有的6个属性源中配置的，比如通过命令行参数指定。

	private Set<Profile> getProfilesActivatedViaProperty() {
			if (!this.environment.containsProperty(ACTIVE_PROFILES_PROPERTY)
					&& !this.environment.containsProperty(INCLUDE_PROFILES_PROPERTY)) {
				return Collections.emptySet();
			}
			Binder binder = Binder.get(this.environment);
			Set<Profile> activeProfiles = new LinkedHashSet<>();
			activeProfiles.addAll(getProfiles(binder, INCLUDE_PROFILES_PROPERTY));
			activeProfiles.addAll(getProfiles(binder, ACTIVE_PROFILES_PROPERTY));
			return activeProfiles;
		}

查看 Binder 的静态实例化可知，该对象包含了环境上下文已有的属性源配置，以及属性解析器对象，所以可以获取并解析属性配置。

TODO：松散绑定还需花大力气研读

	public static Binder get(Environment environment) {
		return new Binder(ConfigurationPropertySources.get(environment),
				new PropertySourcesPlaceholdersResolver(environment));
	}

接着，从环境上下文获取其他激活状态的profile，过滤掉前面激活的， 添加到对象的 profiles 队列。

这里获取的profile实际也还是取的 ```spring.profiles.active``` 配置，没明白这样做的目的。仅仅是environment是可以自行配置，configfile只能读取写死的？

**Notice:** 此时，尚未加载配置文件，所以该配置只能是环境上下文已有的6个属性源中配置的，比如通过命令行参数指定。

	private List<Profile> getOtherActiveProfiles(Set<Profile> activatedViaProperty) {
			return Arrays.stream(this.environment.getActiveProfiles()).map(Profile::new)
					.filter((profile) -> !activatedViaProperty.contains(profile)).collect(Collectors.toList());
		}

激活profile，这里默认情况下，没有指定激活的profile，所以直接返回了

	void addActiveProfiles(Set<Profile> profiles) {
			if (profiles.isEmpty()) {
				return;
			}
			if (this.activatedProfiles) {
				if (this.logger.isDebugEnabled()) {
					this.logger.debug("Profiles already activated, '" + profiles + "' will not be applied");
				}
				return;
			}
			this.profiles.addAll(profiles);
			if (this.logger.isDebugEnabled()) {
				this.logger.debug("Activated activeProfiles " + StringUtils.collectionToCommaDelimitedString(profiles));
			}
			this.activatedProfiles = true;
			removeUnprocessedDefaultProfiles();
		}

如果最后，队列 profiles 大小为1，代表只有默认的profile，则从环境上下文(environemnt)获取默认的profile name，实例化并添加到profiles列表。



#### 加载激活状态profile的配置

	while (!this.profiles.isEmpty()) {
				Profile profile = this.profiles.poll();
				if (profile != null && !profile.isDefaultProfile()) {
					addProfileToEnvironment(profile.getName());
				}
				load(profile, this::getPositiveProfileFilter, addToLoaded(MutablePropertySources::addLast, false));
				this.processedProfiles.add(profile);
			}

如果profile不为空，并且不是默认的profile，也就是应用自行指定了profile，则需要将定义的profile添加到环境上下文(environment)中去。

#### 获取需要加载的配置文件

	private void load(Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
			getSearchLocations().forEach((location) -> {
				boolean isFolder = location.endsWith("/");
				Set<String> names = isFolder ? getSearchNames() : NO_SEARCH_NAMES;
				names.forEach((name) -> load(location, name, profile, filterFactory, consumer));
			});
		}

看看 ```getSearchLocations``` 方法，用于获取配置文件的路径，可以看到，应用可以通过 ```spring.config.location``` 自行定义配置文件所在的位置，如果没有额外定义，则默认将从如下四个路径去查找配置文件 **classpath:/,classpath:/config/,file:./,file:./config/**

**Notice:** 此时，尚未加载配置文件，所以该配置只能是环境上下文已有的6个属性源中配置的，比如通过命令行参数指定。

	private Set<String> getSearchLocations() {
			if (this.environment.containsProperty(CONFIG_LOCATION_PROPERTY)) {
				return getSearchLocations(CONFIG_LOCATION_PROPERTY);
			}
			Set<String> locations = getSearchLocations(CONFIG_ADDITIONAL_LOCATION_PROPERTY);
			locations.addAll(
					asResolvedSet(ConfigFileApplicationListener.this.searchLocations, DEFAULT_SEARCH_LOCATIONS));
			return locations;
		}

找到配置文件所在路径后，接下来就是获取需要加载的配置文件。 ```getSearchNames``` 就是获取配置文件名称的逻辑，可以看到，这里同样可以通过 ```spring.config.name``` 自定义配置文件的名称，如果没有配置，则默认的名称为：**application**

**Notice:** 此时，尚未加载配置文件，所以该配置只能是环境上下文已有的6个属性源中配置的，比如通过命令行参数指定。

	private Set<String> getSearchNames() {
			if (this.environment.containsProperty(CONFIG_NAME_PROPERTY)) {
				String property = this.environment.getProperty(CONFIG_NAME_PROPERTY);
				return asResolvedSet(property, null);
			}
			return asResolvedSet(ConfigFileApplicationListener.this.names, DEFAULT_NAMES);
		}

有了文件路径和文件名称，接下来就是去这些路径下加载这些名称的配置文件。如下面代码逻辑，它是遍历每个路径以及每个文件，通过指定的属性加载器进行加载

这个所谓的属性加载器在实例化 Loader 对象时，从 spring.factories 中获取，默认加载的有 ```PropertiesPropertySourceLoader``` 和 ```YamlPropertySourceLoader```，所以，应用的配置文件可以是 .yml 或者 .properties 都能识别。

	private void load(String location, String name, Profile profile, DocumentFilterFactory filterFactory,
				DocumentConsumer consumer) {
			if (!StringUtils.hasText(name)) {
				for (PropertySourceLoader loader : this.propertySourceLoaders) {
					if (canLoadFileExtension(loader, location)) {
						load(loader, location, profile, filterFactory.getDocumentFilter(profile), consumer);
						return;
					}
				}
			}
			Set<String> processed = new HashSet<>();
			for (PropertySourceLoader loader : this.propertySourceLoaders) {
				for (String fileExtension : loader.getFileExtensions()) {
					if (processed.add(fileExtension)) {
						loadForFileExtension(loader, location + name, "." + fileExtension, profile, filterFactory,
								consumer);
					}
				}
			}
		}

看到这里，实际上还没有将 profile 于 配置文件 关联起来，他们关联的逻辑如下，**prefix + "-" + profile + fileExtension** 的结合产物，在实际使用中是比如 ```application-dev.yml```

	private void loadForFileExtension(PropertySourceLoader loader, String prefix, String fileExtension,
				Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
			DocumentFilter defaultFilter = filterFactory.getDocumentFilter(null);
			DocumentFilter profileFilter = filterFactory.getDocumentFilter(profile);
			if (profile != null) {
				// Try profile-specific file & profile section in profile file (gh-340)
				String profileSpecificFile = prefix + "-" + profile + fileExtension;
				load(loader, profileSpecificFile, profile, defaultFilter, consumer);
				load(loader, profileSpecificFile, profile, profileFilter, consumer);
				// Try profile specific sections in files we've already processed
				for (Profile processedProfile : this.processedProfiles) {
					if (processedProfile != null) {
						String previouslyLoaded = prefix + "-" + processedProfile + fileExtension;
						load(loader, previouslyLoaded, profile, profileFilter, consumer);
					}
				}
			}
			// Also try the profile-specific section (if any) of the normal file
			load(loader, prefix + fileExtension, profile, profileFilter, consumer);
		}

从配置文件加载时，对于配置配置的 ```spring.profiles.active``` 以及 ```spring.profiles.include``` ，会添加到当前实例的 profiles 中。


#### 重置环境上下文的激活状态的profile

	private void resetEnvironmentProfiles(List<Profile> processedProfiles) {
			String[] names = processedProfiles.stream()
					.filter((profile) -> profile != null && !profile.isDefaultProfile()).map(Profile::getName)
					.toArray(String[]::new);
			this.environment.setActiveProfiles(names);
		}

#### 将加载的应用配置添加到环境上下文
应用配置的属性，优先级很低。

这里之后，environment已有7个属性源，其中这里添加的应用配置，优先级最低。

	private void addLoadedPropertySources() {
			MutablePropertySources destination = this.environment.getPropertySources();
			List<MutablePropertySources> loaded = new ArrayList<>(this.loaded.values());
			Collections.reverse(loaded);
			String lastAdded = null;
			Set<String> added = new HashSet<>();
			for (MutablePropertySources sources : loaded) {
				for (PropertySource<?> source : sources) {
					if (added.add(source.getName())) {
						addLoadedPropertySource(destination, lastAdded, source);
						lastAdded = source.getName();
					}
				}
			}
		}

	private void addLoadedPropertySource(MutablePropertySources destination, String lastAdded,
				PropertySource<?> source) {
			if (lastAdded == null) {
				if (destination.contains(DEFAULT_PROPERTIES)) {
					destination.addBefore(DEFAULT_PROPERTIES, source);
				}
				else {
					destination.addLast(source);
				}
			}
			else {
				destination.addAfter(lastAdded, source);
			}
		}


### 配置需要忽略的Bean
spring.beaninfo.ignore 环境属性配置

### 打印banner

### 创建应用上下文
根据当前应用的类型，创建对应的应用上下文。我们这里是web应用，所以创建的是  ```org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext```, 这里只实例化了对象

是否还记得使用编程方式创建spring容器时  new classpathxmlapplicationcontext(""); 这里等价

	protected ConfigurableApplicationContext createApplicationContext() {
		Class<?> contextClass = this.applicationContextClass;
		if (contextClass == null) {
			try {
				switch (this.webApplicationType) {
				case SERVLET:
					contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
					break;
				case REACTIVE:
					contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
					break;
				default:
					contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
				}
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Unable create a default ApplicationContext, " + "please specify an ApplicationContextClass",
						ex);
			}
		}
		return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
	}

### 获取SpringBoot应用异常报告器
从 spring.factories 文件中找出所有 SpringBootExceptionReporter 的实现，当前示例包括：

- spring-boot.jar
	- org.springframework.boot.diagnostics.FailureAnalyzers


### 准备上下文

这个步骤包括，加载Bean定义，执行ApplicationContextInitializer初始化

	private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment,
			SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
		context.setEnvironment(environment);
		postProcessApplicationContext(context);
		applyInitializers(context);
		listeners.contextPrepared(context);
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// Add boot specific singleton beans
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		if (printedBanner != null) {
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		// Load the sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		load(context, sources.toArray(new Object[0]));
		listeners.contextLoaded(context);
	}

#### 应用ApplicationContextInitializer进行初始化
ApplicationContextInitializer 来源于前面

### 加载bean配置BeanDefinitionLoader
BeanDefinitionLoader 是一个集大成者，相当于 AnnotatedBeanDefinitionReader ， XmlBeanDefinitionReader ， ClassPathBeanDefinitionScanner 多种来源的Bean配置的门面加载器

	BeanDefinitionLoader(BeanDefinitionRegistry registry, Object... sources) {
		this.sources = sources;
		this.annotatedReader = new AnnotatedBeanDefinitionReader(registry);
		this.xmlReader = new XmlBeanDefinitionReader(registry);
		if (isGroovyPresent()) {
			this.groovyReader = new GroovyBeanDefinitionReader(registry);
		}
		this.scanner = new ClassPathBeanDefinitionScanner(registry);
		this.scanner.addExcludeFilter(new ClassExcludeFilter(sources));
	}


### 刷新应用上下文

	private void refreshContext(ConfigurableApplicationContext context) {
		refresh(context);
		if (this.registerShutdownHook) {
			try {
				context.registerShutdownHook();
			}
			catch (AccessControlException ex) {
				// Not allowed in some environments.
			}
		}
	}

实际执行的是 AbstractApplicationContext 类的 refresh() 方法，这个方法会完成如下事情：

- 初始化前的准备工作。比如系统属性或者环境变量的准备和验证
- 刷新容器，对加载的Bean进行注册，此时已具备 BeanFactory 容器的一切动能
- 对 BeanFactory 进行填充。SPEL，属性编辑器等
- 由子类实现的额外处理，
- 激活 BeanFactoryPostProcessor，比如 propertyplaceHolder
- 注册 BeanPostProcessor
- 国际化资源初始化
- 注册事件广播器
- 留给子类扩展的 onRefersh
- 注册事件，可以由事件广播器调用进行广播
- 初始化非惰性的单例，包括conversation的处理
- 完成刷新过程，发出 ContextRefershEvent 事件