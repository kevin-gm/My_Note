#centos7虚拟机安装elasticsearch5.0.x-安装篇
##安装jdk
安装步骤见“在CentOS 7中安装与配置JDK8”

##创建新用户（非root用户）
elasticsearch只能用非root启动，这里我创建了一个叫seven的用户

	[root@localhost ~]# useradd kevin
	[root@localhost ~]# passwd kevin

##下载,解压并运行elasticsearch

下载

	[root@localhost ~]# su kevin
	[kevin@localhost root]$ cd /usr/kevin
	[kevin@localhost kevin]$ mkdir opt
	[kevin@localhost kevin]$ cd opt

将下载的elasticsearch-5.0.0.tar.gz文件上传到opt目录下

解压并运行elasticsearch

	[seven@localhost opt]$ tar -zxvf elasticsearch-5.0.0.tar.gz

重命名解压文件

	mv elasticsearch-5.0.0.tar.gz elasticsearch

切换到root用户

	su root

设置用户权限

	chown -R kevin /usr/kevin/opt/elasticsearch

新建必要文件夹：（在安装过程中，不建这2个文件夹，启动时会报错，找不到这两个路径。）

	mkdir -p /usr/local/elasticsearch/plugins
	mkdir -p /usr/local/elasticsearch/config/scripts

修改配置文件，修改访问elasticsearch的IP及端口（注意，设置参数的时候:后面要有空格！）

	[seven@localhost config]$ vim /usr/java/elasticsearch/config/elasticsearch.yml

	# 换个集群的名字，免得跟别人的集群混在一起
	cluster.name: wlt-es5.0-application   
	#
	 换个节点名字
	node.name: node-1
	     
	path.data: /data/elasticsearch/data
	path.logs: /data/elasticsearch/logs
	
	 # 修改一下ES的监听地址，这样别的机器也可以访问
	network.host: 0.0.0.0
	 # 默认的就好
	http.port: 9200
	# 增加新的参数，这样head插件可以访问es
	#http.cors.enabled: true
	#http.cors.allow-origin: “*"

启动服务

	./elasticsearch

此时启动异常：

	[2017-03-12T03:28:50,530][INFO ][o.e.n.Node               ] [] initializing ...
	[2017-03-12T03:28:50,643][INFO ][o.e.e.NodeEnvironment    ] [g_bkxC0] using [1] data paths, mounts [[/ (rootfs)]], net usable_space [13.1gb], net total_space [17.6gb], spins? [unknown], types [rootfs]
	[2017-03-12T03:28:50,643][INFO ][o.e.e.NodeEnvironment    ] [g_bkxC0] heap size [1.9gb], compressed ordinary object pointers [true]
	[2017-03-12T03:28:50,646][INFO ][o.e.n.Node               ] [g_bkxC0] node name [g_bkxC0] derived from node ID; set [node.name] to override
	[2017-03-12T03:28:50,664][INFO ][o.e.n.Node               ] [g_bkxC0] version[5.0.0], pid[39660], build[253032b/2016-10-26T04:37:51.531Z], OS[Linux/3.10.0-514.6.2.el7.x86_64/amd64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_121/25.121-b13]
	[2017-03-12T03:28:52,650][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [aggs-matrix-stats]
	[2017-03-12T03:28:52,651][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [ingest-common]
	[2017-03-12T03:28:52,651][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [lang-expression]
	[2017-03-12T03:28:52,651][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [lang-groovy]
	[2017-03-12T03:28:52,651][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [lang-mustache]
	[2017-03-12T03:28:52,651][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [lang-painless]
	[2017-03-12T03:28:52,651][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [percolator]
	[2017-03-12T03:28:52,651][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [reindex]
	[2017-03-12T03:28:52,651][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [transport-netty3]
	[2017-03-12T03:28:52,651][INFO ][o.e.p.PluginsService     ] [g_bkxC0] loaded module [transport-netty4]
	[2017-03-12T03:28:52,652][INFO ][o.e.p.PluginsService     ] [g_bkxC0] no plugins loaded
	[2017-03-12T03:28:56,773][INFO ][o.e.n.Node               ] [g_bkxC0] initialized
	[2017-03-12T03:28:56,773][INFO ][o.e.n.Node               ] [g_bkxC0] starting ...
	[2017-03-12T03:28:57,072][INFO ][o.e.t.TransportService   ] [g_bkxC0] publish_address {10.201.23.210:9300}, bound_addresses {10.201.23.210:9300}
	[2017-03-12T03:28:57,075][INFO ][o.e.b.BootstrapCheck     ] [g_bkxC0] bound or publishing to a non-loopback or non-link-local address, enforcing bootstrap checks
	ERROR: bootstrap checks failed
	max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536]
	max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
	[2017-03-12T03:28:57,165][INFO ][o.e.n.Node               ] [g_bkxC0] stopping ...
	[2017-03-12T03:28:57,180][INFO ][o.e.n.Node               ] [g_bkxC0] stopped
	[2017-03-12T03:28:57,181][INFO ][o.e.n.Node               ] [g_bkxC0] closing ...
	[2017-03-12T03:28:57,197][INFO ][o.e.n.Node               ] [g_bkxC0] closed

修改sysctl.conf文件，添加vm.max_map_count配置

	vi /etc/sysctl.conf
	##文件追加配置
	vm.max_map_count = 262144

解决“max file descriptors [4096] for elasticsearch process likely too low”

	sudo vim /etc/security/limits.conf
加入以下两行：(kevin这里为用户名)

	kevin hard nofile 65536
	kevin soft nofile 65536

再次启动就正常了。

以上过程中，如果遇到网络端口不可用，还需要开发linux的端口访问权限，比如：

	firewall-cmd --zone=public --add-port=9200/tcp --permanent
	firewall-cmd --reload