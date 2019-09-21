1. 先检查SELinux状态，如果已关闭则无需相关修改

    [root@vultr ~]# sestatus
    SELinux status:                 disabled

  如果状态为 enable，则执行 2，否则执行 3
  
2. 安装SELinux的管理工具 semanage(如果已安装，则无需此步骤)

    [root@vultr ~]# yum -y install semanage
    [root@vultr ~]# yum provides semanage
    
  安装运行semanage所需依赖工具包 policycoreutils-python
  
    [root@vultr ~]# yum -y install policycoreutils-python
    
  查询当前 ssh 服务端口
  
    [root@vultr ~]# semanage port -l|grep ssh
    
  向 SELinux 中添加 ssh 端口
  
    [root@vultr ~]# semanage port -a -t ssh_port_t -p tcp 22345
    
  重启 ssh 服务
  
    [root@vultr ~]# systemctl restart sshd.service    或者     service sshd restart
    
3. 从ssh的配置文件中，找到 port配置，并复制添加一行记录

    [root@vultr ~]# vi /etc/ssh/sshd_config
    
  结果为：
  >
      
    Port 22
    Port 22345
    #AddressFamily any
    
  重启sshd服务
  
    [root@vultr ~]# systemctl restart sshd
    
4. 防火墙配置，开放端口

  [root@vultr ~]# firewall-cmd --permanent--add-port=62231/tcp
  [root@vultr ~]# firewall-cmd --reload
