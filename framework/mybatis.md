# MyBatis

### 简介

[MyBatis](https://mybatis.org/mybatis-3/) 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

1. [MyBatis 和 Spring Boot 整合](https://www.ibm.com/developerworks/cn/java/j-spring-boot-integrate-with-mybatis/index.html)
2. [Mybatis spring boot starter](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)

基本使用

配置：[https://mybatis.org/mybatis-3/configuration.html](https://mybatis.org/mybatis-3/configuration.html)

如何深入学习 MyBatis？

1. [MyBatis 源码分析导读](https://www.tianxiaobo.com/2018/07/16/MyBatis-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%E5%AF%BC%E8%AF%BB/)
2. [MyBatis 源码结构](https://my.oschina.net/wenjinglian/blog/1625437)
3. [MyBatis 框架](https://www.jianshu.com/p/15781ec742f2)

源码阅读计划

* [ ] 整体了解 MyBatis，如基本使用、项目结构、前置知识、基本原理、解决什么问题、有什么优势、有什么不足、最佳实践等
* [ ] 从 MyBatis 的模型、接口出发来分析，产出 MyBatis 的核心模型（领域）和核心的编程接口
* [ ] 分析 MyBatis 的初始化流程
* [ ] 分析 MyBatis 的 SQL 执行流程
* [ ] 分析 MyBatis 和 Spring 的集成
* [ ] 总结 MyBatis 的设计模式和设计思想，有哪些可以借鉴的地方

### JDBC 规范

1. [JDBC 4.2 规范](https://download.oracle.com/otn-pub/jcp/jdbc-4_2-mrel2-spec/jdbc4.2-fr-spec.pdf?AuthParam=1591492626_c2317df8fac0675b5c4243a58cac9ebd)
2. [JTA 规范](https://download.oracle.com/otn-pub/jcp/jta-1.1-spec-oth-JSpec/jta-1_1-spec.pdf?AuthParam=1591492792_9b9e886a24a5d84c67f5c44a204e2bbd)
3. [JNDI 规范](https://docs.oracle.com/cd/E17802_01/products/products/jndi/javadoc/)

想要学好 MyBatis，JDBC 规范是必备的。

### MyBatis 框架设计

![MyBatis &#x5206;&#x5C42;&#x67B6;&#x6784;](../.gitbook/assets/image%20%285%29.png)

1. API接口层
2. 数据处理层：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。
3. 框架支撑层：包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。

### MyBatis 的常用配置

* [MyBatis 常用 API 及 注解](https://mybatis.org/mybatis-3/java-api.html)

### MyBatis 的设计思想和设计原则



### MyBatis 小技巧

* MyBatis 插入数据时返回自增主键

开发过程中，我们往往需要在插入数据后获取当前记录自增后的id，可以使用 `select last_insert_id();` 来获取到；但是用了 MyBatis 之后怎么做呢？需要配合 `useGeneratedKeys` 和 `keyProperty` ,  keyProperty的值为传入参数的属性名，Mybatis会自动把自增id的值赋给该属性， 而返回值仍为影响行数：

> useGeneratedKeys: insert and update only, This tells MyBatis to use the JDBC getGeneratedKeys method to retrieve keys generated internally by the database, default is false
>
> keyProperty: insert and update only, Identifies a property into which MyBatis will set the key value returned by getGeneratedKeys, or by a selectKey child element of the insert statement. Default: unset. Can be a comma separated list of property names if multiple generated columns are expected.

```text
// xml 的方式
<insert id="insertAuthor" keyProperty="id" useGeneratedKeys="true">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>

// or
<selectKey
  keyProperty="id"
  resultType="int"
  order="BEFORE"
  statementType="PREPARED">
  ......
</selectKey>

/// ---
// 实现原理参考：https://cofcool.github.io/tech/2017/11/06/Mybatis-insert-get-id
// 可见 MyBatis 帮我们处理了这种场景
```

* MyBatis Plus

Todo 探索 MyBatis Plus 的增强

{% embed url="https://www.cnblogs.com/wunian7yulian/p/10276642.html" %}

```text
// MyBatis 的 BaseMapper 接口
com.baomidou.mybatisplus.core.enums.SqlMethod 定义了很多的 SQL 模版，根据模版来自动
生成 SQL 语句
```

{% embed url="https://github.com/baomidou/mybatis-plus" %}

### MyBatis 源码分析

#### MyBatis 初始化流程分析

```text
// SqlSessionFactoryBuilder
// 核心功能：
//  1. XML(XMLConfigBuilder) -> org.apache.ibatis.session.Configuration
//  2. Configuration -> SqlSessionFactory (DefaultSqlSessionFactory)

Configuration 是 Mybatis 的配置类，Mybatis 支持的所有配置都定义在这里
```



