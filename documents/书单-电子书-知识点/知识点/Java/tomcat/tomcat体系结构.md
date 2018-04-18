- server
	- service
		- engine
			- host
				- context
					- wapper
		- connector


## servlet容器
容器是一个能执行从客户端接收的请求，并且基于这些请求返回响应的对象。

一个容器可以有选择性的支持一个Valve(阀)的流水线，他可以通过实现Pipeline接口，以根据运行时的配置顺序处理请求。

Servlet容器是org.apache.catalina.Container接口的实例

有四种类型的容器：Engine，Host，Context，Wrapper
>
- Engine：代表整个Servlet引擎，可能包含一个或者多个Host或者Context子容器，或者其他自定义组。
- Host：代表虚拟主机，包含一个或多个Context子容器
- Context：代表一个独立的web应用程序，可以有多个Wrapper
- Wrapper：表示一个独立的servlet

Catalina的给定部署并不需要包含所有的容器类型。因此，容器的实现需要设计成即使缺少父容器也能正确运行。

容器也可能跟一些提供其他功能的组件相关联，不如：Loader，Logger，Manager，Realm，Resources

## 管道Pipeline
Pipeline接口描述了当invoke方法调用时，需要顺序执行的Valve(阀)列表。

需要某个Valve(阀)必须处理Request(请求)并创建响应的Response(响应)，通常是最后一个Valve，而不是尝试传递请求。

通常有一个单一的Pipeline跟每一个Container(容器)关联。

可以通过修改Tomcat的配置文件server.xml来动态的添加Valve。

Valve需要与Container关联，一个Servlet容器可以有一条Pipeline，当调用了容器的invoke方法后，容器会将处理工作交给Pipeline完成