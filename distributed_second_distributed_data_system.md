# 分布式数据系统

## 概要
```shell
    1. 分布式系统部署目的
        (1). 扩展性，当数据量和负载超过了单机处理上限，需要将负载分散到多台机器上。
        (2). 容错和高可用，单点故障会导致系统不可用，需要多台机器进行冗余。
        (3). 延迟考虑，客户可以访问最近数据中心提供的服务，降低响应的延迟。
    2. 复制和分区
        将数据分布到多个节点的方式有"复制"和"分区"。「复制」是在多个节点保存相同的数据，达到数据冗余，使得系统可用，
        同时也可以提高系统的性能。「分区」将大数据分到不同的节点。
    3. 分布式系统核心的基本问题，节点失效、网络不可靠、副本一致性、持久性、可用性与延迟之间各种细微的权衡
```

## 数据复制
```shell
    1. 复制势必涉及到变化数据的同步问题。通常有 3 中方案，第一种主从复制。第二种多主节点复制。第三种无主节点复制。
    2. 复杂过程中的问题。
           (1). 采用同步复制还是异步复制。
           (2). 如何复制失败的副本
```

### 主从复制
```shell
    1. 主从复制的原理
            「第一步」指定一个副本为主副本（主节点），当客户端写数据库时，必须将写请求发生到主副本，主副本将新数据写到自己的本地存储。
            「第二步」其他副本为从副本（从节点），主副本将数据更新到自己的本地存储后，再将数据更改作为复制的日志或者
            更改流发生给其他从副本，从副本根据日志内容进行进行相应的操作，其写入的顺序与主副本要严格保持一致。
            「第三步」当客户端读数据库中数据时，直接去从副本进行读取。只有主副本才能接受写请求。
    2. 主从复制应用范围很广，关系型数据库：PostgreSQL, MySQL. 非关系型数据库：MongoDB,RethinkDB。
       分布式消息队列：Kafka, rabbitmq.
    3. 同步复制与异步复制，同步复制是主副本阻塞等待从副本的更新成功。同步复制的优点：一旦同步复制成功，则主从副本数据一致，
       都是最新的数据，所以客户端从数据库读的都是最新数据（不管是从主副本还是从副本）。同步复制的缺点：如果从副本无法及时完成
       同步（从副本节点崩溃，网络延迟），那么写入就不能视为成功。主副本就一直阻塞导致后续写入操作无法进行。
       所以系统一般不会将所有节点设置为同步复制，所以数据库「同步复制设置」的选项就是半同步模式，只将一个节点作为同步复制，
       其他节点作为异步复制。
    4. 增加新的从节点，保证新的从节点数据与主节点数据保持一致。简单的常规数据文件拷贝会导致不同的节点呈现不同的时间点数据，
       因为在这种拷贝方式过程中有数据写入更新操作。
       正常的逻辑过程：
            「第一步」在某个时间点对主节点的副本数据产生一致性快照。
            「第二步」将该快照拷贝到新的从副本节点上。
            「第三步」从节点连接主节点并请求该节点对应的快照之后的更新数据日志（内部的原理是在创建快照时，
                    快照与系统复制日志的位置保持关联）。
            「第四步」从节点在快照之后根据日志进行数据的更新。这个过程称为追赶。
    5. 处理节点失效情况。
            (1). 从节点失效，使用追赶式恢复。从节点的本地磁盘会保留副本收到的数据变更日志。那么在从节点恢复时就可以根据本地的更新日志
                 向主节点请求故障之后的数据。
            (2). 主节点失效，节点切换。判断主节点是否失效，通常采用超时的机制，各个节点间互发心跳包，如果主节点没有响应，
                 则认为主节点失效了。接下来进行选举（通过超过多数的节点共识），主要在从节点中选举出一个作为新的主节点，
                 该节点最好与旧的主节点数据差异小，同时客户端以后写请求都有路由到新的主节点中（请求路由），
                 其他节点接受来着新的主节点的变更数据。这时如果原来的主节点重新上线，系统要使其降为从节点.
    6. 主节点失效时进行节点切换存在的问题
            (1) 当使用了异步复制，并且在失效之前，新的主节点并没有收到原主节点的所有数据．这时如果原主节点恢复重新
            　　 加入到集群中，新的主节点会收到冲突的写请求，因为原主节点还未意识到角色的变化，还向其他的节点同步
            　　　消息．常见的解决方案是，原主节点上未完成复制的写请求丢弃，这可能会违背数据更新持久化的特性。
            (2) 在某些故障下，可能发生两个节点都自认为是主节点(脑裂)，两个主节点都可能接受写请求，并且没
                有很好解决冲突的办撞（“多主节点复制技术”），最后数据可能会丢失或者破坏
            (3) 如何设置合适的超时时间来判断主节点的失效．超时时间设置太长，如果是主节点真的失效，则整体恢复的时间长．
            　　　如果设置超时时间短，则可能由于突发的峰值导致主节点的响应时间变长甚至超时，或则是网络故障，这些都
            　　　有可能造成不必要的切换，导致结果更糟．
    7. 复制日志的实现(主从复制复制方式)
            (1) 基于语句的复制
                    主节点记录所执行的每个写请求(操作语句)并将该操作语句作为日志发送给从节点，从节点再根据语句执行．优点是
                    传输的数据量少，缺点是执行语句内有非确定性的函数(例如 NOW(). Rand())将会导致每个副本的值都不一样．
                    如果语句中使用自增列，或者依赖于数据库的现有数据，则所有副本必须按照完全相同的顺序执行，否则其执行的结果不同。
                    那么如果有多个同时并发执行的事务会有很大的限制。解决的方案：主节点可以在记录操作语句时将非确定性函数替换为
                    执行之后的确定的结果，这样所有节点直接使用相同的结果值。但是，这里面存在太多边界条件需要考虑，
                    因此目前通常首选的是其他复制实现方案。
            (2) 基于预写日志(WAL)的传输
                    预写日志记录的是存储引擎的磁盘数据结构数据，是底层的数据，如果主从节点采用相同的存储引擎，则可以通过
                    传输存预写日志来建立和主节点内容完全相同的数据副本。
                    应用: PostgreSQL, Oracle
                    缺点: 日志的记录的数据是很底层的(像哪些磁盘块哪些字节发送变化)，使复制方案与存储引擎紧密耦合在一起，
                    　　　如果主从节点的存储引擎不一致，则该复制方案无法工作
            (3) 基于行的逻辑日志复制
                    逻辑日志是区分与物理存储引擎，它与存储引擎进行解耦，更容易保持向后兼容，从而使主从节
                    点能够运行不同版本的软件甚至是不同的存储引擎。关系型数据库的逻辑日志是以行为单位，每一行
                    能够标识数据更删改的有效信息． MySQL 的基于行的复制的 binlog 日志就是这种方式．对于外部
                    程序而言，逻辑日志要更容易解析。如果要将数据库的内容发送到外部系统（如用于离线分析的数据仓库），
                    或构建自定义索引和缓存等，基于逻辑日志的复制更有优势，该技术也被称为变更数据捕获
            (4) 基于触发器的复制
                    比较: 以上 3 种复制方式是由数据库系统实现的，缺乏灵活性，比如数据变更时想复制一部分数据，或则想从
                    　　　一种数据库复制到另一种数据库．那么就可以采用触发器和存储过程(关系数据库支持的功能)
                    原理: 触发器支持注册应用层代码，当数据库系统发生数据更改（写事务）时自动执行上述自定义代码。
                    　　　通过触发器技术，可以将数据更改记录到一个单独的表中，然后外部处理逻辑访问该表，
                    　　　实施必要的自定义应用层逻辑，例如将数据更改复制到另一个系统。
                    应用: Oracle 的 Databus 和 Postgres 的 Bucardo
                    优缺点: 基于触发器的复制比其他复制开销大，更容易出错．

```
