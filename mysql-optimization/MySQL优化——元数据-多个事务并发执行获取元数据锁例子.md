当多个事务并发执行时，元数据锁获取顺序会对操作结果产生影响，如下面的示例所示。
Client 1:
	LOCK TABLE x WRITE, x_new WRITE;
	该语句在x和x_new上按名称顺序请求和获取写锁。

Client 2:
	INSERT INTO x VALUES(1);
	该语句请求和阻塞等待x上的写锁。
	
Client 3:
	RENAME TABLE x TO x_old, x_new TO x;
	该语句以名称顺序请求x、x_new和x_old上的独占锁，但是阻塞等待x上的锁

Client 1:
	UNLOCK TABLES;

该语句释放x和x_new上的写锁。客户机3对x的排他锁请求比客户机2的写锁请求具有更高的优先级，因此客户机3在x上获得它的锁，然后在x_new和x_old上也获得锁，
执行重命名并释放锁。然后客户2获得它在x上的锁，执行插入并释放锁。

锁获取顺序导致重命名表在插入之前执行。当客户端2发出插入命令时，x被命名为x_new，并被客户端3重命名为x:

```sql
mysql> SELECT * FROM x;
	+------+
	| i    |
	+------+
	|    1 |
	+------+

mysql> SELECT * FROM x_old;
Empty set (0.01 sec)

```



现在从具有相同结构的名为x和new_x的表开始。同样，有三个客户机发出涉及以下表的语句:
Client 1:
	LOCK TABLE x WRITE, new_x WRITE;
	该语句在new_x和x上按名称顺序请求和获取写锁。
	
Client 2:
	INSERT INTO x VALUES(1);
	该语句请求和阻塞等待x上的写锁。
	
Client 3:
	RENAME TABLE x TO old_x, new_x TO x;
	该语句以名称顺序请求new_x、old_x和x上的独占锁，但是阻塞等待new_x上的锁。
	
Client 1:
	UNLOCK TABLES;
	
该语句释放x和new_x上的写锁。对于x，唯一挂起的请求是由客户机2发出的，因此客户机2获得它的锁，执行插入，然后释放锁。对于new_x，
唯一挂起的请求是由客户机3发出的，它被允许获取该锁(以及old_x上的锁)。重命名操作仍然阻塞x上的锁，直到客户端2插入完成并释放锁为止。
然后客户3获得x上的锁，执行重命名，并释放锁。

在这种情况下，锁获取顺序导致INSERT在重命名表之前执行。插入到的x是原来的x，现在通过重命名操作重命名为old_x:

```sql
mysql> SELECT * FROM x;
Empty set (0.01 sec)

mysql> SELECT * FROM old_x;
+------+
| i    |
+------+
|    1 |
+------+

```



如果并发语句中的锁获取顺序对应用程序的操作结果产生影响(如前面的示例所示)，则可以调整表名以影响锁获取顺序。