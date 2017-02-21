#mysql 5.7.16 安装配置方法图文教程

##一、下载
从官网下载社会版本（MySQL Community Server ）：https://dev.mysql.com/downloads/
<div  align="center">    
	<img src="./images/1.png" width = "600" height = "150" alt="图片名称" 	align=center />
</div>

##二、安装
ZIP Archive版是免安装的。只要解压就行了。不需要安装

PS:解压后是没有“data”文件夹以及“my.ini”配置文件的，这个后续配置的
<div  align="center">    
	<img src="./images/2.png" width = "600" height = "150" alt="图片名称" 	align=center />
</div>

##三、配置
新建“my.ini”文件，并编辑如下内容：

	[mysql]
	# 设置mysql客户端默认字符集
	default-character-set=utf8 
	[mysqld]
	#设置3306端口
	port = 3306 
	# 设置mysql的安装目录
	basedir=D:\mysql\mysql-5.6.17-winx64
	# 设置mysql数据库的数据的存放目录
	datadir=D:\mysql\mysql-5.6.17-winx64\data
	# 允许最大连接数
	max_connections=200
	# 服务端使用的字符集默认为8比特编码的latin1字符集
	character-set-server=utf8
	# 创建新表时将使用的默认存储引擎
	default-storage-engine=INNODB

##四、安装mysql服务并启动（此处会生成data文件夹）
>以管理员身份打开cmd窗口
>
>切换目录至mysql解压文件的bin目录
>
>出入mysqld install回车
>
>输入net start mysql启动mysql服务
>
	>>此时很可能会提示“服务器无法启动，服务器没有报告任何错误”
	>>输入mysqld --initialize-insecure --user=mysql
	>>这时，应该会创建上述的data文件夹，并创建好默认数据库（用户名为root，密码为空），如果提示data文件夹不为空，则清空此文件夹重新执行命令
	>>再次启动mysql服务
<div  align="center">    
	<img src="./images/3.png" width = "500" height = "200" alt="图片名称" 	align=center />
</div>

##五、添加环境变量
上述步骤后，可以使用mysql -u root -p进行进入mysql数据库了，但是需要每次都切换到mysql的bin目录。为了方便，将该bin目录添加到环境变量。