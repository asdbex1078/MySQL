# MySQL

## MySQL优化

## MySQL存储引擎

### InnoDB

1.  #### 从MySQL架构到InnoDB架构

   1. [MySQL架构]()

   2. [InnoDB架构]()

   3. [InnoDB的多线程模型]()
- [Master Thread]()
      
- [IO Thread]()
      
- [Purge Thread]()
      
- [Page Cleaner Thread]()
      
4. InnoDB简介
   
   - [使用InnoDB的好处]()
   
   - [如何使InnoDB性能更高]()
   
   - [查看当前库支持的存储引擎，以及默认的存储引擎]()
   
5. InnoDB关键特性
   
   - [Insert Buffer——插入缓冲区]()
   
   - [Change Buffer——变更缓冲区]()
   
   - [Double Write Buffer——双写缓冲区]()
   
   - [Adaptive Hash Index——自适应哈希索引]()
   
   - [AIO——异步IO]()
   
   - [刷新临近页]()
   
   - 预读
   
2.  #### InnoDB内存结构
   
   1. 缓冲池 - Buffer Pool
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
   2. 日志缓冲区 - Log Buffer
      - redo log buffer
      - undo log buffer
   3. 变更缓冲区 - Change Buffer 
      - 简介
      - 什么时候合并到数据页
      - 在架构中的位置
      - 作用
      - 组成部分
      - 参数配置
      - 监控变更缓冲区
      - 其他信息
   4. 额外内存池
   
3. #### InnoDB磁盘结构

   1. 页逻辑存储结构
      - 页的结构
        - 页中插入数据的过程
        - COMPACT行格式
        - Page Directory
        - Page Header
        - File Header
        - File Trailer
      - 总结
   2. 表 - table
   3. 索引 - index
      - 聚簇索引和二级索引
      - InnoDB索引的物理结构
      - 排序索引的创建
   4. 表空间 - TableSpaces
      - 系统表空间
      - 每个表的表空间
      - InnoDB内存中对 .ibd 文件的管理
      - 数据字典和 .idb 文件的关系
      - 通用表空间
      - 创建通用表空间
      - 撤销表空间
      - 临时表空间
   5. 数据字典 - Data Dictionary

4.  #### MySQL文件

   1. 参数文件
      - 作用
      - 参数文件类型
      - 参数类型
      - 参数文件加载顺序
   2. 日志文件
      - 错误日志
      - 慢查询日志
        - 作用
        - 查看慢日志是否开启 / 慢日志位置
        - 满足什么条件会记录到慢日志中
        - 测试慢日志记录
        - mysqldumpslow的高级用法
        - 慢查询日志的输出方式
      - 二进制日志
        - 简介
        - 参数控制
        - 二进制日志的切换
        - 删除二进制日志文件
        - 相关参数
        - 查看二进制日志内容
      - 查询日志
   3. socket文件
   4. pid文件
   5. MYISAM引擎文件
   6. InnoDB常规表文件
   7. InnoDB Redo Log
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
   8. InnoDB Undo Log
      - 了解几个概念
      - 简介
      - 作用
      - 存储方式
      - 关键结构体
      - purge线程
      - group commit
      - 存储位置
   9. LSN
      - 什么是 LSN 
      - 根据 LSN 能获得什么信息
      - LSN 存在的位置
      - 查看 LSN 信息
      - 举例
   10. bin log 和 redo log 的区别
   11. bin log、redo log、undo log写入顺序 - 案例

5.  #### InnoDB锁

   1. latch - 线程锁
      - 简介
      - 作用
      - 举例
      - 如何查看
      - 分类
      - latch争用发生的原因
      - 如何降低latch争用
      - InnoDB Buffer Pool并发控制加锁过程
   2. lock介绍
      - 简介与latch区别
      - 加锁位置
      - 加锁原则
   3. latch 和 lock 的区别
   4. lock分类
      - 表级锁
        - 意向锁
        - 自增自动上锁
      - 行级锁
        - 共享读锁
        - 排他写锁
        - 记录锁
        - 间隙锁
        - next-key锁
        - 插入意向锁
        - 空间所以你谓词锁
      - 页级锁
   5. lock加锁案例分析
   6. 不同SQL加的不同lock锁
   7. 死锁
      - 死锁简单案例
      - 死锁本质
      - InnoDB死锁实例
      - 死锁必要条件
      - InnoDB死锁检测
      - 死锁检测机制 - wait-for graph
      - 禁用死锁检测
      - 如何避免死锁

6.  #### InnoDB事务

   1. 事务概念及分类
      - 概念
      - 分类
        - 扁平事务
        - 带有保存点的扁平事务
        - 链事务
        - 嵌套事务
        - 分布式事务
   2. 一些前提知识点
      - 自动提交、提交和回滚
      - 将 DML 操作与事务分组
      - 一致性非锁定读 - read view / snapshot
      - 锁定读
      - 幻影行
   3. 事务的状态
   4. 事务的隔离级别
   5. 事务的特性
   6. 事务实现的基础
      - redo log
      - undo log
   7. 事务的实现
   8. 多版本并发控制
      - 简介
      - snapshot - 快照 与 read view
      - MVCC快照读案例
      - MVCC的快照读 与 当前读的例子
      - MVCC有没有解决幻读的问题
      - MVCC对聚簇索引和二级索引处理方式的区别
   9. 分布式事务
      - XA

7.  #### 其他知识点

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