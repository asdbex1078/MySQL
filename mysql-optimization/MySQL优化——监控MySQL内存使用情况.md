下面的示例演示如何使用 性能架构 和sys架构来监视MySQL内存使用情况

默认情况下，大多数性能架构内存插装都是禁用的。可以通过更新性能模式setup_instruments表中已启用的列来启用工具。
内存工具的名称以Memory /code_area/instrument_name的形式存在，其中code_area是一个值，比如sql或innodb，而instrument_name是工具的详细信息。

	1、要查看可用的MySQL内存工具，查询性能模式setup_instruments表。下面的查询将为所有代码区域返回数百个内存工具。

		mysql> SELECT * FROM performance_schema.setup_instruments
			   WHERE NAME LIKE '%memory%';
			   
		您可以通过指定代码区域来缩小结果。例如，你可以通过指定InnoDB作为代码区域来限制结果到InnoDB内存工具。

		mysql> SELECT * FROM performance_schema.setup_instruments
			   WHERE NAME LIKE '%memory/innodb%';
		+-------------------------------------------+---------+-------+
		| NAME                                      | ENABLED | TIMED |
		+-------------------------------------------+---------+-------+
		| memory/innodb/adaptive hash index         | NO      | NO    |
		| memory/innodb/buf_buf_pool                | NO      | NO    |
		| memory/innodb/dict_stats_bg_recalc_pool_t | NO      | NO    |
		| memory/innodb/dict_stats_index_map_t      | NO      | NO    |
		| memory/innodb/dict_stats_n_diff_on_level  | NO      | NO    |
		| memory/innodb/other                       | NO      | NO    |
		| memory/innodb/row_log_buf                 | NO      | NO    |
		| memory/innodb/row_merge_sort              | NO      | NO    |
		| memory/innodb/std                         | NO      | NO    |
		| memory/innodb/trx_sys_t::rw_trx_ids       | NO      | NO    |
		...

		根据你的MySQL安装，代码区域可能包括performance_schema, sql，客户端，innodb, myisam, csv，内存，黑洞，存档，分区，和其他。
		
	2、要启用内存工具，请在MySQL配置文件中添加一个性能模式工具规则。例如，要启用所有内存工具，请将此规则添加到您的配置文件并重新启动服务器:
	
		performance-schema-instrument='memory/%=COUNTED'
		
		注意：在启动时启用内存工具可确保对启动时发生的内存分配进行计数
		
		重新启动服务器后，对于已启用的内存工具，性能模式setup_instruments表中的ENABLED列应该报告为YES。对于内存工具，setup_instruments表中的计时列
		将被忽略，因为内存操作没有计时。
		
		mysql> SELECT * FROM performance_schema.setup_instruments
			   WHERE NAME LIKE '%memory/innodb%';
		+-------------------------------------------+---------+-------+
		| NAME                                      | ENABLED | TIMED |
		+-------------------------------------------+---------+-------+
		| memory/innodb/adaptive hash index         | NO      | NO    |
		| memory/innodb/buf_buf_pool                | NO      | NO    |
		| memory/innodb/dict_stats_bg_recalc_pool_t | NO      | NO    |
		| memory/innodb/dict_stats_index_map_t      | NO      | NO    |
		| memory/innodb/dict_stats_n_diff_on_level  | NO      | NO    |
		| memory/innodb/other                       | NO      | NO    |
		| memory/innodb/row_log_buf                 | NO      | NO    |
		| memory/innodb/row_merge_sort              | NO      | NO    |
		| memory/innodb/std                         | NO      | NO    |
		| memory/innodb/trx_sys_t::rw_trx_ids       | NO      | NO    |
		...
		
	3、查询存储仪表数据。在本例中，在性能模式memory_summary_global_by_event_name表中查询内存工具数据，该表通过EVENT_NAME总结数据。EVENT_NAME是工具的名称。

		下面的查询返回InnoDB缓冲池的内存数据。有关列的说明，请参阅第25.12.15.9节“内存汇总表”。	
		
		mysql> SELECT * FROM performance_schema.memory_summary_global_by_event_name
			   WHERE EVENT_NAME LIKE 'memory/innodb/buf_buf_pool'\G
						  EVENT_NAME: memory/innodb/buf_buf_pool
						 COUNT_ALLOC: 1
						  COUNT_FREE: 0
		   SUM_NUMBER_OF_BYTES_ALLOC: 137428992
			SUM_NUMBER_OF_BYTES_FREE: 0
					  LOW_COUNT_USED: 0
				  CURRENT_COUNT_USED: 1
					 HIGH_COUNT_USED: 1
			LOW_NUMBER_OF_BYTES_USED: 0
		CURRENT_NUMBER_OF_BYTES_USED: 137428992
		   HIGH_NUMBER_OF_BYTES_USED: 137428992
		
		可以使用sys模式memory_global_by_current_bytes表查询相同的底层数据，该表显示了服务器中按分配类型划分的当前全局内存使用情况。
		
		mysql> SELECT * FROM sys.memory_global_by_current_bytes
			   WHERE event_name LIKE 'memory/innodb/buf_buf_pool'\G
		*************************** 1. row ***************************
			   event_name: memory/innodb/buf_buf_pool
			current_count: 1
			current_alloc: 131.06 MiB
		current_avg_alloc: 131.06 MiB
			   high_count: 1
			   high_alloc: 131.06 MiB
		   high_avg_alloc: 131.06 MiB
		
		这个sys模式查询根据代码区域聚合当前分配的内存(current_alloc):
		
		mysql> SELECT SUBSTRING_INDEX(event_name,'/',2) AS
			   code_area, sys.format_bytes(SUM(current_alloc))
			   AS current_alloc
			   FROM sys.x$memory_global_by_current_bytes
			   GROUP BY SUBSTRING_INDEX(event_name,'/',2)
			   ORDER BY SUM(current_alloc) DESC;
		+---------------------------+---------------+
		| code_area                 | current_alloc |
		+---------------------------+---------------+
		| memory/innodb             | 843.24 MiB    |
		| memory/performance_schema | 81.29 MiB     |
		| memory/mysys              | 8.20 MiB      |
		| memory/sql                | 2.47 MiB      |
		| memory/memory             | 174.01 KiB    |
		| memory/myisam             | 46.53 KiB     |
		| memory/blackhole          | 512 bytes     |
		| memory/federated          | 512 bytes     |
		| memory/csv                | 512 bytes     |
		| memory/vio                | 496 bytes     |
		+---------------------------+---------------+
		
		