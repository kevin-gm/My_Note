转载：https://www.cnblogs.com/bigbrotherer/p/7241845.html

### 下载并安装MySQL官方的 Yum Repository

	[root@localhost ~]# wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm

### 使用yum安装Repository

	[root@localhost ~]# yum -y install mysql57-community-release-el7-10.noarch.rpm

### 安装mysql

	[root@localhost ~]# yum -y install mysql-community-server

最后显示：

	Installed:
	  mysql-community-libs.x86_64 0:5.7.21-1.el7                  mysql-community-libs-compat.x86_64 0:5.7.21-1.el7                  mysql-community-server.x86_64 0:5.7.21-1.el7                 
	
	Dependency Installed:
	  mysql-community-client.x86_64 0:5.7.21-1.el7                      mysql-community-common.x86_64 0:5.7.21-1.el7                      numactl-libs.x86_64 0:2.0.9-6.el7_2                     
	
	Replaced:
	  mariadb-libs.x86_64 1:5.5.52-1.el7                                                                                                                                                           
	
	Complete!

### 数据源设置
启动mysql

	systemctl start  mysqld.service

查看MySQL运行状态

	[root@VM_0_14_centos mysql]# systemctl status mysqld.service
	● mysqld.service - MySQL Server
	   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
	   Active: active (running) since Tue 2018-04-03 17:09:31 CST; 8s ago
	     Docs: man:mysqld(8)
	           http://dev.mysql.com/doc/refman/en/using-systemd.html
	  Process: 15558 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
	  Process: 15480 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
	 Main PID: 15562 (mysqld)
	   CGroup: /system.slice/mysqld.service
	           └─15562 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

	Apr 03 17:09:20 VM_0_14_centos systemd[1]: Starting MySQL Server...
	Apr 03 17:09:31 VM_0_14_centos systemd[1]: Started MySQL Server.

获取初始密码

	[root@VM_0_14_centos mysql]# grep "password" /var/log/mysqld.log
	2018-04-03T09:09:21.923707Z 1 [Note] A temporary password is generated for root@localhost: ws_LmrkaR9Z;

登录并修改root密码

	[root@VM_0_14_centos mysql]# mysql -uroot -p
	mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new password';

此时还有一个问题，就是因为安装了Yum Repository，以后每次yum操作都会自动更新，需要把这个卸载掉：

[root@localhost ~]# yum -y remove mysql57-community-release-el7-10.noarch