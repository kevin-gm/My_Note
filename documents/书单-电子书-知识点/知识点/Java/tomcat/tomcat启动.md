一、 启动涉及的类

1. Bootstrap：启动主类，即main方法所在的类。
	> 该类在加载时，通过static静态块，会初始化设置系统变量catalina.home以及catalina.base属性。
	> 
	> catalina.home和catalina.base默认情况下，指向的路径是一致的，都是Tomcat主目录
	> 
	> 可以通过启动参数，即java -Dkey=value方式自定义设置路径
2. Catalina：命令执行主体
3. CatalinaProperties：公共配置属性对象
	> * 该类在加载时，通过static静态块，会初始化设置必要的启动参数，并设置为为系统变量。
	> * 首先，获取通过catalina.config设置的系统变量，来加载对应路径上的配置文件，也就是可以通过-Dcatalina.config=path 来修改catalina.properties文件的路径
	> * 如果没有，则加载默认的配置文件，包括
		* ${catalina.base}/conf/catalina.properties
		* /org/apache/catalina/startup/catalina.properties
4. ClassLoaderFactory：类加载器创建工厂，用于创建Tomcat自定义的类加载器
5. SecurityClassLoad：用于加载Tomcat所需的基本类
6. SecurityConfig：Catalina安全访问工具类
7. Security：集中了所有的安全属性和通用安全方法。


二、 启动步骤

1. 执行main方法
2. 实例化``Bootstrap``对象，此时会对Bootstrap进行类的加载初始化。
3. 执行Bootstrap实例对象的init方法
	* 初始化类加载器
		* commonLoader：公共的类加载器，父加载器为null，Tomcat和应用都可以使用
			* ${catalina.base}/lib
			* ${catalina.base}/lib/*.jar
			* ${catalina.home}/lib
			* ${catalina.home}/lib/*.jar
		* catalinaLoader：其父加载器为commonLoader
		* sharedLoader：其父加载器为commonLoader
	* 设置当前线程的类加载器为catalinaLoader
	* 使用当前线程的类加载器加载Tomcat基础类
		* loadCorePackage：加载核心类，包路径为：org.apache.catalina.core
		* loadCoyotePackage：http请求相关，包路径：org.apache.coyote
		* loadLoaderPackage：加载相关，包路径：org.apache.catalina.loader
		* loadRealmPackage：用户角色相关，包路径：org.apache.catalina.realm
		* loadServletsPackage：加载默认的servlet，包路径：org.apache.catalina.servlets
		* loadSessionPackage：session相关，包路径：org.apache.catalina.session
		* loadUtilPackage：公用工具相关，包路径：org.apache.catalina.util
		* loadValvesPackage：阀相关，类似于过滤器的一种对象，包路径：org.apache.catalina.valves
		* loadWebResourcesPackage：识别web资源相关，包路径：org.apache.catalina.webresources
		* loadJavaxPackage：cookie相关，加载类javax.servlet.http.Cookie
		* loadConnectorPackage：连接相关，包路径：org.apache.catalina.connector
		* loadTomcatPackage：Tomcat自身使用的，包路径：org.apache.tomcat
	* 加载并实例化启动命令执行主体org.apache.catalina.startup.Catalina
	* 反射设置Catalina的parentClassLoader属性为sharedLoader。（为什么使用反射，此时已经实例化了对象，为什么不直接设置？）
4. 根据main参数指令，执行不同的操作
	* startd / start：启动Tomcat
		* 调用Bootstrap的load方法，实际是调用Catalina的load方法
			* initDirs():查看系统变量java.io.tmpdir是否有指定临时目录。
			* initNaming()：初始化命名服务naming信息，用于解析xml为java对象的。包括设置系统变量catalina.useNaming、java.naming.factory.initial、java.naming.factory.url.pkgs
			* createStartDigester()：创建用于解析xml的Digester对象，创建该对象时，会指定需要解析的server.xml的各个节点以及对应的java对象
			* 读取conf/server.xml文件，如果为空，则读取classpath下的server-embed.xml文件
			* 解析读取的文件
			* 服务器Server 的catalina属性设置，Server对象是通过xml解析生成的
			* 初始化console日志输出对象
			* Server对象初始化。此时的Server的实例对象为StandardServer，继承了LifecycleBase，所以实际调用的是LifecycleBase的init方法。
				* 检查Server状态
				* 执行lifecycleEvent
	* stopd / stop：停止tomcat
	* configtest：加载参数，然后退出
	* 其余情况，提示命令不存在