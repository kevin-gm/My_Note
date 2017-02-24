#Tomcat优化之配置线程池
原文链接：
<a>https://bbs.aliyun.com/read/307481.html?utm_campaign=wenzhang&utm_medium=article&utm_source=QQ-qun&utm_content=m_11609
</a>

##简介 
 
&emsp;&emsp;线程池作为提高程序处理数据能力的一种方案，应用非常广泛。大量的服务器都或多或少的使用到了线程池技术，不管是用Java还是C++实现，线程池都有如下的特点,线程池一般有三个重要参数：
>1.最大线程数。在程序运行的任何时候，线程数总数都不会超过这个数。如果请求数量超过最大数时，则会等待其他线程结束后再处理。<br/>
>2.最大共享线程数，即最大空闲线程数。如果当前的空闲线程数超过该值，则多余的线程会被杀掉。<br/>
>3.最小共享线程数，即最小空闲线程数。如果当前的空闲数小于该值，则一次性创建这个数量的空闲线程，所以它本身也是一个创建线程的步长。 <br/>

线程池有两个概念：
>1.Worker线程。工作线程主要是运行执行代码，有两种状态：空闲状态和运行状态。在空闲状态时，类似“休眠”，等待任务；处理运行状态时，表示正在运行任务（Runnable）。<br/>
>2.辅助线程。主要负责监控线程池的状态：空闲线程是否超过最大空闲线程数或者小于最小空闲线程数等。如果不满足要求，就调整之。
 
&emsp;&emsp;来看一下线程池究竟是怎么一回事？其实线程池的原理很简单，类似于操作系统中的缓冲区的概念，它的流程如下：先启动若干数量的线程，并让这些线程都处于睡眠 状态，当客户端有一个新请求时，就会唤醒线程池中的某一个睡眠线程，让它来处理客户端的这个请求，当处理完这个请求后，线程又处于睡眠状态。可能你也许会 问：为什么要搞得这么麻烦，如果每当客户端有新的请求时，我就创建一个新的线程不就完了？这也许是个不错的方法，因为它能使得你编写代码相对容易一些，但 你却忽略了一个重要的问题??性能！就拿我所在的单位来说，我的单位是一个省级数据大集中的银行网络中心，高峰期每秒的客户端请求并发数超过100，如果 为每个客户端请求创建一个新线程的话，那耗费的CPU时间和内存将是惊人的，如果采用一个拥有200个线程的线程池，那将会节约大量的的系统资源，使得更 多的CPU时间和内存用来处理实际的商业应用，而不是频繁的线程创建与销毁。 
 
##配置 
&emsp;&emsp;使用线程池，用较少的线程处理较多的访问，可以提高tomcat处理请求的能力。使用方式：<br/> 

首先。打开/conf/server.xml，增加 

	<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="500" minSpareThreads="20" maxIdleTime="60000" prestartminSpareThreads="true" maxQueueSize="100" />
 
>name: 线程名称<br/>
>namePrefix: 线程前缀<br/>
>maxThreads : 最大并发连接数，不配置时默认200，一般建议设置500~ 800,要根据自己的硬件设施条件和实际业务需求而定。 
minSpareThreads：Tomcat启动初始化的线程数，默认值25
prestartminSpareThreads：在tomcat初始化的时候就初始化minSpareThreads的值， 不设置true时minSpareThreads
maxQueueSize: 最大的等待队列数，超过则拒绝请求
minSpareThreads：线程最大空闲时间60秒。

然后，修改<connector ...="" style="box-sizing: border-box;">节点，增加executor属性，如: 
 
	<Connector port="8080" 	protocol="org.apache.coyote.http11.Http11Nio2Protocol"  
        connectionTimeout="20000"  
        redirectPort="8443"  
        executor="tomcatThreadPool"  
        enableLookups="false"  
        acceptCount="100"  
        maxPostSize="10485760"  
        compression="on"  
        disableUploadTimeout="true"  
        compressionMinSize="2048"  
        noCompressionUserAgents="gozilla, traviata"  
        acceptorThreadCount="2"  
        compressableMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript"  
        URIEncoding="utf-8"/>
 
>port：连接端口。<br/>
protocol：连接器使用的传输方式。Tomcat8设置 nio2 更好：	org.apache.coyote.http11.Http11Nio2Protocolprotocol；Tomcat 6、7 设置 nio 更	好：org.apache.coyote.http11.Http11NioProtocol <br/>
注：每个web客户端请求对于服务器端来说就一个单独的线程，客户端的请求数量增多将会导致线程数就上去了，CPU就忙着 跟线程切换。 
 
 
&emsp;&emsp;而NIO则是使用单线程(单个CPU)或者只使用少量的多线程(多CPU)来接受Socket，而由线程池来处理堵塞在pipe 或者队 列里的请求.这样的话，只要OS可以接受TCP的连接，web服务器就可以处理该请求，大大提高了web服务器的可伸缩性。 
 
 
&emsp;&emsp;executor： 连接器使用的线程池名称enableLookups：禁用DNS 查询acceptCount：指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理，默认设置 100 。maxPostSize：限制 以FORM URL 参数方式的POST请求的内容大小，单位字节，默认是 2097152(2兆)，10485760 为 10M。 
 
 
&emsp;&emsp;如果要禁用限制，则可以设置为 -1。acceptorThreadCount： 用于接收连接的线程的数量，默认值是1。一般这个指需要改动的时候是因为该服务器是一个多核CPU，如果是多核 CPU 一般配置为 2。 
 
 
&emsp;&emsp;compression：传输时是压缩。compressionMinSize：压缩的大小noCompressionUserAgents：不启用压缩的浏览器 
 
 
&emsp;&emsp;提示：压缩会增加Tomcat负担，最好采用Nginx + Tomcat 或者 Apache + Tomcat 方式，压缩交由Nginx/Apache 去做。 
 
 
&emsp;&emsp;Tomcat 的压缩是在客户端请求服务器对应资源后，从服务器端将资源文件压缩，再输出到客户端，由客户端的浏览器负责解压缩并浏览。相对于普通的 浏览过程 HTML、CSS、Javascript和Text，它可以节省40% 左右的流量。更为重要的是，它可以对动态生成的，包括CGI、PHP、JSP、ASP、Servlet,SHTML等输出的网页也能进行压缩，压缩效率也很高。