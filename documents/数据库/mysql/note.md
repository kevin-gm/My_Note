1. 添加用户

  create user 'userName'@'localhost' identified by 'password';
  
  > 指定为localhost时，只能再本地登录，远程无法登录访问。此时可以指定需要登录MySQL服务器的IP地址，比如 234.23.34.45。或者直接将其设置为 %，允许任何远程连接。
    相关文档：https://dev.mysql.com/doc/refman/5.7/en/adding-users.html
    
 2. 给用户授权
 
   grant all privileges on ApolloPortalDB.* to 'apollo'@'234.23.34.45';
   
   > 为从234.23.34.45登录的apollo这个用户，授予ApolloPortalDB下所有数据表所有权限
   
   grant SELECT,INSERT,UPDATE,DELETE,CREATE,DROP on ApolloPortalDB.* to 'apollo'@'234.23.34.45';
   
   > 授予增删改查建表权限。
   
   相关文档：https://dev.mysql.com/doc/refman/5.7/en/adding-users.html
   
 3. 修改数据库的字符编码
 
    alter database ApolloConfigDB character set utf8mb4;


4. 在 /etc/my.cnf 文件指定binglog后，启动报错

   ```mysqld: File './mysql-bin.index' not found (Errcode: 13 - Permission denied)```

  原因是：
  > 通过yum方式安装mysql,会生成mysql:mysql用户组和用户，启动mysql默认是使用mysql用户。如果我们开启了慢log日志，而且我们使用service mysqld start启动mysql,会报如题所示的错误，根据提示我们知道在my.cnf默认配置指定的/var/lib/mysql这个目录下，存放着数据文件，/var/lib/mysql权限虽然是归mysql:mysql用户组和用户拥有，但是这个目录下的大多数文件，权限都是700,也就是说通过mysql用户来启动，却少权限，我们可以改变这些文件的权限，从而通过使用service mysqld start命令来启动mysql,但是也可以有另外一种方法来启动mysql,而不用改变/var/lib/mysql目录下的文件权限，这个命令就是mysqld --user=root,如果需要让这个命令在后台执行，可以使用mysqld --user=root &.
  
  使用 ```mysqld --user=root &``` 启动！！！！
