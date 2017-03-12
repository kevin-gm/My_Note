#在CentOS 7中安装与配置JDK8

##安装说明

>系统环境：centos7

>安装方式：rpm安装

>软件：jdk-8u25-linux-x64.rpm

>下载地址：http://www.oracle.com/technetwork/java/javase/downloads/index.html

##检验系统原版本
	[root@zck ~]# java -version
	java version "1.7.0_"
	OpenJDK Runtime Environment (IcedTea6 1.11.1) (rhel-1.45.1.11.1.el6-x86_64)
	OpenJDK 64-Bit Server VM (build 20.0-b12, mixed mode)

##进一步查看JDK信息：
	[root@localhost ~]#  rpm -qa | grep java
	javapackages-tools-3.4.1-6.el7_0.noarch
	tzdata-java-2014i-1.el7.noarch
	java-1.7.0-openjdk-headless-1.7.0.71-2.5.3.1.el7_0.x86_64
	java-1.7.0-openjdk-1.7.0.71-2.5.3.1.el7_0.x86_64
	python-javapackages-3.4.1-6.el7_0.noarch

##卸载OpenJDK，执行以下操作：
	[root@localhost ~]# rpm -e --nodeps tzdata-java-2014i-1.el7.noarch
	[root@localhost ~]# rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.71-2.5.3.1.el7_0.x86_64
	[root@localhost ~]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.71-2.5.3.1.el7_0.x86_64




##安装JDK
上传新的jdk-8u25-linux-x64.rpm软件到/usr/local/执行以下操作：

	[root@zck local]# rpm -ivh jdk-8u25-linux-x64.rpm

JDK默认安装在/usr/java中。


##验证安装
执行以下操作，查看信息是否正常：

	[root@localhost ~]# java
	[root@localhost ~]# javac
	[root@localhost ~]# java -version
	java version "1.8.0_25"
	Java(TM) SE Runtime Environment (build 1.8.0_25-b17)
	Java HotSpot(TM) 64-Bit Server VM (build 25.25-b02, mixed mode)




##配置环境变量
修改系统环境变量文件

	vi + /etc/profile


向文件里面追加以下内容：

	JAVA_HOME=/usr/java/jdk1.8.0_25
	JRE_HOME=/usr/java/jdk1.8.0_25/jre
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
	CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
	export JAVA_HOME JRE_HOME PATH CLASSPATH


使修改生效

	[root@localhost ~]# source /etc/profile   //使修改立即生效
	[root@localhost ~]#        echo $PATH   //查看PATH值


查看系统环境状态

	[root@localhost ~]# echo $PATH
	/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/usr/java/jdk1.8.0_121/bin:/usr/java/jdk1.8.0_121/jre/bin