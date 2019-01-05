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
