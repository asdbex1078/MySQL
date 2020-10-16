由于SHOW警告显示的语句可能包含特殊标记，以提供有关查询重写或优化器操作的信息，因此该语句不一定是有效的SQL，也不打算执行。
输出还可能包括带有消息值的行，这些行提供关于优化器所采取的操作的附加非sql解释性注释。

```sql
mysql> EXPLAIN
       SELECT t1.a, t1.a IN (SELECT t2.a FROM t2) FROM t1\G
********* 1. row *********
           id: 1
  select_type: PRIMARY
        table: t1
         type: index
possible_keys: NULL
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using index
********* 2. row *********
           id: 2
  select_type: SUBQUERY
        table: t2
         type: index
possible_keys: a
          key: a
      key_len: 5
          ref: NULL
         rows: 3
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
********* 1. row *********
  Level: Note
   Code: 1003
Message: /* select#1 / select test.t1.a AS a,
         <in_optimizer>(test.t1.a,test.t1.a in
         ( <materialize> (/ select#2 */ select test.t2.a
         from test.t2 where 1 having 1 ),
         <primary_index_lookup>(test.t1.a in
         <temporary table> on <auto_key>
         where ((test.t1.a = materialized-subquery.a))))) AS t1.a
         IN (SELECT t2.a FROM t2) from test.t1
1 row in set (0.00 sec)

```



说明：
	1、该查询有两个select，即select#1和select#2
	2、<in_optimizer>：这是一个内部优化器对象，对用户没有任何意义。
	3、<materialize>：使用子查询实现
	4、<primary_index_lookup>：使用主键查找来处理查询片段以查找合格的行
	5、<temporary table>：这表示为缓存中间结果而创建的内部临时表。
	6、<auto_key>：为临时表自动生成的key
	7、`materialized-subquery`.`a` ： 实现了col_name对内部临时表中列的引用， 以保存评估子查询的结果。
	简化版的sql如下：

```sql
select t1.a,
         <in_optimizer>(
					 t1.a,t1.a in
					 ( <materialize> (/* select#2 */ select t2.a
					 from t2 where 1 having 1 ),
					 <primary_index_lookup>(t1.a in
					 <temporary table> on <auto_key>
					 where ((t1.a = materialized-subquery.a))))
				 ) AS t1.a
         IN (SELECT t2.a FROM t2) from t1
```



结论：		 
	一、结合EXPLAIN可以看出，先执行子查询，后执行外层select
	二、先查出select t2.a from t2 where 1 having 1；将结果缓存到内部临时表中，
	三、利用t1.a去in查询这个临时表
	四、最后返回结果