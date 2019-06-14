# 获取CentOS SNMP数据的方法(Get)

## 一、	SNMP官方OID及数据获取方法

### 1、	获取系统信息

>
	OID	.1.3.6.1.2.1.1.1.0
	MIB	RFC1213-MIB
	备注	sysDescr
	节点	1(iso).3(org).6(dod).1(internet).2(mgmt).1(mib-2).1(system).1(sysDescr).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.1.1.0
	SNMPv2-MIB::sysDescr.0 = STRING: Linux localhost.localdomain 2.6.18-194.3.1.el5 #1 SMP Thu May 13 13:09:10 EDT 2010 i686

### 2、	获取系统运行时间

>
	OID	.1.3.6.1.2.1.25.1.1.0
	MIB	HOST-RESOURCES-MIB
	备注	hrSystemUptime
	节点	1(iso).3(org).6(dod).1(internet).2(mgmt).1(mib-2).25(host).1(hrSystem).1(hrSystemUptime).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.25.1.1.0
	HOST-RESOURCES-MIB::hrSystemUptime.0 = Timeticks: (62644520) 7 days, 6:00:45.20

### 3、	获取SNMP服务运行时间

>
	OID	.1.3.6.1.2.1.1.3.0
	MIB	RFC1213-MIB
	备注	sysUpTime
	节点	1(iso).3(org).6(dod).1(internet).2(mgmt).1(mib-2).1(system).3(sysUpTime).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.1.3.0
	DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (1568396) 4:21:23.96

### 4、	获取主机名

>
	OID	.1.3.6.1.2.1.1.5.0
	MIB	RFC1213-MIB
	备注	sysName
	节点	1(iso).3(org).6(dod).1(internet).2(mgmt).1(mib-2).1(system).5(sysName).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.1.5.0
	SNMPv2-MIB::sysName.0 = STRING: localhost.localdomain

### 5、	获取登录用户数

>
	OID	.1.3.6.1.2.1.25.1.5.0
	MIB	HOST-RESOURCES-MIB
	备注	hrSystemNumUsers
	节点	1(iso).3(org).6(dod).1(internet).2(mgmt).1(mib-2).25(host).1(hrSystem).5(hrSystemNumUsers).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.25.1.5.0
	HOST-RESOURCES-MIB::hrSystemNumUsers.0 = Gauge32: 2

### 6、	获取总进程数

>
	OID	.1.3.6.1.2.1.25.1.6.0
	MIB	HOST-RESOURCES-MIB
	备注	hrSystemProcesses
	节点	1(iso).3(org).6(dod).1(internet).2(mgmt).1(mib-2).25(host).1(hrSystem).6(hrSystemProcesses).0
	请求方式	GET
	获取数据：
	/bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.25.1.6.0
	HOST-RESOURCES-MIB::hrSystemProcesses.0 = Gauge32: 121

### 7、	获取空闲CPU百分比

>
	OID	.1.3.6.1.4.1.2021.11.11.0
	MIB	UCD-SNMP-MIB
	备注	ssCpuIdle
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).11(systemStats).11(ssCpuIdle).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.11.11.0
	UCD-SNMP-MIB::ssCpuIdle.0 = INTEGER: 99

### 8、	获取系统占用CPU百分比

>
	OID	.1.3.6.1.4.1.2021.11.10.0
	MIB	UCD-SNMP-MIB
	备注	ssCpuSystem
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).11(systemStats).10(ssCpuSystem).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.11.10.0
	UCD-SNMP-MIB::ssCpuSystem.0 = INTEGER: 0

### 9、	获取用户占用CPU百分比

>
	OID	.1.3.6.1.4.1.2021.11.9.0
	MIB	UCD-SNMP-MIB
	备注	ssCpuUser
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).11(systemStats).9(ssCpuUser).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.11.9.0
	UCD-SNMP-MIB::ssCpuUser.0 = INTEGER: 0

### 10、	获取1分钟内CPU负载平均值

>
	OID	.1.3.6.1.4.1.2021.10.1.3.1
	MIB	UCD-SNMP-MIB
	备注	ssCpuUser
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).10(laTable).1(laEntry).3(laLoad).1(1minLoad)
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.10.1.3.1
	UCD-SNMP-MIB::laLoad.1 = STRING: 0.01

### 11、	获取5分钟内CPU负载平均值

>
	OID	.1.3.6.1.4.1.2021.10.1.3.2
	MIB	UCD-SNMP-MIB
	备注	ssCpuUser
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).10(laTable).1(laEntry).3(laLoad).2(5minLoad)
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.10.1.3.2
	UCD-SNMP-MIB::laLoad.2 = STRING: 0.03

### 12、	获取15分钟内CPU负载平均值

>
	OID	.1.3.6.1.4.1.2021.10.1.3.3
	MIB	UCD-SNMP-MIB
	备注	ssCpuUser
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).10(laTable).1(laEntry).3(laLoad).3(15minLoad)
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.10.1.3.3
	UCD-SNMP-MIB::laLoad.3 = STRING: 0.00

### 13、	获取内存总数

>
	OID	.1.3.6.1.4.1.2021.4.5.0
	MIB	UCD-SNMP-MIB
	备注	memTotalReal
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).4(memory).5(memTotalReal).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.4.5.0
	UCD-SNMP-MIB::memTotalReal.0 = INTEGER: 2065888 kB

### 14、	获取可用内存数

>
	OID	.1.3.6.1.4.1.2021.4.6.0
	MIB	UCD-SNMP-MIB
	备注	memAvailReal
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).4(memory).6(memAvailReal).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.4.6.0
	UCD-SNMP-MIB::memAvailReal.0 = INTEGER: 740188 kB

### 15、	获取缓冲区（buffer）内存数

>
	OID	.1.3.6.1.4.1.2021.4.14.0
	MIB	UCD-SNMP-MIB
	备注	memBuffer
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).4(memory).14(memBuffer).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.4.14.0
	UCD-SNMP-MIB::memBuffer.0 = INTEGER: 172296 kB

### 16、	获取缓存（cache）内存数

>
	OID	.1.3.6.1.4.1.2021.4.15.0
	MIB	UCD-SNMP-MIB
	备注	memCached
	节点	1(iso).3(org).6(dod).1(internet).4(private).1(enterprises).2021(ucdavis).4(memory).15(memCached).0
	请求方式	GET
	获取数据：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.4.15.0
	UCD-SNMP-MIB::memCached.0 = INTEGER: 781712 kB

### 17、	获取5分钟内硬盘读写I/O数

>
	获取数据方法一：
	[root@localhost ~]# cd /var/lib/cacti/scripts
	[root@localhost scripts]# php ./ss_net_snmp_disk_io.php 192.9.203.80
	reads:0 writes:60883
>
	获取数据方法二：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.13.15.1.1.2 | grep sd
	UCD-DISKIO-MIB::diskIODevice.17 = STRING: sda
	UCD-DISKIO-MIB::diskIODevice.18 = STRING: sda1
	UCD-DISKIO-MIB::diskIODevice.19 = STRING: sda2
	UCD-DISKIO-MIB::diskIODevice.20 = STRING: sda3
>
	由此得知sda对应的id为17
>
	使用如下命令查询过去5分钟硬盘I/O读取次数：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.13.15.1.1.5.17
	UCD-DISKIO-MIB::diskIOReads.17 = Counter32: 19467
>
	使用如下命令查询过去5分钟硬盘I/O写入次数：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.13.15.1.1.6.17
	UCD-DISKIO-MIB::diskIOWrites.17 = Counter32: 6952616

### 18、	获取5分钟内硬盘读写字节数

>
	获取数据方法一：
	[root@localhost ~]# cd /var/lib/cacti/scripts
	[root@localhost scripts]# php ./ss_net_snmp_disk_bytes.php 192.9.203.80
	bytesread:0 byteswritten:14123008
>
	获取数据方法二：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.13.15.1.1.2 | grep sd
	UCD-DISKIO-MIB::diskIODevice.17 = STRING: sda
	UCD-DISKIO-MIB::diskIODevice.18 = STRING: sda1
	UCD-DISKIO-MIB::diskIODevice.19 = STRING: sda2
	UCD-DISKIO-MIB::diskIODevice.20 = STRING: sda3
>
	由此得知sda对应的id为17
>
	使用如下命令查询过去5分钟硬盘I/O读取字节数：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.13.15.1.1.12.17
	UCD-DISKIO-MIB::diskIONReadX.17 = Counter64: 584299008
>
	使用如下命令查询过去5分钟硬盘I/O写入字节数：
	[root@localhost ~]# /bin/snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.13.15.1.1.13.17
	UCD-DISKIO-MIB::diskIONWrittenX.17 = Counter64: 126959739904

### 19、	获取分区“/”总容量、已使用容量及剩余容量

>
	获取数据方法：
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.25.2.3.1.3
	HOST-RESOURCES-MIB::hrStorageDescr.1 = STRING: Physical memory
	HOST-RESOURCES-MIB::hrStorageDescr.3 = STRING: Virtual memory
	HOST-RESOURCES-MIB::hrStorageDescr.6 = STRING: Memory buffers
	HOST-RESOURCES-MIB::hrStorageDescr.7 = STRING: Cached memory
	HOST-RESOURCES-MIB::hrStorageDescr.10 = STRING: Swap space
	HOST-RESOURCES-MIB::hrStorageDescr.31 = STRING: /
	HOST-RESOURCES-MIB::hrStorageDescr.35 = STRING: /boot
	HOST-RESOURCES-MIB::hrStorageDescr.36 = STRING: /dev/shm
>
	由此得知sda对应的id为31
>
	使用如下命令查询分区“/”的分配单元大小：
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80  .1.3.6.1.2.1.25.2.3.1.4.31
	HOST-RESOURCES-MIB::hrStorageAllocationUnits.31 = INTEGER: 4096 Bytes
>
	使用如下命令查询分区“/”的总容量：
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80  .1.3.6.1.2.1.25.2.3.1.5.31
	HOST-RESOURCES-MIB::hrStorageSize.31 = INTEGER: 58506191
	分区/的总容量为 58506191 × 4096 Bytes ÷ 1024 ÷ 1024 ÷ 1024 ≈ 223GB
>
	使用如下命令查询分区“/”已使用的容量：
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80  .1.3.6.1.2.1.25.2.3.1.6.31
	HOST-RESOURCES-MIB::hrStorageUsed.31 = INTEGER: 2150645
	分区“/”已使用的容量为 2150645 × 4096 Bytes ÷ 1024 ÷ 1024 ÷ 1024 ≈ 8GB
>
	分区“/”剩余容量为223GB - 8GB = 215GB

### 20、	获取网络设备“eth0”工作情况

>
	获取数据方法：
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.2.2.1.2
	IF-MIB::ifDescr.1 = STRING: lo
	IF-MIB::ifDescr.2 = STRING: eth0
	IF-MIB::ifDescr.3 = STRING: sit0
>
	由此得知eth0对应的id为2
>
	使用如下命令查询网络设备“eth0”的状态：
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.2.2.1.7.2
	IF-MIB::ifAdminStatus.2 = INTEGER: up(1)
	网卡状态为“up(1)”时表示工作正常，为“down (2)”时表示异常
>
	使用如下命令查询网络设备“eth0”的连接速度：
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.2.2.1.5.2
	IF-MIB::ifSpeed.2 = Gauge32: 100000000
	100000000b÷1000÷1000=100Mb
>
	使用如下命令查询网络设备“eth0”的总输入字节数：
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.2.2.1.10.2
	IF-MIB::ifInOctets.2 = Counter32: 1342296114
	1342296114b÷1000÷1000÷1000≈1GB
>
	使用如下命令查询网络设备“eth0”的总输出字节数：
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.2.1.2.2.1.16.2
	IF-MIB::ifOutOctets.2 = Counter32: 836563304
	836563304b÷1000÷1000≈836MB

## 二、	SNMP自定义OID及数据获取方法

### 1、	自定义OID方法

在文件“/etc/snmp/snmpd.conf”中添加形如“extend .1.3.6.1.4.1.2021.1 xuCheckHttpd /bin/sh /etc/snmp/xuCheckHttpd.sh”的代码以自定义OID，其中：

- extend表示SNMP采用EXTEN扩展方法
- .1.3.6.1.4.1.2021.1为OID节点，其中.1.3.6.1.4.1.2021.是专用扩展之用，不可更改，后面的可自定义，如1可改成任意数字
- xuCheckHttpd为自定义OID名，可任意
- /bin/sh /etc/snmp/xuCheckHttpd.sh为脚本绝对路径

### 2、	获取httpd服务运行情况

在文件“/etc/snmp/snmpd.conf”中添加 “extend .1.3.6.1.4.1.2021.1 xuCheckHttpd /bin/sh /etc/snmp/xuCheckHttpd.sh”

/etc/snmp/xuCheckHttpd.sh内容如下：

	#!/bin/sh
	NUMPIDS=`pgrep httpd | wc -l`
	echo "Httpd processes Num:$NUMPIDS"
	exit $NUMPIDS

httpd服务启动时获取数据如下：

>
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.1
	UCD-SNMP-MIB::ucdavis.1.1.0 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.2.1.2.12.120.117.67.104.101.99.107.72.116.116.112.100 = STRING: "/bin/sh"
	UCD-SNMP-MIB::ucdavis.1.2.1.3.12.120.117.67.104.101.99.107.72.116.116.112.100 = STRING: "/etc/snmp/xuCheckHttpd.sh"
	UCD-SNMP-MIB::ucdavis.1.2.1.4.12.120.117.67.104.101.99.107.72.116.116.112.100 = ""
	UCD-SNMP-MIB::ucdavis.1.2.1.5.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 5
	UCD-SNMP-MIB::ucdavis.1.2.1.6.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.2.1.7.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.2.1.20.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 4
	UCD-SNMP-MIB::ucdavis.1.2.1.21.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.3.1.1.12.120.117.67.104.101.99.107.72.116.116.112.100 = STRING: "Httpd processes Num:9"
	UCD-SNMP-MIB::ucdavis.1.3.1.2.12.120.117.67.104.101.99.107.72.116.116.112.100 = STRING: "Httpd processes Num:9"
	UCD-SNMP-MIB::ucdavis.1.3.1.3.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.3.1.4.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 9
	UCD-SNMP-MIB::ucdavis.1.4.1.2.12.120.117.67.104.101.99.107.72.116.116.112.100.1 = STRING: "Httpd processes Num:9"

httpd服务停止时获取数据如下：

>
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.1
	UCD-SNMP-MIB::ucdavis.1.1.0 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.2.1.2.12.120.117.67.104.101.99.107.72.116.116.112.100 = STRING: "/bin/sh"
	UCD-SNMP-MIB::ucdavis.1.2.1.3.12.120.117.67.104.101.99.107.72.116.116.112.100 = STRING: "/etc/snmp/xuCheckHttpd.sh"
	UCD-SNMP-MIB::ucdavis.1.2.1.4.12.120.117.67.104.101.99.107.72.116.116.112.100 = ""
	UCD-SNMP-MIB::ucdavis.1.2.1.5.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 5
	UCD-SNMP-MIB::ucdavis.1.2.1.6.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.2.1.7.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.2.1.20.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 4
	UCD-SNMP-MIB::ucdavis.1.2.1.21.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.3.1.1.12.120.117.67.104.101.99.107.72.116.116.112.100 = STRING: "Httpd processes Num:0"
	UCD-SNMP-MIB::ucdavis.1.3.1.2.12.120.117.67.104.101.99.107.72.116.116.112.100 = STRING: "Httpd processes Num:0"
	UCD-SNMP-MIB::ucdavis.1.3.1.3.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.1.3.1.4.12.120.117.67.104.101.99.107.72.116.116.112.100 = INTEGER: 0
	UCD-SNMP-MIB::ucdavis.1.4.1.2.12.120.117.67.104.101.99.107.72.116.116.112.100.1 = STRING: "Httpd processes Num:0"

3、	获取MySQL服务运行情况

在文件“/etc/snmp/snmpd.conf”中添加 “extend .1.3.6.1.4.1.2021.2 xuCheckMysqld /bin/sh /etc/snmp/xuCheckMysqld.sh”

/etc/snmp/xuCheckMysqld.sh内容如下：

	#!/bin/sh
	NUMPIDS=`pgrep mysqld | wc -l`
	echo "MySQL processes Num:$NUMPIDS"
	exit $NUMPIDS

MySQL服务启动时获取数据如下：

>
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.2
	UCD-SNMP-MIB::prEntry.0 = INTEGER: 1
	UCD-SNMP-MIB::prTable.2.1.2.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = STRING: "/bin/sh"
	UCD-SNMP-MIB::prTable.2.1.3.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = STRING: "/etc/snmp/xuCheckMysqld.sh"
	UCD-SNMP-MIB::prTable.2.1.4.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = ""
	UCD-SNMP-MIB::prTable.2.1.5.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 5
	UCD-SNMP-MIB::prTable.2.1.6.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 1
	UCD-SNMP-MIB::prTable.2.1.7.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 1
	UCD-SNMP-MIB::prTable.2.1.20.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 4
	UCD-SNMP-MIB::prTable.2.1.21.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 1
	UCD-SNMP-MIB::prTable.3.1.1.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = STRING: "MySQL processes Num:2"
	UCD-SNMP-MIB::prTable.3.1.2.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = STRING: "MySQL processes Num:2"
	UCD-SNMP-MIB::prTable.3.1.3.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 1
	UCD-SNMP-MIB::prTable.3.1.4.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 2
	UCD-SNMP-MIB::prTable.4.1.2.13.120.117.67.104.101.99.107.77.121.115.113.108.100.1 = STRING: "MySQL processes Num:2"

MySQL服务停止时获取数据如下：

>
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.2
	UCD-SNMP-MIB::prEntry.0 = INTEGER: 1
	UCD-SNMP-MIB::prTable.2.1.2.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = STRING: "/bin/sh"
	UCD-SNMP-MIB::prTable.2.1.3.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = STRING: "/etc/snmp/xuCheckMysqld.sh"
	UCD-SNMP-MIB::prTable.2.1.4.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = ""
	UCD-SNMP-MIB::prTable.2.1.5.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 5
	UCD-SNMP-MIB::prTable.2.1.6.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 1
	UCD-SNMP-MIB::prTable.2.1.7.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 1
	UCD-SNMP-MIB::prTable.2.1.20.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 4
	UCD-SNMP-MIB::prTable.2.1.21.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 1
	UCD-SNMP-MIB::prTable.3.1.1.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = STRING: "MySQL processes Num:0"
	UCD-SNMP-MIB::prTable.3.1.2.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = STRING: "MySQL processes Num:0"
	UCD-SNMP-MIB::prTable.3.1.3.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 1
	UCD-SNMP-MIB::prTable.3.1.4.13.120.117.67.104.101.99.107.77.121.115.113.108.100 = INTEGER: 0
	UCD-SNMP-MIB::prTable.4.1.2.13.120.117.67.104.101.99.107.77.121.115.113.108.100.1 = STRING: "MySQL processes Num:0"

4、	获取PBX服务运行情况

在文件“/etc/snmp/snmpd.conf”中添加 “extend .1.3.6.1.4.1.2021.3 xuCheckPbx /bin/sh /etc/snmp/xuCheckPbx.sh”

/etc/snmp/xuCheckPbx.sh内容如下：

	#!/bin/sh
	NUMPIDS=`pgrep asterisk | wc -l`
	echo "PBX processes Num:$NUMPIDS"
	exit $NUMPIDS

PBX服务启动时获取数据如下：

>
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.3
	UCD-SNMP-MIB::ucdavis.3.1.0 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.2.1.2.10.120.117.67.104.101.99.107.80.98.120 = STRING: "/bin/sh"
	UCD-SNMP-MIB::ucdavis.3.2.1.3.10.120.117.67.104.101.99.107.80.98.120 = STRING: "/etc/snmp/xuCheckPbx.sh"
	UCD-SNMP-MIB::ucdavis.3.2.1.4.10.120.117.67.104.101.99.107.80.98.120 = ""
	UCD-SNMP-MIB::ucdavis.3.2.1.5.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 5
	UCD-SNMP-MIB::ucdavis.3.2.1.6.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.2.1.7.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.2.1.20.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 4
	UCD-SNMP-MIB::ucdavis.3.2.1.21.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.3.1.1.10.120.117.67.104.101.99.107.80.98.120 = STRING: "PBX processes Num:2"
	UCD-SNMP-MIB::ucdavis.3.3.1.2.10.120.117.67.104.101.99.107.80.98.120 = STRING: "PBX processes Num:2"
	UCD-SNMP-MIB::ucdavis.3.3.1.3.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.3.1.4.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 2
	UCD-SNMP-MIB::ucdavis.3.4.1.2.10.120.117.67.104.101.99.107.80.98.120.1 = STRING: "PBX processes Num:2"

MySQL服务停止时获取数据如下：

>
	[root@localhost ~]# snmpwalk -v 2c -c public 192.9.203.80 .1.3.6.1.4.1.2021.3
	UCD-SNMP-MIB::ucdavis.3.1.0 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.2.1.2.10.120.117.67.104.101.99.107.80.98.120 = STRING: "/bin/sh"
	UCD-SNMP-MIB::ucdavis.3.2.1.3.10.120.117.67.104.101.99.107.80.98.120 = STRING: "/etc/snmp/xuCheckPbx.sh"
	UCD-SNMP-MIB::ucdavis.3.2.1.4.10.120.117.67.104.101.99.107.80.98.120 = ""
	UCD-SNMP-MIB::ucdavis.3.2.1.5.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 5
	UCD-SNMP-MIB::ucdavis.3.2.1.6.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.2.1.7.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.2.1.20.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 4
	UCD-SNMP-MIB::ucdavis.3.2.1.21.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.3.1.1.10.120.117.67.104.101.99.107.80.98.120 = STRING: "PBX processes Num:0"
	UCD-SNMP-MIB::ucdavis.3.3.1.2.10.120.117.67.104.101.99.107.80.98.120 = STRING: "PBX processes Num:0"
	UCD-SNMP-MIB::ucdavis.3.3.1.3.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 1
	UCD-SNMP-MIB::ucdavis.3.3.1.4.10.120.117.67.104.101.99.107.80.98.120 = INTEGER: 0
	UCD-SNMP-MIB::ucdavis.3.4.1.2.10.120.117.67.104.101.99.107.80.98.120.1 = STRING: "PBX processes Num:0"
