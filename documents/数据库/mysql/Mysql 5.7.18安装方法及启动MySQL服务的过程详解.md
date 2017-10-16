MySQL 是一个非常强大的关系型数据库。但有些初学者在安装配置的时候，遇到种种的困难，在此就不说安装过程了，说一下配置过程。在官网下载的MySQL时候，有msi格式和zip格式。Msi直接运行安装即可，zip则解压在自己喜欢的目录地址即可。在安装这两种的时候，都需要配置才能用。以下介绍主要是msi格式默认的地址：C:\Program Files\ mysql-5.7.18-win32。
         一．在安装或者解压后，需要配置环境变量，过程如下：我的电脑->属性->高级系统设置->高级->环境变量。在系统变量中点击path，然后选择编辑。把”C:\ProgramFiles\mysql-5.7.18-win32\bin;”追加进去。 这里需要意的是，不是覆盖！还有就是最好在最前面追加。不能漏掉英式分号“;”。（在中间也可以插进去，但切记一定要在“;”后面追加。）

二．点击开始菜单，搜索cmd.exe，左击以管理员身份运行。一定要以管理员身份运行！！！

如果直接运行cmd的话，输入mysqld -install时会出现因为权限不够而出现错误：Install/Remove of theService Denied!

正确输入mysqld –install按回车键时，有显示The service already exists!

三．接着输入net start mysql启动服务器，如果显示启动服务器失败，如下图。

这是因为5.7以上版本中，C:\Program Files\mysql-5.7+目录下没有data文件夹，在这，切记不要拷贝mysql其他版本的data文件夹，而是在窗口输入mysqld--initialize-insecure --user=mysql，需要注意的是“--”前面有一个空格，然后回车即可。（等待需要等半分钟，看电脑快慢）
  
（在C:\ProgramFiles\mysql-5.7.18-win32目录中，显示没有data文件夹。如下图）
       
 注：在5.7以下的版本中，需要在C:\ProgramFiles\mysql-5.*目录下的my-default.ini修改，右击编辑即可，把
basedir=……
datadir=…..
修改为
?
1
2
basedir= C:\Program Files\mysql-5.*
datadir= C:\Program Files\mysql-5.*\data
即可，其中“C:\Program Files\mysql-5.*”以实际安装位置确定
        四．在C:\ProgramFiles\mysql-5.7.18-win32目录中，显示data文件夹创建成功。

创建完成data后，在输入mysqld –install，然后按回车键（如果在步骤三启动就成功的，就不用再次输入）

     五。服务启动成功之后，就可以登录了，输入mysql -u root –p按回车键，出现Enter password，因为没有设置登录密码，所以什么都不用输入，直接按回车键即可

直接后回车后，如下图，到此mysql配置已经成功了！

在服务启动后，因为刚创建的 root 用户是空密码的，因此，需要先进行密码设定。可执行如下命令：
mysqladmin -u root -p password 此处输入新的密码
Enter password: 此处输入旧的密码
执行完以上两条命令后，只要 Enter password: 后输入的旧密码正确，则 root 用户的新密码就算设定成功了。此后，要想登录 root 用户，则都需要使用新密码。
注意：刚创建的 root 用户是空密码的，因此，在第一次修改 root 用户的密码时，在 Enter password: 后面不需要输入任何密码，直接回车即可。
至此，MySQL v5.7.18 的解压安装就已经全部完成，因此，需要把先前已经启的 MySQL 服务给停止掉，执行如下命令：
?
1
net stop mysql
登录并使用MySQL
前面已经完成对MySQL数据库的安装，只要安装成功后，就可以正常登录 root 用户，并进行数据的相关操作，如：建表、增、删、改、查等等。下面是简单的流程：
以管理员身份打开 cmd，并切到 mysql 安装目录的 bin 目录下
?
1
2
net start mysql        // 说明：该命令是启动 mysql 服务
mysql -u root -p        // 说明：该命令是登录 root 用户

Enter password: 先前设置的 root 用户的密码
正确登录后，就可以对数据进行操作了如：增、删、改、查等等。示例：
?
1
2
3
mysql> show databases;  // 显示所有数据库
mysql> select 语句............
...
不再使用数据库时，要退出用户，并停止服务，执行如下命令：
?
1
2
mysql> quit;
net stop mysql
删除数据库
如果不再想用mysql了，则可以执行如下命令：
?
1
mysqld --remove
以上所述是小编给大家介绍的Mysql 5.7.18安装方法及启动MySQL服务的过程详解，希望对大家有所帮助，如果大家有任何疑问请给我留言，小编会及时回复大家的。在此也非常感谢大家对脚本之家网站的支持！
原文链接：http://blog.csdn.net/qq_18145031/article/details/71335722
