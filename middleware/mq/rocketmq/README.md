# RocketMQ

### 纲要

* MQ 通用性问题：MQ 使用场景，怎么保证消息不丢失，怎么处理重复消息问题，消息积压了怎么办，怎么选择消息队列产品并做个比较，怎么保证消息的顺序性
* MQ 的使用经验问题：在项目中使用 MQ 具体解决什么问题？使用过程中有没有遇到过什么坑？结合你的应用场景来分析一下引入 MQ 后提升了哪些地方？
  * 用 MQ 怎么解决分布式事务问题？
  * 怎么解决双写问题？写 DB 和写 MQ
* MQ 的技术细节问题
  * RocketMQ 的整体介绍、技术架构、部署架构等
  * Producer 方面的问题
  * Broker 方面的问题
    * 消息存储设计和实现
    * 刷盘方式
    * HA 设计和实现，同步方式
    * 消息转发设计和实现
    * 延迟消息设计和实现
    * 事务消息设计和实现
  * Consumer 方面的问题
  * Namesrv 方面的问题
  * HA 怎么实现的？高吞吐和高并发怎么实现的？低延迟是怎么实现的？
  * 从而引入更底层的问题
    * OS 方面的问题，如内存管理，mmap，堆外内存和零拷贝，磁盘顺序写和随机写，文件系统和 IO 调度算法等
    * 网络方面的问题，如 Netty，线程模型，高性能 IO 模型，tcp/ip 协议，tls/https 等
    * JVM 方面的问题，Namesrv 和 Broker 用了什么 GC 算法，有没有对 Broker 做 GC 调优等
    * DLedger 方面的问题，如 raft 协议，其他的分布式一致性算法等
  * 调优
  * 基准测试和性能测试
    * [https://blog.csdn.net/zshake/article/details/64594139](https://blog.csdn.net/zshake/article/details/64594139)
    * [https://cloud.tencent.com/developer/article/1456397](https://cloud.tencent.com/developer/article/1456397)
* MQ 运维，云原生方向

### RocketMQ 的设计原则

RocketMQ 的设计追求简单与性能第一，主要体现为：

* 因为 Topic 路由信息能容忍分钟级的不一致。所有 RocketMQ 的 NameServer 集群之间互不通信，降低了 NameServer 的复杂程度以及对网络的要求，同时性能比 zookeeper 有了极大的提升
* 高效的 IO 存储机制，消息存储文件设计成文件组的概念，组内单个文件大小固定，方便引入内存映射机制，所有 topic 的消息存储基于顺序写，极大提升了消息的写性能，同时为了兼顾消息消费与消息查找，引入了消息消费队列文件与索引文件
* 幂等性问题：RocketMQ 不处理，留给用户处理

### MQ 通用性问题 

#### RocketMQ 的消息生产和消息消费的顺序性问题

* 对于全局有序，所有消息需要进入同一个 Broker 的同一个 Topic 下的同一个 Queue，也就是单 Broker 单 Topic 单 Queue 的方式，这样就没有办法做到高可用，而且性能和吞吐量是有上限的；但是很少有场景是需要这样的
* 局部有序，这个是大部分场景可能会遇到的；一般的做法是在 Producer 端投递消息时把需要保证顺序的消息投递到同一个 Queue 中；在消费时由一个消费者顺序进行消费；但是值得注意的是：**在严格局部顺序的场景下，要保证 Broker 的可用性，如果需要投递到某个 Broker 的 Queue 上，在超时或者服务无法访问时需要等待重发，这个时候可能需要牺牲掉一定的可用性**
* 并发无序，有些时候我们并不在意消息的顺序性，这个就比较灵活了

### RocketMQ 基本使用

#### 理解 RocketMQ 的基本概念和特性

1. [MQ 中涉及到的概念看这里](https://github.com/apache/rocketmq/blob/master/docs/cn/concept.md)
2. [RocketMQ 的功能特性看这里](https://github.com/apache/rocketmq/blob/master/docs/cn/features.md)

### RocketMQ 的架构设计

#### 技术架构

![RocketMQ &#x6280;&#x672F;&#x67B6;&#x6784;](../../../.gitbook/assets/image%20%2877%29.png)

主要由 4 个部分组成：

* Producer：消息发布的角色，支持分布式集群方式部署。Producer通过MQ的负载均衡模块选择相应的Broker集群队列进行消息投递，投递的过程支持快速失败并且低延迟。（Producer 如何实现的，以及如何做的快速失败和低延迟？）
* Consumer：消息消费的角色，支持分布式集群方式部署。支持以push推，pull拉两种模式对消息进行消费。同时也支持集群方式和广播方式的消费，它提供实时消息订阅机制，可以满足大多数用户的需求。
* NameServer：NameServer是一个非常简单的Topic路由注册中心，其角色类似Dubbo中的zookeeper，支持Broker的动态注册与发现。主要包括两个功能：Broker管理，NameServer接受Broker集群的注册信息并且保存下来作为路由信息的基本数据。然后提供心跳检测机制，检查Broker是否还存活；路由信息管理，每个NameServer将保存关于Broker集群的整个路由信息和用于客户端查询的队列信息。然后Producer和Conumser通过NameServer就可以知道整个Broker集群的路由信息，从而进行消息的投递和消费。NameServer通常也是集群的方式部署，各实例间相互不进行信息通讯。Broker是向每一台NameServer注册自己的路由信息，所以每一个NameServer实例上面都保存一份完整的路由信息。当某个NameServer因某种原因下线了，Broker仍然可以向其它NameServer同步其路由信息，Producer,Consumer仍然可以动态感知Broker的路由的信息。
* BrokerServer：Broker主要负责消息的存储、投递和查询以及服务高可用保证，为了实现这些功能，Broker包含了以下几个重要子模块。
  1. Remoting Module：整个Broker的实体，负责处理来自clients端的请求。
  2. Client Manager：负责管理客户端\(Producer/Consumer\)和维护Consumer的Topic订阅信息
  3. Store Service：提供方便简单的API接口处理消息存储到物理硬盘和查询功能。
  4. HA Service：高可用服务，提供Master Broker 和 Slave Broker之间的数据同步功能。
  5. Index Service：根据特定的Message key对投递到Broker的消息进行索引服务，以提供消息的快速查询。

![Broker &#x67B6;&#x6784;](../../../.gitbook/assets/image%20%2879%29.png)

#### 部署架构

![RocketMQ &#x90E8;&#x7F72;&#x67B6;&#x6784;](../../../.gitbook/assets/image%20%2878%29.png)

* NameServer是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。
* Broker部署相对复杂，Broker分为Master与Slave，一个Master可以对应多个Slave，但是一个Slave只能对应一个Master，Master与Slave 的对应关系通过指定相同的BrokerName，不同的BrokerId 来定义，BrokerId为0表示Master，非0表示Slave。Master也可以部署多个。每个Broker与NameServer集群中的所有节点建立长连接，定时注册Topic信息到所有NameServer。 注意：当前RocketMQ版本在部署架构上支持一Master多Slave，但只有BrokerId=1的从服务器才会参与消息的读负载 （？什么意思）。
* Producer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer获取Topic路由信息，并向提供Topic 服务的Master建立长连接，且定时向Master发送心跳。Producer完全无状态，可集群部署。
* Consumer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer获取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向Master、Slave发送心跳。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，消费者在向Master拉取消息时，Master服务器会根据拉取偏移量与最大偏移量的距离（判断是否读老消息，产生读I/O），以及从服务器是否可读等因素建议下一次是从Master还是Slave拉取。

结合部署架构图，描述集群工作流程：

* 启动NameServer，NameServer起来后监听端口，等待Broker、Producer、Consumer连上来，相当于一个路由控制中心。
* Broker启动，跟所有的NameServer保持长连接，定时发送心跳包。心跳包中包含当前Broker信息\(IP+端口等\)以及存储所有Topic信息。注册成功后，NameServer集群中就有Topic跟Broker的映射关系。
* 收发消息前，先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，也可以在发送消息时自动创建Topic。
* Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic存在哪些Broker上，轮询从队列列表中选择一个队列，然后与队列所在的Broker建立长连接从而向Broker发消息。
* Consumer跟Producer类似，跟其中一台NameServer建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，开始消费消息。

via: [https://github.com/apache/rocketmq/blob/master/docs/cn/architecture.md](https://github.com/apache/rocketmq/blob/master/docs/cn/architecture.md)

### RocketMQ 的功能设计

#### 消息存储设计

主要从存储整体架构、PageCache与Mmap内存映射以及RocketMQ中两种不同的刷盘方式来介绍。

**存储整体架构**

* CommitLog: 消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容,消息内容不是定长的。单个文件默认是 1G，文件名长度为20位，左边补零，剩余为起始偏移量，比如00000000000000000000代表了第一个文件，起始偏移量为0，文件大小为1G=1073741824；当第一个文件写满了，第二个文件为00000000001073741824，起始偏移量为1073741824，以此类推。消息主要是顺序写入日志文件，当文件满了，写入下一个文件；（**这样命名文件的好处是可以快速定位某个偏移量的消息在哪个文件中存储，比如可以使用二分查找等**）
* ConsumeQueue: 消息消费队列，引入的目的主要是提高消息消费的性能，由于RocketMQ是基于主题topic的订阅模式，消息消费是针对主题进行的，如果要遍历commitlog文件中根据topic检索消息是非常低效的。Consumer即可根据ConsumeQueue来查找待消费的消息。其中，ConsumeQueue（逻辑消费队列）作为消费消息的索引，保存了指定Topic下的队列消息在CommitLog中的起始物理偏移量offset，消息大小size和消息Tag的HashCode值。consumequeue文件可以看成是基于topic的commitlog索引文件，故consumequeue文件夹的组织方式如下：topic/queue/file三层组织结构，具体存储路径为：$HOME/store/consumequeue/{topic}/{queueId}/{fileName}。同样consumequeue文件采取定长设计，每一个条目共20个字节，分别为8字节的commitlog物理偏移量、4字节的消息长度、8字节tag hashcode，单个文件由30W个条目组成，可以像数组一样随机访问每一个条目，每个ConsumeQueue文件大小约5.72M；
* IndexFile: IndexFile（索引文件）提供了一种可以通过key或时间区间来查询消息的方法。Index文件的存储位置是：$HOME \store\index${fileName}，文件名fileName是以创建时的时间戳命名的，固定的单个IndexFile文件大小约为400M，一个IndexFile可以保存 2000W个索引，IndexFile的底层存储设计为在文件系统中实现HashMap结构，故rocketmq的索引文件其底层实现为hash索引。

存储模块的设计还可以参考 [http://zjykzk.github.io/post/cs/rocketmq/store/](http://zjykzk.github.io/post/cs/rocketmq/store/)

#### 通信机制

* 协议设计
* 通信方式和流程
* 网络模型：Reactor 多线程设计

via: [https://github.com/apache/rocketmq/blob/master/docs/cn/design.md](https://github.com/apache/rocketmq/blob/master/docs/cn/design.md) （todo\)

### RocketMQ 源码实现

#### 存储模块实现 \(rocketmq/store 模块\)

存储模块的实现依赖于 `io.openmessaging.storage:dledger` , 有关 DLedger 的解读看：

1. [阿里数据一致性实践：DLedger 技术在消息领域的应用](https://www.infoq.cn/article/f6y4QRiDitBN6uRKp*fq) 
2. [DLedger - 基于 raft 协议的 CommitLog 存储库](https://juejin.im/post/6844903913045360654)
3. [https://yq.aliyun.com/articles/718344](https://yq.aliyun.com/articles/718344)
4. 代码仓库: [https://github.com/openmessaging/openmessaging-storage-dledger](https://github.com/openmessaging/openmessaging-storage-dledger)

#### 基础模块 - remoting

remoting 对服务端和客户端通信做了抽象，提供了 `RemotingClient` 和 `RemotingServer` 两个接口，RocketMQ 的网络相关的功能都由该模块承担。

```java
// 通信的基础
public interface RemotingService {
    void start();

    void shutdown();

    void registerRPCHook(RPCHook rpcHook);
}

// Client 的抽象
public interface RemotingClient extends RemotingService {

    void updateNameServerAddressList(final List<String> addrs);

    List<String> getNameServerAddressList();

    RemotingCommand invokeSync(final String addr, final RemotingCommand request,
        final long timeoutMillis) throws InterruptedException, RemotingConnectException,
        RemotingSendRequestException, RemotingTimeoutException;

    void invokeAsync(final String addr, final RemotingCommand request, final long timeoutMillis,
        final InvokeCallback invokeCallback) throws InterruptedException, RemotingConnectException,
        RemotingTooMuchRequestException, RemotingTimeoutException, RemotingSendRequestException;

    void invokeOneway(final String addr, final RemotingCommand request, final long timeoutMillis)
        throws InterruptedException, RemotingConnectException, RemotingTooMuchRequestException,
        RemotingTimeoutException, RemotingSendRequestException;

    void registerProcessor(final int requestCode, final NettyRequestProcessor processor,
        final ExecutorService executor);

    void setCallbackExecutor(final ExecutorService callbackExecutor);

    ExecutorService getCallbackExecutor();

    boolean isChannelWritable(final String addr);
}

// Server 端的抽象
public interface RemotingServer extends RemotingService {

    void registerProcessor(final int requestCode, final NettyRequestProcessor processor,
        final ExecutorService executor);

    void registerDefaultProcessor(final NettyRequestProcessor processor, final ExecutorService executor);

    int localListenPort();

    Pair<NettyRequestProcessor, ExecutorService> getProcessorPair(final int requestCode);

    RemotingCommand invokeSync(final Channel channel, final RemotingCommand request,
        final long timeoutMillis) throws InterruptedException, RemotingSendRequestException,
        RemotingTimeoutException;

    void invokeAsync(final Channel channel, final RemotingCommand request, final long timeoutMillis,
        final InvokeCallback invokeCallback) throws InterruptedException,
        RemotingTooMuchRequestException, RemotingTimeoutException, RemotingSendRequestException;

    void invokeOneway(final Channel channel, final RemotingCommand request, final long timeoutMillis)
        throws InterruptedException, RemotingTooMuchRequestException, RemotingTimeoutException,
        RemotingSendRequestException;

}
```



#### Client - Producer 实现

Note: 以设计和实现 Producer 的角度去总结。

在 RocketMQ 的设计和实现中有很多共用的东西，比如 remoting 就是一个共用模块，其中实现了通信双方的 client 和 server，也实现了基本的通信协议（编解码、序列化反序列化、命令等），解决了网络通信问题。

Producer 是作为客户端存在去主动连接 Namesrv 和 Broker，建立一个全双工的  TCP 长连接。

Producer 实现

RocketMQ 在实现时使用了门面设计模式，为用户使用提供了简单易操作的接口，屏蔽了 Producer 内部的复杂实现，但是在实际调用某些功能时委托给内部的实现进行处理。



* Producer 里的 MessageQueue List 是如何构建的？因为在发送消息时要选择一个 MessageQueue

Producer 在 send message 时，会先获取到 topicPublishInfo，这里边包含了两个重要的数据：topicRouteData 和 MessageQueueList，其中 topicRouteData 中包括了 topicData 的列表和 brokerData 的列表，而 MessageQueueList 的数据就是从 topicData 列表数据中转化而来的，如果一个 Topic 分布在两个 Broker 之上，而且读写队列是 4 的话，那么就会有 8 个 MessageQueue，分别对应到 broker1 的 0 1 2 3 这 4 个队列和 broker2 的 0 1 2 3 这 4 个队列。所以说，如果某个 Broker 挂掉，那么这个 Broker 上的队列都不能访问了，在我们举的例子中，就只剩下 4 个 Queue，在一个短暂的时间内，本来要发给某个 Queue 的数据需要发给另外的 Queue，即使我们使用了 id 取余的方式，也不能严格的保证局部顺序；当 Broker 端做了高可用的 DLedger 集群后，能缩短故障时间，并增加了自动恢复的能力，要么我们就需要牺牲一定的可用性，如果某个 id 投递到某个 Queue，把这个对应关系记下来，如果这个 Queue 不可用，就后面再发送，这无疑增加了系统实现的复杂度，而且还会影响吞吐量

* 如果某个 Topic 分布在 3 个 Broker 上，Producer 从 Namesrv 中拉取到 3 个 Broker 的信息，这个时候发送消息默认情况下是通过 RoundRobin 的方式做的，但是如果某个 Broker 挂掉了，原本发往这个 Broker MessageQueue 上的消息可能发送到了其他的 Broker 上，如果是要保证局部顺序的场景下，会不会出现乱序的问题呢？怎么解决？这个问题还要看 Consumer 是如何消费的
* Producer 发送消息的主要方式是同步发送、OneWay 发送、类似于 RPC 的请求-响应、异步发送+回调、事务消息发送、批量消息，各有什么优缺点以及适用场景

```text
org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl
 > defaultAsyncSenderExecutor: 创建了一个队列大小为 50000 的线程池，线程数量为cpu核数
 > asyncSenderExecutor: 还可以自定义线程池
 
 // 默认的异步发送流程
 1. getAsyncSenderExecutor() 获取到异步发送线程池
 2. 提交任务到线程池，用 Runnable 包裹，注册 callback，判断超时时间等
 3. 线程池调度到发送任务后，调用 sendDefaultImpl()
 4. 消息发送的核心逻辑
   - 4.1 判断服务状态是否 RUNNING 等
   - 4.2 根据 topic 找路由信息
   - 4.3 org.apache.rocketmq.client.latency.MQFaultStrategy#selectOneMessageQueue 选择一个 MessageQueue
     a. 默认策略是随机选择一个 MessageQueue，对每个线程维护一个index，每次取的时候+1，最终的效果有点像轮询
     b. 容错退避策略
```

消息发送的负载均衡

生产者在发送消息时，会首先查找 `TopicPublishInfo` ，如果本地找不到就去 NameSrv 去请求，最后还是没有找到就会异常退出；`TopicPublishInfo` 包括了 MessageQueue 的列表、是否顺序消息、本地线程的索引计数器、Topic 路由数据等，可以从 `TopicPublishInfo` 来选择一个 MessageQueue，具体选择的策略有两种：一种是如果没有 enable latencyFaultTolerance，就用递增取模的方式选择。一种是如果 enable 了，在递增取模的基础上，再过滤掉 not available 的。这里所谓的 latencyFaultTolerance, 是指对之前失败的，按一定的时间做退避，如果上次请求的 latency 超过 550L ms, 就退避 3000L ms；超过 1000L，就退避 60000L。参考 `org.apache.rocketmq.client.latency.MQFaultStrategy`

#### Namesrv 设计与实现

Namesrv 作为 RocketMQ 的核心组件之一，承担了路由注册中心的作用。而 Namesrv 的设计和实现追求简单和高效，多个 Namesrv 之间不需要同步状态，也不需要有任何的通信，一般情况下为了可用性的考虑都会在生产环境中部署多个 Namesrv（3 ～5个），Producer 和 Counsmer 会从其中一个 Namesrv 上拉取路由信息等，Broker 会把自己注册到所有的 Namesrv 上；Namesrv 的整体结构如下：

![Namesrv &#x7684;&#x7ED3;&#x6784;&#x56FE;](../../../.gitbook/assets/image%20%28136%29.png)

从上图看，可以把 Namesrv 分为两层，一层是基础的网络通信层，这一层包括对 Netty 的封装使用、命令协议的实现、长连接的维护管理、心跳的实现；再上一层是 Namesrv 的路由信息管理，这也是 Namesrv 的主要业务逻辑，它主要实现在 `RouteInfoManager` 中；此外，在启动 Namesrv 时会把 NettyServer 启动起来，并初始化 RouteInfo 相关的信息，根据配置初始化一些后台线程池等

回答下面几个问题：

1. 有哪些信息注册到了 Namesrv ?  **主要有 Topic 数据，Broker 数据、Cluster 信息、Broker Live 数据等，详细的看下面的路由信息**
2. Namesrv 一般是以集群部署，且集群中的各个节点不互通，那么 Namesrv 集群中各节点路由信息不一致时，RocketMQ 如何保证可用性？**路由信息不一致时交给 Producer 或者 Customer 自己去解决，简化了 Namesrv 的实现难度，比如 Broker 挂掉了，Client 没有及时获取到这个信息，做了请求，Client 会采取重试和无效 Broker 规避的方式，而在 Customer 端需要使用幂等处理来消除掉可能的重复消息**
3. Broker 不可用后，Namesrv 并不会立即将变更后的注册信息推送至 Producer or Consumer，那 RocketMQ 是如何保证 Producer 和 Consumer 正常发送/消费消息的呢 ？**采用重试的策略，在 Producer 端遇到无效的 Broker 会暂时规避掉**

梳理的流程图等在 [Google  Drive](https://app.diagrams.net/#G1KnciHJxwyYAvI9d4CFO8EKCx3XKxxGLd) 上。

* [谈谈 RocketMQ NameServer 的设计与实现](http://tinylcy.me/2019/rocketmq-nameserver/) - 本文结合源码分析了 Namesrv 在设计上的权衡，追求简单高效、复杂度低、高性能的实现，而由于网络分区等问题引起的数据一致性问题交给了 Producer、Broker、Consumer 去解决；Namesrv 并不会把路由信息的变化主动推送给客户端，降低了技术实现复杂度，要靠客户端拉取来感知变化，当 broker 不可用时，使用重试和无效 Broker 规避的方式解决。

```java
// Namesrv 上维护的路由信息
// Topic 数据
topicQueueTable: {
    "topic1": [
        {
          "brokerName": "broker-a",
          "readQueueNums": 4,
          "writeQueueNums": 4,
          "perm": 6,
          "topicSysFlag": 0
        },
        {
          "brokerName": "broker-b",
          "readQueueNums": 4,
          "writeQueueNums": 4,
          "perm": 6,
          "topicSysFlag": 0
        }
    ]
}

// Broker 数据
brokerAddrTable: {
    "broker-a": {
      "cluster": "cluster1",
      "brokerName": "broker-a",
      "brokerAddrs": {
        0: "192.168.1.100:10911",
        1: "192.168.1.101:10911"
      }
    },
    "broker-b": {
      "cluster": "cluster1",
      "brokerName": "broker-b",
      "brokerAddrs": {
        0: "192.168.1.102:10911",
        1: "192.168.1.103:10911"
      }
    }
}

// Broker live info
brokerLiveTable: {
    "192.168.1.100:10911": {
        "lastUpdateTimestamp": 1519082838232,
        "dataVersion": dataVersionObj,
        "channel": channelObj,
        "haServerAddr": ""
    }
}

// Cluster addr
clusterAddrTable: {
  "cluster1": ["broker-a", "broker-b"]
}
```

#### Broker 设计与实现

TODO: Broker 的整体处理流程，从接收到生产者的消息到消息被消费者消费掉，这中间涉及到很多的环节。

Broker 的核心是消息存储、消息转发和消息过滤等，最核心的设计理念是基于 commitLog 的，在 commitLog 上构建索引，把 commitLog 高效持久化和高效复制到 Slave Broker 上；然后就是实现 Broker 的高可用，自动 failover，这个用到了 DLedger，DLedger 是结合分布式一致性算法的带有选主能力的 commitLog 实现，解决了 Master Broker 挂掉后，无法自动替换 Broker 的问题。

Broker 是整个 RocketMQ 的核心，至少重点了解如下内容：

1. broker 的消息存储是怎么实现的
2. broker 的 ha 是怎么实现的，怎么做故障切换
3. broker 的消息过滤怎么实现的

* MessageStore 服务的设计

RocketMQ 的 Broker 其中一个非常重要的功能是消息存储，MessageStore 是 Broker 中定义的接口规范，对于消息的存储做了抽象，RocketMQ 给出了一个默认的 Store 实现，我们完全可以根据规范实现自己的 Store 引擎，比如 DLedger 就可以替换掉 DefaultMessageStore，从而做到自动选主。

CommitLog 涉及到的深层知识：内存映射，顺序写盘，堆外内存与堆内内存，FileChannel

TODO：延伸到操作系统层面的实现原理

* [谈谈 RocketMQ 的消息存储设计](http://tinylcy.me/2019/the-design-of-rocketmq-message-storage-system/)

### 性能调优

#### 关于刷盘策略

> A spin lock is recommended for asynchronous disk flush, a reentrant lock is recommended for synchronous disk flush, configuration item is `useReentrantLockWhenPutMessage`, default is false; Enable `TransientStorePoolEnable` is recommended when use asynchronous disk flush; Recommend to close `transferMsgByHeap` to improve fetch efficiency; Set a little larger `sendMessageThreadPoolNums`, when use synchronous disk flush.
>
> 异步刷盘建议使用自旋锁，同步刷盘建议使用重入锁，调整Broker配置项`useReentrantLockWhenPutMessage`，默认为false；异步刷盘建议开启`TransientStorePoolEnable`；建议关闭transferMsgByHeap，提高拉消息效率；同步刷盘建议适当增大`sendMessageThreadPoolNums`，具体配置需要经过压测。

为什么在异步刷盘下关闭 transferMsgByHeap 呢？异步刷盘模式下，是建议开启 transientStorePoolEnable 的，这个时候在写 CommitLog 时使用的是堆外内存 + FileChannel 的方式，如果在开启transferMsgByHeap 时，需要将数据复制到堆内后，然后再发送出去，至少多了两次的内存拷贝，性能会有所下降，关闭后确实会提高拉取消息的效率。

```java
// PullMessageProcessor.java
// ...
final GetMessageResult getMessageResult =
            this.brokerController.getMessageStore().getMessage(requestHeader.getConsumerGroup(), requestHeader.getTopic(),
                requestHeader.getQueueId(), requestHeader.getQueueOffset(), requestHeader.getMaxMsgNums(), messageFilter);

// org.apache.rocketmq.store.DefaultMessageStore#getMessage
public GetMessageResult getMessage(final String group, final String topic, final int queueId, final long offset,
        final int maxMsgNums,
        final MessageFilter messageFilter) {
    // ...
    SelectMappedBufferResult selectResult = this.commitLog.getMessage(offsetPy, sizePy);
    // ...
    getResult.addMessage(selectResult);
    
    getResult.setStatus(status);
    getResult.setNextBeginOffset(nextBeginOffset);
    getResult.setMaxOffset(maxOffset);
    getResult.setMinOffset(minOffset);
    return getResult;
}

// org.apache.rocketmq.store.CommitLog#getMessage
public SelectMappedBufferResult getMessage(final long offset, final int size) {
    MappedFile mappedFile = this.mappedFileQueue.findMappedFileByOffset(offset, offset == 0);
    if (mappedFile != null) {
        int pos = (int) (offset % mappedFileSize);
        return mappedFile.selectMappedBuffer(pos, size);
    }
    // ...
}

// org.apache.rocketmq.store.MappedFile#selectMappedBuffer(int, int)
public SelectMappedBufferResult selectMappedBuffer(int pos, int size) {
    ByteBuffer byteBuffer = this.mappedByteBuffer.slice();
    byteBuffer.position(pos);
    ByteBuffer byteBufferNew = byteBuffer.slice();
    byteBufferNew.limit(size);
    return new SelectMappedBufferResult(this.fileFromOffset + pos, byteBufferNew, size, this);
}


```

在消费者拉取消息时，消息是从 MessageStore 中获取的，这个实现是在 DefaultMessageStore 中的，在这里会从 commitLog 中拉取需要的消息，然后返回消息的结果 GetMessageResult；在 CommitLog 中返回的是 SelectMappedBufferResult，这个数据是从 mappedFile 中获取的；而 SelectMappedBufferResult 中两个重要的属性 byteBuffer 和 mappedFile 都是和当前的 mappedFile 实例相关的，而 mappedFile 是对内存映射的封装，mappedByteBuffer 就是内存映射的那块可以直接操作的内存，也就是说，最终返回的 GetMessageResult 是一块或者多块被内存映射的内存区域，因为这些数据是磁盘上的文件和虚拟内存之间的直接映射，并且可以被用户态程序直接访问，所以不需要内存拷贝就可以直接写数据进去，也可以读数据出来；如果开启 transferMsgByHeap，就会将 pageCache 中的数据 copy 到堆内存中，然后发送到 socket buffer 中，这样就至少多了2次的内存 copy；如果不开启 transferMsgByHeap 就会使用 Zero-Copy 的方式，利用 DMA 直接把数据发送到网卡。



#### 在 RocketMQ 的 Broker 中关闭偏向锁，可以提升性能



### Reference

* [https://rocketmq.apache.org/](https://rocketmq.apache.org/)
* [RocketMQ 官方中文架构文档](https://github.com/apache/rocketmq/tree/master/docs/cn)
* [https://www.jianshu.com/p/7f772c3eccd6](https://www.jianshu.com/p/7f772c3eccd6) consumer 流程
* [如何保证rocketmq不丢失消息](https://juejin.im/post/6844904102011338760)
* [如何保证kafka不丢失消息](https://mp.weixin.qq.com/s/qttczGROYoqSulzi8FLXww)
* [快手基于 RocketMQ 的在线消息系统建设实践](https://mp.weixin.qq.com/s/ljSktiZYh_5W93m3yB4M-g)
* [RocketMQ 如何在双11下0故障](https://mp.weixin.qq.com/s/nkNT2CvPHiWZF95NWzd3Ug)
* [DLedger 主从切换实现平滑升级的技巧](https://yq.aliyun.com/articles/720413)
* [RocketMQ 概念和设计](http://qyb.cool/archives/rocketmq%E4%B8%80%E6%A6%82%E5%BF%B5%E5%92%8C%E8%AE%BE%E8%AE%A1#%E4%B8%80%E3%80%81%E6%A6%82%E5%BF%B5%E5%92%8C%E7%89%B9%E6%80%A7)
* [文件 IO 操作的一些最佳实践](https://www.cnkirito.moe/file-io-best-practise/)
* [天池中间件大赛百万队列存储设计总结](https://www.cnkirito.moe/mq-million-queue/)

性能测试

* [RocketMQ vs Kafka](https://alibaba-cloud.medium.com/kafka-vs-rocketmq-multiple-topic-stress-test-results-d27b8cbb360f)

