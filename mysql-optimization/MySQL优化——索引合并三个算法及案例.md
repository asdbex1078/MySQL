优化器在不同的可能的索引合并算法和基于各种可用选项的成本估计的其他访问方法之间进行选择。

索引合并的使用受优化器中系统变量optimizer_switch 的影响。index_merge, index_merge_intersection, index_merge_union, index_merge_sort_union。
见第8.9.2节，“可切换优化”。

默认情况下，所有这些标志都是打开的。若只启用某些算法，请将index_merge设置为off，并仅启用应该允许的其他算法。
(例如，查询optimizer_switch变量结果：
	mysql> show VARIABLES like 'optimizer_switch';

|  Variable_name   |                            Value                             |
| :--------------: | :----------------------------------------------------------: |
| optimizer_switch | index_merge=on,index_merge_union=on,index_merge_sort_union=on,index_merge_intersection=on,<br />engine_condition_pushdown=on,index_condition_pushdown=on,mrr=on,mrr_cost_based=on,<br />block_nested_loop=on,batched_key_access=off,materialization=on,semijoin=on,loosescan=on,<br />firstmatch=on,duplicateweedout=on,subquery_materialization_cost_based=on,use_index_extensions=on,<br />condition_fanout_filter=on,derived_merge=on |



一、索引合并交集 (Extra中显示 ：Using intersect)     重点：  ----- AND 条件

	当一个WHERE子句与AND组合在一起被转换为多个范围条件时，此访问算法适用，并且每个条件是下列条件之一：
	(表示使用and的各个索引的条件时，该信息表示是从处理结果获取交集)
	
		1、这种形式的N部分表达式，其中索引正好有N个部分（即覆盖所有索引部分）：
	
			key_part1 = const1 AND key_part2 = const2 ... AND key_partN = constN
			(需满足特殊条件——索引合并成本 < 单列索引成本。后期会讨论)
		
			(
				mysql> explain select * from user1 where user_province = '山西省' and user_age = 31;
				+----+-------------+-------+------------+-------------+--------------------------------------------------+---------------------------+---------+------+------+----------+---------------------------------------------------------+
				| id | select_type | table | partitions | type        | possible_keys                                    | key                       | key_len | ref  | rows | filtered | Extra                                                   |
				+----+-------------+-------+------------+-------------+--------------------------------------------------+---------------------------+---------+------+------+----------+---------------------------------------------------------+
				|  1 | SIMPLE      | user1 | NULL       | index_merge | idx_user_age,idx_province_city_area,idx_province | idx_user_age,idx_province | 2,99    | NULL |  164 |   100.00 | Using intersect(idx_user_age,idx_province); Using where |
				+----+-------------+-------+------------+-------------+--------------------------------------------------+---------------------------+---------+------+------+----------+---------------------------------------------------------+
				1 row in set, 1 warning (0.00 sec)
			)
		2、InnoDB表 主键上的任何范围条件。
		
			①、SELECT * FROM innodb_table
				WHERE primary_key < 10 AND key_col1 = 20;	
				
			(
				举个例子：user_id为主键，user_age为单列索引。
				mysql> explain select * from user1 where user_id < 14000 and user_age = 30;
				+----+-------------+-------+------------+-------------+----------------------+----------------------+---------+------+------+----------+----------------------------------------------------+
				| id | select_type | table | partitions | type        | possible_keys        | key                  | key_len | ref  | rows | filtered | Extra                                              |
				+----+-------------+-------+------------+-------------+----------------------+----------------------+---------+------+------+----------+----------------------------------------------------+
				|  1 | SIMPLE      | user1 | NULL       | index_merge | PRIMARY,idx_user_age | idx_user_age,PRIMARY | 6,4     | NULL |    1 |   100.00 | Using intersect(idx_user_age,PRIMARY); Using where |
				+----+-------------+-------+------------+-------------+----------------------+----------------------+---------+------+------+----------+----------------------------------------------------+
				1 row in set, 1 warning (0.00 sec)
			)
			
			②、SELECT * FROM tbl_name
				WHERE key1_part1 = 1 AND key1_part2 = 2 AND key2 = 2;
	
				（覆盖索引 + 另一个索引的组合。考虑一下为什么要加 force index，成本问题）
				（	
					mysql> explain select user_province,user_age from user1 force index(idx_user_age,idx_province) where user_province = '山西省' and user_city = '沧州市' AND user_area = '汤旺河区' AND user_age = 31;
					+----+-------------+-------+------------+-------------+---------------------------+---------------------------+---------+------+------+----------+---------------------------------------------------------+
					| id | select_type | table | partitions | type        | possible_keys             | key                       | key_len | ref  | rows | filtered | Extra                                                   |
					+----+-------------+-------+------------+-------------+---------------------------+---------------------------+---------+------+------+----------+---------------------------------------------------------+
					|  1 | SIMPLE      | user1 | NULL       | index_merge | idx_user_age,idx_province | idx_user_age,idx_province | 2,99    | NULL |  164 |     1.00 | Using intersect(idx_user_age,idx_province); Using where |
					+----+-------------+-------+------------+-------------+---------------------------+---------------------------+---------+------+------+----------+---------------------------------------------------------+
					1 row in set, 1 warning (0.00 sec)
				）
				
	如果所使用的索引没有复盖查询中使用的所有列，则只有在满足所有使用key的范围条件时，才会检索完整的行。
	（即，最终需要返回的列，不在索引中，就需要回表查询完整的行信息。）
	
	如果合并条件之一是InnoDB表的主键上的条件，则该条件不用于行检索，而是用于筛选使用其他条件检索的行。
	（可见上边2中①，主键并不是用来检索完整的行信息的，而是过滤不用的行。）

二、索引合并联合 (Using union(...))  -- 重点 or查询
	
	此算法的标准是类似的指数合并交算法。该算法适用于将表的WHERE子句与OR组合在一起的不同键上转换为多个范围条件的情况，并且每个条件是下列条件之一：
	(表示使用or连接各个使用索引的条件时，该信息表示从处理结果获取并集)
	
	1、这种形式的N部分表达式，其中索引正好有N个部分（即覆盖所有索引部分）：
	(同一中的1，只不过这个地方的条件是or，官网是错的)
	
	(
		mysql> explain select * from user1 where user_province = '山西省' or user_age = 31;
		+----+-------------+-------+------------+-------------+--------------------------------------------------+---------------------------+---------+------+-------+----------+-----------------------------------------------------+
		| id | select_type | table | partitions | type        | possible_keys                                    | key                       | key_len | ref  | rows  | filtered | Extra                                               |
		+----+-------------+-------+------------+-------------+--------------------------------------------------+---------------------------+---------+------+-------+----------+-----------------------------------------------------+
		|  1 | SIMPLE      | user1 | NULL       | index_merge | idx_user_age,idx_province_city_area,idx_province | idx_province,idx_user_age | 99,2    | NULL | 18679 |   100.00 | Using union(idx_province,idx_user_age); Using where |
		+----+-------------+-------+------------+-------------+--------------------------------------------------+---------------------------+---------+------+-------+----------+-----------------------------------------------------+
		1 row in set, 1 warning (0.00 sec)
	)
	
	2、InnoDB表 主键上的任何范围条件。
	
	(	条件中含有主键，且 条件为 or
		mysql> explain select * from user1 where user_id < 14000 or user_age = 30;
		+----+-------------+-------+------------+-------------+----------------------+----------------------+---------+------+------+----------+------------------------------------------------+
		| id | select_type | table | partitions | type        | possible_keys        | key                  | key_len | ref  | rows | filtered | Extra                                          |
		+----+-------------+-------+------------+-------------+----------------------+----------------------+---------+------+------+----------+------------------------------------------------+
		|  1 | SIMPLE      | user1 | NULL       | index_merge | PRIMARY,idx_user_age | PRIMARY,idx_user_age | 4,2     | NULL | 2432 |   100.00 | Using union(PRIMARY,idx_user_age); Using where |
		+----+-------------+-------+------------+-------------+----------------------+----------------------+---------+------+------+----------+------------------------------------------------+
		1 row in set, 1 warning (0.00 sec)
	)
	
	3、满足索引合并交集算法适用的条件。
		
		(
			mysql> explain select * from user1 where (user_id = 13703 AND user_name = '赵子龙') or (user_age = 30 AND user_province = '山东省');
			+----+-------------+-------+------------+-------------+------------------------------------------------------------------------+-----------------------------------+---------+------+------+----------+------------------------------------------------------------------------+
			| id | select_type | table | partitions | type        | possible_keys                                                          | key                               | key_len | ref  | rows | filtered | Extra                                                                  |
			+----+-------------+-------+------------+-------------+------------------------------------------------------------------------+-----------------------------------+---------+------+------+----------+------------------------------------------------------------------------+
			|  1 | SIMPLE      | user1 | NULL       | index_merge | PRIMARY,idx_user_name,idx_user_age,idx_province_city_area,idx_province | PRIMARY,idx_user_age,idx_province | 4,2,99  | NULL |  172 |   100.00 | Using union(PRIMARY,intersect(idx_user_age,idx_province)); Using where |
			+----+-------------+-------+------------+-------------+------------------------------------------------------------------------+-----------------------------------+---------+------+------+----------+------------------------------------------------------------------------+
			1 row in set, 1 warning (0.00 sec)
		)

三、索引合并排序联合 (using sort_union和using sort_intersection)

	当WHERE子句被转换为由OR组合的多个范围条件时，此访问算法是适用的，但索引合并联合算法是不适用的。
	(与前面两个对应的类似，只是他们是出现在用and和or查询信息量大时，先查询主键，然后进行排序合并后，才能读取记录并返回。)
	
	例子：
		1、mysql> EXPLAIN select * from user1 force index(idx_user_name,idx_user_age) where user_age < 10 or user_name < "丁";
			+----+-------------+-------+------------+-------------+----------------------------+----------------------------+---------+------+-------+----------+-----------------------------------------------------------+
			| id | select_type | table | partitions | type        | possible_keys              | key                        | key_len | ref  | rows  | filtered | Extra                                                     |
			+----+-------------+-------+------------+-------------+----------------------------+----------------------------+---------+------+-------+----------+-----------------------------------------------------------+
			|  1 | SIMPLE      | user1 | NULL       | index_merge | idx_user_name,idx_user_age | idx_user_age,idx_user_name | 2,303   | NULL | 41731 |   100.00 | Using sort_union(idx_user_age,idx_user_name); Using where |
			+----+-------------+-------+------------+-------------+----------------------------+----------------------------+---------+------+-------+----------+-----------------------------------------------------------+
			1 row in set, 1 warning (0.00 sec)
	
		2、mysql> EXPLAIN select * from user1  force index(idx_user_name,idx_user_age) where (user_age < 47 or user_name = "赵子龙") AND create_time = '2020-08-18 15:45:08';
			+----+-------------+-------+------------+-------------+----------------------------+----------------------------+---------+------+--------+----------+-----------------------------------------------------------+
			| id | select_type | table | partitions | type        | possible_keys              | key                        | key_len | ref  | rows   | filtered | Extra                                                     |
			+----+-------------+-------+------------+-------------+----------------------------+----------------------------+---------+------+--------+----------+-----------------------------------------------------------+
			|  1 | SIMPLE      | user1 | NULL       | index_merge | idx_user_name,idx_user_age | idx_user_age,idx_user_name | 2,303   | NULL | 104345 |    10.00 | Using sort_union(idx_user_age,idx_user_name); Using where |
			+----+-------------+-------+------------+-------------+----------------------------+----------------------------+---------+------+--------+----------+-----------------------------------------------------------+
			1 row in set, 1 warning (0.00 sec)
		
		(思考一下，为什么要加force index ——成本模型，比例值)
		
	排序联合算法和联合算法之间的区别在于排序联合算法必须首先为所有行提取行Id，并在返回任何行之前对它们进行排序。