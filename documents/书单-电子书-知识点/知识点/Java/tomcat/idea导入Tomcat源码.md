个人环境
> * JDK - 1.8
> * tomcat - 8.0.49
> * idea - 2017.2
> * ant - 1.9.9


1. 从Tomcat官网下载对应版本的src源码，比如apache-tomcat-8.0.49-src
2. idea的代码结构是project下分多个module，我们把apache-tomcat-8.0.49-src当成project下的一个module。所以需要新建一个父级project
3. 新建一个文件夹，命名为tomcat-8-study，将apache-tomcat-8.0.49-src拷贝到该文件夹下，并新建pom.xml文件，引入maven管理。（下面的pom文件包括后面解决问题后，完整的pom配置）

		<?xml version="1.0" encoding="UTF-8"?>  
		<project xmlns="http://maven.apache.org/POM/4.0.0"  
		         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
		         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
		  
		  
		    <modelVersion>4.0.0</modelVersion>  
		    <groupId>org.apache.tomcat</groupId>  
		    <artifactId>Tomcat8.0</artifactId>  
		    <name>Tomcat8.0</name>  
		    <version>8.0</version>  
		  
		    <build>  
		        <finalName>Tomcat8.0</finalName>  
		        <sourceDirectory>java</sourceDirectory>  
		        <testSourceDirectory>test</testSourceDirectory>  
		        <resources>  
		            <resource>  
		                <directory>java</directory>  
		            </resource>  
		        </resources>  
		        <testResources>  
		            <testResource>  
		                <directory>test</directory>  
		            </testResource>  
		        </testResources>  
		        <plugins>  
		            <plugin>  
		                <groupId>org.apache.maven.plugins</groupId>  
		                <artifactId>maven-compiler-plugin</artifactId>  
		                <version>3.3</version>  
		  
		                <configuration>  
		                    <encoding>UTF-8</encoding>  
		                    <source>1.8</source>  
		                    <target>1.8</target>  
		                </configuration>  
		            </plugin>  
		        </plugins>  
		    </build>  
		  
		    <dependencies>  
		        <dependency>  
		            <groupId>junit</groupId>  
		            <artifactId>junit</artifactId>  
		            <version>4.12</version>
		            <scope>test</scope>  
		        </dependency>
		        <dependency>
		            <groupId>org.easymock</groupId>
		            <artifactId>easymock</artifactId>
		            <version>3.4</version>
		            <scope>test</scope>
		        </dependency>
		        <dependency>
		            <groupId>org.apache.ant</groupId>
		            <artifactId>ant</artifactId>
		            <version>1.9.9</version>
		        </dependency>
		        <dependency>  
		            <groupId>wsdl4j</groupId>  
		            <artifactId>wsdl4j</artifactId>  
		            <version>1.6.2</version>  
		        </dependency>  
		        <dependency>  
		            <groupId>javax.xml</groupId>  
		            <artifactId>jaxrpc</artifactId>  
		            <version>1.1</version>  
		        </dependency>  
		        <dependency>  
		            <groupId>org.eclipse.jdt.core.compiler</groupId>  
		            <artifactId>ecj</artifactId>  
		            <version>4.2.2</version>  
		        </dependency>  
		    </dependencies>  
		  
		</project>
4. 在tomcat-8-study文件夹下，新建catalina-home文件夹

	将apache-tomcat-8.0.49-src下的conf、bin、webapps目录拷贝到catalina-home文件夹下。

	此时catalina-home就相当于我们平时使用的Tomcat
5. 在apache-tomcat-8.0.49-src下新建pom.xml文件，module有需要maven管理

		<?xml version="1.0" encoding="UTF-8"?>  
		<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
		         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">  
		  
		    <modelVersion>4.0.0</modelVersion>  
		    <groupId>site.coloured</groupId>  
		    <artifactId>tomcat-8-study</artifactId>  
		    <name>Tomcat 8.0 Study</name>  
		    <version>1.0</version>  
		    <packaging>pom</packaging>  
		  
		    <modules>  
		        <module>apache-tomcat-8.0.49-src</module>  
		    </modules>  
		</project>
6. 最终的目录结构如下

		--tomcat-8-study
		------catalina-home
		------------conf
		------apache-tomcat-8.0.49-src
		------------bin
		------------java
		------------modules
		------------webapps
		------------pom.xml
		------pom.xml
7. 设置Main class为：org.apache.catalina.startup.Bootstrap

		idea -->  run/debug configuration
8. 设置VM options为：

		-Dcatalina.home=catalina-home -Dcatalina.base=catalina-home  
		-Djava.endorsed.dirs=catalina-home/endorsed -Djava.io.tmpdir=catalina-home/temp  
		-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager  
		-Djava.util.logging.config.file=catalina-home/conf/logging.properties
9. 此时尝试启动
	
	个人搭建的结果，启动报错：找不到org.apache.tools.ant.BuildException这个类。

	查看idea-->project structure-->modules-->dependencies发现，引用的ant项目报错，提示信息为library has broken path。

	ant的包是通过maven引入的，查看pom.xml果然是引入的依赖有问题。

	原因是，原本网上资料使用的ant版本是1.6.5的
		
		<dependency>
            <groupId>ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.6.5</version>
        </dependency>

	而由于我使用的是高版本1.9.9，此时的依赖跟之前不一样，所以没能正确引用ant的包导致的。

		<dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.9.9</version>
        </dependency>

10. 此时再启动，提示找不到类```easymock```

	这个类在test文件夹下，可以干脆选择将apache-tomcat-8.0.49-src下的test文件夹删除，然后将pom文件里面testSource相关的配置也删除掉，这个并不影响，因为即使处理了easymock的问题，也还有另一个问题。

	自行导入easymock依赖。

		<dependency>
		    <groupId>org.easymock</groupId>
		    <artifactId>easymock</artifactId>
		    <version>3.4</version>
		    <scope>test</scope>
		</dependency>

11. 再启动，报错"找不到符号:TestCookieFilter"，

	可以发现，这个不是第三方依赖的类，而是test目录下，util包里面的类，但是代码里面却没有。此时就很尴尬了

	万能的度娘找到了答案，有人在github找到了这个类的代码。于是将其添加到对应的util包内。

		/*
		 * Licensed to the Apache Software Foundation (ASF) under one or more
		 * contributor license agreements.  See the NOTICE file distributed with
		 * this work for additional information regarding copyright ownership.
		 * The ASF licenses this file to You under the Apache License, Version 2.0
		 * (the "License"); you may not use this file except in compliance with
		 * the License.  You may obtain a copy of the License at
		 *
		 *     http://www.apache.org/licenses/LICENSE-2.0
		 *
		 * Unless required by applicable law or agreed to in writing, software
		 * distributed under the License is distributed on an "AS IS" BASIS,
		 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
		 * See the License for the specific language governing permissions and
		 * limitations under the License.
		 */
		package util;
		
		import java.util.Locale;
		import java.util.StringTokenizer;
		
		/**
		 * Processes a cookie header and attempts to obfuscate any cookie values that
		 * represent session IDs from other web applications. Since session cookie names
		 * are configurable, as are session ID lengths, this filter is not expected to
		 * be 100% effective.
		 *
		 * It is required that the examples web application is removed in security
		 * conscious environments as documented in the Security How-To. This filter is
		 * intended to reduce the impact of failing to follow that advice. A failure by
		 * this filter to obfuscate a session ID or similar value is not a security
		 * vulnerability. In such instances the vulnerability is the failure to remove
		 * the examples web application.
		 */
		public class CookieFilter {
		
		    private static final String OBFUSCATED = "[obfuscated]";
		
		    private CookieFilter() {
		        // Hide default constructor
		    }
		
		    public static String filter(String cookieHeader, String sessionId) {
		
		        StringBuilder sb = new StringBuilder(cookieHeader.length());
		
		        // Cookie name value pairs are ';' separated.
		        // Session IDs don't use ; in the value so don't worry about quoted
		        // values that contain ;
		        StringTokenizer st = new StringTokenizer(cookieHeader, ";");
		
		        boolean first = true;
		        while (st.hasMoreTokens()) {
		            if (first) {
		                first = false;
		            } else {
		                sb.append(';');
		            }
		            sb.append(filterNameValuePair(st.nextToken(), sessionId));
		        }
		
		
		        return sb.toString();
		    }
		
		    private static String filterNameValuePair(String input, String sessionId) {
		        int i = input.indexOf('=');
		        if (i == -1) {
		            return input;
		        }
		        String name = input.substring(0, i);
		        String value = input.substring(i + 1, input.length());
		
		        return name + "=" + filter(name, value, sessionId);
		    }
		
		    public static String filter(String cookieName, String cookieValue, String sessionId) {
		        if (cookieName.toLowerCase(Locale.ENGLISH).contains("jsessionid") &&
		                (sessionId == null || !cookieValue.contains(sessionId))) {
		            cookieValue = OBFUSCATED;
		        }
		
		        return cookieValue;
		    }
		}

12. 再次启动，此时报错"找不到符号：VERSION_1_8"

	报错的位置为，org.eclipse.jdt.internal.compiler.impl.CompilerOptions.

	注释掉所有如下代码，此处是检测虚拟机版本的：

		else if(opt.equals("1.8")) {
                settings.put(CompilerOptions.OPTION_Source,
                             CompilerOptions.VERSION_1_8);
            } else if(opt.equals("1.9")) {
                settings.put(CompilerOptions.OPTION_Source,
                             CompilerOptions.VERSION_1_9);
            }

13. 此时，Tomcat可以完整启动。

14. 访问，http://127.0.0.1:8080/，此时个人的环境又报错了

	有一个nullpointerexpcetion异常，根据报错位置查看代码可知

	JspFactory.getDefaultFactory()次数获取的JSPFactory是null，

		ValidateVisitor(Compiler compiler) {
	            this.pageInfo = compiler.getPageInfo();
	            this.err = compiler.getErrorDispatcher();
	            this.loader = compiler.getCompilationContext().getClassLoader();
	            // Get the cached EL expression factory for this context
	            expressionFactory =
	                    JspFactory.getDefaultFactory().getJspApplicationContext(
	                    compiler.getCompilationContext().getServletContext()).
	                    getExpressionFactory();
	        }

	查看对应的类和方法可知，默认返回的就是一个null。

		private static volatile JspFactory deflt = null;

		public static JspFactory getDefaultFactory() {
	        //return new JspFactoryImpl();
	        return deflt;
	    }

	此时个人的处理方式是，查看JspFactory抽象类对应的实现，将其改成对应的实现

		return new JspFactoryImpl();

15. 重启，访问，得到了想要的"小猫"

16. 此时可能查看localhost.log日志，发现有报异常

	classnotfound:contextlistener。

	不影响使用，不过可以将webapps下的example删掉，启动就没关系了。