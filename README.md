# MySQL

## MySQL优化

## MySQL存储引擎

### InnoDB

1.  #### 从MySQL架构到InnoDB架构

   1. [MySQL架构](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#mysql%E4%BD%93%E7%B3%BB%E6%9E%B6%E6%9E%84)

   2. [InnoDB架构](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#innodb)

   3. [InnoDB的多线程模型](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#innodb%E7%9A%84%E4%B8%80%E4%B8%AA%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B)
- [Master Thread](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#master-thread---%E6%A0%B8%E5%BF%83%E7%BA%BF%E7%A8%8B)
  
- [IO Thread](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#io-thread)
  
- [Purge Thread](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#purge-thread)
  
- [Page Cleaner Thread](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.0.MySQL%E6%9E%B6%E6%9E%84%E5%88%B0innoDB%E6%9E%B6%E6%9E%84.md#page-cleaner-thread)
4. InnoDB简介

5. InnoDB关键特性
   
   - [Insert Buffer——插入缓冲区](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.1.InnoDB%E2%80%94%E2%80%94%E5%85%B3%E9%94%AE%E7%89%B9%E6%80%A7.md#insert-buffer---%E6%8F%92%E5%85%A5%E7%BC%93%E5%86%B2)
   
   - [Double Write Buffer——双写缓冲区](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.1.InnoDB%E2%80%94%E2%80%94%E5%85%B3%E9%94%AE%E7%89%B9%E6%80%A7.md#double-write---%E4%B8%A4%E6%AC%A1%E5%86%99)
   
   - [Adaptive Hash Index——自适应哈希索引](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.1.InnoDB%E2%80%94%E2%80%94%E5%85%B3%E9%94%AE%E7%89%B9%E6%80%A7.md#adaptive-hash-index---%E8%87%AA%E9%80%82%E5%BA%94%E5%93%88%E5%B8%8C%E7%B4%A2%E5%BC%95)
   
   - [AIO——异步IO]()
   
   - [刷新临近页](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.1.InnoDB%E2%80%94%E2%80%94%E5%85%B3%E9%94%AE%E7%89%B9%E6%80%A7.md#%E5%88%B7%E6%96%B0%E4%B8%B4%E8%BF%91%E9%A1%B5)
   
   - [预读](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.1.1.InnoDB%E2%80%94%E2%80%94%E5%85%B3%E9%94%AE%E7%89%B9%E6%80%A7.md#%E9%A2%84%E8%AF%BB)
   
6. #### InnoDB内存结构

   1. [缓冲池 - Buffer Pool](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.2.0.InnoDB%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E7%BC%93%E5%86%B2%E6%B1%A0.md#%E7%BC%93%E5%86%B2%E6%B1%A0)
      - 为什么会出现
      - 是什么
      - 架构
      - 关键概念 - 数据页
      - 怎么做的
      - 服务器的内存大小很重要
      - 缓冲池中缓存的数据页的类型
      - 怎么识别数据在哪个缓冲页中
      - 缓冲池实例
      - Buffer Pool如何应对高并发场景
      - Buffer Pool的初始化
      - LRU List
      - Free List
      - Flush List
      - 总结
   2. [日志缓冲区 - Log Buffer](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.2.1.InnoDB%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94log%20buffer.md#log-buffer)
      - redo log buffer
      - undo log buffer
   3. [变更缓冲区 - Change Buffer](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.2.2.InnoDB%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94change%20buffer.md)
      - 简介
      - 什么时候合并到数据页
      - 在架构中的位置
      - 作用
      - 组成部分
      - 参数配置
      - 监控变更缓冲区
      - 其他信息
   4. [额外内存池](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.2.3.InnoDB%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E9%A2%9D%E5%A4%96%E5%86%85%E5%AD%98%E6%B1%A0.md)

7. #### InnoDB磁盘结构

   1. [页逻辑存储结构](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.3.0.InnoDB%E7%A3%81%E7%9B%98%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E9%80%BB%E8%BE%91%E5%AD%98%E5%82%A8%E7%BB%93%E6%9E%84.md#innodb%E9%A1%B5%E9%80%BB%E8%BE%91%E5%AD%98%E5%82%A8%E7%BB%93%E6%9E%84)
      - 页的结构
        - 页中插入数据的过程
        - COMPACT行格式
        - Page Directory
        - Page Header
        - File Header
        - File Trailer
      - 总结
   2. [表 - table](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.3.1.InnoDB%E7%A3%81%E7%9B%98%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E8%A1%A8.md#%E8%A1%A8)
   3. [索引 - index](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.3.2.InnoDB%E7%A3%81%E7%9B%98%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E7%B4%A2%E5%BC%95.md#%E7%B4%A2%E5%BC%95)
      - 聚簇索引和二级索引
      - InnoDB索引的物理结构
      - 排序索引的创建
   4. [表空间 - TableSpaces](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.3.3.InnoDB%E7%A3%81%E7%9B%98%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E8%A1%A8%E7%A9%BA%E9%97%B4.md#tablespaces)
      - 系统表空间
      - 每个表的表空间
      - InnoDB内存中对 .ibd 文件的管理
      - 数据字典和 .idb 文件的关系
      - 通用表空间
      - 创建通用表空间
      - 撤销表空间
      - 临时表空间
   5. [数据字典 - Data Dictionary]()

8. #### MySQL文件

   1. [参数文件](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.0.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94%E5%8F%82%E6%95%B0%E6%96%87%E4%BB%B6.md#%E5%8F%82%E6%95%B0%E6%96%87%E4%BB%B6)
      - 作用
      - 参数文件类型
      - 参数类型
      - 参数文件加载顺序
   2. 日志文件
      - [错误日志](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.1.0.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6%E4%B9%8B%E9%94%99%E8%AF%AF%E6%97%A5%E5%BF%97.md)
      - [慢查询日志](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.1.1.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6%E4%B9%8B%E6%85%A2%E6%9F%A5%E8%AF%A2%E6%97%A5%E5%BF%97.md#%E6%85%A2%E6%9F%A5%E8%AF%A2%E6%97%A5%E5%BF%97---slow-query-log)
        - 作用
        - 查看慢日志是否开启 / 慢日志位置
        - 满足什么条件会记录到慢日志中
        - 测试慢日志记录
        - mysqldumpslow的高级用法
        - 慢查询日志的输出方式
      - [二进制日志](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.1.2.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6%E4%B9%8B%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%97%A5%E5%BF%97.md#%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%97%A5%E5%BF%97)
        - 简介
        - 参数控制
        - 二进制日志的切换
        - 删除二进制日志文件
        - 相关参数
        - 查看二进制日志内容
      - [查询日志](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.1.3.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6%E4%B9%8B%E6%9F%A5%E8%AF%A2%E6%97%A5%E5%BF%97.md#%E6%9F%A5%E8%AF%A2%E6%97%A5%E5%BF%97)
   3. [socket文件](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.2.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94socket%E6%96%87%E4%BB%B6.md)
   4. [pid文件](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.3.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94pid%E6%96%87%E4%BB%B6.md)
   5. [MYISAM引擎文件](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.4.0.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94MYISAM%E6%96%87%E4%BB%B6.md)
   6. [InnoDB常规表文件](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.4.1.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94InnoDB%E5%B8%B8%E8%A7%84%E8%A1%A8%E6%96%87%E4%BB%B6.md#innodb%E7%9A%84%E8%A1%A8%E6%96%87%E4%BB%B6)
   7. [InnoDB Redo Log](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.4.2.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94InnoDB%20Redo%20Log.md#%E9%87%8D%E5%81%9A%E6%97%A5%E5%BF%97redo-log)
      - 概念及作用
      - redo log的组成部分
      - 内存中的 redo log buffer
      - 磁盘上的 redo log file 文件
      - 日志块 log block
      - log group 和redo log file
      - redo log格式
      - 什么时候产生redo log？什么时候日志刷盘？
      - 脏页刷盘机制
      - 什么时候释放redo log
      - InnoDB的恢复行为
      - 举例
      - 总结
   8. [InnoDB Undo Log](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.4.3.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94InnoDB%20Undo%20Log.md#%E5%9B%9E%E6%BB%9A%E6%97%A5%E5%BF%97-undo-log)
      - 了解几个概念
      - 简介
      - 作用
      - 存储方式
      - 关键结构体
      - purge线程
      - group commit
      - 存储位置
   9. [LSN](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.5.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94LSN.md#lsn)
      - 什么是 LSN 
      - 根据 LSN 能获得什么信息
      - LSN 存在的位置
      - 查看 LSN 信息
      - 举例
   10. [bin log 和 redo log 的区别](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.6.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94bin%20log%E5%92%8Credo%20log%E7%9A%84%E5%8C%BA%E5%88%AB.md#%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6-%E5%92%8C-%E9%87%8D%E5%81%9A%E6%97%A5%E5%BF%97-%E4%B9%8B%E9%97%B4%E7%9A%84%E5%8C%BA%E5%88%AB)
   11. [bin log、redo log、undo log写入顺序 - 案例](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.4.8.Mysql%E6%96%87%E4%BB%B6%E2%80%94%E2%80%94bin%20log%E3%80%81redo%20log%E3%80%81undo%20log%E5%86%99%E5%85%A5%E9%A1%BA%E5%BA%8F-%E6%A1%88%E4%BE%8B.md#%E4%B8%BE%E4%B8%AA%E4%BE%8B%E5%AD%90---bin-logredo-logundo-log%E5%86%99%E5%85%A5%E9%A1%BA%E5%BA%8F)

9. #### InnoDB锁

   1. [latch - 线程锁](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.5.0.InnoDB%E9%94%81%E2%80%94%E2%80%94latch.md#latch---%E7%BA%BF%E7%A8%8B%E9%94%81)
      - 简介
      - 作用
      - 举例
      - 如何查看
      - 分类
      - latch争用发生的原因
      - 如何降低latch争用
      - InnoDB Buffer Pool并发控制加锁过程
   2. [lock - 事务锁 介绍](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.5.1.InnoDB%E9%94%81%E2%80%94%E2%80%94lock%E4%B9%8B%E4%BB%8B%E7%BB%8D.md#lock---%E4%BA%8B%E5%8A%A1%E9%94%81)
      - 简介与latch区别
      - 加锁位置
      - 加锁原则
   3. [latch 和 lock 的区别](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.5.2.InnoDB%E9%94%81%E2%80%94%E2%80%94latch%E5%92%8Clock%E7%9A%84%E5%8C%BA%E5%88%AB.md#latch-%E5%92%8C-lock-%E7%9A%84%E5%8C%BA%E5%88%AB)
   4. [lock分类](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.5.3.InnoDB%E9%94%81%E2%80%94%E2%80%94lock%E4%B9%8B%E5%88%86%E7%B1%BB.md#%E9%94%81%E7%9A%84%E5%88%86%E7%B1%BB)
      - [表级锁](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.5.3.InnoDB%E9%94%81%E2%80%94%E2%80%94lock%E4%B9%8B%E5%88%86%E7%B1%BB.md#%E8%A1%A8%E7%BA%A7%E9%94%81)
        - 意向锁
        - 自增自动上锁
      - [行级锁](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.5.3.InnoDB%E9%94%81%E2%80%94%E2%80%94lock%E4%B9%8B%E5%88%86%E7%B1%BB.md#%E8%A1%8C%E7%BA%A7%E9%94%81)
        - 共享读锁
        - 排他写锁
        - 记录锁
        - 间隙锁
        - next-key锁
        - 插入意向锁
        - 空间所以你谓词锁
      - 页级锁
   5. [lock加锁案例分析](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.5.4.InnoDB%E9%94%81%E2%80%94%E2%80%94lock%E4%B9%8B%E5%8A%A0%E9%94%81%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90.md#%E5%8A%A0%E9%94%81%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90)
   6. [不同SQL加的不同lock锁](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.5.5.InnoDB%E9%94%81%E2%80%94%E2%80%94lock%E4%B9%8B%E4%B8%8D%E5%90%8CSQL%E5%8A%A0%E9%94%81%E5%88%86%E6%9E%90.md#innodb%E4%B8%AD%E7%94%B1%E4%B8%8D%E5%90%8C%E7%9A%84sql%E7%94%9F%E6%88%90%E7%9A%84%E9%94%81)
   7. [死锁](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.5.6.InnoDB%E9%94%81%E2%80%94%E2%80%94%E6%AD%BB%E9%94%81.md#%E6%AD%BB%E9%94%81)
      - 死锁简单案例
      - 死锁本质
      - InnoDB死锁实例
      - 死锁必要条件
      - InnoDB死锁检测
      - 死锁检测机制 - wait-for graph
      - 禁用死锁检测
      - 如何避免死锁

10. #### InnoDB事务

   1. [事务概念及分类](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.6.0.InnoDB%E4%BA%8B%E5%8A%A1%E2%80%94%E2%80%94%E6%A6%82%E5%BF%B5%E5%8F%8A%E5%88%86%E7%B1%BB.md#%E6%A6%82%E5%BF%B5)
      - 概念
      - 分类
        - 扁平事务
        - 带有保存点的扁平事务
        - 链事务
        - 嵌套事务
        - 分布式事务
   2. [一些前提知识点](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.6.1.InnoDB%E4%BA%8B%E5%8A%A1%E2%80%94%E2%80%94%E5%89%8D%E6%8F%90%E7%9F%A5%E8%AF%86%E7%82%B9.md#%E4%B8%80%E4%BA%9B%E5%89%8D%E6%8F%90%E7%9F%A5%E8%AF%86)
      - 自动提交、提交和回滚
      - 将 DML 操作与事务分组
      - 一致性非锁定读 - read view / snapshot
      - 锁定读
      - 幻影行
   3. [事务的状态](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.6.2.InnoDB%E4%BA%8B%E5%8A%A1%E2%80%94%E2%80%94%E4%BA%8B%E5%8A%A1%E7%9A%84%E7%8A%B6%E6%80%81.md#%E4%BA%8B%E5%8A%A1%E7%9A%84%E7%8A%B6%E6%80%81)
   4. [事务的隔离级别](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.6.3.InnoDB%E4%BA%8B%E5%8A%A1%E2%80%94%E2%80%94%E4%BA%8B%E5%8A%A1%E7%9A%84%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB.md#%E4%BA%8B%E5%8A%A1%E7%9A%84%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB)
   5. [事务的特性](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.6.4.InnoDB%E4%BA%8B%E5%8A%A1%E2%80%94%E2%80%94%E4%BA%8B%E5%8A%A1%E7%9A%84%E7%89%B9%E6%80%A7.md#%E4%BA%8B%E5%8A%A1%E7%9A%84%E7%89%B9%E6%80%A7)
   6. [事务实现的基础](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.6.5.InnoDB%E4%BA%8B%E5%8A%A1%E2%80%94%E2%80%94%E4%BA%8B%E5%8A%A1%E5%AE%9E%E7%8E%B0%E7%9A%84%E5%9F%BA%E7%A1%80.md#%E4%BA%8B%E5%8A%A1%E5%AE%9E%E7%8E%B0%E7%9A%84%E5%9F%BA%E7%A1%80)
      - redo log
      - undo log
   7. [事务的实现](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.6.6.InnoDB%E4%BA%8B%E5%8A%A1%E2%80%94%E2%80%94%E4%BA%8B%E5%8A%A1%E7%9A%84%E5%AE%9E%E7%8E%B0.md#%E4%BA%8B%E5%8A%A1%E7%9A%84%E5%AE%9E%E7%8E%B0)
   8. [多版本并发控制](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.6.7.InnoDB%E4%BA%8B%E5%8A%A1%E2%80%94%E2%80%94%E5%A4%9A%E7%89%88%E6%9C%AC%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6.md#%E5%A4%9A%E7%89%88%E6%9C%AC%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6---mvcc)
      - 简介
      - snapshot - 快照 与 read view
      - MVCC快照读案例
      - MVCC的快照读 与 当前读的例子
      - MVCC有没有解决幻读的问题
      - MVCC对聚簇索引和二级索引处理方式的区别
   9. 分布式事务
      - [XA](https://github.com/asdbex1078/MySQL/blob/master/mysql-storage-engines/innodb/1.6.8.0.InnoDB%E4%BA%8B%E5%8A%A1%E2%80%94%E2%80%94%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E4%B9%8BXA.md#%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)

11. #### 其他知识点

### MYISAM

### Memory

### TempTable

## 涉及MySQL其他

> 参考文献：
>
> 1. MySQL官方网站
>
> 2. 阿里数据库月报
>    
>    - [InnoDB undo log漫游](http://mysql.taobao.org/monthly/2015/04/01/)
>    - [MySQL · 引擎特性 · Innodb 表空间](http://mysql.taobao.org/monthly/2019/10/01/)
>    - [MySQL · 引擎特性 · Latch 持有分析](http://mysql.taobao.org/monthly/2020/03/07/)
>    - [MySQL · 引擎特性 · InnoDB 数据文件简述](http://mysql.taobao.org/monthly/2020/08/06/)
>    
>3. 阿里云开发者社区
> 
>   - [InnoDB的read view，回滚段和purge过程简介](https://developer.aliyun.com/article/560506)
> 
>4. 《高性能MySQL 第3版》
> 
>5. 《MySQL技术内幕 InnoDB存储引擎》
> 
>6. 简书博客
> 
>    - [MySQL中InnoDB引擎中页的概念](https://www.jianshu.com/p/e5e3f8a823c3)
>   
> 7. CSDN博客
>
>    - [InnoDB引擎--存储结构与文件](https://blog.csdn.net/john_lw/article/details/80306122)
>    - [详细分析MySQL事务日志(redo log和undo log)](https://www.cnblogs.com/f-ck-need-u/p/9010872.html)
>    - [InnoDB引擎--存储结构与文件](https://blog.csdn.net/john_lw/article/details/80306122)
>    - [MySQL学习总结](https://blog.csdn.net/howinfun/category_9704174.html)
>    - [InnoDB锁分析](https://www.cnblogs.com/crazylqy/p/7611069.html)
>
> 8. 知乎
>
>    - [redo log检查点](https://zhuanlan.zhihu.com/p/148414574)
>    - [深入理解InnoDB](https://zhuanlan.zhihu.com/p/161737133)
>
> 9. 其他博客
>
>    - [详解redo log 和 undo log](https://www.cnblogs.com/f-ck-need-u/p/9010872.html)
>    - [MySQL中一条更新语句是如何执行的](https://www.cnblogs.com/wangchunli-blogs/p/10393139.html)
>    - [InnoDB存储引擎关键特性](https://www.it610.com/article/5037373.htm)
>





> 本文一些知识点未深入研究，链接摘录如下，感兴趣可以继续研究。
>
> 延申阅读：
>
> 1. [MySQL · 内核分析 · InnoDB mutex 实现分析](http://mysql.taobao.org/monthly/2020/03/05/)
> 2. [MySQL · 内核特性 · InnoDB btree latch 优化历程](http://mysql.taobao.org/monthly/2020/06/02/)
> 3. [MySQL · 引擎特性 · InnoDB UNDO LOG写入 - 源码分析](http://mysql.taobao.org/monthly/2020/08/05/)