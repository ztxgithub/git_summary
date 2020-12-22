# kafka

## 概念
```shell
    1. 应对数据量激增的情况, Kafka 能够有效隔离上下游业务, 将上游突增的流量缓存起来, 以平滑的方式传导到下游子系统中, 
       避免了流量的不规则冲击
       
    2. kafka 客户端, libkafka
    3. 框架
            (1) kafka 入门, 消息引擎基础, kafka 基本术语, kafka 角色定位, kafka 版本选择.
            (2) 基本使用, 线上方案确定, 集群配置参数
            (3) 客户端
                    生产者, 分区机制, 压缩算法, 无消息丢失配置, TCP 连接管理, 幂等性生产者与事务
                    消费者, 消费者组, 位移主题(consumer_offset), Rebalance, 位移提交, 异常处理, 多线程开发, TCP 连接管理, 
                           group 监控.
            (4) kafka 原理, 备份机制, 请求处理, Rebalace 全流程, Controller, 高水位
            (5) 运维与监控, 主题管理, 动态配置, 消费者组位移管理, KafkaAdminClient, 认证机制, MirrorMaker, 监控框架, 授权管理, 
                           kafka 调优, 流处理应用搭建
                           
            (6) 高级 kafka 应用, kafka streams, kafka DSL 开发
    4. kafka 是一款开源的消息引擎系统. 消息引擎传输的对象是消息(要进行消息序列化), 
       如何传输消息属于消息引擎设计机制的一部分(点对点模型, 发布/订阅模型), 
       消息引擎系统的作用是, 第一处理瞬时高流量的场景, 防止一下子给下游的系统造成压力, 是流量控制在平滑的状态.
                          第二解耦, 减少了系统间不必要的交互
                          
    5. Kafka 是消息引擎系统, 也是分布式流处理平台, 它可以保证实现端到端的精确一次处理语义, kafka 的流处理组件 Kafka Streams 是一个
    　　用于搭建实时流处理的客户端库而不是一个完整的流处理平台, 没有集群调度, 弹性部署等运维特性, 适用于中小型项目.
    
    6. kafka 种类
            (1) Apache Kafka: 也被称为社区版 Kafka, 最常用的, 开发人数最多, 但 Apache Kafka 本身不提供任何监控框架和工具. 不过
                            　有一些开源监控框架可以监控 kafka, 例如 Kafka manager, kafka eagle, JMXTrans + InfluxDB + Grafana
                              仅仅需要一个消息引擎系统, 
                            　可以考虑 Apache Kafka:
            (2) Confluent Kafka: Confluent Kafka 提供了一些 Apache Kafka 没有的高级特性, 比如跨数据中心备份、Schema 注册中心
                                 以及集群监控工具
            (3) Cloudera/Hortonworks Kafka: 通过便捷化的界面操作将 Kafka 的安装、运维、管理、监控全部统一在控制台中, 降低对 Kafka
                                            集群的掌控程度, 缺陷在于把控度低，演进速度较慢
                          
    7. Kafka 版本迭代
            (1) Scala 2.11 - kafka_2.11-2.2.1.tgz,  其中 2.11 是编译 Kafka 源代码的 Scala 编译器版本, Kafka 服务器端的代码由
                scala 语言编写的, 那么 kafka 的版本号为 2.2.1 -> 大版本 - 小版本 - Patch 号
            (2) 0.8 : 引入了副本机制, 提供了数据的高可靠, 较好得做到了数据不丢失, 要升级到 0.8.2.2 这个版本, 因为该版本
                      中老版本消费者 API 是比较稳定的, 但不要使用新版本的 Producer API(bug 较多)
                      
                0.9: 增加基础的安全认证/权限功能, 使用 java 重写了新版本的消费者 API, 引入 Kafka Connect 组件用于实现高性能的
                　　　数据抽取, 可以使用新版本的 Producer, 不用使用新版本 Consumer API(bug 较多), 位移保存由原来的 zookeeper
                     变为 kafka 内部的位移主题.
                0.10: 引入 kafka stream, 可以进行分布式流处理. 升级到 0.10.2.2 然后使用新版本 Consumer API(已经稳定), 修复了
                　　　　可能导致 Producer 性能减低的 bug
                0.11: 提供了幂等性 Producer API 以及事务 API, 对 Kafka 消息格式做了重构, 目前最主流的版本之一, 可以更新到
                　　　　0.11.0.3
                1.0 和 2.0 : 主要还是针对 Kafka Streams 进行改进.
                
            (3) 建议尽量保持服务器端版本和客户端版本一致
```

### 技巧
```shell
    1. Kafka broker CPU 偏高的情况
            (1) server 和 client 使用不同的压缩算法.
            (2) server 和 client 版本不一致造成消息格式转换.
            (3) broker 端解压缩校验
            
    2. kafka-producer-perf-test 性能测试脚本还不错
    3. 解决消息队列重复消费问题, 可以用 kafka 0.11 的事务
```

## 基础知识
```shell
    1. 备份机制中, kafka 定义了领导者副本(Leader Replica)和追随者副本(Follower Replica), 领导者副本是对外服务, 追随者副本用于进行
    　　数据冗余, 不对外服务(包括读写操作)
    2. 分区, 也就是分片, 将数据分布在不同服务器上, 形成完整的数据. Kafka 中的分区机制指的是将每个主题划分成多个分区(Partition),
       每个分区是一组有序的消息日志. 生产者生产的每条消息只会被发送到一个分区.每个分区下可以配置若干个副本, 其中只能有 1 个领
       导者副本和 N-1 个追随者副本. 生产者向分区写入消息, 每条消息在分区中的位置信息由位移（Offset）来表示. 分区位移总是从 0 开始, 
       假设一个生产者向一个空分区写入了 10 条消息, 那么这 10 条消息的位移依次是 0、1、2、…、9
       
       整体架构:
            第一层是主题层, 每个主题可以配置 M 个分区, 而每个分区又可以配置 N 个副本
            第二层是分区层, 每个分区的 N 个副本中只能有一个充当领导者角色, 对外提供服务；
                          其他 N-1 个副本是追随者副本, 只是提供数据冗余之用.
            第三层是消息层, 分区中包含若干条消息, 每条消息的位移从 0 开始, 依次递增。
            客户端程序只能与分区的领导者副本进行交互.
            
    3. 持久化, Kafka 使用消息日志(Log)来保存数据, 只能进行追加写, 进行性能较好的顺序 I/O 写操作(高吞吐量的原因)
              定期删除旧消息, 一个日志细分成多个日志段, 消息被追加写到当前最新的日志段中, 当写满了一个日志段后, Kafka 会自动切分出
              一个新的日志段, 并将老的日志段封存起来. Kafka 在后台还有定时任务会定期地检查老的日志段是否能够被删除, 
              从而实现回收磁盘空间的目的
              
    4. kafka 的 Rebalance(重平衡), 消费组内的某个消费者实例异常了, Kafka 能够自动检测到, 然后把这个异常实例之前负责的分区转移给
       其他活着的消费者
    5. topic 创建的时候可以指定分区数
```

### 线上集群部署
```shell
    1. 操作系统
            I/O 模型
                linux 操作系统要比其他操作系统合适. 在 I/O 模型上 kafka 在 linux 上使用的是 epoll , 
                而部署在 windows 上使用的是 select
             
            网络传输效率
                linux 下的 kafka 的数据传输可以通过零拷贝(Zero Copy)技术, 避免使用内核数据进行拷贝.
                
    2. 磁盘
            使用普通机械磁盘就可以了, 因为 kafak 的存储是顺序读写操作, 不需要随机读写, 很大规避普通机械磁盘劣势.由于 kafka 在软件层面
            提供了数据冗余, 以及负载均衡(分区技术), 也可以不用磁盘阵列(RAID)
            
            磁盘容量
                新增消息数(平均每天), 消息留存的时间, 每条消息平均大小, 备份数, 是否启用压缩.
    3. 带宽
            根据带宽计算出需要部署几台 kafka 服务器, 假设是 1Gbps 的千兆网络, 而给 kafka 使用的带宽则是 70%, 所以单台服务器收到最多
            为 700Mb 的带宽资源, 不能让 Kafka 服务器常规性使用这么多资源, 故通常要再额外预留出 2/3 的资源, 即单台服务器使用带宽
            700Mb / 3 ≈ 240Mbps,  而业务需要 1 秒处理 1Gb 的数据. 那么就有 1Gb / 240M = 10 台, 再加上再备份 2 份数据, 则需要
            30 台.
```

### 集群配置参数
```shell
    1. Broker 参数
        (1) 磁盘存储
                log.dirs : 指定 kafka 文件存储的具体目录, 指定若干个目录, 例如 /home/kafka1,/home/kafka2,/home/kafka3
                           注意: 指定目录最好挂载在不同物理磁盘上. 可以提升读写磁盘, 多块磁盘同时读写提高吞吐量, 同时 Broker 自动在
                                好的路径上重建副本, 然后从 leader 同步 . 或则　Kafka　支持工具能够将某个路径上的数据拷贝到其他路径上
                                在 kafka 1.1 版本后, 坏掉磁盘的数据可以自动转移到其他正常磁盘上, Broker 还能正常工作, 在 1.1 版本
                                以前挂掉任何一个磁盘, Broker 都无法正常工作.
                                
        (2)ZooKeeper 相关
                zookeeper.connect: zk1:2181,zk2:2181,zk3:2181, 指向 zookeeper 的端口信息, 
               
        (3) Broker 连接相关的
                listeners：　<协议名称, 主机名, 端口号>
                            监听器, 通过什么协议, 指定的主机名和端口访问 kafka 服务, 
                            协议名称：　
                                    PLAINTEXT : 明文传输 
                                    SSL: SSL 加密传输
                                    
                            主机名: 这个最好使用主机名, 不要使用 ip
                            
                advertised.listeners 
                        主要是用于外网访问, kafka 部署在内网不需要设置, 常见使用是 Kafka Broker 机器上配置了双网卡,
                        一块网卡用于内网访问（即内网IP）；另一个块用于外网访问. 可以配置 listeners 为内网 IP, 
                        advertised.listeners 为外网 IP
                            
        (4) topic 相关
                auto.create.topics.enable
                    是否运行自动创建 topic, 最好设置为 false, 要事先规定好 topic
                    
                unclean.leader.election.enable
                    是否允许 Unclean Leader 选举, 分区内有多个副本, 只有一个副本对外服务, 这个副本一般是数据保存多的, 如果有些副本
                   数据保存太少称为 Unclean 副本, 不能参加领导副本竞选. 
                   设置为 false(建议), 代表其他保存多的副本都故障了,也不能让 Unclean 副本参与竞选, 会使得 kafka 不可用.
                   设置为 true , 则数据会丢失.
                   
                auto.leader.rebalance.enable(false)
                    是否允许定期进行 Leader 选举, 由于切换掉对应的 leader, 需要一定成本, 并且所有客户端都要切换.
                    
        (5) 数据留存
                log.retention.{hour|minutes|ms}
                    就是 log.retention.hour, log.retention.minutes, log.retention.ms, log.retention.ms 的优先级最高, 代表
                    消息数据被保存多久时间, 之后就删除掉.
                    
                log.retention.bytes
                    可以保存多少容量的数据消息, -1 表示无限制. 这个参数用于云上 kafka 服务, 限制每个租户使用的磁盘空间
                    
                message.max.bytes
                    约定 Broker 控制消息最大字节数, 默认的 1000012 太少了, 需要设置.
                    
    2. Topic 级别参数(会覆盖全局参数)
    
        (1) retention.ms: 规定该 Topic 消息被保存的时长, 一旦设置了这个值, 它会覆盖掉 Broker 端的全局参数值, 该 topic 保存按
        　　　　　　　　　　 retention.ms 的值保存.
        (2) retention.bytes：规定要为该 Topic 预留多大的磁盘空间, -1 : 代表无限制.
        (3) max.message.bytes
        (4) 需用同时修改 Broker 的 replica.fetch.max.bytes 保证复制正常, 消费还要修改配置 fetch.message.max.bytes
        
        设置 Topic 级别参数方法:
            第一种: 创建 Topic 时进行设置, 
                      > bin/kafka-topics.sh  --create 
                        --topic transaction(topic 名字) 
                        --bootstrap-server localhost:9092
                      　--partitions1 
                        --replication-factor1
                        --config retention.ms=15552000000 --config max.message.bytes=5242880
                        
            第二种: 修改 Topic 时设置(推荐使用)
                    > bin/kafka-configs.sh 
                       --zookeeper localhost:2181
                       --entity-type topics 
                       --entity-name transaction
                       --alter 
                       --add-config max.message.bytes=10485760
                       
                       
    3. JVM 参数
            (1) kafka 最好部署在 java 8 环境中
            (2) 将 JVM 堆大小设置成 6 GB, 因为 Kafka Broker 在与客户端进行交互时会在 JVM 堆上创建大量的 ByteBuffer 实例,
                Heap Size 不能太小, 正常情况可以监控堆上的 live data, 设置 JVM 堆为 1.5 倍, 可以手动触发Full GC, 然后查看一下
                堆上存活的数据大小, 比如说是 1500MB, 那么你可以设置heap size为 2.25 GB, 没必要越大越好, 这样会影响页缓存的空间
            (3) GC 设置(垃圾回收器), 
                    如果是 java 7 
                        cpu 充裕, 则使用 CMS 收集器(-XX:+UseCurrentMarkSweepGC)
                        cpu 不充裕, 则使用吞吐量收集器(-XX:+UseParallelGC)
                        
                    如果是 java 9, 使用默认的 G1 收集器
                    
            (4) 具体设置
                    在启动 Kafka Broker 之前, 设置 KAFKA_HEAP_OPTS(堆大小), KAFKA_JVM_PERFORMANCE_OPTS(GC 参数) 这两个环境变量
                    > export 　KAFKA_HEAP_OPTS=--Xms6g  --Xmx6g
                    > export  KAFKA_JVM_PERFORMANCE_OPTS= -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 
                    　　　　　　　　　　　　　　　　　　　　　　-XX:InitiatingHeapOccupancyPercent=35 
                    　　　　　　　　　　　　　　　　　　　　　　-XX:+ExplicitGCInvokesConcurrent
                     　　　　　　　　　　　　　　　　　　　　　 -Djava.awt.headless=true
                    > bin/kafka-server-start.sh config/server.properties
                        
                        

    4. 操作系统参数
            (1) 文件描述符限制, 常见的错误 "Too many open files" 跟文件描述符有关, 设置好 ulimit -n 1000000
            (2) 文件系统类型, 选择 xfs 的日志型文件系统要好于 ext4 类型.
            (3) swap 空间, 将其设置为很小值(接近于 0), 如果是 0, 则进程的物理内存不够, 会直接 kill 掉进程, 可以用很小的值, 使得进程不
                           被 kill 掉, 又能快速发现问题(Broker 性能急速下降)
            (4) Flush 落盘时间, kafka 调用系统函数, 只是将数据写到操作系统的页缓存中, 后续操作系统根据 LRU 算法会定期将页缓存数据
                存到磁盘上. 
                    
                    
    5.其他
        (1) auto.offset.reset
                earliest: 从头开始读取消息
                latest: 从当前新位移处读取
                auto.offset.reset 被用到的时机是, 当 consumer 启动后会从 Kafka 读取它上次消费的位移,有以下两种情况
                    情况1： 如果 Kafka broker 端没有保存这个位移值
                    情况2： consumer 拿到位移值开始消费, 如果后面发现它要读取消息的位移在 Kafka 中不存在(可能对应的消息已经被删除了)        
```

## broker
### 概念
```shell
    1. offsets.topic.num.partitions, 位移主题分区数据, 默认是 50
    2. offsets.topic.replication.factor, 位移分区的副本数.
```

## 生产者

### 生产者参数
```shell
    1. compression.type, 指定压缩类型, GZIP, Snappy 和 LZ4, zstd
    2. acks = all, 已提交的定义
    3. retries, 其他除 callback 外的故障重试的次数.
    4. bootstrap.servers, 指定 producer 启动时要向　bootstrap.servers　中所有的 broker 发起 tcp 连接.
    5. connections.max.idle.ms 指连接到 broker 在　connections.max.idle.ms　时间内没有数据通信, 则 kafka 主动断开连接. 值为 -1 
       表示永久连接.
    6. enable.idempotence 设置幂等性, true 为幂等
      
```
### 分区机制
```shell
    1. 概念
            (1) 分区的作用
                    a. 提供数据负载均衡.
                    b. 可以实现系统的高伸缩性, 后续容量不够了都可以增加服务器扩展分区.
                    c. 不同分区部署在不同的服务器上, 数据读写操作也是按分区来进行的, 这样可以同时多个分区读写, 提高系统吞吐量.
                    
            (2)  Kafka 默认分区策略是
                    如果指定 Key, 那么默认分区策略按消息键保序策略
                    如果没有指定 Key, 则默认分区策略使用轮询策略
                    
            (3) 选择策略
                    如果有业务要确保消息的顺序(因果关系), 则可以通过消息特点设置在 key 中, 使 key 分配到某一分区, 既达到顺序性.
                    又使其他的消息并发读取.
                    
                    
    2. 分区机制策略 - 轮询策略(Round-robin)
            (1) 生产者放置消息的策略是由 partitioner.class 参数进行指定, kafka java api, 默认就是轮询策略.
            (2) 优点: 保证消息最大限度地被平均分配到所有分区上
            (3) 缺点: 增加分区后, 需要重新进行数据再分配
            
    3. 随机策略(Randomness)
            
    4. 按消息键保序策略(Key-ordering)
            (1) kafka 可以为每个消息指定 key, 这个消息的 key 可以是业务相关的,例如业务 id, 部门编号. 也可以表示消息本身数据, 例如
                将 key 设置为时间戳. 一旦消息被定义了 Key, 可以保证同一个 Key 的所有消息都进入到相同的分区里面, 分区内的消息也是顺序.
```

### 生产者压缩算法
```shell
    1. 概念
            (1) kafak 以消息集合层面进行写入, 而消息集合又是由多条日志项(record item)组成.
            (2) V1 版的消息格式与 V2 版的区别(0.11.0.0 引入)
                    a. crc 校验变化
                       v1 版每条消息都要执行 CRC 校验, 有些情况又得重新执行 crc 校验,  例如 broker 端可能会对消息时间戳进行更新,
                       时间戳更新完后, 每条消息又得进行 crc 校验. 还有 broker 端有时会对消息格式进行转换(主要兼容老版半客户端), 这时
                       每条消息的 crc 又得计算,　浪费空间, cpu 的使用.
                       
                       v2 版本将 crc 校验计算上移到消息集合层, 减少每个消息的校验计算.
                       
                    b. 压缩变化
                        v1 版本是把多条消息进行压缩, 然后保存到外层消息的消息体字段中.
                        v2 版本是对整个消息集合进行压缩.
                        
    2. 压缩侧
            大部分压缩侧是在生产者(product)进行的, 但也有部分情况是在 broker 端进行
            
            生产者:
                    在生产者指定  compression.type 配置选项, 可以选择压缩的种类(gzip, snappy)
                    
            broker:
                    在 broker 进消息的压缩有 2 种情况
                    情况一: broker 中的压缩算法和生产者的压缩算法不一致, 通常 broker 中的 compression.type 默认是与生产者一致.
                           如果不一致, 就得先解压缩再压缩.
                           
                    情况二: broker 进行消息格式转换.Kafka 集群中可能会同时保存多种版本的消息格式. 为了兼容
                           老版本的格式, Broker 端会对新版本消息格式向老版本格式的进行转换转换.这个会涉及消息的解压缩和重新压缩
                           
                    情况三: broker 将消息保存时, 要先进行解压, 进行消息的校验.(kafka 2.4 已经修复了, 不需要解压验证)
                    
    3. 压缩算法
            (1) kafka 2.1.0 以前的, GZIP, Snappy 和 LZ4
                kafka 2.1.0 以后,  zstd(Zstandard), 是 Facebook 开源的一个算法, 提供超高的压缩比.
                
            (2) 在压缩比方面, zstd > LZ4 > GZIP > Snappy, 压缩比最高的是 zstd
                在解压缩速度上, LZ4 > Snappy > zstd 和 GZIP, LZ4 速度最快.
                
    4. 策略
            (1) 如果 product 侧的 CPU 充足, 并且网络带宽有限, 则建议开启压缩.
```

### 配置无消息丢失
```shell
    1. 概念
            (1) kafka 只对 "已提交"的消息(committed message) 做有限度的持久化保证. 其中"已提交"是指生产者已将消息提交给 broker 
                并保存, 并给生产者响应报文.
                
    2. 消息丢失情况
            (1) 生产者 product , 使用错误的 api, 例如 producer.send(msg), 这个是异步的发送, 成功返回不代表消息持久化到 kafka 中.
                可以使用 producer.send(msg, callback), 发送的结果都在 callback 中.
                
            (2) 对于消费者 comsumer 而言, 要先读消息, 成功后再更新位移, 防止更新完位移后, 客户端出现异常消息丢失.
            (3) 同个客户端多线程异步处理消息, consume 不能开启自动提交位移, 需要应用程序手动提交位移. 多线程处理不太好控制, 会存在消息
                重复消费的可能.
                
    3. 实践
            Producer 的参数
                (1) 设置 acks = all,  代表"已提交"的条件, all 是生产者认为所有的副本 Broker 都要收到消息, 
                    才认为消息是 "已提交", 如果是 all, 则时间花在了各个副本的同步上.
                (2) 设置 retries 的值,  代表消息发送失败时, 进行消息重发的次数. 与  producer.send(msg, callback) 中的 callback
                    不冲突, 先对于可重试的错误, retries 触发, 否则直接进入 callback
            
            broker 的参数
                (1) 设置 unclean.leader.election.enable = false, 防止副本 broker 数据同步太少也竞争成为 leader.
                (2) 设置 replication.factor >= 3, 代表 broker 集群中数据冗余的份数.
                (3) 设置 min.insync.replicas > 1, 代表对于 borker 来说消息至少要被写入到多少个副本才算是“已提交
                (4) 确保 replication.factor > min.insync.replicas, 推荐 replication.factor = min.insync.replicas + 1
                
            Consumer 的参数
                (1) 设置 enable.auto.commit 为 false, 取消消费者的自动提交位移,改为手动提交位移, 防止多线程中某一个线程处理
                    异常的情况
```

### 拦截器
```shell
    1. 概念
    2. 生产者拦截器
            (1) 生产者拦截的是在发送消息前, 或则消息提交成功后进行拦截处理.
            (2) 生产者和消费者都有一个相同参数(interceptor.classes), 指定是一组类的列表, 每个类都是特定逻辑的拦截器实现类.
            (3) 
                onSend：该方法会在消息发送之前被调用
                onAcknowledgement：该方法会在消息成功提交或发送失败之后被调用, onAcknowledgement 的调用要早于 callback 调用. 和
                                   onSend 不在同一个线程中, 需要考虑到对临界资源的互斥访问, 同时 onAcknowledgement 方法是在
                                    Producer 发送的主路劲下, 所以 onAcknowledgement 方法业务不能太重
                                    
    3. 消费者
            (1) 消费者拦截的是在消费消息前, 或则提交位移后编写特定逻辑
            (2) 
                 onConsume: 该方法在正式消费消息前调用, 可以对消息进行预处理
                 onCommit：　Consumer 在提交位移后调用该方法. 可以在该方法中做一些记账类的动作, 比如打日志等
                 
    4. 使用场景
            (1) 端到端系统性能检测, 通过实现拦截器的逻辑以及可插拔的机制, 可以观测, 验证以及监控集群间的客户端性能指标, 从具体的消息层面
            　　上收集这些数据.
            (2) 消息审计, 即接入 kafka 服务时, 需要随时查看每条消息是哪个业务方在什么时候发布的, 之后又被那些业务方在什么时刻消费的.
                可以编写一个拦截器类, 实现相应的消息审计逻辑, 同时规定所有接入 kafka 服务的客户端需要设置这个拦截器.
                 
```

### 生产者管理 tcp 连接
```shell
    1. 概念
    2. 开发生成者步骤
            (1) 构造参数对象(生产者需要的)
            (2) 将参数对象作为传入参数, 创建 KafkaProducer 对象实例
            (3) 使用 kafkaProducer 的 send() 方法发送消息
            (4) 调用 KafkaProducer 的 close 方法关闭生产者, 并释放各种系统资源
            
            Properties props = new Properties ();
            props.put(“参数 1”, “参数 1 的值”)；
            props.put(“参数 2”, “参数 2 的值”)；
            ……
            try (Producer<String, String> producer = new KafkaProducer<>(props)) {
                        producer.send(new ProducerRecord<String, String>(……), callback);
             ……
            }
            
    3. 生产者创建 tcp 连接
            (1) 在创建生产者时(KafkaProducer　实例), 会启动　Sender 线程, 该线程会根据  bootstrap.servers 参数, 向每个
                bootstrap.servers 参数列表中的 broker 进行 tcp 连接, 同时发送　metadata　请求, 获取集群的元数据信息.
            (2) 同时在更新了集群的元数据后, 也会存在向新的 broker 进行 tcp 连接, 更新集群元数据的时机是, 
                    第一:  Producer 尝试给一个不存在的主题发送消息, producer 收到 broker 响应, 此时 Producer 会发送 METADATA 
                          请求给 Kafka 集群, 去尝试获取最新的元数据信息
                    第二   Producer 每隔 metadata.max.age.ms 时间定期去向 broker 拉取最新的元数据.
            (3) 要发送消息时, Producer 发现与目标 Broker 的连接不存在, 也会创建一个
            
    4. 生产者关闭 tcp 连接
            (1) 生产者主动关闭连接, 推荐使用 producer.close() 
            (2) 生产者被动关闭, product 设置 connections.max.idle.ms, 默认是 9 分钟, 如果该连接在 9 分种内没有进行任何数据传输, 
                kafka 会主动断开 tcp 连接. 
                
    5. 小结
            (1) KafkaProducer 实例首次更新元数据信息之后, 还会再次创建与集群中所有 Broker 的 TCP 连接
            (2) 如果设置 Producer 端 connections.max.idle.ms 参数大于 0, 则初始化 Sender 线程创建的 TCP 连接在没有数据通信时会被
            　　自动关闭, 如果设置 connections.max.idle.ms =-1, 那么初始化 Sender 线程创建的 TCP 连接将无法被关闭, 从而成为空连接
```

### 消息可靠性
```shell
    1. 概念
            (1) 消息交付可靠性保障
                    a. 最多一次(at most once)：消息可能会丢失, 但绝不会被重复发送.
                    b. 至少一次(at least once)： 消息不会丢失, 但有可能被重复发送.
                    c. 精确一次(exactly once)：消息不会丢失, 也不会被重复发送
                    
                kafka 使用的是 至少一次
                
    2. 精准一次
            (1) 幂等性 Producer
                    原理:
                        producer 默认是不具备幂等性的, 会出现 producer 发送多条相同的消息, 在 kafka 服务没有进行过滤,
                        在 0.11 版本后, 支持 Producer 的幂等性, 设置 enable.idempotence 参数, 
                        props.put(“enable.idempotence”, ture), 设置完幂等性后, producer 逻辑不变, kafka 服务负责消息的去重.
                        
                    注意:
                        (1) 只保证单分区的幂等性, 只保证某个主题的一个分区上不出现重复消息, 它无法实现多个分区的幂等性
                        (2) 只能实现单会话上的幂等性, 不能实现跨会话的幂等性, 如果 producer 重启还会出现多条消息的可能.
                        
            (2) 事务性 Producer
                    原理:
                        解决了单分区的幂等性问题, 和 单会话的幂等性问题.
                    
                        隔离性表明并发执行的事务彼此相互隔离, 互补影响, 隔离性也称为为可串行化(假设每个事务都是数据库中唯一的事务).
                        kafka 支持的是已提交读(read committed)隔离级别, 当消费者读数据时, 只能看到已提交的数据.
                        保证多条消息原子性地写入到目标分区.
                        
                        事务型 Producer 能够保证将消息原子性地写入到多个分区中, 这批消息要么全部写入成功, 要么全部失败.
                        producer 重启后, kafka 依然可以处理重复消息.
                        
                    操作:
                        第一步: 先设置 Producer 参数, enable.idempotence = true, 设置　transctional.id 为唯一的值.
                        第二步: producer 端调用事务 api, 
                        
                                    producer.initTransactions();  // 事务初始化
                                    try {
                                                producer.beginTransaction();  // 开始事务
                                                producer.send(record1);       // 操作
                                                producer.send(record2);       // 操作
                                                producer.commitTransaction(); // 提交事务
                                    } catch (KafkaException e) {
                                                producer.abortTransaction();
                                    }
                                    
                                    
                        consumer 端也需要针对事务 producer ,设置对应的参数 isolation.level 参数, 
                            isolation.level 的值:
                                read_uncommitted：这是默认值，表明 Consumer 能够读取到 Kafka 写入的任何消息, 不论事务型 
                                                  Producer 提交事务还是终止事务, 其写入的消息都可以读取. 如果你用了事务型 
                                                  Producer, 那么对应的 Consumer 就不要使用这个值
                                read_committed：表明 Consumer 只会读取事务型 Producer 成功提交事务写入的消息. 它也能看到
                                                非事务型 Producer 写入的所有消息
                                                
                                                
                    应用场景:
                        a. 事务更多用在 kafka Stream 中, 主要是实现流处理的精准一次.
                                                
                                                
            (3) 区别
                    事务性 Producer 比幂等性 Producer 更精准一次消息, 同时更消耗性能.
```

## 消费者
### 消费者参数
```shell
    1. enable.auto.commit 为 false, 表示不进行自动位移更新, 需要手动更新, 
    　　　　　　　　　　　　　如果新版本 consumer(0.9 以后)想要提交到 zookeeper, enable.auto.commit 设置为 false, 并且调用
     　　　　　　　　　　　　 zookeeper api.
     
                          为 true(默认值),  Consumer 在后台定期提交位移, 提交间隔由 auto.commit.interval.ms 参数控制
                          
    2. session.timeout.ms :  Coordinator 在 session.timeout.ms 秒之内没有收到 Group 下某 Consumer 实例的心跳, 就会认为这个 
                             Consumer 实例已经挂了, 推荐使用 6 秒, 这个值如果在 consumer group 不一致, 则取 consumer 最大的
    3. heartbeat.interval.ms : consumer 发送心跳的间隔, 推荐使用 2 秒
    4. max.poll.interval.ms : 默认是 5 分钟, 表示 Consumer 程序如果在 5 分钟之内无法消费完 poll 方法返回的消息, 那么 Consumer 
                             会主动发起“离开组”的请求, Coordinator 也会开启新一轮 Rebalance
    5. max.poll.records : poll 返回的数据量的值
    6. connection.max.idle.ms : 默认是 9 分钟, 用于自动关闭, 当该消费者连接持续 9 分钟没有数据通信时, kafka 自动关闭该连接.
    　　　　　　　　　　　　　　　　　　　　　　　
```
### 消费者组
```shell
    1. 概念
            (1) 消费者组(consumer group)
                    a. 消费者组可以有一个或则多个 consumer 实例, 实例可以是一个进程也可以是线程, 不过一般是进程比较多.
                    b. 在一个 Kafka 集群中, group id 是标识唯一的一个 Consumer Group.
                    c. 每个订阅主题下的每个分区的同一个消息只能有同一个消费者组的一个 consumer 实例进行消费
                    
            (2) kafka 的消费者组(Consumer Group)可以很好得解决点对点模型缺点(多个消费者争抢消息), 订阅/发布模型的缺点(每个订阅者
                都必须订阅主题下的所有分区), 具体的操作是消费者组(Consumer Group)订阅多个主题, 组内的每个实例不要求一定都要订阅主题的
                所有分区, 可以只消费部分分区中的消息.
                
            (3) 理想情况下, 消费者组(Consumer group)内的实例的数量等于该 Group 订阅所有主题的分区总数.
            (4) 消费者组(Consumer group)内的实例的数量主要看启动了多少个具有相同 group.id 的 consumer 实例.
            (5) Coordinator(协调者), 为 Consumer Group 服务, 负责为 Consumer Group 的 Rebalance, 位移管理和组成员的管理.
                consumer 在提交位移时, 是向 Coordinator 所在的 broker 进行提交位移. Consumer 启动时, 也是向 Coordinator 所在的
                Broker 发送各种请求, 然后由 Coordinator 负责执行消费者组的注册, 成员管理记录等元数据管理操作. 
                
                当 Consumer Group 完成 Rebalance 后, 每个 Consumer 实例都会定期地向 Coordinator 发送心跳请求. consumer 端
                参数 session.timeout.ms, Coordinator 在 session.timeout.ms 后每个收到 consumer 心跳, 则会认定该 consumer
                离线, 重新进行 Rebalance
                
            (6) 消费者组调用 KafkaConsumer.subscribe(), 独立消费者则调用的是 KafkaConsumer.assign() 
    2. Rebalance 
            (1) 概念
                    Rebalance 规定了消费者组(Consumer Group) 下所有消费者(Consumer)都能较公平得均分所有的分区数.
                    
            (2) 触发时机
                    a. 消费者组(Consumer Group)的消费者数量发生变化, 例如有新的消费者加入, 有消费者异常退出等.
                    b. 订阅主题数量发生变化, 消费者组(Consumer Group)通过正则表达式订阅主题, 如 
                       consumer.subscribe(Pattern.compile(“t.*c”)), 即订阅以 't' 开头, 以 'c' 结尾的主题.
                       如果新创建一个满足这样条件的主题, 那么该 Group 就会发生 Rebalance
                    c. 订阅主题下的分区数改变, kafka 目前只支持一次性增加一个主题的分区, 增加后会导致订阅该主题的所有 Group 进行
                       Rebalance. 
                       
            (3) 缺点
                    a. 进行 Rebalance 过程中, 所有消费组的消费者都停止消费.
                    b. 现在 Rebalance 策略是所有 Consumer 实例共同参与, 全部重新分配分区, 这样切换过程比较耗时, 最好能够复用原来的
                    　　分区.
                    c. 当 Group 主题和分区多, 消费者组的消费数量多时, Rebalance 时间很久.
                    
            (4) 避免 Rebalance
                    第一类避免 consumer 未能及时发送心跳, 导致 Consumer 被“踢出”Group 而引发的, 主要设置 session.timeout.ms = 6s 和 
                          heartbeat.interval.ms = 2s 的值, 最好超时时间是心跳的 3 倍.
                          
                    第二类避免 Consumer 消费时间过长导致的, 设置 max.poll.interval.ms 参数.
                        
                    
    3. Coordinator
            (1) 确定某个 Consumer Group 对应的 Coordinator 所在的 Broker
                    第一步: 根据 consume.id 计算出其哈希值, 再取模位移分区数(__consumer_offsets, consumer group 的配置信息保存在
                    　　　　位移主题中.), 得到对应的位移分区序号
                    第二步: 再找出位移主题分区需要的 Leader 副本在哪个 Broker 上就可以
                    
```

### 位移
```shell
    1. 概念
            (1) 在消费者组(Consumer Group)中, 位移体现在 KV 中, <group_id + 主题 + 分区, 该分区下的位移>
            (2) 新老版本的 Consumer 的位移管理
            
                    老版本 Consumer 的位移管理,是将位移的存储保存在 Zookeeper 中, 使得 kafka 内部尽量的无状态. 但 Zookeeper 本身
                    不适用频繁写/更新操作, 这会极大减低 Zookeeper 集群的性能.
                    
                    新版本 Consumer 的位移管理, 是将位移保存在 kafka 内部定义的主题(topic), 这个 topic 叫 "__consumer_offsets",
                    将消费者的位移数据作为 kafka 的消息提交到 "__consumer_offsets" 的 topic 中.
                    
            (3) __consumer_offsets 就是个内部定义的主题, 这个主题具备特定的格式的消息, producer 不能往该主题上发送其他格式的消息, 
               会导致 broker 崩溃.
               
            (4) Consumer 需要为分配给它的每个分区提交各自的位移数据
    2. 消息格式
            第一种(最常用):
                消息格式由一个个 KV 组成
                    key : group_id + 主题(topic) + 分区
                    value:　位移值, 时间戳等
                     
            第二种:
                消费者组(Consumer Group) 的信息, 用来注册  Consumer Group 
                
            第三种:
                tombstone 消息(墓碑消息, delete mark), 当某个 Consumer Group 下的所有 Consumer 实例都停止了, 而且它们的位移数据
                都已被删除时, Kafka 会向位移主题的对应分区写入 tombstone 消息(null 空消息), 表明要彻底删除这个 Group 的信息
                
    3. 位移主题
            (1) kafka 集群中第一个 consumer 程序启动时, 其位移主题自动创建, 该主题默认的分区数 50(offsets.topic.num.partitions 参数),
                默认副本数 3 (offsets.topic.replication.factor 参数)
                
            (2) kafka 可以进行手动创建位移, 调用 kafka api 进行, 但不推荐, 老的版本 kafka 源码中硬代码编写 50 的分区数.
            
    4. 位移提交
            (1) 自动提交
                    a. 将 enable.auto.commit 参数设置为 true, 定期隔 auto.commit.interval.ms 时间提交位移量. 只要是 consumer 
                        一直启动者, 就会不断隔一段时间向位移主题发送位移消息(不管 kafka 中有没有收到新的消息)
                        
                    b. 缺点
                            存在重复消费数据, 在 consumer 已经消费了最新的数据, auto.commit.interval.ms 提交位移周期还没有到时, 
                            consumer 重启了或则发生 Rebalance , 此时对应的位移主题还没有更新, 所以还会消费到相同的数据.
                 
            (2) 手动提交
                   a.  enable.auto.commit 参数设置为 false, 程序自己手动调用 consumer api(例如 consumer.commitSync, 这个会提交
                       poll() 返回的最新位移)
                       
                       // commitSync 同步提交(阻塞, 影响 tps, 支持自动错误重试)
                       while (true) {
                           ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
                           process(records); // 处理消息
                           try 
                           {
                                consumer.commitSync();  // 同步提交
                           } 
                           catch (CommitFailedException e) 
                           {
                                handle(e); // 处理提交失败异常
                           }
                       }
                       
                       
                       // commitAsync 异步提交(不阻塞, 但由于是异步重试没意义)
                       while (true) {
                           ConsumerRecords<String, String> records =　consumer.poll(Duration.ofSeconds(1));
                           process(records); // 处理消息
                           consumer.commitAsync((offsets, exception) -> 
                                {
                                    if (exception != null)
                                    handle(exception);
                                }
                                );
                       }
                       
                       // commitSync 和　commitAsync 结合使用, 正常情况下都是执行异步位移提交, 但在 Consumer 要关闭前,
                       // 调用 commitSync() 方法执行同步阻塞式的位移提交, 以确保 Consumer 关闭前能够保存正确的位移数据
                       
                       try 
                       {
                           while (true) 
                           {
                               ConsumerRecords<String, String> records =　consumer.poll(Duration.ofSeconds(1));
                               process(records); // 处理消息
                               consumer.commitAysnc(); // 使用异步提交规避阻塞
                           }
                       } 
                       catch (Exception e) 
                       {
                            handle(e); // 处理异常
                       } 
                       finally // 当 while 退出就会执行
                       {
                           try 
                           {
                                consumer.commitSync(); // 最后一次提交使用同步阻塞式提交
                           } 
                           finally 
                           {
                                consumer.close();
                           } 
                       }
                      
                   b. 不使用 poll 返回的消息总数进行位移提交
                      　　　通过　commitSync(Map<TopicPartition, OffsetAndMetadata>) 和　
                      　　　commitAsync(Map<TopicPartition, OffsetAndMetadata>)
                                TopicPartition: 消费的分区
                                OffsetAndMetadata: 位移数据
                   
                       
                       
            (3) 由于自动提交位移, 位移主题的消息会不断堆积, 导致磁盘爆满, 所以 kafka 采用 Compact 策略, 定期删除过期的数据
                (各个主题下相同的 key), 提供了专门的后台线程(Log Cleaner)定期地检查待 Compact 的主题, 
                看看是否存在满足条件的可删除数据
                
    5. CommitFailException 异常
            (1) 场景一: 消息处理的总时间超过预设的 max.poll.interval.ms 参数值
            
                        …
                        Properties props = new Properties();
                        …
                        props.put("max.poll.interval.ms", 5000);
                        // 订阅一个 "test-topic" 主题
                        consumer.subscribe(Arrays.asList("test-topic"));
                        
                        while (true) 
                        {
                            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
                            // 使用 Thread.sleep 模拟真实的消息处理逻辑
                            Thread.sleep(6000);  // 超过 5 秒, 抛出 CommitFailException 异常
                            consumer.commitSync();
                        }
                        
                      解决方案
                            第一种: 优化代码, 缩短单体消息的处理时间
                            第二种: 增大 max.poll.interval.ms 参数的值, 默认是 5 分钟
                            第三种: 减少 max.poll.records 的值(默认是 500), 这时调用一次 consumer.poll 方法返回的消息数据量
                            第四种: 使用多线程来增强消费的能力, 但是会存在多线程间处理位移提交的问题.
                            
            (2) 场景二: 消费者组(consumer group)和独立消费者(standalone consumer) 同时设置相同的 group.id 就会抛出　
            　　　　　　　CommitFailException 异常
```

### 多线程消费者
```shell
    1. 概念
            (1) KafkaConsumer 类不是线程安全的, 多个线程要互斥使用同一个 KafkaConsumer 实例
         
    2. 方案一: 多线程 + 多 KafkaConsumer 实例
            
            优点:
                a. 程序方便实现, 在每个线程创建专属的 KafkaConsumer 实例就可以
                b. 线程间无交互
                c. kafka 主题中的每个分区只被一个线程处理(进行消息的获取, 处理), 容易实现分区内的消息消费顺序.
                
            缺点:
                a. 占用更多资源, 每个线程都需要占用内存, cpu 等资源
                b. 线程数受限于主题分区数, 扩展性差
                c. 线程本身处理消息容易超时(业务逻辑重), 引发 Rebalance 
                 
    3. 方案二:  KafkaConsumer 线程池 + 消息处理池
            
            优点:
                a. 可独立扩展消费获取线程数和消息处理池
                b. 伸缩性好
                
            缺点:
                a. 实现难度高
                b. 难以维护分区内的消息消费顺序, 由于有消息处理池的存在, 先发送的消息不一定先处理
                c. 处理链路长, 不易位移提交管理
                
```

### 消费者管理 tcp 连接
```shell
    1. 概念
    2. 消费者创建 tcp
            (1) 和生产者不同, 构建 KafkaConsumer 实例时是不会创建 TCP 连接的, 而是调用 KafkaConsumer.poll 方法时被创建, 而
                KafkaConsumer.poll 内部有 3 种情况进行 tcp 创建.
                
            (2) 情况一: 发起 FindCoordinator 请求
                          消费者程序在首次启动 poll 方法时, 向 kafka 集群中的某个 broker 请求 FindCoordinator, 获取哪个 broker
                       是这个消费者的协调者.
                       
            (3) 情况二: 连接协调者时
                            消费者收到 broker 的 FindCoordinator 响应报文, 获取到协调者信息(broker 信息), 再向协调进行连接.
                       只有成功连入协调者, 协调者才能开启正常的组协调操作, 比如加入组、等待组分配方案、心跳请求处理、位移获取、
                       位移提交等
                       
            (4) 情况三: 消费数据
                            如果消费者需要消费多个主题下的分区, 则需要连接各个分区所在的 broker
                            
            (5) 生产者连接流程
                    第一步: 先连接任何一个 broker, 发送元数据请求以获取整个集群的信息, 再发送 FindCoordinator 请求, 
                    　　　　获取协调者所在的 Broker 信息 
                    第二步: 再发起第二个 tcp 连接, 连接协调者, 令其执行组成员管理操作
                    第三步: 又分别创建了新的 TCP 连接, 主要用于实际的消息获取, 要消费的分区的 leader 副本在哪台 Broker 上, 消费者
                           就要创建连向哪台 Broker 的 TCP
                           
    3. 消费者关闭 tcp
            (1) 主动关闭
                    手动调用　KafkaConsumer.close() 方法, 或者是执行 Kill 命令
            (2) 自动关闭
                    由消费者端参数 connection.max.idle.ms　控制的(默认是 9 分钟), 某个 Socket 连接上连续 connection.max.idle.ms
                都没有任何请求, 自动断开该连接.
                
                
            一般情况三获取数据连接和情况二与协调者连接会保留, 情况一的连接会自动关闭.
```

### 消费者组消费进度监控
```shell
    1. 概念
            (1) 消费者 Lag: 消费者消费滞后程度. 如 Kafka 生产者向某主题生产 100 万条消息, 而消费者当前消费 80 万条消息, 则 Lag
                           为 20 万条.
            (2)  Lead 值:  指消费者最新消费消息的位移与分区当前第一条消息位移的差值, 如果消费者消费很慢, lead 就会越来越小, 接近于
                           0 时, 就要注意还没消费的数据要被删除了.
    2. 方法
            (1) 方法一: kafka 自带的命令(kafka-consumer-groups 脚本)
                       > bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092(kafka 连接信息) --describe 
                         --group testgroup(group_id)
                         
                         --partition参数 // 独立消费者需要加上
                         
            (2) 方法二: Kafka Java Consumer API(Kafka 2.0.0 之后的)
                        
                       简介:
                            用程序的方式自动化监控, 
                            
                       第一步: 调用 AdminClient.listConsumerGroupOffsets 方法获取消费者组的最新消费消息的位移 consumer_offset
                              (Kafka 2.0.0 之后有) 
                              
                       第二步: 调用 consumer.endOffsets() 获取订阅分区的最新消息位移 msg_offset
                       第三步: 做差值 msg_offset - consumer_offset
                       
            (3) 方法三: Kafka JMX 监控指标
                        kafka.consumer:type= consumer-fetch-managermetrics, partition=“{partition}”,
                                             topic=“{topic}”, client-id=“{client-id}”                    
```
