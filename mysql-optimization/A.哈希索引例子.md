-- 建表语句
CREATE TABLE testhash (
	fname VARCHAR(50) not null,
	lname varchar(50) not null,
	key using hash(fname)
)engine=memory;

-- 准备数据
INSERT INTO `testmybatis`.`testhash`(`fname`, `lname`) VALUES ('Afaewf', 'Ofnina');
INSERT INTO `testmybatis`.`testhash`(`fname`, `lname`) VALUES ('Bfqoe', 'Vfeqrg');
INSERT INTO `testmybatis`.`testhash`(`fname`, `lname`) VALUES ('Pfqeff', 'Ugfoerg');
INSERT INTO `testmybatis`.`testhash`(`fname`, `lname`) VALUES ('Rerfw', 'Hfqewf');

mysql> select * from testhash;
+--------+---------+
| fname  | lname   |
+--------+---------+
| Afaewf | Ofnina  |
| Bfqoe  | Vfeqrg  |
| Pfqeff | Ugfoerg |
| Rerfw  | Hfqewf  |
+--------+---------+
4 rows in set (0.00 sec)


假设f()函数，可以对fname进行哈希计算，
f(Afaewf) = 2345
f(Bfqoe) = 3825
f(Pfqeff) = 1298
f(Rerfw) = 5627

则索引的分布是这样的：
	+---------+----------------+
	| 槽(slot)| 值(value)      |
	+---------+----------------+
	| 1298    | 指向第3行数据  |
	| 2345    | 指向第1行数据  |
	| 3825    | 指向第2行数据  |
	| 5627    | 指向第4行数据  |
	+---------+----------------+

对于一个 SELECT 来说
	select * from testhash where fname = 'Pfqeff';
	1、先计算哈希值，f(fname) = 1298
	2、去哈希表中查找lot值，拿到指针，去表中看，fname 是否 等于 'Pfqeff',
		相等的话，则返回结果。
		不等的话，继续来哈希表，拿下一个指针，再去判断
		