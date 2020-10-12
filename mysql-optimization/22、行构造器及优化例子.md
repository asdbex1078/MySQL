考虑下表，该表具有一个主键 (c1, c2, c3)：

	CREATE TABLE t1 (
	  c1 INT, c2 INT, c3 INT, c4 CHAR(100),
	  PRIMARY KEY(c1,c2,c3)
	);
	
在此查询中，WHERE子句使用索引中的所有列。但是，行构造器本身不包含索引前缀，因此优化器仅使用c1（key_len=4，大小为c1）：

	mysql> EXPLAIN SELECT * FROM t1
		   WHERE c1=1 AND (c2,c3) > (1,1)\G
	*************************** 1. row ***************************
			   id: 1
	  select_type: SIMPLE
			table: t1
	   partitions: NULL
			 type: ref
	possible_keys: PRIMARY
			  key: PRIMARY
		  key_len: 4
			  ref: const
			 rows: 3
		 filtered: 100.00
			Extra: Using where
			
在这种情况下，使用等效的非构造函数表达式重写行构造函数表达式可能会导致更完整的索引使用。对于给定的查询，行构造函数和等效的非构造函数表达式为：

	(c2,c3) > (1,1)
	c2 > 1 OR ((c2 = 1) AND (c3 > 1))
	
使用索引（key_len=12）中的所有三列重写查询以使用非构造函数表达式在优化器中导致结果：


	mysql> EXPLAIN SELECT * FROM t1
		   WHERE c1 = 1 AND (c2 > 1 OR ((c2 = 1) AND (c3 > 1)))\G
	*************************** 1. row ***************************
			   id: 1
	  select_type: SIMPLE
			table: t1
	   partitions: NULL
			 type: range
	possible_keys: PRIMARY
			  key: PRIMARY
		  key_len: 12
			  ref: NULL
			 rows: 3
		 filtered: 100.00
			Extra: Using where
			
因此，为获得更好的结果，请避免将行构造函数与AND/ OR 表达式混合使用 。使用其中一个或另一个。