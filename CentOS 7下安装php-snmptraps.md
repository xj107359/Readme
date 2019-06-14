# CentOS 7 下安装 php-snmptraps

>https://github.com/phpipam/php-snmptraps

## CentOS 7 关闭 SELinux

### 临时关闭

	##设置SELinux 成为permissive模式
	##setenforce 1 设置SELinux 成为enforcing模式
	[root@localhost ~]# setenforce 0

### 永久关闭

	vi /etc/selinux/config

>
将SELINUX=enforcing改为SELINUX=disabled 
>
设置后需要重启才能生效

## setup

### 1. install net-snmp, net-snmp-utils, apache, php, mysql, zip and unzip

	[root@localhost ~]# yum install net-snmp-utils zip unzip -y

### 2. unzip php-snmptraps-master.zip

	[root@localhost ~]# cd /var/www/html
	[root@localhost html]# unzip php-snmptraps-master.zip
	[root@localhost html]# chmod -R 777 php-snmptraps-master
	[root@localhost ~]# cd /var/www/html/php-snmptraps-master
	[root@localhost php-snmptraps-master]# mv -f ./* ../
	[root@localhost php-snmptraps-master]# mv .htaccess ../

### 3. vi /etc/snmp/snmptrapd.conf

	[root@localhost html]# which php
	/usr/bin/php

>
	...
	# listen on
	agentaddress my_ip_address:162
	# set php-snmptraps as default trap handler
	traphandle default /usr/bin/php /var/www/html/traphandler.php
	...

### 5. Import database schema

	[root@localhost ~]# mysql
	MariaDB [(none)]> create database snmptraps;
	Query OK, 1 row affected (0.01 sec)

	MariaDB [(none)]> GRANT ALL on snmptraps.* to snmptraps@localhost identified by "snmptraps";
	Query OK, 0 rows affected (0.03 sec)
	
	MariaDB [(none)]> flush privileges;
	Query OK, 0 rows affected (0.00 sec)
	
	MariaDB [(none)]> exit
	Bye
	
	[root@localhost ~]# mysql snmptraps < /var/www/html/db/SCHEMA.sql


### 6. Prepare and edit config file

	[root@localhost html]# cd /var/www/html/
	[root@localhost php-snmptraps-master]# cp config.dist.php config.php

vi /var/www/html/config.php
>
	$db['user'] = "snmptraps";
	$db['pass'] = "snmptraps";
	$db['name'] = "snmptraps";

### 7. Set mod_rewrite

mod_rewrite is required for snmptraps. Example here:

> http://phpipam.net/documents/prettified-links-with-mod_rewrite/

	[root@localhost ~]# httpd -M | grep rewrite_module
	 rewrite_module (shared)

	[root@localhost ~]# vi /etc/httpd/conf/httpd.conf
>
	<Directory "/var/www/html">
	...
	 AllowOverride All
	...
	</Directory>

	[root@localhost ~]# vi /var/www/html/.htaccess
>
	RewriteEngine On

### 8. start service

	[root@localhost ~]# systemctl restart httpd

	[root@localhost ~]# systemctl enable snmptrapd
	[root@localhost ~]# systemctl restart snmptrapd

### 9. view website
That should be it, fire up browser and login. Default user/pass is ***Admin/snmptraps***.

### 10. Debug

	[root@localhost ~]# systemctl status snmptrapd


## Send test message and debugging

To test snmptrap you can use the following command:

```snmptrap -v 2c -c public 192.9.203.192 '' .1.3.6.1.4.1.2636.4.1.1 .1.3.6.1.4.1.2636.4.1.1 s "Power supply failure"```

Check also the file you set in config.php for possible errors:

```tail -f /tmp/trap.txt```
