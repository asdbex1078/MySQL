# InnoDB简介

## 使用InnoDB的好处

1. 如果您的服务器由于硬件或软件问题而意外退出，无论当时数据库中发生了什么，重新启动数据库后都不需要执行任何特殊操作。InnoDB`崩溃恢复`会自动完成**崩溃之前已提交**的所有更改，并撤消所有**正在处理但尚未提交**的更改。只需重新启动并从上次中断的地方继续即可。
2. InnoDB存储引擎有自己的`缓冲池`，在访问数据时可以在主存中缓存**表**和**索引数据**。经常使用的数据直接从**内存**中处理。此缓存适用于许多类型的信息，并加快处理速度。在专用数据库服务器上，通常会将高达**80%**的物理内存分配给缓冲池。
3. 如果将相关数据分割到不同的表中，则可以设置执行参照完整性的`外键`。更新或删除数据，其他表中的相关数据将**自动**更新或删除。如果尝试将数据插入到一个辅助表中，而主表中没有相应的数据，那么**错误的数据将自动被踢出**。
4. 如果数据在磁盘或内存中损坏，则`校验机制`会在使用前提醒您注意虚假数据。
5. 当您为每个表使用适当的`主键`列设计数据库时，涉及这些列的操作将**自动优化**。在**WHERE**子句、**ORDER BY**子句、**GROUP BY**子句和**join操作**中引用主键列非常快。
6. 插入、更新和删除由一种称为`change buffering`的自动机制进行优化。InnoDB不仅允许对同一个表并发读和写访问，它还**缓存修改后的数据**以简化磁盘I/O。
7. `自适应哈希索引`。当从一个表中一遍又一遍访问相同的数据时，InnoDB会自动对这些数据建立自适应哈希索引，就像从哈希表中查出来一样。
8. 您可以压缩表和关联的索引。
9. 您可以创建和删除索引，而对性能和可用性的影响要小得多。
10. 截断表空间文件非常快，可以释放磁盘空间供`操作系统`重用，而不是释放只有InnoDB才能重用的`系统表空间`内的空间。
11. 对于`BLOB`和长`TEXT`字段，使用`动态行格式`的表数据的存储布局更有效。
12. 您可以通过查询`INFORMATION_SCHEMA`表来监视存储引擎的内部工作。
13. 您可以通过`查询性能模式`表来监视存储引擎的性能细节。
14. 您可以自由地将`InnoDB`表与其他MySQL存储引擎的表混合使用，即使在同一条语句中也是如此。例如，您可以使用JOIN操作在单个查询中合并来自`InnoDB`和 `MEMORY`表的数据。
15. InnoDB是为处理大数据量时的CPU效率和最高性能而设计的。
16. InnoDB表可以处理大量数据，即使是在文件大小限制在2GB的操作系统上。

## 如何使InnoDB性能更高

1. 为每个表中查询最频繁的一列或几列指定一个主键，如果没有明显的主键，则指定一个**自动递增的值**。
2. 使用join连接从多个表中根据相同的ID值提取数据时，为了提高连接性能，可以在连接列上定义`外键`，并在每个表中使用**相同的数据类型**声明这些列。**添加外键可以确保引用的列被索引**，这可以提高性能。外键还将**删除**或**更新**传播到所有受影响的表，并防止在父表中没有对应id的情况下在子表中插入数据。
3. 关闭`自动提交`。每秒提交数百次会限制性能(受存储设备的写入速度限制)。
4. 通过使用`START TRANSACTION`和`COMMIT`语句将相关的DML操作sql分组到事务中。不建议过于频繁地提交，也不建议发出一个批次中含有大量的INSERT、UPDATE或DELETE语句，这些语句可能运行几个小时。
5. 不建议使用`LOCK TABLES` 语句。`InnoDB`可以同时处理对同一个表进行读写的多个会话，而无需牺牲可靠性或高性能。要获得对一组行的排他性写访问权限，请使用 [`SELECT ... FOR UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)语法仅锁定要更新的行。
6. 启用`innodb_file_per_table`选项，或者使用常规`表空间`将表的`数据`和`索引`放在**单独的文件**中，而不是`系统表空间`中。
   默认情况下`innodb_file_per_table`选项是启用的。
7. 评估您的数据和访问模式是否可以从`InnoDB`表或页面压缩功能中受益。您可以在`InnoDB`不牺牲读/写功能的情况下压缩表。
8. 使用选项`sql_mode=NO_ENGINE_SUBSTITUTION`运行服务器，以防止在CREATE TABLE的engine =子句中指定的引擎出现问题时使用不同的存储引擎创建表。

## 查看当前库支持的存储引擎，以及默认的存储引擎

> mysql> SHOW ENGINES;
>|       Engine       | Support |                           Comment                            | Transactions |  XA  | Savepoints |
> | :----------------: | :-----: | :----------------------------------------------------------: | :----------: | :--: | :--------: |
> |       InnoDB       | DEFAULT |  Supports transactions, row-level locking, and foreign :-:   |     eys      | YES  |    YES     |
> |     MRG_MYISAM     |   YES   |            Collection of identical MyISAM tables             |      NO      |  NO  |     NO     |
> |       MEMORY       |   YES   |  Hash based, stored in memory, useful for temporary tables   |      NO      |  NO  |     NO     |
> |     BLACKHOLE      |   YES   | /dev/null storage engine (anything you write to it disappears) |      NO      |  NO  |     NO     |
> |       MyISAM       |   YES   |                    MyISAM storage engine                     |      NO      |  NO  |     NO     |
> |        CSV         |   YES   |                      CSV storage engine                      |      NO      |  NO  |     NO     |
> |      ARCHIVE       |   YES   |                    Archive storage engine                    |      NO      |  NO  |     NO     |
> | PERFORMANCE_SCHEMA |   YES   |                      Performance Schema                      |      NO      |  NO  |     NO     |
> |     FEDERATED      |   NO    |                Federated MySQL storage engine                |     NULL     | NULL |    NULL    |

> mysql> SELECT * FROM INFORMATION_SCHEMA.ENGINES;
>
> |       Engine       | Support |                           Comment                            | Transactions |  XA  | Savepoints |
> | :----------------: | :-----: | :----------------------------------------------------------: | :----------: | :--: | :--------: |
> |       InnoDB       | DEFAULT |  Supports transactions, row-level locking, and foreign :-:   |     eys      | YES  |    YES     |
> |     MRG_MYISAM     |   YES   |            Collection of identical MyISAM tables             |      NO      |  NO  |     NO     |
> |       MEMORY       |   YES   |  Hash based, stored in memory, useful for temporary tables   |      NO      |  NO  |     NO     |
> |     BLACKHOLE      |   YES   | /dev/null storage engine (anything you write to it disappears) |      NO      |  NO  |     NO     |
> |       MyISAM       |   YES   |                    MyISAM storage engine                     |      NO      |  NO  |     NO     |
> |        CSV         |   YES   |                      CSV storage engine                      |      NO      |  NO  |     NO     |
> |      ARCHIVE       |   YES   |                    Archive storage engine                    |      NO      |  NO  |     NO     |
> | PERFORMANCE_SCHEMA |   YES   |                      Performance Schema                      |      NO      |  NO  |     NO     |
> |     FEDERATED      |   NO    |                Federated MySQL storage engine                |     NULL     | NULL |    NULL    |