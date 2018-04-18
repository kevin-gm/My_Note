1. 查看当前系统内核

		uname -sr

2. CentOS 允许使用 ELRepo，这是一个第三方仓库，可以将内核升级到最新版本，要在CentOS中允许使用ELRepo，需要执行以下两条命令

		rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
		rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

3. 查看当前可用的内核包

		yum --disablerepo="*" --enablerepo="elrepo-kernel" list available

4. 选择一个进行安装，一般选择主线的稳定内核。

		yum --enablerepo=elrepo-kernel install kernel-ml

5. 当前重启机器，已经可以生效，但是为了让新安装的内核成为默认的启动项，先继续进行grub配置。

6. 查看当前系统有几个内核

		cat /boot/grub2/grub.cfg |grep menuentry

7. 设置默认启动内核

		grub2-set-default "CentOS Linux (3.10.0-327.el7.x86_64)7 (Core)"

8. 验证是否设置成功，下述命令应该显示修改配置后的内核版本。

	grub2-editenv list

9. 重启机器，再次查看当前系统内核，已经生效。

		init 6
		或者
		reboot