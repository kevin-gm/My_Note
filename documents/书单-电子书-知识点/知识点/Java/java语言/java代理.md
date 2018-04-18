## 概念
为某个对象提供一个代理，以控制对这个对象的访问。 代理类和委托类有共同的父类或父接口，这样在任何使用委托类对象的地方都可以用代理对象替代。

代理类负责请求的预处理、过滤、将请求分派给委托类处理、以及委托类执行完请求后的后续处理。

## 静态代理（设计模式中"代理模式"的实现方式）

所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。

代理类和委托类必须在编译前就存在。

##### 代理接口

	/**  
	 * 代理接口。处理给定名字的任务。 
	 */  
	public interface Subject {  
	  /** 
	   * 执行给定名字的任务。 
	    * @param taskName 任务名 
	   */
	   public void dealTask(String taskName);   
	}

##### 委托类，具体处理业务

	/** 
	 * 真正执行任务的类，实现了代理接口。 
	 */  
	public class RealSubject implements Subject {  
	  
	   /** 
	    * 执行给定名字的任务。这里打印出任务名，并休眠500ms模拟任务执行了很长时间 
	    * @param taskName  
	   */  
	   @Override  
	   public void dealTask(String taskName) {  
	      System.out.println("正在执行任务："+taskName);  
	      try {  
	         Thread.sleep(500);  
	      } catch (InterruptedException e) {  
	         e.printStackTrace();  
	      }  
	   }  
	}

##### 静态代理类，持有实际业务处理类的引用

	/** 
	 *　代理类，实现了代理接口。 
	 */  
	public class ProxySubject implements Subject {  
		 //代理类持有一个委托类的对象引用  
		 private Subject delegate;  
		   
		 public ProxySubject(Subject delegate) {  
		  this.delegate = delegate;  
		 }  
		  
		 /** 
		  * 将请求分派给委托类执行，记录任务执行前后的时间，时间差即为任务的处理时间 
		  *  
		  * @param taskName 
		  */  
		 @Override  
		 public void dealTask(String taskName) {  
		 	long stime = System.currentTimeMillis();   
		 	//将请求分派给委托类处理  
		 	delegate.dealTask(taskName);  
		 	long ftime = System.currentTimeMillis();   
		 	System.out.println("执行任务耗时"+(ftime - stime)+"毫秒");
		}
	}

##### 静态代理类工厂

	public class SubjectStaticFactory {  
		 //客户类调用此工厂方法获得代理对象。  
		 //对客户类来说，其并不知道返回的是代理类对象还是委托类对象。  
		 public static Subject getInstance(){   
		  	return new ProxySubject(new RealSubject());  
		 }  
	} 

##### 客户类

	public class Client1 {  
	  
	 public static void main(String[] args) {
		//获取代理类
	 	Subject proxy = SubjectStaticFactory.getInstance();
		//调用代理类方法
	 	proxy.dealTask("DBQueryTask");  
	 }
	}

##### 静态代理类优缺点
>优点：业务类只需要关注业务逻辑本身，保证了业务类的重用性。这是代理的共有优点。
>
> 缺点：
> 
> 1. 代理对象的一个接口只服务于一种类型的对象，如果要代理的方法很多，势必要为每一种方法都进行代理，静态代理在程序规模稍大时就无法胜任了。
> 2. 如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。
> 3. 另外，如果要按照上述的方法使用代理模式，那么真实角色(委托类)必须是事先已经存在的，并将其作为代理对象的内部属性。但是实际使用时，一个真实角色必须对应一个代理角色，如果大量使用会导致类的急剧膨胀；此外，如果事先并不知道真实角色（委托类），该如何使用代理呢？这个问题可以通过Java的动态代理类来解决。

## 动态代理
动态代理类的源码是在程序运行期间由JVM根据反射等机制动态的生成，缓存在JVM内存中，所以不存在代理类的实际字节码文件。代理类和委托类的关系是在程序运行时确定。

**java.lang.reflect.Proxy**
这是 Java 动态代理机制生成的所有动态代理类的父类，它提供了一组静态方法来为一组接口动态地生成代理类及其对象。

	// 方法 1: 该方法用于获取指定代理对象所关联的调用处理器  
	static InvocationHandler getInvocationHandler(Object proxy)   
	  
	// 方法 2：该方法用于获取关联于指定类装载器和一组接口的动态代理类的类对象  
	static Class getProxyClass(ClassLoader loader, Class[] interfaces)   
	  
	// 方法 3：该方法用于判断指定类对象是否是一个动态代理类  
	static boolean isProxyClass(Class cl)   
	  
	// 方法 4：该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例  
	static Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)

**java.lang.reflect.InvocationHandler**
这是调用处理器接口，它自定义了一个 invoke 方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问。每次生成动态代理类对象时都要指定一个对应的调用处理器对象。

	// 该方法负责集中处理动态代理类上的所有方法调用。第一个参数既是代理类实例，第二个参数是被调用的方法对象  
	// 第三个方法是调用参数。调用处理器根据这三个参数进行预处理或分派到委托类实例上反射执行  
	Object invoke(Object proxy, Method method, Object[] args)

**java.lang.ClassLoader**
这是类装载器类，负责将类的字节码装载到 Java 虚拟机（JVM）中并为其定义类对象，然后该类才能被使用。

Proxy 静态方法生成动态代理类同样需要通过类装载器来进行装载才能使用，它与普通类的唯一区别就是其字节码是由 JVM 在运行时动态生成的而非预存在于任何一个 .class 文件中。

每次生成动态代理类对象时都需要指定一个类装载器对象

##### 动态代理实现步骤

>a. 实现InvocationHandler接口创建自己的调用处理器
>
>b. 给Proxy类提供ClassLoader和代理接口类型数组创建动态代理类
>
>c. 以调用处理器类型为参数，利用反射机制得到动态代理类的构造函数
>
>d. 以调用处理器对象为参数，利用动态代理类的构造函数创建动态代理类对象

##### 代理接口

	/**  
	 * 代理接口。处理给定名字的任务。 
	 */  
	public interface Subject {  
	  /** 
	   * 执行给定名字的任务。 
	    * @param taskName 任务名 
	   */
	   public void dealTask(String taskName);   
	}

##### 业务实现类

	/** 
	 * 真正执行任务的类，实现了代理接口。 
	 */  
	public class RealSubject implements Subject {  
	  
	   /** 
	    * 执行给定名字的任务。这里打印出任务名，并休眠500ms模拟任务执行了很长时间 
	    * @param taskName  
	   */  
	   @Override  
	   public void dealTask(String taskName) {  
	      System.out.println("正在执行任务："+taskName);  
	      try {  
	         Thread.sleep(500);  
	      } catch (InterruptedException e) {  
	         e.printStackTrace();  
	      }  
	   }  
	}

##### 代理处理器，持有业务接口引用

	/**
	 * 动态代理类对应的调用处理类
	 */
	public class SubjectInvocationHandler implements InvocationHandler {
	
		// 代理类持有一个委托类的对象引用
		private Object delegate;
	
		public SubjectInvocationHandler(Object delegate) {
			this.delegate = delegate;
		}
	
		@Override
		public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
			long stime = System.currentTimeMillis();
			// 利用反射机制将请求分派给委托类处理。Method的invoke返回Object对象作为方法执行结果。
			// 因为示例程序没有返回值，所以这里忽略了返回值处理
			method.invoke(delegate, args);
			long ftime = System.currentTimeMillis();
			System.out.println("执行任务耗时" + (ftime - stime) + "毫秒");
			return null;
		}
	}

##### 代理实例工厂类

	/**
	 * 生成动态代理对象的工厂
	 */
	public class ProxyFactory {
	
		// 客户类调用此工厂方法获得代理对象。
		// 对客户类来说，其并不知道返回的是代理类对象还是委托类对象。
		public static Subject getInstance() {
			Subject delegate = new RealSubject();
			InvocationHandler handler = new SubjectInvocationHandler(delegate);
			Subject proxy = null;
			proxy = (Subject)Proxy.newProxyInstance(delegate.getClass().getClassLoader(),
					delegate.getClass().getInterfaces(), handler);
			return proxy;
		}
	}

##### 客户端

	public static void main(String[] args) {
		Subject proxy = ProxyFactory.getInstance();  
		proxy.dealTask("DBQueryTask");
	}

### 动态代理的原理
通过上面的内容可知，代理类的生成是在运行时，通过Proxy类的静态方法newProxyInstance生成的。

	Proxy.newProxyInstance(delegate.getClass().getClassLoader(), delegate.getClass().getInterfaces(), dynamicProxy);

需要指定生成代理类的类加载器，代理类代理的接口，以及对应的处理器。

查看 ```newProxyInstance``` 方法，只保留关键代码。

	public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException {

        final Class<?>[] intfs = interfaces.clone();
		// 此处生成代理类的类对象
        Class<?> cl = getProxyClass0(loader, intfs);
        try {
			// 获取对应的构造方法
            final Constructor<?> cons = cl.getConstructor(constructorParams);
			// 创建具体的实例
            return cons.newInstance(new Object[]{h});
        }
    }

可以看到，生成对象实例时，就是普通的反射代码，所以关键点就在于如何生成代理类的类对象。

	private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        // 实现的接口数量不能大于65535.
		if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }
		// 从缓存获取代理对象
        return proxyClassCache.get(loader, interfaces);
    }

查看 ```proxyClassCache``` 的定义可知，

	// proxyClassCache的定义
	private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

接下来关注下 ```ProxyClassFactory``` 的 ```apply``` 方法

	private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>> {
        // 定义代理类的前缀，所有生成的代理类文件名称都是 包名+$proxy+序号
        private static final String proxyClassNamePrefix = "$Proxy";
		// 代理类的序号，使用AtomicLong实现并发时的原子操作，值从0开始
        private static final AtomicLong nextUniqueNumber = new AtomicLong();

        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

			// 省略验证代码...

			// 代理类的包路径
            String proxyPkg = null;
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

			// 对于非公共接口，代理类的包名与接口一致
            for (Class<?> intf : interfaces) {
				// 获取接口的修饰符
                int flags = intf.getModifiers();
				// 如果接口不是Public的，则代理类的包名设置与接口包名一致。
                if (!Modifier.isPublic(flags)) {
                    // 省略包名获取逻辑
                }
            }
			// 如果是公共接口，则包名设置为ReflectUtil.PROXY_PACKAGE，即com.sun.proxy
            if (proxyPkg == null) {
                proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
            }
			// 生成代理类序号
            long num = nextUniqueNumber.getAndIncrement();
			// 构建代理类名称， 包名+前缀+序号。比如，com.sun.proxy.$Proxy0
            String proxyName = proxyPkg + proxyClassNamePrefix + num;
			// 生成代理类Class实例的字节码
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces, accessFlags);
            try {
				// 返回代理类的类对象实例
                return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {}
        }
    }

ProxyGenerator，其中saveGeneratedFiles为是否需要将class文件保存到磁盘的标识，该标识可以通过设置系统变量进行更改。	```System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");```

	public static byte[] generateProxyClass(final String var0, Class<?>[] var1, int var2) {
        ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
		// 此处会生产对应的二进制字节码，逻辑看着头晕，此处不表。
        final byte[] var4 = var3.generateClassFile();
		// 如果需要生成对应的文件，则将二进制内容输出到本地。
        if (saveGeneratedFiles) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    try {
                        int var1 = var0.lastIndexOf(46);
                        Path var2;
                        if (var1 > 0) {
                            Path var3 = Paths.get(var0.substring(0, var1).replace('.', File.separatorChar));
							// JDK1.8，如果路径和文件不存在，会自动创建
                            Files.createDirectories(var3);
                            var2 = var3.resolve(var0.substring(var1 + 1, var0.length()) + ".class");
                        } else {
                            var2 = Paths.get(var0 + ".class");
                        }
						// 输出二进制内容到本地，生成class文件
                        Files.write(var2, var4, new OpenOption[0]);
                        return null;
                    } catch (IOException var4x) {
                        throw new InternalError("I/O exception saving generated file: " + var4x);
                    }
                }
            });
        }
        return var4;
    }

通过反编译工具，查看生成的代理类的class文件内容，大致如下：

	public final class $Proxy0 extends Proxy implements Subject {
		// method 的数量为，Object的基本方法，比如equals等+类本身的方法数量
	    private static Method m1;
	    private static Method m2;
	    private static Method m3;
	    private static Method m4;
	    private static Method m0;
	
	    public $Proxy0(InvocationHandler var1) throws  {
	        super(var1);
	    }
		// equals、toString、hashCode 都是直接调用的处理器对应的invoke方法。
		// 此处说明，equals等方法也可以被代理，也就是如果执行代理类的equals等方法，也会执行代理逻辑
	    public final boolean equals(Object var1) throws  {
	        try {
	            return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
	        } catch (RuntimeException | Error var3) {
	            throw var3;
	        } catch (Throwable var4) {
	            throw new UndeclaredThrowableException(var4);
	        }
	    }
		// 实际方法也是调用处理器的invoke方法
	    public final void printString() throws  {
	        try {
	            super.h.invoke(this, m3, (Object[])null);
	        } catch (RuntimeException | Error var2) {
	            throw var2;
	        } catch (Throwable var3) {
	            throw new UndeclaredThrowableException(var3);
	        }
	    }
		// 静态块，反射创建对应的method实例
	    static {
	        try {
	            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
	            m2 = Class.forName("java.lang.Object").getMethod("toString");
	            m3 = Class.forName("site.coloured.learning.spring.practice.proxy.common.Subject").getMethod("printString");
	            m4 = Class.forName("site.coloured.learning.spring.practice.proxy.common.Subject").getMethod("printInt");
	            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
	        } catch (NoSuchMethodException var2) {
	            throw new NoSuchMethodError(var2.getMessage());
	        } catch (ClassNotFoundException var3) {
	            throw new NoClassDefFoundError(var3.getMessage());
	        }
	    }
	}

总结：

- JDK动态代理是在程序运行过程中，生成代理类的实例。
- 代理类的Class对象，会在第一次生成代理类的时候生成，并缓存在JVM内存中，之后生成代理类时，直接从内存获取改Class对象并创建代理类对象实例。
- 程序执行到代理类时，首先static块通过反射获取对应处理器(Invocationhandler)的invoke方法，执行额外逻辑后，然后再通过反射调用被代理对象的实际业务方法
- 因为代理类继承了Proxy类，而Java是单继承的，所以只能实现其余接口，故JDK动态代理只能支持基于接口的代理，不能支持基于实现类的代理。
- 代理类，同样可以代理equals、toString、hashCode 等方法。

## Cglib代理
静态代理和动态代理模式都是要求目标对象是实现一个接口的目标对象,但是有时候目标对象只是一个单独的对象,并没有实现任何的接口,这个时候就可以使用以目标对象子类的方式类实现代理,这种方法就叫做:Cglib代理

CGLIB是一个强大的、高性能的代码生成库。**Cglib代理,也叫作子类代理,它是在内存中构建一个子类对象从而实现对目标对象功能的扩展.**

- JDK的动态代理有一个限制,就是使用动态代理的对象必须实现一个或多个接口,如果想代理没有实现接口的类,就可以使用Cglib实现.
- Cglib是一个强大的高性能的代码生成包,它可以在运行期扩展java类与实现java接口.它广泛的被许多AOP的框架使用,例如Spring AOP和synaop,为他们提供方法的interception(拦截)
- Cglib包的底层是通过使用一个小而块的字节码处理框架ASM来转换字节码并生成新的类.不鼓励直接使用ASM,因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉.

**Cglib子类代理实现方法:**

1. 需要引入cglib的jar文件,但是Spring的核心包中已经包括了Cglib功能,所以直接引入pring-core-3.2.5.jar即可.
2. 引入功能包后,就可以在内存中动态构建子类
3. 代理的类不能为final,否则报错
4. 目标对象的方法如果为final/static,那么就不会被拦截,即不会执行目标对象额外的业务方法.

##### 目标对象类

	/**
	 * 目标对象,没有实现任何接口
	 */
	public class UserDao {
	
	    public void save() {
	        System.out.println("----已经保存数据!----");
	    }
	}

##### Cglib代理工厂: ProxyFactory.java

	/**
	 * Cglib子类代理工厂
	 * 对UserDao在内存中动态构建一个子类对象
	 */
	public class ProxyFactory implements MethodInterceptor{
	    //维护目标对象
	    private Object target;
	
	    public ProxyFactory(Object target) {
	        this.target = target;
	    }
	
	    //给目标对象创建一个代理对象
	    public Object getProxyInstance(){
	        //1.工具类
	        Enhancer en = new Enhancer();
	        //2.设置父类
	        en.setSuperclass(target.getClass());
	        //3.设置回调函数
	        en.setCallback(this);
	        //4.创建子类(代理对象)
	        return en.create();
	
	    }
	
	    @Override
	    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
	        System.out.println("开始事务...");
	
	        //执行目标对象的方法
	        Object returnValue = method.invoke(target, args);
	
	        System.out.println("提交事务...");
	
	        return returnValue;
	    }
	}

##### 客户端

	public static void main(String[] args)　｛
		//目标对象
        UserDao target = new UserDao();
        //代理对象
        UserDao proxy = (UserDao)new ProxyFactory(target).getProxyInstance();
        //执行代理对象的方法
        proxy.save();
	}

### cglib介绍

Enhancer
> 增强器，cglib通过其创建代理对象。