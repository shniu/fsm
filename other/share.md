# 分享

先想一个 topic：聊聊写代码这件事（我不要再写烂代码）

开篇词：自己一直对技术有一个方面的追求，那就是希望自己写出的是优秀的高质量的代码，所以在编码时也会格外注意；经过几十年的发展，在计算机工程领域不乏有很多大牛，在设计和实现的阶段，总结出了非常多的优秀的原则和模式，但是由于自己的理解和领悟着实有限，并不对这些优秀的设计原则、设计模式等做很细致的罗列，仅仅分享一些自己的看法和实践。

分享我对写代码这件事情的看法和实践，我自己也在不断的学习中。

好的代码是重构出来的，好的设计是需要有经验积累的。写代码是作为工程师的最本职的工作，写出优秀的、易维护的、易扩展的代码应该是每个工程师的追求。



---

另一个 Topic：聊聊 MySQL 的使用和原理

后端存储是后端服务开发中最重要、最容易出错且最容易产生性能瓶颈的地方，所以对使用的存储技术做深入理解是非常必要的，我们需要知道我们写的一个语句，在不同场景下是怎么执行的、数据是怎么流转的、不同量级的数据下会不会有非常大的性能差距；而除了性能的问题，数据规模增大后的扩展性和可用性也是非常重要的问题。

在当下，MySQL 都是大部分公司构建后端服务的首选存储方案，所以就聊聊 MySQL 的使用和原理。

* 介绍下MySQL 在大厂的处理数据的能力，来证明 MySQL 其实很强
* 总结一些结论性的东西，也是作为最佳实践的check list
* 用实际案例切入，来一步步探索 MySQL 的索引、锁、事务等，再更进一步分析 MySQL 的核心 Server 和 存储引擎 \(InnoDB\)，重点是要讲的明白，容易理解
* 讲一下MySQL 的两阶段协议（如日志的两阶段提交，锁的两阶段锁），引申一下两阶段思想在实际开发中的应用，拿最近在项目中的例子，例子1：充币交易的监听与同步，画图来说明如何使用两阶段的思路来同步数据；例子2 提币请求发送时引入 Pending 状态，画图来说明；再找一些其他的场景
* 分析一下 MySQL 的主从同步原理，并介绍主从复制的实际应用，如 缓存更新 / binlog 订阅做数据分发（CDC 数据变更模式 / 事务发件箱模式 / 双写 MySQL 和 MQ 的问题） / 分析一下 Canal 的实现原理和架构设计

不同隔离级别，以及解决的不同问题（脏读、不可重复度、幻读）；重点介绍 读提交和可重复读隔离级别，参考：[https://tech.meituan.com/2014/08/20/innodb-lock.html](https://tech.meituan.com/2014/08/20/innodb-lock.html)

索引：Innodb在内存中如何组织数据，Innodb在磁盘如何组织数据，聚簇索引和非聚簇索引，回表的过程，最左前缀原则，索引下推，索引覆盖，索引使用的注意事项；参考：[https://tech.meituan.com/2014/06/30/mysql-index.html](https://tech.meituan.com/2014/06/30/mysql-index.html)

在系统设计时，面对普通用户的索引选择和面对管理后台的索引选择有时候是冲突的，如果混在一起，索引建立的过多，也会影响主站的性能，因为建立索引的过程需要花费时间，一般做法是使用数据分发的方式，根据不同的聚合需求建立不同的索引机制，如主从复制做数据同步，使用一个slave库来解决；也可以使用es来解决

锁机制，对于热点数据的更新，如何提高并发能力？举例：平台账户的 trader 和 普通用户之间，就会形成热点账户，很容易造成锁等待，降低了系统整体的并发能力，可以将平台账户拆成很多个子账户，来增加并发度；MySQL 的锁可以理解为有两类：隐式锁\(一致性非锁定读\)和显式锁\(一致性锁定读\)

幻读问题是如何被解决的？

解决更新丢失的方式：使用一致性锁定读的方式和使用乐观锁的方式

![](../.gitbook/assets/image%20%2828%29.png)

```text
///// 一致性锁定读
begin;
select id, name, balance from account where id = 2 for update;
// 应用代码：balance += 10;  balance = 110
update account set balance = 110 where id = 2;
commit;

///// 乐观锁
begin;
select id, name, balance, version from account where id = 2;

// 假设 balance = 100, version = 1
// 应用代码：balance += 10;  balance = 110
update account set balance = 110, version = version + 1 where id = 2 and version = 1;
commit;

///// 利用 MySQL 的一致性非锁定读
begin;
select id, balance from account where id = 2;
update account set balance = balance + 10 where id = 2;
commit;
```

一些结论：

1. 


