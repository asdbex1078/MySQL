表定义如下：
	CREATE TABLE t (id INT NOT NULL PRIMARY KEY, col_a VARCHAR(100));
	
有以下两个条件
	SELECT * FROM t WHERE id = POW(1,2);
	SELECT * FROM t WHERE id = FLOOR(1 + RAND() * 49);
	
这两个查询似乎都使用了主键查找，这是由于与主键的相等比较，但这只对第一个在优化阶段可以优化成常量，值为TRUE:

	第一个查询总是生成一行的最大值，因为带有常量参数的POW()是一个常量值，可以用索引查找。

	第二个查询包含一个表达式,使用不确定性函数RAND(),他不是一个常数，每次都会有不同的新值。因此,他可能匹配0行、1行或多行，
	具体取决于id列值和RAND()序列中的值。

非确的影响并不局限于选择语句。这个UPDATE语句使用一个非确定性函数来选择要修改的行:
	UPDATE t SET col_a = some_expr WHERE id = FLOOR(1 + RAND() * 49);
	其目的大概是至多更新主键与表达式匹配的单行。但是，它可能更新零行、一行或多行，具体取决于id列值和RAND()序列中的值。
