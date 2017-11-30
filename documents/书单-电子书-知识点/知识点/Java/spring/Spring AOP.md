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

## AOP概念

aspect-oriented programming，面向切面编程。

作为面向对象编程的一种补充，广泛应用于处理一些具有横切性质的系统级服务，如事务管理、安全检查、权限校验等。

AOP实现的关键就在于 AOP 框架自动创建的 AOP 代理，AOP 代理则可分为**静态代理**和**动态代理**两大类，其中静态代理是指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为**编译时增强**；而动态代理则在运行时借助于 JDK 动态代理、CGLIB 等在内存中“临时”生成 AOP 动态代理类，因此也被称为**运行时增强**。

* 切面（Aspect）：一个关注点的模块化，这个关注点可能会横切多个对象。事务管理是J2EE应用中一个关于横切关注点的很好的例子。在Spring AOP中，切面可以使用基于Schema的AOP支持或者基于@Aspect注解的方式来实现。
* 连接点（Joinpoint）：在程序执行过程中某个特定的点，比如某方法调用的时候或者处理异常的时候。在Spring AOP中，一个连接点总是表示一个方法的执行。
* 通知（Advice）：在切面的某个特定的连接点上执行的动作。其中包括了“around”、“before”和“after”等不同类型的通知（通知的类型将在后面部分进行讨论）。许多AOP框架（包括Spring）都是以拦截器做通知模型，并维护一个以连接点为中心的拦截器链。
* 切入点（Pointcut）：匹配连接点的断言。通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行（例如，当执行某个特定名称的方法时）。切入点表达式如何和连接点匹配是AOP的核心：Spring缺省使用AspectJ切入点语法。
* 引入（Introduction）：用来给一个类型声明额外的方法或属性（也被称为连接类型声明（inter-type declaration））。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用引入来使一个bean实现IsModified接口，以便简化缓存机制。
* 目标对象（Target Object）： 被一个或者多个切面所通知的对象。也被称做被通知（advised）对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个被代理（proxied）对象。
* AOP代理（AOP Proxy）：AOP框架创建的对象，用来实现切面契约（例如通知方法执行等等）。在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。
* 织入（Weaving）：把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。

**通知类型**：

* 前置通知（Before advice）：在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。
* 后置通知（After returning advice）：在某连接点正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。
* 异常通知（After throwing advice）：在方法抛出异常退出时执行的通知。
* 最终通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
* 环绕通知（Around Advice）：包围一个连接点的通知，如方法调用。这是最强大的一种通知类型。环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它自己的返回值或抛出异常来结束执行。

## spring aop
Spring AOP 与ApectJ 的目的一致，都是为了统一处理横切业务，但与AspectJ不同的是，Spring AOP 并不尝试提供完整的AOP功能(即使它完全可以实现)，Spring AOP 更注重的是与Spring IOC容器的结合，并结合该优势来解决横切业务的问题

Spring注意到AspectJ在AOP的实现方式上依赖于特殊编译器(ajc编译器)，因此Spring很机智回避了这点，转向采用动态代理技术的实现原理来构建Spring AOP的内部机制（动态织入），这是与AspectJ（静态织入）最根本的区别。

Spring缺省使用J2SE 动态代理（dynamic proxies）来作为AOP的代理。 这样任何接口（或者接口集）都可以被代理。
Spring也可以使用CGLIB代理. 对于需要代理类而不是代理接口的时候CGLIB代理是很有必要的。如果一个业务对象并没有实现一个接口，默认就会使用CGLIB

@AspectJ使用了Java 5的注解，可以将切面声明为普通的Java类。@AspectJ样式在AspectJ 5发布的AspectJ project部分中被引入。Spring 2.0使用了和AspectJ 5一样的注解，并使用AspectJ来做切入点解析和匹配。但是，AOP在运行时仍旧是纯的Spring AOP，并不依赖于AspectJ的编译器或者织入器（weaver）

##### 启用@AspectJ支持

	<aop:aspectj-autoproxy/>

##### 定义切面

	@Aspect
	@Component
	public class LogAspect {
	
		@Pointcut(value = "@annotation(site.coloured.sample.aop.LogAnnotation)")
		public void pointcut() {}
		
		@Before(value = "pointcut()")
		public void before() {
			System.out.println("前置通知");
		}
		
		@After(value = "pointcut()")
		public void after() {
			System.out.println("后置通知");
		}
	}

##### 业务方法

	@LogAnnotation
	public String test() {
		System.out.println("test");
	}