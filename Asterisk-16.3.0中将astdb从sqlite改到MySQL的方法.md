# Asterisk-16.3.0中将astdb从sqlite改到MySQL的方法

## 目的
分布式结构下，为了让多台Asterisk共享astdb中的数据，需要将astdb从sqlite改到MySQL，故需修改astdb相关代码。

## Asterisk版本
当前使用asterisk-16.3.0版本

## 相关源码文件
- include/asterisk/astdb.h

	该文件为astdb申明文件，该文件中定义的函数供Asterisk内部使用（**CRUD操作**）

- main/db.c

	该文件为astdb具体实现文件，包含加载、卸载，astdb.h中声明的函数，实现CLI的函数，及实现AMI的函数

- func_db.c

	该文件包含实现FUNCTION的函数

## 修改指南
以下介绍如何修改asterisk的代码并编译。

**1. 修改/main/Makefile**
在/main/Makefile L53

	ASTSSL_LIBS:=$(OPENSSL_LIB)
	AST_LIBS+=$(BKTR_LIB)
	AST_LIBS+=$(LIBXML2_LIB)
	AST_LIBS+=$(LIBXSLT_LIB)
	AST_LIBS+=$(SQLITE3_LIB)
	AST_LIBS+=$(ASTSSL_LIBS)
	AST_LIBS+=$(JANSSON_LIB)
	AST_LIBS+=$(URIPARSER_LIB)
	AST_LIBS+=$(UUID_LIB)
	AST_LIBS+=$(CRYPT_LIB)
	AST_LIBS+=$(AST_CLANG_BLOCKS_LIBS)
	AST_LIBS+=$(RT_LIB)
	AST_LIBS+=$(SYSTEMD_LIB)
后面增加

	AST_LIBS+=$(MYSQLCLIENT_LIB)

**2. 使用db.c替换main/db.c**

**3. 修改配置文件**

/etc/asterisk/res_config_mysql.conf

	[general]
	dbtype = mysql			# mysql或者sqlite
	dbhost = 192.9.203.180
	dbname = asteriskreal
	dbuser = root
	dbpass = 
	dbport = 3306
	dbcharset = latin1

*由于是在不同的设备上，故读取dbsock毫无意义*

## 修改情况
**1. include/asterisk/astdb.h中声明的函数**

| C函数  							| 进度 								 	|
| --------------------------------- | ------------------------------------- |
| *ast_db_put*  					| 修改 								 	|
| *ast_db_del*  					| 修改  									|
| *ast_db_deltree*  				| 修改  									|
| *ast_db_get*  					| 修改  									|
| *ast_db_get_allocated*  			| 修改  									|
| *ast_db_gettree*  				| 修改  									|
| ***ast_db_gettree_by_prefix*** 	| <font color="red">**不修改**</font> 	|
| *ast_db_freetree*  				| 修改  									|

**2. CLI(ast_cli_register_multiple)中声明的函数**

| CLI指令  													| C函数 							| 进度 		|
| --------------------------------------------------------- | ----------------------------- | --------- |
| *database put &lt;family&gt; &lt;key&gt; &lt;value&gt;* 	| *handle_cli_database_put* 	| 修改  		|
| *database del &lt;family&gt; &lt;key&gt;* 				| *handle_cli_database_del* 	| 修改  		|
| *database deltree &lt;family&gt; [key]* 					| *handle_cli_database_deltree* | 修改  		|
| *database get &lt;family&gt; &lt;key&gt;* 				| *handle_cli_database_get* 	| 修改  		|
| *database gettree &lt;family&gt; [key]* 					| *handle_cli_database_gettree* | **新增** 	|
| *database show [family [keytree]]* 						| *handle_cli_database_show* 	| 修改  		|
| *database showkey &lt;keytree&gt;* 						| *handle_cli_database_showkey* | 修改  		|
| ***database query "&lt;SQL Statement&gt;"*** | *handle_cli_database_query* | <font color="red">**屏蔽**</font> |

**3. FUNCTION (ast_custom_function_register)中声明的函数**

| FUNCTION函数  		| C函数 						| 进度  	|
| ----------------- | ------------------------- | ----- |
| *DB(read)* 		| *function_db_read* 		| 修改  	|
| *DB(write)* 		| *function_db_write* 		| 修改 	|
| *DB_EXISTS(read)* | *function_db_exists*		| 修改  	|
| *DB_DELETE(read)* | *function_db_delete* 		| 修改  	|
| *DB_DELETE(write)*| *function_db_delete_write*| 修改  	|
| *DB_KEYS(read)*	| *function_db_keys* 		| 修改  	|

**4. AMI (ast_manager_register_xml_core)中声明的函数**

| AMI指令  		| C函数 					| 进度  	|
| ------------- | --------------------- | ----- |
| *DBGet* 		| *manager_dbget* 		| 修改  	|
| *DBPut* 		| *manager_dbput* 		| 修改  	|
| *DBDel* 		| *manager_dbdel* 		| 修改  	|
| *DBDelTree* 	| *manager_dbdeltree* 	| 修改  	|

**5. 遗留问题**

- 需要花费少量时间确保与res_config_mysql.c统一配置文件及格式。

	已解决，现在统一使用***/etc/asterisk/res_config_mysql.conf***
>
	[general]
	dbtype = mysql			# mysql或者sqlite
	dbhost = 192.9.203.180
	dbname = asteriskreal
	dbuser = root
	dbpass = 
	dbport = 3306
	dbcharset = latin1

- 目前写死使用mysql代替sqlite，将来需读取配置文件，启动时选择数据库类型。
	
	已解决，***/etc/asterisk/res_config_mysql.conf***的***[general]***->***dbtype***为***mysql***时，将使用MySQL连接

- mysql断开不会重连，后续需要增加相关处理

	已增加代码，使MySQL连接断开时会重连

- 统一数据库锁
	
	已解决，目前统一使用***dblock***

- 会存在两个mysql连接，一个来自Asterisk Daemon的db.c，一个来自Asterisk的realtime

	无法解决

- 继续检查代码，确保所有用到锁的函数，都正确释放了锁

	已修复几处会导致死锁的代码，将继续检查代码

- 是否需提供工具，将sqlite数据库内容导入mysql？

	不提供，可使用工具导数据

## 代码简单分析

### 引用头文件
在main/db.c L44行代码
	#include <sqlite3.h>
后增加

	// Add MySQL header
	#include <mysql/mysql.h>
	#include <mysql/errmsg.h>

	// Add syslog header
	#include <syslog.h>

### 增加宏定义及控制数据库类型的变量
在main/db.c L107行代码
	#define MAX_DB_FIELD 256
前增加

	#ifndef MYSQL_PORT
	# ifdef MARIADB_PORT
	#  define MYSQL_PORT MARIADB_PORT
	# else
	#  define MYSQL_PORT 3306
	# endif
	#endif

	// Define database type
	static const int XU_DB_SQLITE3 = 0;
	static const int XU_DB_MYSQL = 1;
	static int xuDBType;// default is XU_DB_MYSQL;

### 锁
在main/db.c L108行代码

	AST_MUTEX_DEFINE_STATIC(dblock);
中声明并初始化了锁dblock（具体参见*include/asterisk/lock.h*）

之后需要加锁时调用

	ast_mutex_lock(&dblock);
需要解锁时调用

	ast_mutex_unlock(&dblock);

### 增加配置文件相关的常量
在main/db.c L115行代码

	static void db_sync(void);
后增加

	// Use same config file as "res_config_mysql.c"
	static const char XU_CONFIG_MYSQL_CONF[] = "res_config_mysql.conf";
	static const char XU_CONFIG_MYSQL_SECTION[] = "general";
	static const char XU_CONFIG_MYSQL_TYPE[] = "dbtype";
	static const char XU_CONFIG_MYSQL_HOST[] = "dbhost";
	static const char XU_CONFIG_MYSQL_PORT[] = "dbport";
	static const char XU_CONFIG_MYSQL_NAME[] = "dbname";
	static const char XU_CONFIG_MYSQL_USER[] = "dbuser";
	static const char XU_CONFIG_MYSQL_PASS[] = "dbpass";
	//	static const char XU_CONFIG_MYSQL_SOCK[] = "dbsock";
	static const char XU_CONFIG_MYSQL_CHAR[] = "dbcharset";
	static const char XU_CONFIG_MYSQL_TIMEOUT[] = "dbtimeout";

### 增加存储数据库连接及连接参数的变量
在main/db.c L115行代码

	static void db_sync(void);
后增加

	struct mysql_conn {
		MYSQL       handle;
		char        host[50];	// "localhost"
		int         port;		// MYSQL_PORT
		char        name[50];	// "asteriskreal"
		char        user[50];	// "root"
		char        pass[50];	// ""
	//	char        sock[50];	// ""
		char        charset[50];// "latin1"
		int         connected;	//
		time_t      connect_time;
	}xu_mysql_conn;

这个变量在*astdb_init*中初始化，在*astdb_atexit*中释放

之后需要进行数据库查询时，调用*&xu_mysql_conn.handle*

	res = mysql_query(&xu_mysql_conn.handle, fullSql)

### 数据库语句
在main/db.c L117行代码
	#define DEFINE_SQL_STATEMENT(stmt,sql) static sqlite3_stmt *stmt; \
		const char stmt##_sql[] = sql;
前增加

	// 用于 ast_db_put
	const char xu_put_stmt_sql[] = "INSERT INTO `astdb` (`key`, `value`) VALUES ('%s', '%s') ON DUPLICATE KEY UPDATE `value`='%s';";
	// 用于 db_get_common
	const char xu_get_stmt_sql[] = "SELECT `value` FROM `astdb` WHERE `key` = '%s' LIMIT 0,1;";
	// 用于 ast_db_del 及 ast_db_deltree
	const char xu_del_stmt_sql[] = "DELETE FROM `astdb` WHERE `key` = '%s';";
	// 用于 ast_db_deltree
	const char xu_deltree_stmt_sql[] = "DELETE FROM `astdb` WHERE `key` LIKE '%s/%%';";
	// 用于 ast_db_deltree
	const char xu_deltree_all_stmt_sql[] = "DELETE FROM `astdb`;";
	// 用于 ast_db_gettree 及 ast_db_gettree_by_prefix
	const char xu_gettree_stmt_sql[] = "SELECT `key`,`value` FROM `astdb` WHERE `key` LIKE '%s/%%' ORDER BY `key`;";
	// 用于 ast_db_gettree
	const char xu_gettree_all_stmt_sql[] = "SELECT `key`,`value` FROM `astdb`;";
	// 用于 handle_cli_database_showkey
	const char xu_showkey_stmt_sql[] = "SELECT `key`,`value` FROM `astdb` WHERE `key` LIKE '%%/%s' ORDER BY `key`;";
	// 用于 xu_db_create_astdb
	const char xu_create_astdb_stmt_sql[] = "CREATE TABLE IF NOT EXISTS `astdb`(`key` VARCHAR(256) NOT NULL DEFAULT '',`value` VARCHAR(256) DEFAULT '',PRIMARY KEY (`key`)) ENGINE=MYISAM CHARSET=latin1 COLLATE=latin1_general_ci;";
	// gettree_prefix_stmt 对应的SQL语句由 xu_gettree_stmt_sql 实现
	// const char xu_gettree_prefix_stmt[] = "SELECT key, value FROM astdb WHERE key > ?1 AND key <= ?1 || X'ffff'";

### 日志
注意：astdb是在Asterisk Daemon中初始化的，故无法使用*ast_log*记录日志，需使用*syslog*写到*/var/log/message*中。

	syslog(LOG_INFO, "db.c ast_db_put(family(%s), key(%s), value(%s))", family, key, value);

	syslog(LOG_ERR, "db.c ast_db_put: mysql_query failed. Error: %s", mysql_error(&xu_mysql_conn.handle));

### 加载模块

	int astdb_init(void)
	{
		// 读取配置文件，并确定数据库类型
		xuDBType = XU_DB_SQLITE3;
		xu_parse_config();
		
		// 初始化数据库
		if (db_init()) {
			return -1;
		}
		
		// sqlite3才需要
		if (xuDBType == XU_DB_SQLITE3)
		{
			ast_cond_init(&dbcond, NULL);
			if (ast_pthread_create_background(&syncthread, NULL, db_sync_thread, NULL)) {
				return -1;
			}
		}
		
		// 注册程序退出时调用astdb_atexit
		ast_register_atexit(astdb_atexit);
		
		// 注册CLI指令及AMI函数
		ast_cli_register_multiple(cli_database, ARRAY_LEN(cli_database));
		ast_manager_register_xml_core("DBGet", EVENT_FLAG_SYSTEM | EVENT_FLAG_REPORTING, manager_dbget);
		ast_manager_register_xml_core("DBPut", EVENT_FLAG_SYSTEM, manager_dbput);
		ast_manager_register_xml_core("DBDel", EVENT_FLAG_SYSTEM, manager_dbdel);
		ast_manager_register_xml_core("DBDelTree", EVENT_FLAG_SYSTEM, manager_dbdeltree);

		return 0;
	}

PS：记得增加数据库重连
	
	#if MYSQL_VERSION_ID >= 50013
		/* Add option for automatic reconnection */
		if (mysql_options(&xu_mysql_conn.handle, MYSQL_OPT_RECONNECT, &my_bool_true) != 0) {
			ast_log(LOG_ERROR, "db.c xu_db_open: mysql_options MYSQL_OPT_RECONNECT returned (%d) %s\n", mysql_errno(&xu_mysql_conn.handle), mysql_error(&xu_mysql_conn.handle));
		}
	#endif


PS2：函数*astdb_init*是在*main/asterisk.c L4049的asterisk_daemon*调用的：

	static void asterisk_daemon(int isroot, const char *runuser, const char *rungroup){
		...
		check_init(astdb_init(), "ASTdb");
	}

### 卸载模块

	static void astdb_atexit(void)
	{
		// 注销CLI指令及AMI函数
		ast_cli_unregister_multiple(cli_database, ARRAY_LEN(cli_database));
		ast_manager_unregister("DBGet");
		ast_manager_unregister("DBPut");
		ast_manager_unregister("DBDel");
		ast_manager_unregister("DBDelTree");
			
		if (xuDBType == XU_DB_MYSQL)
		{
			if (xu_mysql_conn.connected)
			{
				// 关闭mysql连接
				ast_mutex_lock(&dblock);
				mysql_close(&xu_mysql_conn.handle);
				xu_mysql_conn.connected = 0;
				ast_mutex_unlock(&dblock);
			}
		}
		else
		{
			/* Set doexit to 1 to kill thread. db_sync must be called with
			 * mutex held. */
			ast_mutex_lock(&dblock);
			doexit = 1;
			db_sync();
			ast_mutex_unlock(&dblock);
			
			pthread_join(syncthread, NULL);
			ast_mutex_lock(&dblock);
			clean_statements();
			if (sqlite3_close(astdb) == SQLITE_OK) {
				astdb = NULL;
			}
			ast_mutex_unlock(&dblock);
		}
	}

### 初始化数据库

	static int db_init(void)
	{
		if (xuDBType == XU_DB_MYSQL)
		{
			if (xu_mysql_conn.connected)
				return 0;
			
			// xu_db_open创建数据库连接
			// xu_db_create_astdb初始化数据库表
			if (xu_db_open() || xu_db_create_astdb()) {
				return -1;
			}
		}
		else // xuDBType == XU_DB_SQLITE3
		{
			if (astdb)
				return 0;
			
			if (db_open() || db_create_astdb() || init_statements()) {
				return -1;
			}
		}
		return 0;
	}

### include/asterisk/astdb.h中声明的函数

	int ast_db_put(const char *family, const char *key, const char *value)
	{
		fullSql = 拼接sql语句
		ast_mutex_lock(&dblock);
		mysql_query(&xu_mysql_conn.handle, fullSql);
		ast_mutex_unlock(&dblock);
	}
	
	int ast_db_get(const char *family, const char *key, char *value, int valuelen)
	int ast_db_get_allocated(const char *family, const char *key, char **out)
	static int db_get_common(const char *family, const char *key, char **buffer, int bufferlen)
	
	int ast_db_del(const char *family, const char *key)
	int ast_db_deltree(const char *family, const char *keytree)
	static struct ast_db_entry *db_gettree_common(sqlite3_stmt *stmt)
	static struct ast_db_entry *xu_db_gettree_common(MYSQL *handle)
	struct ast_db_entry *ast_db_gettree(const char *family, const char *keytree)
	struct ast_db_entry *ast_db_gettree_by_prefix(const char *family, const char *key_prefix)
	void ast_db_freetree(struct ast_db_entry *dbe)

### CLI(ast_cli_register_multiple)中声明的函数

	static char *handle_cli_database_put(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)
	{
		(参数不是5个？){显示usage并退出}
		ast_db_put(a->argv[2], a->argv[3], a->argv[4]);
	}
	
	static char *handle_cli_database_get(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)
	{
		(参数不是4个？){显示usage并退出}
		ast_db_get(a->argv[2], a->argv[3], tmp, sizeof(tmp));
		ast_cli(a->fd, "Value: %s\n", tmp);
	}
	
	static char *handle_cli_database_gettree(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)
	{
		(参数小于3个或大于4个？){显示usage并退出}
		(参数为4个？){
			data = ast_db_gettree(a->argv[2], a->argv[3]);
			显示data
		}
		(参数为3个？){
			data = ast_db_gettree(a->argv[2], NULL);
			显示data
		}
	}
	
	static char *handle_cli_database_del(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)
	{
		(参数不是4个？){显示usage并退出}
		res = ast_db_del(a->argv[2], a->argv[3]);
		(res==0?){删除成功}
	}

	static char *handle_cli_database_deltree(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)
	{
		(参数小于3个或大于4个？){显示usage并退出}
		(参数为4个？){
			num_deleted = ast_db_deltree(a->argv[2], a->argv[3]);
			显示num_deleted
		}
		(参数为3个？){
			num_deleted = ast_db_deltree(a->argv[2], NULL);
			显示num_deleted
		}
	}
	
	static char *handle_cli_database_show(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)
	{
		(参数为3个或4个？){
			fullSql = 使用xu_gettree_stmt_sql拼接sql语句
			ast_mutex_lock(&dblock);
			data = mysql_query(&xu_mysql_conn.handle, fullSql);
			显示data
			ast_mutex_unlock(&dblock);
		}
		(参数为3个？){
			fullSql = xu_gettree_all_stmt
			ast_mutex_lock(&dblock);
			data = mysql_query(&xu_mysql_conn.handle, fullSql);
			显示data
			ast_mutex_unlock(&dblock);
		}
	}
	
	static char *handle_cli_database_showkey(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)
	{
		(参数不是3个？){显示usage并退出}
		fullSql = 使用xu_showkey_stmt_sql拼接sql语句
		ast_mutex_lock(&dblock);
		data = mysql_query(&xu_mysql_conn.handle, fullSql);
		显示data
		ast_mutex_unlock(&dblock);
	}
	
	// 仅用于handle_cli_database_query
	static int display_results(void *arg, int columns, char **values, char **colnames)
	// 已屏蔽
	static char *handle_cli_database_query(struct ast_cli_entry *e, int cmd, struct ast_cli_args *a)

	static struct ast_cli_entry cli_database[] = {
		AST_CLI_DEFINE(handle_cli_database_show,    "Shows database contents"),
		AST_CLI_DEFINE(handle_cli_database_showkey, "Shows database contents"),
		AST_CLI_DEFINE(handle_cli_database_get,     "Gets database value"),
		AST_CLI_DEFINE(handle_cli_database_gettree,     "Gets database keytree/value"),
		AST_CLI_DEFINE(handle_cli_database_put,     "Adds/updates database value"),
		AST_CLI_DEFINE(handle_cli_database_del,     "Removes database key/value"),
		AST_CLI_DEFINE(handle_cli_database_deltree, "Removes database keytree/values"),
		// AST_CLI_DEFINE(handle_cli_database_query,   "Run a user-specified query on the astdb"),
	};

### FUNCTION (ast_custom_function_register)中声明的函数
注意：代码位于funcs/func_db.c
	
	static struct ast_custom_function db_function = {
		.name = "DB",
		.read = function_db_read,
		.write = function_db_write,
	};
	function_db_read 调用 ast_db_get
	function_db_write 调用 ast_db_put
	
	static struct ast_custom_function db_exists_function = {
		.name = "DB_EXISTS",
		.read = function_db_exists,
		.read_max = 2,
	};
	function_db_exists 调用 ast_db_get
	
	static struct ast_custom_function db_delete_function = {
		.name = "DB_DELETE",
		.read = function_db_delete,
		.write = function_db_delete_write,
	};
	function_db_delete 调用 ast_db_get 与 ast_db_del
	function_db_delete_write 调用 function_db_delete
	
	static struct ast_custom_function db_keys_function = {
		.name = "DB_KEYS",
		.read2 = function_db_keys,
	};
	function_db_keys 调用 ast_db_gettree
	
	res |= ast_custom_function_register_escalating(&db_function, AST_CFE_BOTH);
	res |= ast_custom_function_register(&db_exists_function);
	res |= ast_custom_function_register_escalating(&db_delete_function, AST_CFE_READ);
	res |= ast_custom_function_register(&db_keys_function);

### AMI (ast_manager_register_xml_core)中声明的函数

	static int manager_dbput(struct mansession *s, const struct message *m)
	调用 ast_db_put
	static int manager_dbget(struct mansession *s, const struct message *m)
	调用 manager_dbget
	static int manager_dbdel(struct mansession *s, const struct message *m)
	调用 ast_db_del
	static int manager_dbdeltree(struct mansession *s, const struct message *m)
	调用 ast_db_deltree

	ast_manager_register_xml_core("DBGet", EVENT_FLAG_SYSTEM | EVENT_FLAG_REPORTING, manager_dbget);
	ast_manager_register_xml_core("DBPut", EVENT_FLAG_SYSTEM, manager_dbput);
	ast_manager_register_xml_core("DBDel", EVENT_FLAG_SYSTEM, manager_dbdel);
	ast_manager_register_xml_core("DBDelTree", EVENT_FLAG_SYSTEM, manager_dbdeltree);


### sqlite statements
以下函数用于初始化及释放sqlite的statements，mysql是否也应该用statements优化操作？

	static int init_stmt(sqlite3_stmt **stmt, const char *sql, size_t len)
	static int clean_stmt(sqlite3_stmt **stmt, const char *sql)
	static void clean_statements(void)
	static int init_statements(void)

### sqlite transaction，用于确保数据实时写入硬盘，mysql无需此操作
	static int ast_db_begin_transaction(void)
	{
		return db_execute_sql("BEGIN TRANSACTION", NULL, NULL);
	}
	
	static int ast_db_commit_transaction(void)
	{
		return db_execute_sql("COMMIT", NULL, NULL);
	}
	
	static int ast_db_rollback_transaction(void)
	{
		return db_execute_sql("ROLLBACK", NULL, NULL);
	}
	
	static void db_sync(void)
	{
		dosync = 1;
		ast_cond_signal(&dbcond);
	}

	static void *db_sync_thread(void *data)
	{
		ast_mutex_lock(&dblock);
		ast_db_begin_transaction();
		for (;;) {
			/* If dosync is set, db_sync() was called during sleep(1),
			 * and the pending transaction should be committed.
			 * Otherwise, block until db_sync() is called.
			 */
			while (!dosync) {
				ast_cond_wait(&dbcond, &dblock);
			}
			dosync = 0;
			if (ast_db_commit_transaction()) {
				ast_db_rollback_transaction();
			}
			if (doexit) {
				ast_mutex_unlock(&dblock);
				break;
			}
			ast_db_begin_transaction();
			ast_mutex_unlock(&dblock);
			sleep(1);
			ast_mutex_lock(&dblock);
		}
		return NULL;
	}

### 将数据从 Berkeley DB 迁移到 SQLite 3

	static int convert_bdb_to_sqlite3(void)
