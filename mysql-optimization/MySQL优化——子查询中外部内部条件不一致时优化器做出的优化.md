outer_expr IN (SELECT inner_expr FROM ... WHERE subquery_where)

当其中一个条件或两个条件都不具备时，优化会更加复杂。

假设已知outer_expr是一个非空值，但是子查询没有生成像outer_expr = inner_expr这样的行。那么outer_expr in (SELECT…)中的计算结果如下:

	1、NULL，当 inner_expr IS NULL，SELECT生成任意行

	2、如果SELECT只生成非空值或不生成任何值，则为FALSE

在这种情况下，使用outer_expr = inner_expr查找行的方法不再有效。有必要查找这样的行，但是如果没有找到，也要查找inner_expr为NULL的行。
粗略地说，子查询可以转换为如下内容:

	EXISTS (SELECT 1 FROM ... WHERE subquery_where AND
        (outer_expr=inner_expr OR inner_expr IS NULL))
	
	需要评估额外IS NULL条件是MySQL具有 ref_or_null访问方法的原因：
	
	mysql> EXPLAIN
		   SELECT outer_expr IN (SELECT t2.maybe_null_key
								 FROM t2, t3 WHERE ...)
		   FROM t1;
	*************************** 1. row ***************************
			   id: 1
	  select_type: PRIMARY
			table: t1
	...
	*************************** 2. row ***************************
			   id: 2
	  select_type: DEPENDENT SUBQUERY
			table: t2
			 type: ref_or_null
	possible_keys: maybe_null_key
			  key: maybe_null_key
		  key_len: 5
			  ref: func
			 rows: 2
			Extra: Using where; Using index
			
在unique_subquery和 index_subquery 子查询，具体的访问方法也有“ or NULL ”变种。

额外的 OR…IS NULL条件使查询执行稍微复杂一些(并且子查询中的一些优化变得不适用)，但通常这是可以容忍的。

如果outer_expr可以为NULL，情况就更糟了。根据SQL将NULL解释为“unknown value”，NULL IN (SELECT inner_expr…)的值应该为:

	NULL，如果选择产生任何行

	FALSE，如果SELECT不生成任何行
	
为了正确评估，必须能够检查SELECT是否生成了任何行，因此不能将outer_expr = inner_expr下推到子查询中。这是一个问题，因为许多真实世界的子查询变得非常慢，
除非等式可以下推。

本质上，必须有不同的方法来执行子查询，具体取决于outer_expr的值。

优化器选择SQL遵从性而不是速度，所以它解释了outer_expr可能为NULL的可能性:

	如果outer_expr为NULL，要对下面的表达式求值，需要执行SELECT来确定是否生成任何行:
		NULL IN (SELECT inner_expr FROM ... WHERE subquery_where)
		这里必须执行原始的选择，而不使用前面提到的那种下推等式。
	另一方面，当outer_expr不是NULL，这是绝对必要的比较:
		outer_expr IN (SELECT inner_expr FROM ... WHERE subquery_where)
		转换为使用下推条件的表达式:
		EXISTS (SELECT 1 FROM ... WHERE subquery_where AND outer_expr=inner_expr)
		如果不进行这种转换，子查询将会很慢。
		
为了解决是否将条件下推到子查询中的两难问题，条件被包装在“触发器”函数中。因此，下列形式的表达式:
	outer_expr IN (SELECT inner_expr FROM ... WHERE subquery_where)
	转化为：
	EXISTS (SELECT 1 FROM ... WHERE subquery_where
                          AND trigcond(outer_expr=inner_expr))
						  
更一般的情况是，如果子查询的比较是基于几对外部和内部表达式，转换采用以下比较:
	(oe_1, ..., oe_N) IN (SELECT ie_1, ..., ie_N FROM ... WHERE subquery_where)
	转换为：
	EXISTS (SELECT 1 FROM ... WHERE subquery_where
                          AND trigcond(oe_1=ie_1)
                          AND ...
                          AND trigcond(oe_N=ie_N)

每个trigcond(X)是一个特殊的函数，计算结果如下:

	当“链接的”外部表达式oe_i不为空为false

	当“链接的”外部表达式oe_i为空时为true

	请注意

	触发器函数不是使用create触发器创建的那种触发器。
	
包装在trigcond()函数中的值不是查询优化器的第一个类谓词。大多数优化不能处理在查询执行时可以打开和关闭的谓词，因此它们假设任何trigcond(X)都是未知的函数，
并忽略它。触发等式可以使用这些优化:

	引用优化:trigcond(X=Y[或Y is NULL])可用于构造ref、eq_ref或ref_or_null表访问。
	
	基于索引查找的子查询执行引擎:trigcond(X=Y)可用于构造unique_subquery或index_subquery访问。
	
	表条件生成器:如果子查询是多个表的联接，则会尽快检查触发的条件。
	
当优化器使用触发条件创建某种基于索引查找的访问时（对于前面列表的前两项），对于条件关闭的情况，优化器必须具有回退策略,这种回退策略总是相同的:
做一个全表扫描。在EXPLAIN输出中，回退显示为对空键的额外列的全扫描:

	mysql> EXPLAIN SELECT t1.col1,
		   t1.col1 IN (SELECT t2.key1 FROM t2 WHERE t2.col2=t1.col2) FROM t1\G
	*************************** 1. row ***************************
			   id: 1
	  select_type: PRIMARY
			table: t1
			...
	*************************** 2. row ***************************
			   id: 2
	  select_type: DEPENDENT SUBQUERY
			table: t2
			 type: index_subquery
	possible_keys: key1
			  key: key1
		  key_len: 5
			  ref: func
			 rows: 2
			Extra: Using where; Full scan on NULL key
			
如果你运行EXPLAIN之后 SHOW WARNINGS，你可以看到触发条件：


*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: select `test`.`t1`.`col1` AS `col1`,
         <in_optimizer>(`test`.`t1`.`col1`,
         <exists>(<index_lookup>(<cache>(`test`.`t1`.`col1`) in t2
         on key1 checking NULL
         where (`test`.`t2`.`col2` = `test`.`t1`.`col2`) having
         trigcond(<is_not_null_test>(`test`.`t2`.`key1`))))) AS
         `t1.col1 IN (select t2.key1 from t2 where t2.col2=t1.col2)`
         from `test`.`t1`
		 
使用触发条件会产生一些性能影响。NULL IN (SELECT…)表达式现在可能会导致全表扫描(速度很慢)，而以前它不会。这是为获得正确的结果所付出的代价
(触发条件策略的目标是提高遵从性，而不是速度)。

对于多表子查询，NULL IN (SELECT…)的执行特别慢，因为联接优化器没有针对外部表达式为NULL的情况进行优化。它假设左侧为NULL的子查询计算非常罕见，
即使有统计数据表明不是这样。另一方面，如果外部表达式可能为NULL，但实际上从未为NULL，则不会影响性能。

为了帮助查询优化器更好地执行您的查询，使用这些建议:

	如果列确实为NOT NULL，则将其声明为NOT NULL。通过简化列的条件测试，这也有助于优化器的其他方面。

	如果不需要区分NULL和FALSE子查询结果，则可以很容易地避免缓慢的执行路径。替换如下所示的比较:
		outer_expr IN (SELECT inner_expr FROM ...)
		转换为：
		(outer_expr IS NOT NULL) AND (outer_expr IN (SELECT inner_expr FROM ...))
	
		然后NULL IN (SELECT…)永远不会被计算，因为MySQL一旦表达式结果清楚就会停止计算。

	另一个可能的修改:
		EXISTS (SELECT inner_expr FROM ...
			WHERE inner_expr=outer_expr)

当您不需要区分NULL和FALSE子查询结果时，这将适用，在这种情况下，可以使用EXISTS


根据成本来选择子查询物化还是转化为exsits.
optimizer_switch系统变量的subquery_materialization_cost_based标志允许控制 子查询物化 和 IN-to-EXISTS 转换之间的选择。参见章节8.9.2“可切换优化”。