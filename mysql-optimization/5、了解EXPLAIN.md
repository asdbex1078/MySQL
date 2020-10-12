EXPLAIN语句提供有关MySQL如何执行语句的信息：

	1、EXPLAIN可解释的语句有 SELECT， DELETE， INSERT， REPLACE，和 UPDATE。

	2、当EXPLAIN与可解释的语句一起使用时，MySQL将显示来自优化器的有关语句执行计划的信息。也就是说，MySQL解释了它将如何处理该语句，
	包括有关如何连接表以及以何种顺序连接表的信息。有关 EXPLAIN用于获取执行计划信息的信息，请参见第8.8.2节“ EXPLAIN输出格式”。

	3、当EXPLAIN与FOR CONNECTION CONNECTION_ID而不是可解释语句一起使用时，它将显示在命名连接中执行的语句的执行计划。
	请参阅第8.8.4节“获取命名连接的执行计划信息”。
	
	4、对于SELECT语句，EXPLAIN会生成附加的执行计划信息，这些信息可以使用SHOW WARNINGS显示。请参见 第8.8.3节“扩展的EXPLAIN输出格式”。
	（可大概知道优化器优化后的SQL，但不是最终执行的SQL）
	
	5、EXPLAIN对于检查涉及分区表的查询很有用。请参见 第22.3.5节“获取有关分区的信息”。

	6、Format选项可用于选择输出格式。Traditional以表格格式显示输出(默认就是这种方式)。如果不存在格式选项，则这是默认设置。JSON格式以JSON格式显示信息。
	例如：explain format=json select user_name,user_age from user1 where user_age <= 20;
	JSON的优势在于，可以看到成本是多少。
	
	7、EXPLAIN返回SELECT语句中使用的每个表的一行信息。它按照MySQL在处理语句时读取的顺序列出输出中的表。MySQL使用嵌套循环联接方法解析所有联接。
	这意味着MySQL从第一个表中读取一行，然后在第二个表、第三个表中查找匹配的行，依此类推。处理完所有表后，MySQL会通过表列表输出选定的列和回溯，
	直到找到有更多匹配行的表。从该表中读取下一行，该过程从下一个表继续。

	8、解释输出包括分区信息。此外，对于SELECT语句，EXPLAIN生成可以在EXPLAIN之后显示警告的扩展信息(请参阅第8.8.3节，“扩展的解释输出格式”)。
	
在EXPLAIN的帮助下，您可以看到应该在何处向表添加索引，以便通过使用索引查找行来加快语句的执行速度。您还可以使用EXPLAIN检查优化器是否以最佳顺序连接表。
要提示优化器使用与SELECT语句中的表命名顺序相对应的联接顺序，语句应以SELECT STRECT_JOIN开头，而不仅仅是SELECT。(请参见第13.2.9节，“SELECT语句”。)。
但是，STRECT_JOIN可能会阻止使用索引，因为它禁用了半连接转换。请参阅第8.2.2.1节“使用半联接转换优化子查询、派生表和视图引用”。
(STRECT_JOIN关键词，用于提示优化器，按照SQL编写顺序来执行。可紧挨着select，也可以在多个表之间使用)

优化器跟踪有时可能提供补充解释的信息。但是，优化器跟踪格式和内容可能会因版本而异。有关详细信息，请参见MySQL内部：跟踪优化器。

如果您在认为应该使用索引时遇到问题，请运行Analyze TABLE来更新表统计信息，例如键的基数，这可能会影响优化器所做的选择。请参阅第13.7.2.1节“分析TABLE语句”。

注意：
EXPLAIN还可用于获取有关表中列的信息。EXPLAIN tbl_name等同于DESCRIBE tbl_name和show column from tbl_name。
有关详细信息，请参阅13.8.1节“DESCRIBE语句”和13.7.5.5节“SHOW COLUMNS语句”。