# Centos 7 下 Nagios 的配置

## 需要监视的资源

- Web服务
- Database服务
- PBX服务
- 硬盘空间
- 中继状态

## OID列表

| OID 							| 描述 				| 级别 			| 举例														|
| ----------------------------- | ----------------- | ------------- | --------------------------------------------------------- |
| UCD-SNMP-MIB::ucdavis.1.101	| Web服务正常 		| informational	| ucdavis.1.101 => Web server is OK.						|
| UCD-SNMP-MIB::ucdavis.1.102	| Web服务停止 		| critical 		| ucdavis.1.102 => Web server Crashed.						|
| UCD-SNMP-MIB::ucdavis.1.103	| Database服务正常 	| informational	| ucdavis.1.103 => Database server is OK.					|
| UCD-SNMP-MIB::ucdavis.1.104	| Database服务停止 	| critical	 	| ucdavis.1.104 => Database server Crashed.					|
| UCD-SNMP-MIB::ucdavis.1.105	| PBX服务正常 		| informational	| ucdavis.1.105 => PBX server is OK.						|
| UCD-SNMP-MIB::ucdavis.1.106	| PBX服务停止 		| critical	 	| ucdavis.1.106 => PBX server Crashed.						|
| UCD-SNMP-MIB::ucdavis.1.107	| 硬盘空间正常 		| informational	| ucdavis.1.107 => There is enough disk space now, free space 428188MB	|
| UCD-SNMP-MIB::ucdavis.1.108	| 硬盘空间不足 		| warning	 	| ucdavis.1.108 => Free disk space is less than 20%, free space 5670MB	|
| UCD-SNMP-MIB::ucdavis.1.109	| 硬盘空间已满 		| critical	 	| ucdavis.1.109 => Free disk space is less than 10%, free space 40MB	|
| UCD-SNMP-MIB::ucdavis.1.110	| PSTN中继状态正常 	| informational	| ucdavis.1.110 => All PSTN trunks are OK.					|
| UCD-SNMP-MIB::ucdavis.1.111	| PSTN中继状态异常 	| critical	 	| ucdavis.1.111 => 1 PSTN trunk(s) are RED.\r\n7,FXO		|
| UCD-SNMP-MIB::ucdavis.1.112	| SIP中继状态正常 	| informational	| ucdavis.1.112 => All SIP trunks are OK.					|
| UCD-SNMP-MIB::ucdavis.1.113	| SIP中继状态异常 	| critical	 	| ucdavis.1.113 => 1 SIP trunk(s) are RED.\r\n192.9.201.50	|

## 提权

- /etc/rc.d/init.d/nagios

在

	start)
		echo -n "Starting nagios:"

后面增加

	rm -f /tmp/nagios_last_*

## 配置文件


- /etc/nagios/commands.cfg

文件如下：



- /etc/nagios/commands.cfg

文件如下：

	define command{
	        command_name    check_apache_xu
	        command_line    $USER1$/send_apache_notify_xu
	}
	
	define command{
	        command_name    check_mysql_xu
	        command_line    $USER1$/send_mysql_notify_xu
	}
	
	define command{
	        command_name    check_asterisk_xu
	        command_line    $USER1$/send_asterisk_notify_xu
	}
	
	define command{
	        command_name    check_disk_xu
	        command_line    $USER1$/send_disk_notify_xu
	}
	
	define command{
	        command_name    check_pstn_trunk_xu
	        command_line    $USER1$/send_pstn_trunk_notify_xu
	        }
	
	define command{
	        command_name    check_sip_trunk_xu
	        command_line    $USER1$/send_sip_trunk_notify_xu
	        }
	
	define command{
	        command_name    null-notify-by-snmp
	        command_line		echo "NULL"
	}

- /etc/nagios/services.cfg

文件如下：

	define service{
	        host_name               nagios-server
	        service_description     apache
	        is_volatile             0
	        check_period            24x7
	        max_check_attempts      1
	        normal_check_interval   1
	        retry_check_interval    1
	        contact_groups          sagroup
	        notification_interval   1
	        notification_period     24x7
	        notification_options    w,u,c,r
		event_handler           null-notify-by-snmp
	        event_handler_enabled   1
	        check_command           check_apache_xu
	}
	
	define service{
	        host_name               nagios-server
	        service_description     mysql
	        is_volatile             0
	        check_period            24x7
	        max_check_attempts      1
	        normal_check_interval   1
	        retry_check_interval    1
	        contact_groups          sagroup
	        notification_interval   1
	        notification_period     24x7
	        notification_options    w,u,c,r
		event_handler           null-notify-by-snmp
	        event_handler_enabled   1
	        check_command           check_mysql_xu
	}
	
	define service{
	        host_name               nagios-server
	        service_description     asterisk
	        is_volatile             0
	        check_period            24x7
	        max_check_attempts      1
	        normal_check_interval   1
	        retry_check_interval    1
	        contact_groups          sagroup
	        notification_interval   1
	        notification_period     24x7
	        notification_options    w,u,c,r
		event_handler           null-notify-by-snmp
	        event_handler_enabled   1
	        check_command           check_asterisk_xu
	}
	
	define service{
	        host_name               nagios-server
	        service_description     disk
	        is_volatile             0
	        check_period            24x7
	        max_check_attempts      1
	        normal_check_interval   1
	        retry_check_interval    1
	        contact_groups          sagroup
	        notification_interval   1
	        notification_period     24x7
	        notification_options    w,u,c,r
		event_handler           null-notify-by-snmp
	        event_handler_enabled   1
	        check_command           check_disk_xu
	}
	
	define service{
	        host_name               nagios-server
	        service_description     pstn trunks
	        is_volatile             0
	        check_period            24x7
	        max_check_attempts      1
	        normal_check_interval   1
	        retry_check_interval    1
	        contact_groups          sagroup
	        notification_interval   1
	        notification_period     24x7
	        notification_options    w,u,c,r
	        event_handler           trunks-notify-by-snmp
	        event_handler_enabled   1
	        check_command           check_pstn_trunk_xu
	}
	
	define service{
	        host_name               nagios-server
	        service_description     sip trunks
	        is_volatile             0
	        check_period            24x7
	        max_check_attempts      1
	        normal_check_interval   1
	        retry_check_interval    1
	        contact_groups          sagroup
	        notification_interval   1
	        notification_period     24x7
	        notification_options    w,u,c,r
	        event_handler           trunks-notify-by-snmp
	        event_handler_enabled   1
	        check_command           check_sip_trunk_xu
	}

- /usr/lib/nagios/plugins/send_apache_notify_xu

文件如下：

	#!/bin/sh
	OID=enterprises.2021.1
	serviceName=Web
	procName=httpd
	bInit=false
	cache="/tmp/nagios_last_${procName}"
	
	if [ ! -f $cache ]; then
		echo "Running for the first time"
		bInit=true
		touch "$cache"
	else
		chmod 777 "$cache"
	fi
	
	for line in `cat $cache`
	do
	    lastStatus=$line
	done
	echo "lastStatus: $lastStatus"
	
	res=$(/usr/lib/nagios/plugins/check_procs -w 1: -c 1:1024 -C $procName)
	echo $res
	status=$(echo $res | awk '{print $2}')
	# echo $status
	procs=$(echo $res | awk '{print $3}')
	echo "status: $status, procs: $procs"
	
	if [ "$lastStatus" != "$status" ];then
		$(echo $status > $cache)
		if [ "$status" == "OK:" ];then
			if [ $bInit == false ];then
				echo "$serviceName server status OK"
				for line in `cat /etc/server.cfg`
				do
				    IP=$line
						/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.101 $OID.101.1 s "$serviceName服务正常。"
				done
			fi
		else
			echo "$serviceName server crashed"
			for line in `cat /etc/server.cfg`
			do
			    IP=$line
					/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.102 $OID.102.1 s "$serviceName服务异常。"
			done
		fi
	else
		echo "status == lastStatus"
	fi

- /usr/lib/nagios/plugins/send_mysql_notify_xu

文件如下：

	#!/bin/sh
	OID=enterprises.2021.1
	serviceName=Database
	procName=mysqld
	bInit=false
	cache="/tmp/nagios_last_${procName}"
	
	if [ ! -f $cache ]; then
		echo "Running for the first time"
		bInit=true
		touch "$cache"
	else
		chmod 777 "$cache"
	fi
	
	for line in `cat $cache`
	do
	    lastStatus=$line
	done
	echo "lastStatus: $lastStatus"
	
	res=$(/usr/lib/nagios/plugins/check_procs -w 1: -c 1:1024 -C $procName)
	echo $res
	status=$(echo $res | awk '{print $2}')
	# echo $status
	procs=$(echo $res | awk '{print $3}')
	echo "status: $status, procs: $procs"
	
	if [ "$lastStatus" != "$status" ];then
		$(echo $status > $cache)
		if [ "$status" == "OK:" ];then
			if [ $bInit == false ];then
				echo "$serviceName server status OK"
				for line in `cat /etc/server.cfg`
				do
				    IP=$line
						/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.103 $OID.103.1 s "$serviceName服务正常。"
				done
			fi
		else
			echo "$serviceName server crashed"
			for line in `cat /etc/server.cfg`
			do
			    IP=$line
					/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.104 $OID.104.1 s "$serviceName服务异常。"
			done
		fi
	else
		echo "status == lastStatus"
	fi

- /usr/lib/nagios/plugins/send_asterisk_notify_xu

文件如下：

	#!/bin/sh
	OID=enterprises.2021.1
	serviceName=PBX
	procName=asterisk
	bInit=false
	cache="/tmp/nagios_last_${procName}"
	
	if [ ! -f $cache ]; then
		echo "Running for the first time"
		bInit=true
		touch "$cache"
	else
		chmod 777 "$cache"
	fi
	
	for line in `cat $cache`
	do
	    lastStatus=$line
	done
	echo "lastStatus: $lastStatus"
	
	res=$(/usr/lib/nagios/plugins/check_procs -w 1: -c 1:1024 -C $procName)
	echo $res
	status=$(echo $res | awk '{print $2}')
	# echo $status
	procs=$(echo $res | awk '{print $3}')
	echo "status: $status, procs: $procs"
	
	if [ "$lastStatus" != "$status" ];then
		$(echo $status > $cache)
		if [ "$status" == "OK:" ];then
			if [ $bInit == false ];then
				echo "$serviceName server status OK"
				for line in `cat /etc/server.cfg`
				do
				    IP=$line
						/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.105 $OID.105.1 s "$serviceName服务正常。"
				done
			fi
		else
			echo "$serviceName server crashed"
			for line in `cat /etc/server.cfg`
			do
			    IP=$line
					/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.106 $OID.106.1 s "$serviceName服务异常。"
			done
		fi
	else
		echo "status == lastStatus"
	fi

- /usr/lib/nagios/plugins/send_disk_notify_xu

文件如下：

	#!/bin/sh
	OID=enterprises.2021.1
	procName=disk
	bInit=false
	cache="/tmp/nagios_last_${procName}"
	
	if [ ! -f $cache ]; then
		echo "Running for the first time"
		bInit=true
		touch "$cache"
	else
		chmod 777 "$cache"
	fi
	
	for line in `cat $cache`
	do
	    lastStatus=$line
	done
	echo "lastStatus: $lastStatus"
	
	res=$(/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /)
	echo $res
	status=$(echo $res | awk '{print $2}')
	# echo $status
	space=$(echo $res | awk '{print $7 $8}')
	echo "status: $status, space: $space"
	
	if [ "$lastStatus" != "$status" ];then
		$(echo $status > $cache)
		if [ "$status" == "OK" ];then
			if [ $bInit == false ];then
				echo "Disk space OK"
				for line in `cat /etc/server.cfg`
				do
				    IP=$line
						/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.107 $OID.107.1 s "可用磁盘空间充裕, 剩余$space"
				done
			fi
		elif [ "$status" == "WARNING" ];then
			echo "Disk space WARNING"
			for line in `cat /etc/server.cfg`
			do
			    IP=$line
					/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.108 $OID.108.1 s "可用磁盘空间不足20%, 剩余$space"
			done
		else
			echo "Disk space CRITICAL"
			for line in `cat /etc/server.cfg`
			do
			    IP=$line
					/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.109 $OID.109.1 s "可用磁盘空间不足10%, 剩余$space"
			done
		fi
	else
		echo "status == lastStatus"
	fi

- /usr/lib/nagios/plugins/send_pstn_trunk_notify_xu

文件如下：

	#!/bin/sh
	OID=enterprises.2021.1
	procName=pstn_trunk
	bInit=false
	cache="/tmp/nagios_last_${procName}"
	
	if [ ! -f $cache ]; then
		echo "Running for the first time"
		bInit=true
		touch "$cache"
	else
		chmod 777 "$cache"
	fi
	
	for line in `cat $cache`
	do
	    lastStatus=$line
	done
	echo "lastStatus: $lastStatus"
	
	status=$(lsdahdi | grep RED | wc -l)
	echo "status: $status"
	
	if [ "$lastStatus" != "$status" ];then
		$(echo $status > $cache)
		if [ $status -eq 0 ];then
			if [ $bInit == false ];then
				echo "PSTN trunk status OK"
				for line in `cat /etc/server.cfg`
				do
				    IP=$line
						/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.110 $OID.110.1 s "所有PSTN中继正常。"
				done
			fi
		else
			echo "PSTN trunk status CRITICAL"
			deads=$(lsdahdi | grep RED |awk '{print $1 "," $2}')
			echo "deads trunks: $deads"
			msg="${status}条PSTN中继异常：
	"
			for dead in $deads
			do
			    msg="${msg}${dead}
	"
			done
			echo $msg
			
			for line in `cat /etc/server.cfg`
			do
			    IP=$line
					/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.111 $OID.111.1 s "$msg"
			done
		fi
	else
		echo "status == lastStatus"
	fi

- /usr/lib/nagios/plugins/send_sip_trunk_notify_xu

文件如下：

	#!/bin/sh
	OID=enterprises.2021.1
	procName=sip_trunk
	bInit=false
	cache="/tmp/nagios_last_${procName}"
	
	if [ ! -f $cache ]; then
		echo "Running for the first time"
		bInit=true
		touch "$cache"
	else
		chmod 777 "$cache"
	fi
	
	for line in `cat $cache`
	do
	    lastStatus=$line
	done
	echo "lastStatus: $lastStatus"
	
	status=$(sudo /usr/sbin/asterisk -rx"customtrunkstat" | grep CTRUNK_DEAD | wc -l)
	echo "status: $status"
	
	if [ "$lastStatus" != "$status" ];then
		$(echo $status > $cache)
		if [ $status -eq 0 ];then
			if [ $bInit == false ];then
				echo "SIP trunk status OK"
				for line in `cat /etc/server.cfg`
				do
				    IP=$line
						/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.112 $OID.112.1 s "所有SIP中继正常。"
				done
			fi
		else
			echo "SIP trunk status CRITICAL"
			deads=$(sudo /usr/sbin/asterisk -rx"customtrunkstat" | grep CTRUNK_DEAD | awk '{print $2}' | sed 's/CTRUNK_DEAD//g')
			echo "deads trunks: $deads"
			msg="${status}条SIP中继异常：
	"
			for dead in $deads
			do
			    msg="${msg}${dead}
	"
			done
			echo $msg
			for line in `cat /etc/server.cfg`
			do
			    IP=$line
					/usr/bin/snmptrap -IR -v 2c -c public $IP '' $OID.113 $OID.113.1 s "$msg"
			done
		fi
	else
		echo "status == lastStatus"
	fi