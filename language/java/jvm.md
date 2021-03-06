---
description: About JVM
---

# JVM

* [ ] 类加载机制
* [ ] JVM 整体结构解析
* [ ] JVM 内存分配机制
* [ ] JVM 字节码文件分析
* [ ] GC 算法和 GC 收集器
* [ ] GC 调优实战
* [ ] 对象引用类型以及应用场景：[https://www.baeldung.com/java-weak-reference](https://www.baeldung.com/java-weak-reference)
* [ ] JVM 面试
  * [ ] [https://www.cnblogs.com/crazymakercircle/p/14365820.html](https://www.cnblogs.com/crazymakercircle/p/14365820.html)
  * [ ] [https://gitbook.cn/books/5d2d822f175a450263e945f9/index.html](https://gitbook.cn/books/5d2d822f175a450263e945f9/index.html)
  * [ ] [https://www.jianshu.com/p/9d729c9c94c4](https://www.jianshu.com/p/9d729c9c94c4)

### 博客

#### [Java GC 精粹](https://mechanical-sympathy.blogspot.com/2013/07/java-garbage-collection-distilled.html)

JVM 的 GC 概念多，控制 JVM 行为的参数也多，怎么在特定场景下权衡 JVM 的参数，让 GC 更好的工作是一项重要且难度很大的工作。使用如下几个指标来衡量垃圾收集器：

1. 吞吐量，一个应用程序完成的工作量与在 GC 中花费的时间之比；用 `-XX:GCTimeRatio=99` 作为目标吞吐量；99 是默认的，相当于 1% 的GC时间
2. 延迟，系统响应事件所花费的时间，它受到垃圾收集引入的暂停的影响。用 `-XX:MaxGCPauseMillis=<n>` 作为 GC  暂停的目标延迟
3. 内存，我们的系统用来存储状态的内存量，这些状态在被管理时经常被复制和移动。应用程序在任何时间点保留的活动对象集被称为 Live Set。最大堆大小 `-Xmx` 是一个调整参数，用于设置应用程序可用的堆大小。

延迟是事件间的分布, 增加平均延迟以减少最坏情况下的延迟，或者使其不那么频繁，这可能是可以接受的。我们不应该将 "实时 "一词解释为意味着尽可能低的延迟；相反，**实时是指无论吞吐量如何，都要有确定性的延迟**。

对于某些应用工作负载，吞吐量是最重要的目标。一个例子是一个长期运行的批处理工作；只要整个工作能更快完成，即使在垃圾收集时偶尔暂停几秒钟，也没有关系.

对于几乎所有其他的工作负载，从面向人类的交互式应用到金融交易系统，如果系统无响应的时间超过几秒钟，甚至在某些情况下超过几毫秒，就会带来灾难。在金融交易中，用一些吞吐量来换取稳定的延迟往往是值得的。我们可能还会有一些应用程序，这些应用程序受到可用物理内存量的限制，并且必须保持占用空间，在这种情况下，我们必须放弃延迟和吞吐量方面的性能.

一般权衡的结果是：

1.  通过为垃圾回收算法提供更多的内存，可以在很大程度上减少作为摊销成本的垃圾回收成本。
2. 通过包含活动集并保持堆大小较小，可以减少由于垃圾收集而导致的观察到的最坏情况的延迟引发的暂停。
3. 通过管理堆和世代大小以及控制应用程序的对象分配速率，可以减少出现暂停的频率。
4. 可以通过同时运行GC和应用程序来减少大停顿的频率，有时会牺牲吞吐量。

### [JVM 问题诊断快速入门](https://gitbook.cn/books/5d2d822f175a450263e945f9/index.html) - GitChat

### 

