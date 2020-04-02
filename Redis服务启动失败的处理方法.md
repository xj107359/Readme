Redis服务启动失败的处理方法

	[root@localhost ~]# find / -name 'dump.rdb'
	/dump.rdb
	/root/dump.rdb

	# Delete dump file
	rm -f /dump.rdb
	rm -f /root/dump.rdb
	rm -rf /var/run/redis.pid
	
	# Restart server
	service redis restart
	
	# check port
	lsof -i:6379
