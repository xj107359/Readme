# CentOS 7 关闭 SELinux

## 查看状态

	[root@localhost ~]# getenforce
	Enforcing
	[root@localhost ~]# /usr/sbin/sestatus -v
	SELinux status:                 enabled

## 临时关闭

	##设置SELinux 成为permissive模式
	##setenforce 1 设置SELinux 成为enforcing模式
	[root@localhost ~]# setenforce 0

## 永久关闭

	vi /etc/selinux/config

>将SELINUX=enforcing改为SELINUX=disabled 

>设置后需要重启才能生效
