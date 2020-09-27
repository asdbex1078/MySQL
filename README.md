# MySQL

## MySQL优化

## MySQL存储引擎

### InnoDB

#### [InnoDB架构](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.InnoDB%E6%9E%B6%E6%9E%84.md)

#### InnoDB简介

1. [使用InnoDB的好处](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.InnoDB%E7%AE%80%E4%BB%8B.md)
2. [如何使InnoDB性能更高](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.InnoDB%E7%AE%80%E4%BB%8B.md)
3. [查看当前库支持的存储引擎，以及默认的存储引擎](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.InnoDB%E7%AE%80%E4%BB%8B.md)
4. InnoDB特性
   - Insert Buffer——插入缓冲区
   - Change Buffer——变更缓冲区
   - Double Write Buffer——双写缓冲区
   - Adaptive Hash Index——自适应哈希索引
   - AIO——异步IO
   - 刷新临近页

#### InnoDB内存结构

1. 缓冲池 - Buffer Pool
2. 变更缓冲区 - Change Buffer 
3. 双写缓冲区 - Double Write Buffer
4. 自适应哈希索引 - Adaptive Hash Index
5. 日志缓冲区 - Log Buffer

#### InnoDB磁盘结构

1. 表 - table
2. 索引 - index
3. 表空间 - TableSpaces
4. 数据字典 - Data Dictionary
5. 双写缓冲区 - DoubleWrite Buffer
6. 重做日志 - Redo Log
7. 撤销日志 - Undo Log

#### 事务

### MYISAM

### Memory

### TempTable

## 涉及MySQL其他

> 参考文献：
>
> 1. [MySQL官方网站](https://dev.mysql.com/doc/refman/5.7/en/innodb-architecture.html)
> 2. 阿里数据库月报
>    - [InnoDB undo log漫游](http://mysql.taobao.org/monthly/2015/04/01/)
> 3. 《高性能MySQL 第3版》
> 4. 《MySQL技术内幕 InnoDB存储引擎》