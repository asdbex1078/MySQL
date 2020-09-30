# MySQL

## MySQL优化

## MySQL存储引擎

### InnoDB

1. 从MySQL架构到InnoDB架构
   1. [MySQL架构](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#mysql%E4%BD%93%E7%B3%BB%E6%9E%B6%E6%9E%84)
   2. [InnoDB架构](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#innodb%E6%9E%B6%E6%9E%84%E5%9B%BE)
   3. [InnoDB的多线程模型](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#innodb%E7%9A%84%E4%B8%80%E4%B8%AA%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B)
2.  InnoDB简介
   1. [使用InnoDB的好处](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.%E6%80%BB-InnoDB%E7%AE%80%E4%BB%8B.md#%E4%BD%BF%E7%94%A8innodb%E7%9A%84%E5%A5%BD%E5%A4%84)
   2. [如何使InnoDB性能更高](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.%E6%80%BB-InnoDB%E7%AE%80%E4%BB%8B.md#%E5%A6%82%E4%BD%95%E4%BD%BFinnodb%E6%80%A7%E8%83%BD%E6%9B%B4%E9%AB%98)
   3. [查看当前库支持的存储引擎，以及默认的存储引擎](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.%E6%80%BB-InnoDB%E7%AE%80%E4%BB%8B.md#%E6%9F%A5%E7%9C%8B%E5%BD%93%E5%89%8D%E5%BA%93%E6%94%AF%E6%8C%81%E7%9A%84%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E%E4%BB%A5%E5%8F%8A%E9%BB%98%E8%AE%A4%E7%9A%84%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E)
3. InnoDB特性
   - [Insert Buffer——插入缓冲区](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.%E6%80%BB-InnoDB%E7%AE%80%E4%BB%8B.md#insert-buffer---%E6%8F%92%E5%85%A5%E7%BC%93%E5%86%B2)
   - [Change Buffer——变更缓冲区]()
   - [Double Write Buffer——双写缓冲区](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.%E6%80%BB-InnoDB%E7%AE%80%E4%BB%8B.md#double-write---%E4%B8%A4%E6%AC%A1%E5%86%99)
   - [Adaptive Hash Index——自适应哈希索引](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.%E6%80%BB-InnoDB%E7%AE%80%E4%BB%8B.md#adaptive-hash-index---%E8%87%AA%E9%80%82%E5%BA%94%E5%93%88%E5%B8%8C%E7%B4%A2%E5%BC%95)
   - [AIO——异步IO](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.%E6%80%BB-InnoDB%E7%AE%80%E4%BB%8B.md#aio---%E5%BC%82%E6%AD%A5io)
   - [刷新临近页](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.%E6%80%BB-InnoDB%E7%AE%80%E4%BB%8B.md#%E5%88%B7%E6%96%B0%E4%B8%B4%E8%BF%91%E9%A1%B5)
4.  InnoDB内存结构
   1. 缓冲池 - Buffer Pool
   2. 变更缓冲区 - Change Buffer 
   3. 双写缓冲区 - Double Write Buffer
   4. 自适应哈希索引 - Adaptive Hash Index
   5. 日志缓冲区 - Log Buffer
5. InnoDB磁盘结构
   1. 表 - table
   2. 索引 - index
   3. 表空间 - TableSpaces
   4. 数据字典 - Data Dictionary
6. 事务

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