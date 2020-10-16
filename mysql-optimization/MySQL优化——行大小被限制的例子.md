1、下面的InnoDB和MyISAM示例演示了MySQL最大行大小限制为65,535字节。无论存储引擎是什么，都会强制执行此限制，即使存储引擎可能能够支持更大的行。
		

```mysql
	mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
			   c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
			   f VARCHAR(10000), g VARCHAR(6000)) ENGINE=InnoDB CHARACTER SET latin1;
		ERROR 1118 (42000): Row size too large. The maximum row size for the used
		table type, not counting BLOBs, is 65535. This includes storage overhead,
		check the manual. You have to change some columns to TEXT or BLOBs
	
	mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
		   c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
		   f VARCHAR(10000), g VARCHAR(6000)) ENGINE=MyISAM CHARACTER SET latin1;
	ERROR 1118 (42000): Row size too large. The maximum row size for the used
	table type, not counting BLOBs, is 65535. This includes storage overhead,
	check the manual. You have to change some columns to TEXT or BLOBs
	
解决方案：将列改为TEXT 或 BOLBS
Ⅰ 在下面的MyISAM示例中，将列更改为TEXT可以避免65,535字节的行大小限制，并允许操作成功，因为BLOB和TEXT列只占行大小的9到12个字节。
	mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
		   c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
		   f VARCHAR(10000), g TEXT(6000)) ENGINE=MyISAM CHARACTER SET latin1;
	Query OK, 0 rows affected (0.02 sec)
	
Ⅱ 对于InnoDB表，这个操作成功了，因为将列更改为文本避免了MySQL的65,535字节的行大小限制，并且InnoDB对可变长度列的脱机存储也避免了InnoDB的行大小限制。
	mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
		   c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
		   f VARCHAR(10000), g TEXT(6000)) ENGINE=InnoDB CHARACTER SET latin1;
	Query OK, 0 rows affected (0.02 sec)
```

2、可变长度列的存储包括长度字节，这些字节按行大小计算。例如，一个VARCHAR(255)字符集utf8mb3列需要两个字节来存储值的长度，因此每个值最多需要767字节。

```mysql
创建表t1的语句成功了，因为列需要32,765 + 2字节和32,766 + 2字节，这符合最大行大小65,535字节:
mysql> CREATE TABLE t1
   (c1 VARCHAR(32765) NOT NULL, c2 VARCHAR(32766) NOT NULL)
   ENGINE = InnoDB CHARACTER SET latin1;
Query OK, 0 rows affected (0.02 sec)

创建表t2的语句失败了，因为尽管列长度在最大长度65,535字节之内，但是需要两个额外的字节来记录长度，这导致行大小超过65,535字节:
mysql> CREATE TABLE t2
   (c1 VARCHAR(65535) NOT NULL)
   ENGINE = InnoDB CHARACTER SET latin1;
ERROR 1118 (42000): Row size too large. The maximum row size for the used
table type, not counting BLOBs, is 65535. This includes storage overhead,
check the manual. You have to change some columns to TEXT or BLOBs

将列长度减少到65,533或更少允许语句成功执行。
mysql> CREATE TABLE t2
   (c1 VARCHAR(65533) NOT NULL)
   ENGINE = InnoDB CHARACTER SET latin1;
Query OK, 0 rows affected (0.01 sec)
```

3、对于MyISAM表，NULL列需要在行中额外的空间来记录它们的值是否为NULL。每个空列多取一位，四舍五入到最近的字节。
	创建t3表的语句失败了，因为MyISAM除了需要可变长度列长度字节的空间外，还需要空列空间，这导致行大小超过65,535个字节:
		

```sql
mysql> CREATE TABLE t3
       (c1 VARCHAR(32765) NULL, c2 VARCHAR(32766) NULL)
       ENGINE = MyISAM CHARACTER SET latin1;
	ERROR 1118 (42000): Row size too large. The maximum row size for the used
	table type, not counting BLOBs, is 65535. This includes storage overhead,
	check the manual. You have to change some columns to TEXT or BLOBs

关于InnoDB空列存储的信息，参见14.11节“InnoDB行格式”。
```

4、InnoDB将行大小(对于存储在本地数据库页中的数据)限制为4KB、8KB、16KB和32KB的innodb_page_size设置为略小于半个数据库页，对于64KB的页限制为略小于16KB。
	创建表t4的语句失败了，因为定义的列超过了16KB InnoDB页的行大小限制。

```sql
mysql> CREATE TABLE t4 (
		   c1 CHAR(255),c2 CHAR(255),c3 CHAR(255),
		   c4 CHAR(255),c5 CHAR(255),c6 CHAR(255),
		   c7 CHAR(255),c8 CHAR(255),c9 CHAR(255),
		   c10 CHAR(255),c11 CHAR(255),c12 CHAR(255),
		   c13 CHAR(255),c14 CHAR(255),c15 CHAR(255),
		   c16 CHAR(255),c17 CHAR(255),c18 CHAR(255),
		   c19 CHAR(255),c20 CHAR(255),c21 CHAR(255),
		   c22 CHAR(255),c23 CHAR(255),c24 CHAR(255),
		   c25 CHAR(255),c26 CHAR(255),c27 CHAR(255),
		   c28 CHAR(255),c29 CHAR(255),c30 CHAR(255),
		   c31 CHAR(255),c32 CHAR(255),c33 CHAR(255)
		   ) ENGINE=InnoDB ROW_FORMAT=COMPACT DEFAULT CHARSET latin1;
	ERROR 1118 (42000): Row size too large (> 8126). Changing some columns to TEXT or BLOB or using
	ROW_FORMAT=DYNAMIC or ROW_FORMAT=COMPRESSED may help. In current row format, BLOB prefix of 768
	bytes is stored inline.
```





