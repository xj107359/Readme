	[root@localhost ~]# watch -n 1 -d "mysql -e 'show full processlist;'"

>
	Every 1.0s: mysql -e 'show full processlist;'                Mon May 27 15:47:32 2019
>
	Id      User    Host    db      Command Time    State   Info
	1       root    192.9.203.192:59900     asteriskreal    Sleep   4               NULL
	21      root    localhost       asterisk        Sleep   24              NULL
	37      root    localhost       NULL    Query   0       NULL    show full processlist