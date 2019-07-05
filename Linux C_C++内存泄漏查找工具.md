# Linux C/C++ 内存泄漏查找工具

前一段时间发现新编译的代码一运行就狂吃内存，铁定是内存泄露了。于是用了好久没用的神奇Valgrind进行检查

	[root@localhost ~]# valgrind --tool=memcheck --leak-check=full asterisk

给出如下提示：

>
	==11040== 264 (168 direct, 96 indirect) bytes in 1 blocks are definitely lost in loss record 10,846 of 12,349
	==11040==    at 0x4C29BC3: malloc (vg_replace_malloc.c:299)
	==11040==    by 0x6BD5E94: my_malloc (in /usr/lib64/mysql/libmysqlclient.so.18.0.0)
	==11040==    by 0x6BA5F65: mysql_store_result (in /usr/lib64/mysql/libmysqlclient.so.18.0.0)
	==11040==    by 0x4CA6D7: xu_db_gettree_common.constprop.10 (db.c:814)
	==11040==    by 0x4CAA93: handle_cli_database_show (db.c:1208)
	==11040==    by 0x4BBFF2: ast_cli_command_full (cli.c:2833)
	==11040==    by 0x4BC172: ast_cli_command_multiple_full (cli.c:2860)
	==11040==    by 0x458C20: netconsole (asterisk.c:1423)
	==11040==    by 0x59CEA8: dummy_start (utils.c:1249)
	==11040==    by 0x7084DD4: start_thread (in /usr/lib64/libpthread-2.17.so)
	==11040==    by 0x801FEAC: clone (in /usr/lib64/libc-2.17.so)

根据提示发现mysql_store_result分配的对象没有释放造成的，添加释放后正常。

	mysql_free_result(result);

总的来说还是对C/C++不熟o(一︿一+)o

