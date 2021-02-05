```sql
mysql> select user_age from user2 GROUP BY user_age;
+----------+
| user_age |
+----------+
|       10 |
|       11 |
|       12 |
|       13 |
|       14 |
|       15 |
|       16 |
|       20 |
|       40 |
+----------+
9 rows in set (0.04 sec)
```

挑中 20 -40之间

在RR隔离级别下，在 user_age 20 和 40之间插入数据。 且user_age为普通索引，有两个事务，

事务1: select * from table where user_age=22 for update。 insert into table (user_age) values (22)，他会在 [20,22) (22,40) 之间加间隙锁，在(22)这个位置申请插入意向锁。 

事务2: select * from table where user_age=30 for update。insert into table (user_age) values (30)。 它会在 [20,30) (30,40) 之间加间隙锁，由于间隙锁互相兼容，故该锁可以获取，另外在(30)这个地方申请插入意向锁。


 当事务1要获取插入意向锁时，发现(22)被事务2的间隙锁锁住了，故等待事务2释放锁； 事务2要获取插入意向锁时，发现(30)被事务1的间隙锁锁住了，故等待事务1释放锁；——死锁形成

```sql
-- 第一步： session 1
set autocommit = 0;
mysql> select * from user2 where user_age = 22 for update;
Empty set

-- 第二步： session 2
set autocommit = 0;
mysql> select * from user2 where user_age = 30 for update;
Empty set

-- 第三步： session 1
mysql> INSERT INTO `testmybatis`.`user2` ( `user_name`, `user_age`, `user_password`, `user_sex`, `user_province`, `user_city`, `user_area`, `create_time`, `modified_time` )
VALUES
	( '陶守改', 22, '58f8414ff5c74a91a00fe99aacd5b2b8', 0, '福建省', '沧州市', '友好区', '2020-08-18 15:45:08', '2020-12-10 11:08:10' );
-- 这里卡住了。。。。。。。。	
	
	
-- 第四步： session 2
mysql> INSERT INTO `testmybatis`.`user2` ( `user_name`, `user_age`, `user_password`, `user_sex`, `user_province`, `user_city`, `user_area`, `create_time`, `modified_time` )
VALUES
	( '陶守改', 30, '58f8414ff5c74a91a00fe99aacd5b2b8', 0, '福建省', '沧州市', '友好区', '2020-08-18 15:45:08', '2020-12-10 11:08:10' );
1213 - Deadlock found when trying to get lock; try restarting transaction

-- 第五步：返回来看 session 1
-- 中间由于获取插入意向锁，等了五秒，五秒后发生死锁，事务2因死锁被取消。事务1获取到锁，执行成功
Query OK, 1 row affected (5.94 sec)
```

