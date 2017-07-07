一、状态或更新必要软件

	yum install -y gcc wget tcl make

二、下载redis
	
	1.wget http://download.redis.io/releases/redis-3.2.1.tar.gz
	2.从官网下载redis-3.2.1.tar.gz并上传

三、解压到当前目录

	tar -zxvf redis-3.2.1.tar.gz .

四、编译与安装

	cd redis-3.2.1
	make
	make PREFIX=/opt/soft/redis-3.2.1 install