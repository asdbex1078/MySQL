1)试着在WHERE子句中对数据库和表名使用常量查找值

你可以利用这一原则如下:

	要查找数据库或表，请使用计算为常量的表达式，例如字面值、返回常量的函数或标量子查询。

	避免使用非常量数据库名称查找值(或不使用查找值)的查询，因为它们需要扫描数据目录以查找匹配的数据库目录名称。

	在数据库中，避免使用非常表名查找值(或不使用查找值)的查询，因为它们需要扫描数据库目录以找到匹配的表文件。
	
这一原则适用于下表中所示的INFORMATION_SCHEMA表，该表显示了通过常量查找值使服务器能够避免目录扫描的列。例如，如果要从表中进行选择，
那么在WHERE子句中为TABLE_SCHEMA使用常量查找值可以避免数据目录扫描。

表							指定要避免数据目录扫描的列		指定要避免数据库目录扫描的列
COLUMNS						TABLE_SCHEMA					TABLE_NAME
KEY_COLUMN_USAGE			TABLE_SCHEMA					TABLE_NAME
PARTITIONS					TABLE_SCHEMA					TABLE_NAME
REFERENTIAL_CONSTRAINTS		CONSTRAINT_SCHEMA				TABLE_NAME
STATISTICS					TABLE_SCHEMA					TABLE_NAME
TABLES						TABLE_SCHEMA					TABLE_NAME
TABLE_CONSTRAINTS			TABLE_SCHEMA					TABLE_NAME
TRIGGERS					EVENT_OBJECT_SCHEMA				EVENT_OBJECT_TABLE
VIEWS						TABLE_SCHEMA					TABLE_NAME

限制特定常量数据库名的查询的好处是只需要检查已命名的数据库目录。例如：
	SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
	WHERE TABLE_SCHEMA = 'test';
	
使用文字数据库名称测试使服务器能够只检查测试数据库目录，而不管可能有多少数据库。相比之下，下面的查询效率较低，
因为它需要扫描数据目录以确定哪个数据库名称与模式'test%'匹配:
	SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
	WHERE TABLE_SCHEMA LIKE 'test%';

对于仅限于特定常量表名的查询，只需要对相应数据库目录中的指定表进行检查。例子:
	SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
	WHERE TABLE_SCHEMA = 'test' AND TABLE_NAME = 't1';

使用字面表名t1使服务器能够只检查t1表的文件，而不管测试数据库中可能有多少个表。相比之下，下面的查询需要扫描测试数据库目录，
以确定哪些表名与模式't%'匹配:
	SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
	WHERE TABLE_SCHEMA = 'test' AND TABLE_NAME LIKE 't%';

下面的查询需要扫描数据库目录以确定与模式“test%”匹配的数据库名称，对于每个匹配的数据库，需要扫描数据库目录以确定与模式“t%”匹配的表名称:
	SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
	WHERE TABLE_SCHEMA = 'test%' AND TABLE_NAME LIKE 't%';
	
2)编写最小化必须打开的表文件数量的查询
对于引用某些INFORMATION_SCHEMA表列的查询，可以使用几种优化方法来最小化必须打开的表文件的数量。例子:
	SELECT TABLE_NAME, ENGINE FROM INFORMATION_SCHEMA.TABLES
	WHERE TABLE_SCHEMA = 'test';

在这种情况下，当服务器扫描数据库目录以确定数据库中表的名称后，无需进一步的文件系统查找就可以使用这些名称。因此，TABLE_NAME不需要打开任何文件。
引擎(存储引擎)的值可以通过打开表的.frm文件来确定，而不需要触及其他表文件，如. myd或. myi文件。

一些值，比如MyISAM表的INDEX_LENGTH，也需要打开. myd或. myi文件。

文件打开优化类型表示为:

	SKIP_OPEN_TABLE:不需要打开表文件。通过扫描数据库目录，已经可以在查询中获得信息。

	OPEN_FRM_ONLY:只需要打开表的.frm文件。

	OPEN_TRIGGER_ONLY:只需要打开表的. trg文件。

	OPEN_FULL_TABLE:未timized信息查找。必须打开.frm、. myd和. myi文件。
	
下面的列表说明了前面的优化类型如何应用于INFORMATION_SCHEMA表列。对于未命名的表和列，不应用任何优化。
	
	列:OPEN_FRM_ONLY应用于所有列

	KEY_COLUMN_USAGE: OPEN_FULL_TABLE 适用于所有列

	分区:OPEN_FULL_TABLE 适用于所有列

	REFERENTIAL_CONSTRAINTS: OPEN_FULL_TABLE 适用于所有列

	统计:
		列					优化类型
	TABLE_CATALOG		OPEN_FRM_ONLY
	TABLE_SCHEMA		OPEN_FRM_ONLY
	TABLE_NAME			OPEN_FRM_ONLY
	NON_UNIQUE			OPEN_FRM_ONLY
	INDEX_SCHEMA		OPEN_FRM_ONLY
	INDEX_NAME			OPEN_FRM_ONLY
	SEQ_IN_INDEX		OPEN_FRM_ONLY
	COLUMN_NAME			OPEN_FRM_ONLY
	COLLATION			OPEN_FRM_ONLY
	CARDINALITY			OPEN_FULL_TABLE
	SUB_PART			OPEN_FRM_ONLY
	PACKED				OPEN_FRM_ONLY
	NULLABLE			OPEN_FRM_ONLY
	INDEX_TYPE			OPEN_FULL_TABLE
	COMMENT				OPEN_FRM_ONLY
	
	TABLE_CONSTRAINTS: OPEN_FULL_TABLE适用于所有列

	触发器:OPEN_TRIGGER_ONLY应用于所有列
	
	视图：
		柱					优化类型
	TABLE_CATALOG			OPEN_FRM_ONLY
	TABLE_SCHEMA			OPEN_FRM_ONLY
	TABLE_NAME				OPEN_FRM_ONLY
	VIEW_DEFINITION			OPEN_FRM_ONLY
	CHECK_OPTION			OPEN_FRM_ONLY
	IS_UPDATABLE			OPEN_FULL_TABLE
	DEFINER					OPEN_FRM_ONLY
	SECURITY_TYPE			OPEN_FRM_ONLY
	CHARACTER_SET_CLIENT	OPEN_FRM_ONLY
	COLLATION_CONNECTION	OPEN_FRM_ONLY
	
3)使用EXPLAIN确定服务器是否可以对查询使用INFORMATION_SCHEMA优化

这尤其适用于从多个数据库搜索信息的INFORMATION_SCHEMA查询，这可能会花费很长时间并影响性能。EXPLAIN输出中的额外值表示服务器可以使用前面描述的哪些优化
(如果有的话)来计算INFORMATION_SCHEMA查询。下面的示例演示了期望在额外值中看到的信息类型。
	mysql> EXPLAIN SELECT TABLE_NAME FROM INFORMATION_SCHEMA.VIEWS WHERE
		   TABLE_SCHEMA = 'test' AND TABLE_NAME = 'v1'\G
	*************************** 1. row ***************************
			   id: 1
	  select_type: SIMPLE
			table: VIEWS
			 type: ALL
	possible_keys: NULL
			  key: TABLE_SCHEMA,TABLE_NAME
		  key_len: NULL
			  ref: NULL
			 rows: NULL
			Extra: Using where; Open_frm_only; Scanned 0 databases
	
使用常量数据库和表查找值使服务器能够避免目录扫描。以获取对视图的引用。TABLE_NAME，只需要打开.frm文件。
	mysql> EXPLAIN SELECT TABLE_NAME, ROW_FORMAT FROM INFORMATION_SCHEMA.TABLES\G
	*************************** 1. row ***************************
			   id: 1
	  select_type: SIMPLE
			table: TABLES
			 type: ALL
	possible_keys: NULL
			  key: NULL
		  key_len: NULL
			  ref: NULL
			 rows: NULL
			Extra: Open_full_table; Scanned all databases

没有提供查找值(没有WHERE子句)，因此服务器必须扫描数据目录和每个数据库目录。对于这样标识的每个表，将选择表名和行格式。TABLE_NAME不需要进一步
打开表文件(应用SKIP_OPEN_TABLE优化)。ROW_FORMAT要求打开所有表文件(应用OPEN_FULL_TABLE)。EXPLAIN会报告OPEN_FULL_TABLE，因为它比SKIP_OPEN_TABLE更昂贵。
	mysql> EXPLAIN SELECT TABLE_NAME, TABLE_TYPE FROM INFORMATION_SCHEMA.TABLES
		   WHERE TABLE_SCHEMA = 'test'\G
	*************************** 1. row ***************************
			   id: 1
	  select_type: SIMPLE
			table: TABLES
			 type: ALL
	possible_keys: NULL
			  key: TABLE_SCHEMA
		  key_len: NULL
			  ref: NULL
			 rows: NULL
			Extra: Using where; Open_frm_only; Scanned 1 database


没有提供表名查找值，因此服务器必须扫描测试数据库目录。对于TABLE_NAME和TABLE_TYPE列，分别应用SKIP_OPEN_TABLE和OPEN_FRM_ONLY优化。
EXPLAIN报告OPEN_FRM_ONLY是因为它更昂贵。
	mysql> EXPLAIN SELECT B.TABLE_NAME
		   FROM INFORMATION_SCHEMA.TABLES AS A, INFORMATION_SCHEMA.COLUMNS AS B
		   WHERE A.TABLE_SCHEMA = 'test'
		   AND A.TABLE_NAME = 't1'
		   AND B.TABLE_NAME = A.TABLE_NAME\G
	*************************** 1. row ***************************
			   id: 1
	  select_type: SIMPLE
			table: A
			 type: ALL
	possible_keys: NULL
			  key: TABLE_SCHEMA,TABLE_NAME
		  key_len: NULL
			  ref: NULL
			 rows: NULL
			Extra: Using where; Skip_open_table; Scanned 0 databases
	*************************** 2. row ***************************
			   id: 1
	  select_type: SIMPLE
			table: B
			 type: ALL
	possible_keys: NULL
			  key: NULL
		  key_len: NULL
			  ref: NULL
			 rows: NULL
			Extra: Using where; Open_frm_only; Scanned all databases;
				   Using join buffer

对于第一个EXPLAIN输出行:常量数据库和表查找值使服务器能够避免对表值进行目录扫描。对表的引用。TABLE_NAME不需要进一步的表文件。

对于第二个EXPLAIN输出行:所有列表值都是OPEN_FRM_ONLY查找，所以列。TABLE_NAME要求打开.frm文件。

	mysql> EXPLAIN SELECT * FROM INFORMATION_SCHEMA.COLLATIONS\G
	*************************** 1. row ***************************
			   id: 1
	  select_type: SIMPLE
			table: COLLATIONS
			 type: ALL
	possible_keys: NULL
			  key: NULL
		  key_len: NULL
			  ref: NULL
			 rows: NULL
			Extra:
在这种情况下，没有应用任何优化，因为COLLATIONS不是可以使用优化的INFORMATION_SCHEMA表之一。