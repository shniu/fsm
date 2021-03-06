---
description: DDD
---

# 领域驱动设计

领域驱动设计是目前为止比较完善的软件设计方法，目前比较流行的做法是结合事件风暴\(Event Storming\)和领域驱动设计在一起，运用事件风暴的工作坊方式将相关人员（业务领域专家、项目负责人、开发、测试、架构师等）聚集在一起，通过类似头脑风暴的形式将系统涉及到的关键事件、动作、业务约束、业务边界等识别出来，并建立统一语言，然后再结合 DDD 的战略设计和战术设计做更近一步的落地；从 OO 的角度来看，DDD 是 OO 的一种延伸，可以指导设计和实现，所以 OO 的思想、设计模式、设计原则同样适用在 DDD 中，系统开发是不断运用设计（更加深刻的理解业务）、不断权衡各种设计方案、不断权衡实现方案等的过程。

NOTE: 可见事件风暴和领域驱动设计是从团队协作出发的，对整个团队的要求其实是挺高的，应该有一个在这方面的专家，带着所有人不断去尝试，有可能前几次会失败，但是一定得坚持下来。

### 基本概念

1. **模型**：模型是能够表达系统业务逻辑和状态的对象；**模型，用来反映事物某部分特征的物件，无论是实物还是虚拟的**
2. **领域**：现实世界中的业务逻辑，在 IT 系统业务分析时，适合某个行业和领域相关的，所以又叫做领域；**指的特定行业或者场景下的业务逻辑**
3. **领域模型**：**DDD 中的模型是指反应 IT 系统的业务逻辑和状态的对象，是从具体业务（领域）中提取出来的，因此又叫做领域模型**
4. **领域驱动设计**：通过对实际业务出发，而非马上关注数据库、程序设计。通过识别出固定的模式，并将这些业务逻辑的承载者抽象到一个模型上。这个模型负责处理业务逻辑，并表达当前的系统状态。这个过程就是领域驱动设计

按照面向对象设计的话，我们的系统是一个电子餐厅。现实餐厅中的实体，应该对应到我们的系统中去，用于承载业务，例如收银员、顾客、厨师、餐桌、菜品，这些虚拟的实体表达了系统的状态，在某种程度上就能指代系统，这就是模型，如果找到了这些元素，就很容易设计出软件

一般方法：

分析业务，设计领域模型，编写代码，建立了领域模型后，我可以考虑使用领域模型指导开发工作，如

* 指导数据库设计
* 指导模块分包和代码设计
* 指导 RESTful API 设计
* 指导事务策略
* 指导权限
* 指导微服务划分（有必要的情况）

### Event Storming & DDD

DDD 拥有一套完整的方法论，最开始接触 DDD 被它的各种概念搞得晕头转向；后来接触到事件风暴，更是不知所云，但是在这条道路上要继续走下去。

* EventStorming

EventStorming 是一种新型的工作坊模式，它可以帮助我们协作探索复杂的业务领域。

> **EventStorming** is a flexible workshop format for collaborative exploration of complex business domains.

* DDD

**DDD其实就是面向对象的一套“方言”，提供了一种基于业务领域的对象划分和分类方法，其最大的价值就在于对于软件开发过程全生命周期使用语言的统一！**

#### DDD

**值对象**：如果这些数据没有ID和生命周期，在修改时是整体替换，也不需要基于它做统计分析，你就可以将它设计为值对象。

如果一个业务行为由多个实体对象参与完成，我们就将这部分业务逻辑放在领域服务中实现。领域服务与实体方法的区别是：实体方法完成单一实体自身的业务逻辑，是相对简单的原子业务逻辑，而领域服务则是多个实体组合出的相对复杂的业务逻辑。两者都在领域层，实现领域模型的核心业务能力。

#### 读[ ThoughtWorks 的领域驱动设计](https://weread.qq.com/web/reader/44f32bb071e1265344f0481ke4d32d5015e4da3b7fbb1fa)的总结

什么是架构设计？软件设计是一个复杂度很高的活动，通过组件化完成关注点分离从而降低局部复杂度很早就是行业共识了。组件化的好处是在不同系统里遇到同样的功能需求时可以复用，在云服务时代，这些组件以服务的形式呈现，更容易被使用，比如云厂商提供的 MQ 服务，RDS，分布式存储服务等。

如果组件化都做的很好了，为什么要在面对新变化和新需求时，需要新的组件和新的组合方式呢？回到架构设计的本质，在写代码之前为什么要做设计？因为设计首先解决的是业务复杂度的问题，其次设计是要解决建立团队协作沟通的共识。架构设计还要应对一个问题，那就是需求在不断的变化，慢慢会发现**软件架构设计的实质是让系统能够更快地响应外界业务的变化，并且使得系统能够持续演进**。在遇到变化时不需要从头开始，保证实现成本得到有效控制。

为了应对业务的变化，我们需要做面线业务变化的架构，从互联网架构中学到三个关键点：

1. 让我们的组件划分尽量靠近变化的原点，对于互联网来说就是用户和业务，这样的划分能够让我们将变化“隔离”在一定的范围（组件）内，从而帮助我们有效减少改变点
2. 组件之间能够互相调用，但彼此之间不应该有强依赖，即各自完成的业务是相对独立的，不会因为一方掉线而牵连另外一方，比如新闻网站挂掉了，聊天网站应该继续正常提供服务，可能提示用户暂时无法提供新闻信息而已。
3. 组件在业务上是鼓励复用的

**从业务出发、面向业务变化是我们现代架构设计成功的关键。架构设计的核心实质是保证面对业务变化时我们能够有足够快的响应能力。**这种响应力体现在新需求（变化）的实现速度上，也体现在我们组件的复用上，在实现过程中现有架构和代码变化点的数量也是技术人员能够切身体会到的。面对日新月异的数字化时代，组织的整体关注点都应该集中到变化的原点，即业务上，而架构应该服务于这种组织模式，让这样的模式落地变得自然。

在 SOA 架构模式中，ESB 是面向技术的组件化，试图通过技术平台的灵活性来解决变化的多样性，短时间内可能有成效，但是长期来看 ESB 必定会成为瓶颈，因为所有的变化都堆积到了这个技术组件来解决，最后 ESB 会变成一个臃肿的含有复杂业务逻辑的复杂系统，对于他的维护和升级变得困难，响应变化的速度也无法得到提升。

面向业务变化而架构就**要求首先理解业务的核心问题**，即有针对性地进行关注点分离来找到相对内聚的业务活动形成子问题域。子问题域内部是相对稳定的，即未来的变化频率不会很高，而子问题边界是很容易变化的，比如在一个物流系统中：计算货物从A地到B地的路径是相对固定的，计算包裹的体积及归类也是相对固定的，但根据包裹的体积优化路径却经常会根据业务条件而变化。

建立一个针对业务变化的高响应力架构是至关重要的，要构建一个可以持续演进的架构。DDD 给我们提供了这种可能性。

### 参考

#### Event Storming

1. [EventStorming resource](https://www.eventstorming.com/resources/)
2. [https://blog.avanscoperta.it/it/tag/event-storming-it/](https://blog.avanscoperta.it/it/tag/event-storming-it/)
3. [http://exploreddd.com/](http://exploreddd.com/)
4. [https://github.com/mariuszgil/awesome-eventstorming](https://github.com/mariuszgil/awesome-eventstorming)
5. [ddd-by-examples](https://github.com/ddd-by-examples)/[**library**](https://github.com/ddd-by-examples/library) ****公共图书馆的例子，A comprehensive Domain-Driven Design example with problem space strategic analysis and various tactical patterns. 非常全面的DDD实践
6. [https://github.com/ddd-by-examples](https://github.com/ddd-by-examples)
7. [x] [EventStorming and Spring with a Splash of DDD](https://spring.io/blog/2018/04/11/event-storming-and-spring-with-a-splash-of-ddd) 介绍 DDD 和 Spring 的结合，principles for building Spring applications with DDD

#### DDD

* [https://www.jdon.com/](https://www.jdon.com/) 软件架构解决之道
* [DDD、EventStorming与业务中台](https://zhuanlan.zhihu.com/p/120896743)
* [使用 DDD 指导业务设计的一点思考](https://insights.thoughtworks.cn/ddd-business-design/)

由浅入深的分析了模型和领域模型，生动的讲述了基于领域模型的驱动开发，和传统的基于数据驱动的开发截然不同，基于领域驱动的设计从业务中分析，提炼出领域模型，然后根据领域模型进行系统设计和开发。

* [DDD 开发落地实践](https://insights.thoughtworks.cn/backend-development-ddd/)
* [后端开发实践：简单可用的 CQRS 编码实践](https://insights.thoughtworks.cn/backend-development-cqrs/)
* [ ] [简单可用的 CQRS 编码实践](https://insights.thoughtworks.cn/backend-development-cqrs)
* [ ] [在微服务中使用领域事件](https://insights.thoughtworks.cn/use-domain-events-in-microservices/)
* [ ] [事件驱动架构EDA编码实践](https://zhuanlan.zhihu.com/p/79095599)
* [ ] [识别领域事件](https://zhuanlan.zhihu.com/p/43776403)
* [ ] DDD 实践案例：[e-commerce-sample](https://github.com/e-commerce-sample)
* [ ] [DDD 战术模型之聚合](https://gitbook.cn/books/5b481d2f3ba8652852051915/index.html)
* [ ] [https://zhuanlan.zhihu.com/c\_137428247](https://zhuanlan.zhihu.com/c_137428247)
* [ ] [https://tech.meituan.com/2017/12/22/ddd-in-practice.html](https://tech.meituan.com/2017/12/22/ddd-in-practice.html) DDD 在美团的实践
* [ ] [https://insights.thoughtworks.cn/backend-development-ddd/\#comment-50693](https://insights.thoughtworks.cn/backend-development-ddd/#comment-50693)
* [ ] [https://www.jdon.com/55003](https://www.jdon.com/55003)  使用ddd实现单体系统的模块化

#### Book

1. [实现领域驱动设计](../../other/reading/ddd-impl.md)
2. [复杂软件设计之道：领域驱动设计全面解析与实战](https://www.jdon.com/54881)  [配套视频](https://pan.baidu.com/disk/home#/all?path=%2FDDD&vmode=list)  [通过JiveJdon入门学习DDD](https://www.jdon.com/ddd/jivejdon/1.html)
3. 解构领域驱动设计 
4. 中台架构与实现：基于DDD和微服务

DDD 实践案例

* [聊一聊聚合的持久化](https://zhuanlan.zhihu.com/p/87074950?utm_source=wechat_session&utm_medium=social&s_r=0)
* [https://app.gitbook.com/@fuinorg/s/ddd-4-java/](https://app.gitbook.com/@fuinorg/s/ddd-4-java/)
* [DDD 在爱奇艺打赏业务中的实践](https://mp.weixin.qq.com/s/9qSvOrOHZsZq5o0LVOZhlw)

#### DDD 会议

* [2020 DDD China](https://scrmtech.gensee.com/webcast/site/vod/play-f548661ed9764816a0b32fdcb053bd32)

