##简介
SPI的全名为Service Provider Interface,是JDK内置的一种服务提供发现机制。

我们系统里抽象的各个模块，往往有很多不同的实现方案，比如日志模块，jdbc模块的方案等。面向对象的设计里，我们一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。为了实现在模块装配的时候能不在程序里动态指明，这就需要一种服务发现机制。 java spi就是提供这样的一个机制：为某个接口寻找服务实现的机制。有点类似IOC的思想，就是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要。

##java spi的具体约定
当服务的提供者，提供了服务接口的一种实现之后，在jar包的<strong>META-INF/services/</strong>目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。 
基于这样一个约定就能很好的找到服务接口的实现类，而不需要再代码里制定。

jdk提供服务实现查找的一个工具类：java.util.ServiceLoader

##示例

### 概述
![Java SPI要点](./images/书单-电子书-知识点/知识点/Java/java语言/SPI/java-spi.png)

### 接口

	package site.coloured.sample.spi.service;
	public interface PeopleService {
	
		void showPeople();
	}

### 实现A

	package site.coloured.sample.spi.impl;

	import site.coloured.sample.spi.service.PeopleService;

	public class BoyService implements PeopleService {

		@Override
		public void showPeople() {
			System.out.println("this is the boy implement");
		}
	
	}

### 实现B

	package site.coloured.sample.spi.impl;

	import site.coloured.sample.spi.service.PeopleService;
	
	public class GirlService implements PeopleService {
	
		@Override
		public void showPeople() {
			System.out.println("this is girl implement");
		}
	
	}

### 代码结构
	
	└── src
	├── site
	│   └── coloured.sample
	│       └── spi
	│			└──service
	│           │	├── PeopleService.java
	│           ├── impl
	│           │   ├── BoyService.java
	│           │   └── GirlService.java
	│           └── Manager.java
	└── META-INF
	    └── services
	        └── site.coloured.sample.spi.service.PeopleService

	site.coloured.sample.spi.service.PeopleService文件内容：
		site.coloured.sample.spi.impl.BoyService
		site.coloured.sample.spi.impl.GirlService

### 客户端

	package site.coloured.sample.spi;

	import java.util.ServiceLoader;
	
	import site.coloured.sample.spi.service.PeopleService;
	
	public class Manager {
	
		public static void main(String[] args) {
			ServiceLoader<PeopleService> loader = ServiceLoader.load(PeopleService.class);
			for (PeopleService service : loader) {
				service.showPeople();
			}
		}
	}

## 数据库驱动java.sql.Driver以及具体实现

### 接口，即java.sql.Driver

	package java.sql;

	import java.util.logging.Logger;
	
	public interface Driver {
	
	    Connection connect(String url, java.util.Properties info)
	        throws SQLException;
	
	    boolean acceptsURL(String url) throws SQLException;
	
	    DriverPropertyInfo[] getPropertyInfo(String url, java.util.Properties info)
	                         throws SQLException;
	
	    int getMajorVersion();
	
	    int getMinorVersion();
	
	    boolean jdbcCompliant();
	
	    public Logger getParentLogger() throws SQLFeatureNotSupportedException;
	}

### mysql驱动中的实现配置（mysql-connector-java-5.1.39)
META-INF/services/java.sql.Driver文件内容

	com.mysql.jdbc.Driver
	com.mysql.fabric.jdbc.FabricMySQLDriver

### 客户端（java.sql.DriverManager）

	static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }

	private static void loadInitialDrivers() {
        String drivers;
        try {
            drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
                public String run() {
                    return System.getProperty("jdbc.drivers");
                }
            });
        } catch (Exception ex) {
            drivers = null;
        }
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
				//SPI 加载接口实现类
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();

                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                // Do nothing
                }
                return null;
            }
        });

        println("DriverManager.initialize: jdbc.drivers = " + drivers);

        if (drivers == null || drivers.equals("")) {
            return;
        }
        String[] driversList = drivers.split(":");
        println("number of Drivers:" + driversList.length);
        for (String aDriver : driversList) {
            try {
                println("DriverManager.Initialize: loading " + aDriver);
                Class.forName(aDriver, true,
                        ClassLoader.getSystemClassLoader());
            } catch (Exception ex) {
                println("DriverManager.Initialize: load failed: " + ex);
            }
        }
    }
