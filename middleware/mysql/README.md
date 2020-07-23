# MySQL

### 为什么会出现数据库？

也就是说数据库用来解决什么问题？针对任何领域，所面对的数据都在日益增多，怎么解决，唯一的办法就是做数据管理，平衡数据的增长和我们对数据本身的需求（如检索需求、更新需求等）。如何管理数据是一个比较难的问题

一些闲言碎语：

日常业务开发中会使用到数据库，作为一个可靠的持久化存储，在做存储架构的设计时，考虑的最多的是：怎么合理使用索引；怎么合理使用事务；会不会遇到锁的性能问题（比如会触发什么锁，表锁/行的读锁/行的写锁/间隙锁，会不会引入死锁，怎么提升并发更新的性能等）；应对什么样的并发量；应对什么样的数据量。后面两个问题更多的属于数据库架构层面的，需要综合业务发展的阶段和应对的业务量来判断

微观层面：弄清楚单个 MySQL 实例是如何管理数据、处理数据的，数据库启动后，可以接收库和表的创建、然后接受数据更新和数据检索，这个过程是怎么完成的？（引入了很多的设计方面的考虑）

宏观层面：数据持久层怎么应对高并发和海量数据存储，这是数据库架构层面的考量，可以使用读写分离的数据库架构，也可以使用分库分表的数据库架构，还可以使用缓存+数据库的架构，这个时候数据不再是单机处理了，就引入了分布式技术（也意味着引入了分布式的优点和缺点）



### MySQL 的基础架构与日志系统

MySQL 的事务隔离级别如何实现的？

### MySQL 资源列表

* [MySQL 技术内幕：InnoDB 存储引擎](https://weread.qq.com/web/reader/611329b059346e611427f1ckc81322c012c81e728d9d180)
* 数据库索引设计与优化 （网盘和本地）
* 高性能 MySQL
* [MySQL 官方手册](https://dev.mysql.com/doc/refman/5.7/en/)
* [How to analyze and tune MySQL query for better performance](https://www.mysql.com/cn/why-mysql/presentations/tune-mysql-queries-performance/)
* MySQL Internals Manual
* 博客
  * [https://www.cnblogs.com/wxw16/p/6105624.html](https://www.cnblogs.com/wxw16/p/6105624.html) in 查询为什么慢
  * [https://juejin.im/post/5c2c53396fb9a04a053fc7fe](https://juejin.im/post/5c2c53396fb9a04a053fc7fe)
  * [https://tech.meituan.com/2014/06/30/mysql-index.html](https://tech.meituan.com/2014/06/30/mysql-index.html) 重点看


