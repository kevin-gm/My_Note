## 概念
为某个对象提供一个代理，以控制对这个对象的访问。 代理类和委托类有共同的父类或父接口，这样在任何使用委托类对象的地方都可以用代理对象替代。

代理类负责请求的预处理、过滤、将请求分派给委托类处理、以及委托类执行完请求后的后续处理。

## 静态代理（设计模式中"代理模式"的实现方式）
> http://blog.csdn.net/giserstone/article/details/17199755

由程序员创建或工具生成代理类的源码，再编译代理类。

所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。

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
动态代理类的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以不存在代理类的字节码文件。代理类和委托类的关系是在程序运行时确定。

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

## Cglib代理
静态代理和动态代理模式都是要求目标对象是实现一个接口的目标对象,但是有时候目标对象只是一个单独的对象,并没有实现任何的接口,这个时候就可以使用以目标对象子类的方式类实现代理,这种方法就叫做:Cglib代理

**Cglib代理,也叫作子类代理,它是在内存中构建一个子类对象从而实现对目标对象功能的扩展.**

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
	｝