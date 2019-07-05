# top 查看特定服务状态

	top -d .5 -p `ps aux | grep asterisk | grep -v 'grep' | awk '{print $2;}'`