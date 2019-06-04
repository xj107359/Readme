# CentOS 7 上 crontab 的使用

## 服务

查看cron的状态，设为开机启动

查看状态

	[root@localhost ~]# systemctl status crond

设为开机启动

	[root@localhost ~]# systemctl enable crond

启动crond服务

	[root@localhost ~]# systemctl start crond

## 配置

cron的主配置文件是/etc/crontab，配置文件基本格式 :

| *  | *    | *  | *    | *  | root | cmd  	|
| -- | ---- | -- | ---- | -- | ---- | ----- |
| 分	 | 时	| 日	 | 月	| 周	 | 用户	| 命令	|

- 第1列表示分钟1～59 每分钟用*或者 */1表示
- 第2列表示小时1～23（0表示0点）
- 第3列表示日期1～31
- 第4列表示月份1～12
- 第5列标识号星期0～6（0表示星期天）
- 第6列运行命令的用户
- 第7列要运行的命令

## 示例

	# 每天，每30分钟执行一次 mycommand命令
	*/30 * * * root /usr/local/mycommand.sh
	
	# 每天凌晨三点，执行命令脚本，PS:这里由于第一个的分钟没有设置，那么就会每天凌晨3点的每分钟都执行一次命令
	* 3 * * * root /usr/local/mycommand.sh

	# 这样就是每天凌晨三点整执行一次命令脚本
	00 3 * * * root /usr/local/mycommand.sh

	# 每天11点到13点之间，每10分钟执行一次命令脚本，这一种用法也很常用
	*/10 11-13 * * * root /usr/local/mycommand.sh

	# 每小时的10-30分钟，每分钟执行一次命令脚本，共执行20次
	10-30 * * * * root /usr/local/mycommand.sh

	# 每小时的10,30分钟，分别执行一次命令脚本，共执行2次
	10,30 * * * * * root /usr/local/mycommand.sh
	
	# 每晚的21:30 重启apache
	30 21 * * * root /usr/local/etc/rc.d/lighttpd restart

	# 每月1、10、22日的4 : 45重启apache
	45 4 1,10,22 * * root /usr/local/etc/rc.d/lighttpd restart

	# 每周六、周日的1 : 10重启apache
	10 1 * * 6,0 root /usr/local/etc/rc.d/lighttpd restart

	# 每天18 : 00至23 : 00之间每隔30分钟重启apache
	0,30 18-23 * * * root /usr/local/etc/rc.d/lighttpd restart

	# 晚上11点到早上7点之间，每隔一小时重启apache
	* 23-7/1 * * * /usr/local/etc/rc.d/lighttpd restart

	# 每一小时重启apache
	* */1 * * * /usr/local/etc/rc.d/lighttpd restart

	# 每月的4号与每周一到周三的11点重启apache
	0 11 4 * mon-wed /usr/local/etc/rc.d/lighttpd restart

	# 一月一号的4点重启apache
	0 4 1 jan * /usr/local/etc/rc.d/lighttpd restart

	# 每半小时同步一下时间
	*/30 * * * * /usr/sbin/ntpdate 210.72.145.44




