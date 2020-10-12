您可能会想重写查询以使算术运算更快，同时又牺牲了可读性。由于MySQL自动进行类似的优化，因此您通常可以避免这项工作，而将查询保留为更易于理解和维护的形式。
MySQL执行的一些优化如下：

1、删除不必要的括号：
	   ((a AND b) AND c OR (((a AND b) AND (c AND d))))
	-> (a AND b AND c) OR (a AND b AND c AND d)
	
2、恒定折叠：
	   (a<b AND b=c) AND a=5
	-> b>5 AND b=c AND a=5
	
3、恒定条件消除：
	   (b>=5 AND b=5) OR (b=6 AND 5=5) OR (b=7 AND 5=6)
	-> b=5 OR b=6
	
4、索引使用的常量表达式只计算一次。

5、在没有WHERE的单个表上的COUNT（*）直接从MyISAM和MEMORY表的表信息中检索。当只查询一个表时，这也适用于任何NOT NULL表达式。

	例如： 
		SELECT COUNT(*) FROM tbl_name;
		
6、对无效常数表达式的早期检测。MySQL很快就发现一些SELECT语句是不成立的，并且不返回任何行。
	
	举例：可查看该语句的追踪过程，在优化阶段，判断到WHERE条件不成立，SQL也不会执行。(追踪过程查看8.1条件不成立的追踪过程.trace)
			select count(1) from user1 where 1=0;
	
7、如果不使用GROUP BY或聚合函数（COUNT（）、MIN（）等），having将于where合并。
	例如：select min(user_age) as user_age from user1 having user_age = 0;
		  select min(user_age) as user_age from user1 where user_age = 0;
		  
8、对于联接中的每个表，构造一个更简单的WHERE,以获得表的快速WHERE计算，并尽快跳过行。
	(写的SQL不一定是最终执行的SQL)
	
9、所有常量表在查询中的任何其他表之前首先读取。常量表是下列任何一种：
	a.空表或具有一行的表。
	b.与主键或唯一索引上的WHERE子句一起使用的表，其中所有索引部分都与常量表达式相比较，并定义为NOT NULL。
	c.索引上的 min() max()
	
	例如：以下所有表均用作常量表：
		SELECT * FROM t WHERE primary_key=1;
		
		SELECT * FROM t1,t2
		  WHERE t1.primary_key=1 AND t2.primary_key=t1.id;
		  
	    SELECT MIN(key_part1),MAX(key_part1) FROM tbl_name;
		
		SELECT MAX(key_part2) FROM tbl_name
		  WHERE key_part1=constant;
  
10、通过尝试所有的可能性，可以找到join表的最佳联接组合。如果ORDER BY和GROUP BY子句中的所有列都来自同一个表，则在连接时优先使用该表。

	例如：
		select * from a
		left join b on a.id = b.id
		left join c on a.id = c.id
		where 
			XXX
		(group by 
			a.id,a.name
		order by
			a.id,a.name);
			
	首先，并不是说，sql会按照a_b_c的方式，依次嵌套查询。
	然后，假设group by 和order by 都来自同一张表，则优先使用该表(尽量使用同一张表进行分组排序)。
	
11、如果有一个ORDER BY子句和一个不同的GROUP BY子句，或者如果ORDER BY或GROUP BY包含连接队列中第一个表以外的表的列，则会创建一个临时表。
	
12、按照索引的排序方式来排序，就不需要临时表执行排序操作
	
	例如以下sql可以用索引排序：
	SELECT ... FROM tbl_name
	  ORDER BY key_part1,key_part2,... limit 10;

	SELECT ... FROM tbl_name
	  ORDER BY key_part1 DESC, key_part2 DESC, ... limit 10;
	(对于DESC，MySQL会倒序查询索引)
	
	以下SQL无法用索引排序：
	SELECT ... FROM tbl_name
	  ORDER BY key_part1, key_part2 DESC, ... ;
	
13、如果明确知道结果很小，可以使用 SQL_SMALL_RESULT 告诉优化器，直接使用内存建临时表。
	select SQL_SMALL_RESULT min(user_age) as user_age from user1 where user_age = 0;
	
14、每个表索引都会被查询，除非优化器认为使用表扫描更有效，否则就会使用最佳索引。一次，基于最佳索引是否跨越了表的30%以上而使用扫描，
	但是固定的百分比不再决定使用索引还是扫描之间的选择。优化器现在更加复杂，并将其估计建立在其他因素上，如表大小、行数和I/O块大小。
	
15、在某些情况下，MySQL甚至不需要查阅数据文件就可以从索引中读取行。如果索引中使用的所有列都是数字的，则仅使用索引树解析查询。
	
	例如：
		SELECT key_part1,key_part2 FROM tbl_name WHERE key_part1=val;

		SELECT COUNT(*) FROM tbl_name
		  WHERE key_part1=val1 AND key_part2=val2;

		SELECT key_part2 FROM tbl_name GROUP BY key_part1;
	
16、在输出每一行之前，将跳过那些与 WHERE/HAVING 子句不匹配的行。

