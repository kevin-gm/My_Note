1.使用root用户执行以下命令:

  
    wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
 
    chmod +x shadowsocks.sh
 
    ./shadowsocks.sh 2>&1 | tee shadowsocks.log
 
 如果是SSR，则将上述命令的shadowsocks  替换为  shadowsocksR，即命令为
 
    wget --no-check-certificate -O shadowsocksR.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
 
- 提示输入密码: 如果不输入密码则默认密码为:teddysun.com
- 提示输入端口: 默认端口8989
- 提示输入加密方式: 选择aes-256-cfb 默认aes-256-gcm
- 如果是SSR，协议选择：auth_sha1_v4_compatible
- obfs(混淆方式)选择：http_simple_compatible

 
 
安装完成后，脚本提示如下：
> 
  Congratulations, Shadowsocks-python server install completed!
  Your Server IP        :your_server_ip
  Your Server Port      :your_server_port
  Your Password         :your_password
  Your Encryption Method:your_encryption_method

  Welcome to visit:https://teddysun.com/342.html
  Enjoy it!
 
 
卸载方法：
使用root用户登录，运行以下命令：
 
    ./shadowsocks.sh uninstall
    
    
单用户配置文件示例，配置文件路径： ```/etc/shadowsocks.json```
 
    {
        "server":"0.0.0.0",
        "server_port":your_server_port,
        "local_address":"127.0.0.1",
        "local_port":1080,
        "password":"your_password",
        "timeout":300,
        "method":"your_encryption_method",
        "fast_open": false
    }
 
多用户多端口配置文件示例，配置文件路径： ```/etc/shadowsocks.json```
 
    {
        "server":"0.0.0.0",
        "local_address":"127.0.0.1",
        "local_port":1080,
        "port_password":{
             "8989":"password0",
             "9001":"password1",
             "9002":"password2",
             "9003":"password3",
             "9004":"password4"
        },
        "timeout":300,
        "method":"your_encryption_method",
        "fast_open": false
    }
 
 
启动:  ```/etc/init.d/shadowsocks start```
停止： ```/etc/init.d/shadowsocks stop```
重启： ```/etc/init.d/shadowsocks restart```
状态： ```/etc/init.d/shadowsocks status```
