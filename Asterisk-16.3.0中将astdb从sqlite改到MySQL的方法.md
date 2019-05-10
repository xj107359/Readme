# Asterisk-16.3.0中将astdb从sqlite改到MySQL的方法
 
- 
## 目的
分布式结构下，为了让多台Asterisk共享astdb中的数据，需要将astdb从sqlite改到MySQL，故需修改astdb相关代码。

## Asterisk版本
当前使用asterisk-16.3.0版本

## 相关文件

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
	dbhost = 192.9.203.180
	dbname = asteriskreal
	dbuser = root
	dbpass = 
	dbport = 3306
	dbsock = /tmp/mysql.sock
	dbcharset = latin1

## 修改进度

**1. include/asterisk/astdb.h中声明的函数**

| First Header  | Second Header | Second Header |
| ------------- | ------------- | ------------- |
| Content Cell  | Content Cell  | Content Cell  |
| Content Cell  | Content Cell  | Content Cell  |

| C函数  | 进度 |
| ------------- | ------------- |
| ast_db_put  | 进度  |
| ast_db_del  | 进度  |
| ast_db_deltree  | 进度  |
| ast_db_get  | 进度  |
| ast_db_get_allocated  | 进度  |
| ast_db_gettree  | 进度  |
| ast_db_gettree_by_prefix  | <font color="red">不修改</font>  |
| ast_db_freetree  | 进度  |


<table>
<thead>
<tr>
  <th>C函数</th>
  <th>进度</th>
</tr>
</thead>
<tbody>
<tr>
  <td><i>ast_db_put</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>ast_db_del</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>ast_db_deltree</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>ast_db_get</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>ast_db_get_allocated</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>ast_db_gettree</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>ast_db_gettree_by_prefix</i></td>
  <td style="color:red"><strong>不修改</strong></td>
</tr>
<tr>
  <td><i>ast_db_freetree</i></td>
  <td>进度</td>
</tr>
</tbody>
</table>

**2. CLI(ast_cli_register_multiple)中声明的函数**

<table>
<thead>
<tr>
  <th>CLI指令</th>
  <th>C函数</th>
  <th>进度</th>
</tr>
</thead>
<tbody>
<tr>
  <td><i>database put &lt;family&gt; &lt;key&gt; &lt;value&gt;</i></td>
  <td><i>handle_cli_database_put</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>database del &lt;family&gt; &lt;key&gt;</i></td>
  <td><i>handle_cli_database_del</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>database deltree &lt;family&gt; [key]</i></td>
  <td><i>handle_cli_database_deltree</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>database get &lt;family&gt; &lt;key&gt;</i></td>
  <td><i>handle_cli_database_get</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>database gettree &lt;family&gt; [key]</i></td>
  <td><i>handle_cli_database_gettree</i></td>
  <td><strong>新增</strong></td>
</tr>
<tr>
  <td><i>database show [family [keytree]]</i></td>
  <td><i>handle_cli_database_show</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>database showkey &lt;keytree&gt;</i></td>
  <td><i>handle_cli_database_showkey</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>database query "&lt;SQL Statement&gt"</i></td>
  <td><i>handle_cli_database_query</i></td>
  <td style="color:red"><strong>屏蔽</strong></td>
</tr>
</tbody>
</table>

**3. FUNCTION (ast_custom_function_register)中声明的函数**

<table>
<thead>
<tr>
  <th>FUNCTION函数</th>
  <th>C函数</th>
  <th>进度</th>
</tr>
</thead>
<tbody>
<tr>
  <td><i>DB(read)</i></td>
  <td><i>function_db_read</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>DB(write)</i></td>
  <td><i>function_db_write</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>DB_EXISTS(read)</i></td>
  <td><i>function_db_exists</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>DB_DELETE(read)</i></td>
  <td><i>function_db_delete</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>DB_DELETE(write)</i></td>
  <td><i>function_db_delete_write</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>DB_KEYS(read)</i></td>
  <td><i>function_db_keys</i></td>
  <td>进度</td>
</tr>
</tbody>
</table>

**4. AMI (ast_manager_register_xml_core)中声明的函数**

<table>
<thead>
<tr>
  <th>AMI指令</th>
  <th>C函数</th>
  <th>进度</th>
</tr>
</thead>
<tbody>
<tr>
  <td><i>DBGet</i></td>
  <td><i>manager_dbget</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>DBPut</i></td>
  <td><i>manager_dbput</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>DBDel</i></td>
  <td><i>manager_dbdel</i></td>
  <td>进度</td>
</tr>
<tr>
  <td><i>DBDelTree</i></td>
  <td><i>manager_dbdeltree</i></td>
  <td>进度</td>
</tr>
</tbody>
</table>


**5. 遗留问题**

1. 需要花费少量时间确保与res_config_mysql.c统一配置文件及格式
2. mysql断开不会重连，后续需要增加相关处理
3. 后续还需花费至少5个工作日！！！确认所有用到锁的函数，都正确释放了锁！！！不然会导致运行时死锁！！！
4. 目前写死使用mysql代替sqlite，将来需读取配置文件，启动时选择数据库类型。
5. 是否需提供工具，将sqlite数据库内容导入mysql？
6. 会存在两个mysql连接，一个来自Asterisk Daemon的db.c，一个来自Asterisk的realtime，目测无法解决。

## 代码结构简介

### 一、模块加载、卸载函数

### 二、include/asterisk/astdb.h中声明的函数

### 三、CLI(ast_cli_register_multiple)中声明的函数

### 四、FUNCTION (ast_custom_function_register)中声明的函数

### 五、AMI (ast_manager_register_xml_core)中声明的函数

### 六、Sqlite特有函数

四、	修改进度

1、	include/asterisk/astdb.h中声明的函数

int ast_db_put(const char *family, const char *key, const char *value);	修改
int ast_db_get(const char *family, const char *key, char *value, int valuelen);	修改
int ast_db_get_allocated(const char *family, const char *key, char **out);	修改
int ast_db_del(const char *family, const char *key);	修改
int ast_db_deltree(const char *family, const char *keytree);	修改
struct ast_db_entry *ast_db_gettree(const char *family, const char *keytree);	修改
struct ast_db_entry *ast_db_gettree_by_prefix(const char *family, const char *key_prefix);	X
void ast_db_freetree(struct ast_db_entry *entry);	修改

2、	CLI(ast_cli_register_multiple)中声明的函数

database show	handle_cli_database_show	修改
database showkey	handle_cli_database_showkey	修改
database get	handle_cli_database_get	修改
database gettree	handle_cli_database_gettree	新增
database put	handle_cli_database_put	修改
database del	handle_cli_database_del	修改
database del	handle_cli_database_deltree	修改
database query	handle_cli_database_query	屏蔽

3、	FUNCTION (ast_custom_function_register)中声明的函数

注意：代码位于funcs/func_db.c

DB	.read = function_db_read
调用ast_db_get	修改
DB	.write = function_db_write
调用ast_db_put	修改
DB_EXISTS	.read = function_db_exists
调用ast_db_get	修改
DB_DELETE	.read = function_db_delete
调用ast_db_get与ast_db_del	修改
DB_DELETE	.write = function_db_delete_write
调用ast_db_get与ast_db_del	修改
DB_KEYS	.read2 = function_db_keys
调用ast_db_gettree与ast_db_freetree	修改

4、	AMI (ast_manager_register_xml_core)中声明的函数

DBPut
manager_dbput	Action: DBPut
Family: fam
Key: key666
Val: 87654

Response: Success
Message: Updated database successfully	修改
DBGet
manager_dbget	Action: DBGet
Family: fam
Key: key666

Response: Success
EventList: start
Message: Result will follow

Event: DBGetResponse
Family: fam
Key: key666
Val: 87654

Event: DBGetComplete
EventList: Complete
ListItems: 1	修改
DBDel
manager_dbdel	Action: DBDel
Family: fam
Key: key666

Response: Success
Message: Key deleted successfully	修改
DBDelTree
manager_dbdeltree	Action: DBDelTree
Family: fam

Response: Success
Message: Key tree deleted successfully	修改
