一、开放端口

  yum install firewalld
  
  systemctl unmask firewalld
  systemctl enable firewalld
  systemctl start firewalld
  
  firewall-cmd --zone=public --add-port=80/tcp --permanent

二、查看端口开放情况

  yum install nmap
  
  nmap 127.0.0.1

三、ls

### ls -l：以列表的形式显示文件

	[root@localhost dev]# ls -l
	total 0
	drwxr-xr-x 2 root root      520 Mar 15 05:56 char

drwxr-xr-x 2 root root      520 Mar 15 05:56 char  依次介绍

- [d][rwx][r-x][r-x]
	- [d]：文件类型
		- -：普通文件
		- d：目录文件
		- p：管理文件
		- l：链接文件
		- b：块设备文件
		- c：字符设备文件
		- s：套接字文件
	- [rwx]：权限。三个依次为，所有者权限 - 组用户权限 - 其他用户权限
		- r：读权限
		- w：写权限
		- x：可执行权限
		- -：无权限
- 2
	- 对于普通文件，链接数
	- 对于目录文件，第一级子目录数
- root
	- 用户名
- root
	- 组名
- 520
	- 文件大小，单位字节
- Mar 15 05:56
	- 最后修改时间
- char
	- 文件名


四、修改密码

	passwd username

五、CentOS8 重启network

	nmcli c reload
