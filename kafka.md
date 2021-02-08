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
                0.11: 目前最主流的版本之一, 可以更新到 0.11.0.3 
                        (1) 提供了幂等性 Producer API 以及事务 API
                        (2) 对 Kafka 消息格式做了重构
                        (3) 消费者程序将心跳单独拉一个线程
                        (4) 重构了控制器底层设计, 将多线程的方案改成单线程加事件队列的方案
                        (5) zookeeper 与控制器之间有同步调用改为异步调用. 
                        (6) 引入 Leader Epoch, 来规避因高水位更新错配导致的各种不一致问题
                        (8) 严格遵守了 offsets.topic.replication.factor 的设置, 如果当前运行的 Broker 数量小于
                            offsets.topic.replication.factor 值, Kafka 会创建主题失败, 并显式抛出异常. 而 0.11 之前是
                            根据运行 broker 数量和 offsets.topic.replication.factor 值取最小值.
                        (9) 实现通过命令行形式重设位移.
                1.0 和 2.0 : 主要还是针对 Kafka Streams 进行改进.
                1.1
                        (1) 引入了动态参数配置, 无需重启 broker
                        
                2.2 : 在 broker 中将控制器请求命令和普通数据请求分优先级, 删除某个主题, 控制器给该主题所有副本所在的 Broker 发送
                      StopReplica的请求会被优先处理.
                
            (3) 建议尽量保持服务器端版本和客户端版本一致
            
    8. kafka 与传统的消息中间件(ActiveMQ, RabbitMQ) 区别是, 传统消息中间件处理消息是破坏性的, 一旦处理成功, 消息就会从 broker 删除.
       而 kafka 记录消息的底层是日记文件形式, 通过位移偏移访问, 可以重复读数据.
       适用场景:
                传统消息中间件: 不关心消息的顺序
                kafka: 需要较高的吞吐量, 但每条消息的处理时间很短, 并且需要关注消息的顺序.
                
    9. 可视化 kafka 管理工具 : kafka manager
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
                    
                unclean.leader.election.enable(建议设置为 false)
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
                      　--partitions 1 
                        --replication-factor 1
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
    3. offsets.retention.minutes : 其保留位移信息时间, 组状态在 empty 的时候, 删除位移信息
    4. num.network.threads, 网络线程池的数量, 默认是 3 , 表示每台 Broker 启动时会创建 3 个网络线程, 
    　　　　　　　　　　　　　　专门处理客户端发送的请求, 发送到共享队列中
    　　　　　　　　　　　　　　
    5. num.io.threads , IO 线程池的线程数据, 默认 8, 表示每台 Broker 启动后自动创建 8 个 IO 线程处理请求, 专门处理实际的请求逻辑,
                        从共享队列中取出请求.
```

## 生产者

### 生产者参数
```shell
    1. compression.type, 指定压缩类型, GZIP, Snappy 和 LZ4, zstd
    2. acks = all(-1), 已提交的定义, 默认值是 1, 代表只要 leader 副本数据被写入, 则生产者认定已提交.
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
                             每次 consumer 发送心跳时会顺带发送 session timeout 时间, 这样 Coordinator 收到后
                             会根据这个 session timeout 时间计算下次 deadline 时间, 如果过了 deadline 还没有收到直接 fail
                             掉该 consumer
    3. heartbeat.interval.ms : consumer 发送心跳的间隔, 推荐使用 2 秒, 这个时间间隔也体现重平衡的时效性
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
                    a. Rebalance 规定了消费者组(Consumer Group) 下所有消费者(Consumer)都能较公平得均分所有的分区数.
                       每次消费者组启动时或则组内消费者实例增加, 必然会触发重平衡过程
                    
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
                    
    3. Rebalance 底层实现
            (1) 重平衡的通知机制
                    如果需要消费者租的所有消费者进行重平衡, 协调者需要通过心跳响应报文中加上 "REBALANCE_IN_PROGRESS" 给各个消费者组内
                 的所有消费者. 
                 
                 kafka 0.11.1.0 之前 消费者心跳定期发送是在消费者主线程中完成的, 这会导致如果主业务逻辑过重, 
                 心跳没法定时准确发送, 导致协调者误判, 认为该消费者下线了. 
                 
                 kafka 0.10.1.0 后, 心跳单独拉一个线程, 按 heartbeat.interval.ms 间隔发送.如果有重平衡通知,会从心跳响应报文中获知.
                 
            (2) 消费者组状态机
                    a. 含义
                           Empty: 消费者组内没有任何成员, 但消费者组可能存在已提交的位移数据, 并且这些位移还没有过期.
                           Dead: 消费者组内没有任何成员, 组的原数据信息(类似于注册信息)已经在协调者中被移除
                           PreparingRebalance(JoinGroup 请求 过程) : 消费者组准备开始重平衡, 此时所有成员都要重新请求加入消费者组.
                           CompletingRebalance(SyncGroup 请求过程): 消费者组下所有成员已经加入, 各个成员正在等待分配方案,
                                                                   老版本叫 AwaitingSync
                           Stable : 消费者组的稳定状态, 表明重平衡已经完成, 组内成员能够正常消费数据.
                           
                    b. 流转
                            一个消费者组最开始是 Empty 状态, 当重平衡过程开启后, 它会被置于 PreparingRebalance 状态等待成员加入, 
                            有成员入组变更到 CompletingRebalance 状态等待分配方案, 最后流转到 Stable 状态完成重平衡.
                            
                            当在 CompletingRebalance/ Stable 状态时, 有成员离开或则有新成员加入, 就会被切换到 
                            PreparingRebalance 状态, 这时所有现存成员必须重新申请加入组.
                            
                    c. 只有 Empty 状态下的组, kafka 才会定期执行过期位移删除的操作
                    
            (3) 消费者端重平衡流程
                    第一步: JoinGroup 请求, 作用是将组成员订阅信息发送给 leader 消费者, 每个加入消费者组的消费者会将自己的订阅主题
                           上报给协调者, 协调者收集所有消费者的主题信息后, 选出 leader 消费者(第一个发送 JoinGroup 的消费者),
                           并将所有成员订阅主题信息作为响应报文给 leader 消费者. 同时剩余消费者响应报文中是带 leader 消费者信息,
                           让它们保存.
                           
                    第二步: SyncGroup 请求, 所有消费者都向协调者发送 SyncGroup 请求, 只不过 leader 消费者 SyncGroup 请求中
                           带有分配方案. 其他消费者 SyncGroup 请求内容为空, 协调者根据分配方案, 分别向对应的消费者发送各自的消费分配.
                           当所有成员都成功接收到分配方案后, 消费者组进入到 Stable 状态, 即开始正常的消费工作
                           
            (4) broker 端重平衡流程
                    场景一: 新成员入组
                                当所有消费者组处于 stable 状态后, 有新的成员加入了消费者组(该消费者发送了 JoinGroup 请求), 
                           协调者检测到, 则通过心跳请求响应的方式通知组内现有的所有成员, 强制它们开启新一轮的重平衡(所有消费者组
                           重新发送 JoinGroup 请求, 之后 SyncGroup )
                            
                    场景二: 组成员主动离组
                                当消费者组某个消费实例主动调用 consumer.close() 时, 会向协调者发送 LeaveGroup 请求, 协调者收到后,
                            则通过心跳请求响应的方式通知组内现有的所有成员, 强制它们开启新一轮的重平衡(所有消费者组重新发送
                            JoinGroup 请求, 之后 SyncGroup )
                            
                    场景三: 组成员崩溃离组
                                当消费者组某个消费实例由于某个原因崩溃了, 则协调者无法立即感知到, 需要等待消费者端的 
                           session.timeout.ms 后才能感知到, 之后进行重平衡流程
                           
                    场景四: 
                        
                            
                    
    4. Coordinator
            (1) 概念
                    a. 协调者组件中保存当前向它注册过的所有组的信息.
            (2) 确定某个 Consumer Group 对应的 Coordinator 所在的 Broker
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

### 测试
```shell
    1. 模拟一个重分区 hang 住的,  在 reassign 的过程中删除 topic
```

## kafka 内核

### kafka 副本机制
```shell
    1. 概念
        (1) 分区下副本是不能对外提供服务(包括读写), 只能从 leader 副本中异步拉取消息, 写入到自己的提交日志, 保持与 leader 副本的同步.
        (2) 当 leader 副本宕机时, zookeeper 会实时监控到, 并进行 leader 选举, 将 follower 副本选出 leader 副本, 老 Leader 副本
            重启后, 只能作为 follower 副本加入到集群中
            
        (3) ISR: In-sync Replicas, 同步副本, 看是否属于同步条件, 是根据 broker 的参数 replica.lag.time.max.ms(默认是 10s), 
                 代表如果 follower 副本与 leader 副本同步的时间不超过 replica.lag.time.max.ms, 则代表 ISR.
                 ISR 副本集合中肯定包含 leader 副本
                 
        (4) 非同步副本: 所有不在 ISR 中的存活副本
                 
            
    2. 副本不提供任何服务(包括读写)
            好处:
                    (1) 方便实现 “Read-your-writes", 可以读取最新的写入消息, 没有延迟.
                    (2) 容易实现单调读一致性, 由于各个副本同步存在延迟, 如果副本提供读服务, 则 副本 1 读到了老数据, 副本 2 又读到了
                    　　新数据, 多次读不一致.
                    
    3. Unclean 领导者选举
            代表 ISC 集合中没有可用的副本, 只能在非同步副本选举出 leader 副本, 但非同步副本落后 Leader 太多, 这导致选出来的 leader
            落后太多.
            
            Broker 端参数 unclean.leader.election.enable 控制是否允许 Unclean 领导者选举
```

### kafka 请求流程
```shell
    1. 概念
            (1) 请求协议
                    数据类请求
                        produce 请求是用于生产消息
                        fetch 请求是消费消息
                        metadata 请求是用于请求 kafka 集群元数据信息
                        
                    控制类请求
                        LeaderAndIsr : 更新 Leader 副本, Follower 副本以及 ISR 集合
                        StopReplica: 停止副本
                    
                    
    2. kafka broker 请求处理
            (1) 处理模型是 Reactor 模型, 有一个 acceptor 线程进行请求接受, 并将其分发到网络线程池中. 这个线程池的线程个数由
                broker 的 num.network.threads 决定的. Acceptor 线程采用轮询的方式将请求发到所有网络线程中.
                
            (2) 请求被转发到网络线程中, 不是自己处理, 而是将请求放到共享请求队列中. Broker 端还有个 IO 线程池, 负责从该队列中取出请求,
                执行真正的处理. 如果是 PRODUCE 生产请求, 则将消息写入到底层的磁盘日志中；如果是 FETCH 请求, 则从磁盘或页缓存中读取消息
                IO 线程池才是真正处理请求逻辑的线程, 线程池个数由 broker 的 num.io.threads 参数(默认是 8 个)
                
            (3) 当 IO 线程处理完请求后, 会将响应报文发送到网络线程对应的响应队列中, 然后由对应的网络线程负责将 Response 返还给客户端
                注意: 保存请求队列是网络线程池共享的,　而请求响应报文则是放到对应网络线程各自的响应队列中.
                
            (4) 其中还有个 Purgatory(炼狱)组件, 用来缓存延迟请求, 即不能立刻处理该请求, 需要满足一定条件.如设置 acks=all 的 
                PRODUCE 请求, 该请求必须等待 ISR 中所有副本都接收消息后才能返回, 此时处理该请求的 IO 线程就必须等待其他 Broker 的
                写入结果. 当请求不能立刻处理时, 它会暂存在 Purgatory 中, 之后一旦满足完成条件，　IO 线程会继续处理该请求,并将 Response
                放入对应网络线程的响应队列中
                
    3. 问题
            (1) 在 kafka 2.3 版本以前, 数据类请求和控制类请求都是统一的对待处理, 这样 控制类请求没法及时得进行处理, 会导致数据无用, 
                例如如果 broker 0 中是 leader 副本, 现在堆积大量的数据类请求, 例如 produce 请求, 并且 acks=all, 虽然 broker 0
                成功写入了数据, 但来了一个控制类请求 LeaderAndIsr , 要变更 leaders 副本, 让 broker 1 称为 leader 副本, 则
                执行显式的日志截断(Log Truncation, 即原 Leader 副本成为 Follower 后, 会将之前写入但未提交的消息全部删除）
                
            (2) kafka 2.4 会分开处理, 数据类请求 和 控制类请求都有各自一套 (网络线程池 + 共享队列 + IO 线程池)
```


### kafka 控制器(Controller)
```shell
    1. 概念
            (1) Controller 组件在 zookeeper 帮助下管理和协调 kafka 集群. 同时 kafka 集群中只有一个 broker 称为 controller.
            (2) zookeeper
                    a. 使用的数据模型类似于文件系统的树形结构, 根目录也是以“/”开始. 该结构上的每个节点被称为 znode, 用来保存一些
                       元数据协调信息
                    b. znode 可分为持久性 znode 和临时 znode. 持久性 znode 不会因为 ZooKeeper 集群重启而消失, 而临时 znode 则与
                       创建该 znode 的 ZooKeeper 会话绑定, 一旦会话结束, 该节点会被自动删除.
                       
                    c. zookeeper 的客户端可以感知到监控 znode 节点的变更(包括删除, 子节点的数据变化, znode 保存的内容)
                    d. ZooKeeper 常被用来实现集群成员管理, 分布式锁, 领导者选举等功能
                    
    2. 控制器的创建
            (1) broker 启动时都会向 zookeeper 创建 /controller 节点, 第一个成功创建 /controller 节点的 broker 是 controller.
            
    3. 控制器的作用
            (1) 主题管理
                    对 Kafka 主题的创建, 删除以及分区增加的操作, kafka-topics 脚本的执行就是控制器在做的.
                    
            (2) 分区重分配
                    对已存在的主题分区进行细粒度的分配功能, 对应与 kafka-reassign-partitions 脚本.
                    
            (3) Preferred 领导者选举
                    避免部分 Broker 负载过重而提供的一种换 Leader 的方案
                    
            (4) 集群成员管理
                    自动检测 broker 的新增, broker 主动关闭, broker 的宕机. 这种自动检测是依赖于 Watch 功能和 ZooKeeper 临时节点
                组合实现的
                    broker 的新增是控制器组件利用 Watch 机制检查 ZooKeeper 的 /brokers/ids 节点下的子节点数量变更.当有新 Broker 
                启动后, 会在 /brokers 下创建专属的 znode 节点. 一旦创建完毕, ZooKeeper 会通过 Watch 机制将消息通知推送给控制器,  
                控制器就能自动地感知到这个变化, 进行新增 Broker 处理.
                    检测 broker 是否存活, 是通过临时节点, 每个 Broker 启动后, 会在 /brokers/ids 下创建一个临时 znode, 如果
                Broker 异常了, 则与 zookeeper 会话断开连接, 临时 node 也会被删除, 则 watch 机制也会通知到控制器.
                
            (5) 数据服务
                    向其他 Broker 提供数据服务. 控制器上保存了最全的集群元数据信息, 定期向其他的 Broker 会发送最新元数据. 
                    
                    控制器组件保存的内容有 
                            所有主题信息, 包括具体的分区信息, 如 leader 副本信息, ISR 集合中有哪些副本等.
                            所有 Broker 信息, 包括当前都有哪些运行中的 Broker, 哪些正在关闭中的 Broker 等
                            所有涉及运维任务的分区, 包括当前正在进行 Preferred 领导者选举以及分区重分配的分区列表
                            
                    这些数据在 ZooKeeper 中也保存一份, 每当控制器初始化时, 会从 ZooKeeper 上读取对应的元数据并填充到自己的缓存中.
                    有了这些数据, 控制器就能对外提供数据服务. 对外主要是指对其他 Broker 而言, 控制器通过向这些 Broker 发送请求的方式
                    将这些数据同步到其他 Broker 上
                    
    4. 控制器故障转移(Failover)
            (1) 当运行中的控制器突然宕机或意外终止时, ZooKeeper 通过 Watch 机制感知到并删除了 /controller 临时节点.
                所有存活的 Broker 开始竞选新的控制器身份, 最终某一个 broker 成功在 zookeeper 创建了 /controller 临时节点,
                在初始化时从 ZooKeeper 中读取集群元数据信息, 并初始化到自己的缓存中, 之后对外提供服务.
                
    5. 实践
            (1) 当主题无法删除, 或者重分区 hang 住, 可能是控制器出问题了, 可以先在 zookeeper 中手动删除该 /controller 临时节点.
                 rmr /controller, 使其进行重新选出控制器.
```

### 高水位
```shell
    1. 概念
            (1) 高水位(HW high Watermark), 用位移表示, 分区高水位以下的消息( 0 <= 位移 < HW)被认为是已提交消息, 反之就是未提交消息.
                                          位移值等于高水位的消息也属于未提交消息, 消费者只能消费已提交的消息.
            (2) 高水位作用
                    a. 定义消息可见性, 用来标识分区下的哪些消息是可以被消费者消费的
                    b. 帮助 Kafka 完成副本同步
                    
            (3) 日志末端位移(Log End Offset, LEO), 表示未来下一条消息要写入的位移值. 介于高水位和 LEO 之间的消息就属于未提交消息
            (4) 每个副本的各个分区都有高水位, 一般分区的高水位就泛指 leader 副本的分区的高水位.
            (5) kafka 利用 LEO, 高水位来进行 leader 副本和　follower 副本同步.
            (6) 副本重启时, 会执行日志截断操作, 会将 LEO 初始化为 HW 的值.
            
    2. 高水位更新机制
            (1) 在 Leader 副本所在的 Broker_leader 上, 还保存了其他 Follower 副本的 LEO 值, 这些保存在 Broker_leader 的
                follower 副本被称为远程副本.
            (2) 更新时机
                     前提: Broker_leader 为保存 leader 副本的 broker, 既保存了 leader 副本的高水位和 LEO, 又保存其他 follower
                           副本的 LEO
                           
                          Broker_follower 只保存了自己的 follower 副本的高水位和 LEO
                          
                          
                     a. Broker_follower 上的 follower 副本 LEO 被更新, 时机在 follower 副本从 leader 副本拉取消息, 写到本地磁盘
                        时, 会更新 follower 副本的 LEO
                        
                     b. Broker_leader 上的 leader 副本 LEO 被更新, 时机在 leader 副本接收到生产者发送的消息, 写入到本地磁盘后,
                        更新 leader 副本的 LEO
                        
                     c. Broker_leader 上的远程副本 LEO 被更新, 时机在 follower 副本从 leader 副本拉取消息时, 会告诉 leader 副本
                        从哪个位移处开始拉取. 那么 leader 副本会使用这个位移值更新对应远程副本的 LEO
                         
                     b. Broker_follower 上的 follower 副本高水位被更新, 时机在 follower 副本成功更新完 LEO 后, 会比较 LEO 和
                        leader 副本发来的高水位值(当 leader 副本的高水位被更新时, leader 副本会发送各个 follower 副本),
                        并用两个的较小值更新自己的高水位.
                        
                     d. Broker_leader 上的 leader 副本高水位被更新, 有 2 个更新时机, 第一个是 leader 副本更新其 LEO 后. 第二个是
                        更新完远程副本 LEO 后. 高水位的取值是 leader 副本和所有与 leader 同步的远程副本 LEO 的最小值.
                        
                        
            (3) 
                    a. 从 Leader 副本纬度
                            
                            处理生产者请求的逻辑如下：
                                1. 将消息写入到本地磁盘
                                2. 更新分区高水位值
                                    i. 获取 Leader 副本所在 Broker 端保存的所有远程副本 LEO 值{LEO-1, LEO-2, ……, LEO-n}
                                    ii. 获取 Leader 副本 LEO
                                    iii. 更新 currentHW = min(LEO, LEO-1, LEO-2, ……, LEO-n)
                                    
                            
                            处理 Follower 副本拉取消息的逻辑如下：
                                1. 读取磁盘(或页缓存)中的消息数据
                                2. 使用 Follower 副本发送请求中的位移值更新远程副本 LEO 值
                                3. 更新分区高水位值
                                
                                
                    b. 从 follower 副本纬度
                    
                            从 Leader 拉取消息的处理逻辑如下：
                                1. 将消息写到本地磁盘
                                2. 更新 LEO 值
                                3. 更新高水位值
                                    i. 获取 Leader 发送的高水位值：currentHW。
                                    ii. 获取 follower 副本更新过的 LEO 值：currentLEO
                                    iii. 更新高水位为 min(currentHW, currentLEO)
                                    
            (4)  副本同步机制
                    第一步:初始化
                            leader 副本, HW = 0, LEO = 0, remote LEO = 0
                            follower 副本, HW = 0, LEO = 0
                            
                    第二步:生产者发送一条消息
                            leader 副本, HW = 0, LEO = 1, remote LEO = 0
                            follower 副本, HW = 0, LEO = 0
                                    
                    第三步: follower 副本拉取消息(向 leader 副本请求位移为 0)
                                leader 副本, HW = 0, LEO = 1, remote LEO = 0
                                follower 副本, HW = 0, LEO = 1
                                
                    第四步: follower 副本拉取消息(向 leader 副本请求位移为 1)
                                leader 副本, HW = 1, LEO = 1, remote LEO = 1(此时　remote LEO　被更新为 1, HW 也被更新为 1)
                                follower 副本, HW = 0, LEO = 1
                                
                    第五步: leader 副本将分区高水位同步给 follower 副本
                            leader 副本, HW = 1, LEO = 1, remote LEO = 1
                            follower 副本, HW = 1, LEO = 1
                            
                           
    3. Leader Epoch
            (1) 背景
                    follower 副本的高水位更新需要一轮额外的拉取请求才能实现, 如果有多个 Follower 副本, 需要拉取多次请求, 这样
                 导致 leader 副本的高水位和 follower 副本的高水位存在不一致的情况, 使得数据不一致, 在这中间如果 broker 宕机, 
                 就会导致数据丢失.
                 
                 数据丢失情况:(条件是 min.insync.replicas 为 1)
                      步骤一:
                        生产者向 leader 写入 2 条消息, leader 副本和 follower 副本都写入 2 条消息(2 个 LEO 都为 2), 在
                 Leader 副本的高水位也已经更新(leader_HW = 2, leader_LEO = 2, leader_remote LEO = 2), 但 Follower 
                 副本高水位还未更新(follower_HW = 1, follower_LEO = 2), 这时 follower 副本宕机了, 重启 follower 副本时, 
                 执行日志截断操作, 将 follower_LEO 调整为 follower_HW 的值, 即　follower_LEO = 1. 此时原来位移为 1 的消息就会
                 被丢弃.
                      步骤二:
                         当 follwer 副本刚要向 leader 副本拉取消息时, leader 副本也宕机了, 此时只能将 follower 上升为 leader 副本,
                 这时原 leader 重启了, 要向现在 leader 副本拉取消息, 会告知现在 leader 副本 HW 为 1, 原来 leader 副本的 HW 也会被
                 更新 1, 数据丢失.
                 
                 
            (2) Leader Epoch 由 2 部分组成 <版本号, 起始位移>
                    第一部分: Epoch. 单调增加的版本号. 每当副本领导权发生变更时, 都会增加该版本号. 小版本号的 Leader 被认为是过期
                             Leader, 不能再行使 Leader 权力
                    第二部分: 起始位移(Start Offset). 切换成 Leader 副本后首条消息的位移.
                    
                    例如, 两个 Leader Epoch<0, 0> 和 <1, 120>, 第一个 Leader Epoch 表示版本号是 0, 这个版本的 Leader 从
                    位移 0 开始保存消息, 一共保存了 120 条消息. 之后 Leader 发生了变更, 版本号增加到 1, 新版本的起始位移是 120
                    
                    Kafka Broker 会在内存中为每个分区都缓存 Leader Epoch 数据, 同时会定期地持久化到一个 checkpoint 文件中.
                    当 Leader 副本写入消息到磁盘时,如果该 Leader 副本是首次写入消息, 那么 Broker 会向缓存中增加一个
                    Leader Epoch 条目, 否则就不做更新. 每次有 Leader 变更时, 新的 Leader 副本会查询这部分缓存, 
                    取出对应的 Leader Epoch 的起始位移, 以避免数据丢失和不一致的情况
                    
            (3) Leader Epoch 解决方案：
                    场景和之前大致是类似的, 在 Follower 副本 B 重启回来后, 需要向 A 发送一个特殊的请求去获取 Leader 的 LEO 值.
                    当获知到 Leader LEO = 2 后, B 发现该 LEO 值不比它自己的 LEO 值小, 而且缓存中也没有保存任何起始位移值 > 2 的 
                    Epoch 条目, 因此 B 无需执行任何日志截断操作. 之后副本 A 宕机, B 成为 Leader. 当 A 重启回来后, 
                    执行与 B 相同的逻辑判断, 发现也不用执行日志截断, 至此位移值为 1 的那条消息在两个副本中均得到保留
                    后面当生产者程序向 B 写入新消息时, 副本 B 所在的 Broker 缓存中, 会生成新的 Leader Epoch 条目：
                    [Epoch=1, Offset=2]. 副本 B 会使用这个条目帮助判断后续是否执行日志截断操作
                            
```

## kafka 管理与监控
### 主题管理
```shell
    1. 概念
            (1) 内部主题 __consumer_offsets 用于位移提交, __transaction_state 主题用于事务. 这 2 个主题默认是 50 分区.
    2. 主题日常管理
            (1) 创建主题
                    > bin/kafka-topics.sh --bootstrap-server broker_host:port --create --topic my_topic_name  
                      --partitions 1 --replication-factor 1
                      
                    --bootstrap-server: kafka 2.2 版本后推荐使用, 不推荐使用 --zookeeper, 因为这个参数绕过了
                                        安全认证(安全认证可以限制主题的创建),  
                    --partitions : 代表分区数
                    --replication-factor : 分区下的副本数, 包含 leader 副本和 follower 副本
                    
            (2) 查询主题列表
                    >  bin/kafka-topics.sh --bootstrap-server broker_host:port --list
                    
            (3) 查询某个主题详细内容
                    > bin/kafka-topics.sh --bootstrap-server broker_host:port --describe --topic <topic_name>
                    
            (4) 修改主题分区
                    目前 kafka 不支持减少某个主题的分区, 支持分区增加.
                    > bin/kafka-topics.sh --bootstrap-server broker_host:port --alter --topic <topic_name> 
                                          --partitions < 新分区数 >
                                          
            (5) 修改主题级别层级的参数
                    bin/kafka-configs.sh --zookeeper zookeeper_host:port 
                    　　　　　　　　　　　　--entity-type topics --entity-name <topic_name> 
                    　　　　　　　　　　　　--alter --add-config max.message.bytes=10485760
                    
            (6) 变更副本数
                     例如创建 __consumer_offsets 主题(默认是 50 分区) 增加为 3 个副本
                        第一步: 创建一个 json 文件, 显式提供 50 个分区对应的副本数. replicas 中的 3 台 Broker 排列顺序不同,
                               目的是将 Leader 副本均匀地分散在 Broker 上. 该文件具体格式如下
                                    {
                                        "version":1, 
                                        "partitions":
                                        [
                                            {"topic":"__consumer_offsets","partition":0,"replicas":[0,1,2]}, 
                                            {"topic":"__consumer_offsets","partition":1,"replicas":[0,2,1]},
                                            {"topic":"__consumer_offsets","partition":2,"replicas":[1,0,2]},
                                            {"topic":"__consumer_offsets","partition":3,"replicas":[1,2,0]},
                                             ...
                                            {"topic":"__consumer_offsets","partition":49,"replicas":[0,1,2]}
                                        ]
                                    }
                                    
                        第二步: 是执行 kafka-reassign-partitions 脚本
                                > bin/kafka-reassign-partitions.sh --zookeeper zookeeper_host:port 
                                                                   --reassignment-json-file reassign.json --execute

                    
            (7) 设置某个主题的副本同步速度
                    目的是为了限制某个主题的 leader 副本和 follower 副本同步的速度.
                    第一步: 先设置 Broker 端参数 leader.replication.throttled.rate 和 follower.replication.throttled.rate
                            每秒不能超过 100MB
                            > bin/kafka-configs.sh --zookeeper zookeeper_host:port --alter --add-config 
                                                   'leader.replication.throttled.rate=104857600, 
                                                   follower.replication.throttled.rate=104857600' 
                                                   --entity-type brokers 
                                                   --entity-name 0
                            参数:
                                --entity-name : Broker ID. 如果该主题的副本分别在 0, 1, 2, 3 多个 Broker 上, 还要依次为
                                                           Broker 1, 2, 3 执行这条命令
                                                           
                    第二步: 为该主题设置要限速的副本
                               为 topic 为 test, 所有副本设置同步速度.
                            > bin/kafka-configs.sh --zookeeper zookeeper_host:port --alter --add-config
                                                   'leader.replication.throttled.replicas=*,
                                                    follower.replication.throttled.replicas=*' 
                                                    --entity-type topics
                                                    --entity-name test
                                                    
            (8) 主题分区迁移
            (9) 删除主题
                    > bin/kafka-topics.sh --bootstrap-server broker_host:port --delete --topic <topic_name>
                    这是 kafka 的后台异步删除.
                    
            (10) 查看主题中的消息
                    
                    查看位移主题的位移值
                    > bin/kafka-console-consumer.sh --bootstrap-server kafka_host:port 
                                                    --topic __consumer_offsets --formatter 
                                                    "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter"
                                                     --from-beginning
                                                     
                    查看消费者组的状态信息                                
                    > bin/kafka-console-consumer.sh --bootstrap-server kafka_host:port 
                                                   --topic __consumer_offsets  --formatter 
                                                   "kafka.coordinator.group.GroupMetadataManager\$GroupMetadataMessageFormatter" 
                                                   --from-beginning
                                                   
    3. 错误排查
            (1) 主题删除失败
                    a. 失败原因
                            第一: 副本所在的 Broker 宕机
                            第二: 待删除主题的部分分区依然在执行迁移过程
                            
                    b. 解决方案
                            第一步: 手动删除 Zookeeper 节点 /admin/delete_topics 下以待删除主题为名的 znode
                            第二步: 手动删除该主题在磁盘上的分区目录
                            
                            
            (2) __consumer_offsets 主题占用太多的磁盘
                    a. 检测方法
                            用 jstack 命令查看一下 kafka-log-cleaner-thread 前缀的线程状态, 看看 Log Cleaner 线程是否挂掉了
                            
                    b. 解决方案
                            重启对应的 broker
```

## 动态配置
```shell
    1. 概念
            (1) confg/server.properties 配置文件里有配置参数, broker 启动时会加载这个文件, server.properties 中配置的参数则
                称为静态参数(Static Configs).
            (2) 动态参数是在 broker 运行过程中, 动态调整参数值而无需重启服务
            (3) Broker Configs 表中增加 Dynamic Update Mode 列
                    read-only: 只有重启 Broker, 才能令修改生效
                    per-broker:  属于动态参数, 修改后, 只会在对应的 Broker 上生效
                    cluster-wide: 也属于动态参数, 修改后, 会在整个集群范围内生效, 即对所有 Broker 都生效
            (4) 参数生效的优先级
                    per-broker 参数 > cluster-wide 参数 > static 参数 > Kafka 默认值
            (5) 动态参数配置后,重启也不会失效,会持久化.
                    
    2. 使用场景
            (1) 动态调整 Broker 端各种线程池大小, 实时应对突发流量.
            (2) 动态调整 Broker 端连接信息或安全配置信息
            (3) 动态更新 SSL Keystore 有效期
            (4) 动态调整 Broker 端 Compact 操作性能
            (5) 实时变更 JMX 指标收集器 (JMX Metrics Reporter)
            
    3. 保存的位置
            (1) 动态参数保存在 zookeeper 中, 
                    a. /config/brokers/<default> 保存设置的 cluster-wide 级别动态参数, 里面的内容是 json 文件, 
                            {
                                "version":1,
                                "config" :
                                {
                                    "num.io.threads" : "12",
                                    .....
                                }
                            }
                    b. /config/brokers/xx 保存 broker xx 的 per-broker 动态参数, 
                            例如: /config/brokers/0 保存 broker 0 的 per-broker 动态参数
                            
    4. 配置命令
            (1) 在集群层面设置全局值( cluster-wide)
                    $ bin/kafka-configs.sh --bootstrap-server kafka-host:port 
                                           --entity-type brokers --entity-default --alter
                                           --add-config unclean.leader.election.enable=true
                                           
                      其中 --entity-default 代表设置集群层面的参数
                      
            (2) 查看集群参数是否配置成功
                    $ bin/kafka-configs.sh --bootstrap-server kafka-host:port 
                                           --entity-type brokers --entity-default --describe
                                           
                    结果:
                        Default config for brokers in the cluster are:
                        
                        unclean.leader.election.enable=true sensitive=false 
                        synonyms={DYNAMIC_DEFAULT_BROKER_CONFIG:unclean.leader.election.enable=true}
                        unclean.leader.election.enable=true 设置成功为 true
                        sensitive=false , 调整的参数不是敏感数据
                        
                删除集群参数
                    $ bin/kafka-configs.sh --bootstrap-server kafka-host:port --entity-type brokers 
                                           --entity-default --alter --delete-config unclean.leader.election.enable
                        
            (3) 设置 per-broker 的参数
                    对 broker 序号为 1 进行设置
                    > bin/kafka-configs.sh --bootstrap-server kafka-host:port 
                                           --entity-type brokers --entity-name 1 
                                           --alter --add-config unclean.leader.election.enable=false
                                           
                      -entity-name 1  配置 broker.id=1 的 broker 参数, 如果有多个 broker 要配置, 那就分别执行对于
                                      broker.id 的命令
                                      
                删除 per-broker 的参数
                    > bin/kafka-configs.sh --bootstrap-server kafka-host:port --entity-type brokers 
                                           --entity-name 1 --alter --delete-config unclean.leader.election.enable
                                           
            (4) 查看 per-broker 的参数
                    > bin/kafka-configs.sh --bootstrap-server kafka-host:port --entity-type brokers 
                                           --entity-name 1 --describe
                    
                    结果:
                        Configs for broker 1 are:
                        
                        unclean.leader.election.enable=false sensitive=false 
                        synonyms={
                                    DYNAMIC_BROKER_CONFIG:unclean.leader.election.enable=false, 
                                    DYNAMIC_DEFAULT_BROKER_CONFIG:unclean.leader.election.enable=true,
                                    DEFAULT_CONFIG:unclean.leader.election.enable=false
                                 }
                                 
    5. 需要调整参数
            (1) log.retention.ms : 日志留存时间
            (2) num.io.threads 和 num.network.threads: 在实际生产环境中, Broker 端请求处理能力经常要按需扩容
            (3) 与 SSL 相关的参数
                    主要是 4 个参数(ssl.keystore.type、ssl.keystore.location、ssl.keystore.password 和
                    ssl.key.password). 允许动态实时调整后, 可以创建那些过期时间很短的 SSL 证书. 每当调整时, Kafka 底层会重新
                    配置 Socket 连接通道并更新 Keystore. 新的连接会使用新的 Keystore, 阶段性地调整这组参数, 有利于增加安全性
            (4) num.replica.fetchers
                    动态解决 Follower 副本拉取速度慢, 增加值相当于增加多个线程进行 follower 副本的拉取.                            
```

## 重设消费者组位移
```shell
    1. 概念
            (1) 
            
    2. 重设位移策略
            (1) 位移纬度
                    位移纬度是直接把消费者的位移重设为给定的位移值.对应的策略有以下 5 种
                    
                    第一种: Earliest 策略
                                将位移调整到主题当前最早位移处, 最早位移不一定就是 0, 可能很久远的消息会被 Kafka 自动删除.
                           如果想要重新消费主题的所有消息, 可以使用 Earliest 策略
                           
                    第二种: Latest 策略
                                把位移重设成最新末端位移, 如果想跳过所有历史消息, 打算从最新的消息处开始消费可以使用 Latest 策略.
                                
                    第三种: Current 策略
                                将位移调整成消费者当前提交的最新位移. 遇到场景:修改消费者程序代码, 并重启消费者, 发现代码有问题,
                           需要回滚之前的代码变更, 同时也要把位移重设到消费者重启时的位置, Current 策略就可以实现这个功能
                           
                    第四种: Specified-Offset 策略
                                把位移值调整到指定的位移处.使用场景是消费者程序在处理某条错误消息时, 可以手动地“跳过”此消息的处理,
                            在实际使用过程中, 可能会出现 corrupted 消息无法被消费, 这时消费者程序会抛出异常, 无法继续工作.
                            可以尝试使用 Specified-Offset 策略来规避
                            
                    第五种: Shift-By-N 策略
                                把位移调整到当前位移 +N 处(N 可以是负值), 把位移重设成当前位移的前 100 条位移处, 你需要指定 N 为
                           -100
                           
            (2) 时间纬度
                    可以给定一个时间, 让消费者把位移调整成这个时间的的位移或则是大于这个时间的最小位移.
                    
                    第一种: DateTime 策略
                                允许指定一个时间, 然后将位移重置到该时间之后的最早位移处. 常见的使用场景是, 想重新消费昨天的数据,
                          可以使用该策略重设位移到昨天 0 点
                     
                    第二种: Duration 策略
                                指给定相对的时间间隔, 然后将位移调整到距离当前给定时间间隔的位移处, 具体格式是 PnDTnHnMnS. 例如
                           如果想将位移调回到 15 分钟前, 那么就可以指定 PT0H15M0S
                           
    3. 重设位移的方式
            (1) 消费者 API 方式
                    void seek(TopicPartition partition, long offset);
                    void seek(TopicPartition partition, OffsetAndMetadata offsetAndMetadata);
                    每次调用 seek 方法只能重设一个分区的位移
                    
                    void seekToBeginning(Collection<TopicPartition> partitions);
                    void seekToEnd(Collection<TopicPartition> partitions);
                    seekToBeginning 和 seekToEnd 则拥有一次重设多个分区的能力
                    
                    Earliest 策略实现
                        Properties consumerProperties = new Properties();
                        consumerProperties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
                        consumerProperties.put(ConsumerConfig.GROUP_ID_CONFIG, groupID);
                        consumerProperties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
                        consumerProperties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer)
                        consumerProperties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserialize)
                        consumerProperties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
                        
                        String topic = "test"; // 要重设位移的 Kafka 主题
                        
                        try (final KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProperties)) 
                        {
                            consumer.subscribe(Collections.singleton(topic));
                            
                            // 一定要调用带长整型的 poll 方法, 而不要调用 consumer.poll(Duration.ofSecond(0))
                            // poll(0), 是阻塞等待成功获取了所需的元数据信息后, 在非阻塞发起 fetch 请求去获取数据
                            // 而 poll(Duration) 是连获取了所需的元数据信息后也是非阻塞的, 
                            // consumer 根本无法在这么短的时间内连接上 coordinator, 就会返回空集合.
                            consumer.poll(0);
                            // 调用 seekToBeginning 方法时, 需要一次性构造主题的所有分区对象
                            consumer.seekToBeginning(
                                    consumer.partitionsFor(topic).stream().map( 
                                                                                partitionInfo -> new TopicPartition(topic, partitionInfo.partition())
                                                                                
                                                                              ) .collect(Collectors.toList())
                                                     );
                        }
                        
                    Latest 策略
                            consumer.seekToEnd(
                                   consumer.partitionsFor(topic).stream().map(
                                                                               partitionInfo -> new TopicPartition(topic, partitionInfo.partition())
                                                                              ) .collect(Collectors.toList())
                                              );
                                              
                    Current 策略
                            consumer.partitionsFor(topic).stream().map(
                                                                        info -> new TopicPartition(topic, info.partition())
                                                                      ) .forEach(
                                                                                  tp -> {
                                                                                            long committedOffset = consumer.committed(tp).offset();
                                                                                            consumer.seek(tp, committedOffset);
                                                                                        }
                                                                                );
                            调用 partitionsFor 方法获取给定主题的所有分区, 然后依次获取对应分区上的已提交位移, 最后通过 seek 方法
                            重设位移到已提交位移处
                            
                    Specified-Offset 策略
                            long targetOffset = 1234L;
                            for (PartitionInfo info : consumer.partitionsFor(topic)) 
                                {
                                    TopicPartition tp = new TopicPartition(topic, info.partition());
                                    consumer.seek(tp, targetOffset);
                                }
                                
                    Shift-By-N 策略
                            for (PartitionInfo info : consumer.partitionsFor(topic)) 
                            {
                                TopicPartition tp = new TopicPartition(topic, info.partition());
                                // 假设向前跳 123 条消息
                                long targetOffset = consumer.committed(tp).offset() + 123L;
                                consumer.seek(tp, targetOffset);
                            }
                            
                    DateTime 策略
                            重设位移到 2019 年 6 月 20 日晚上 8 点
                            long ts = LocalDateTime.of(2019, 6, 20, 20, 0).toInstant(ZoneOffset.ofHours(8)).toEpochMilli();
                            
                             Map<TopicPartition, Long> timeToSearch = consumer.partitionsFor(topic).
                                                                      stream().map(
                              info ->new TopicPartition(topic, info.partition())
                               ).collect(
                                           Collectors.toMap(Function.identity(), tp -> ts)
                                        );
                            
                            for (Map.Entry<TopicPartition, OffsetAndTimestamp> entry : consumer.offsetsForTimes(timeToSearch).entrySet()) 
                            {
                                consumer.seek(entry.getKey(), entry.getValue().offset());
                            }
                            
                            构造了 LocalDateTime 实例, 然后利用它去查找对应的位移值, 最后调用 seek, 实现了重设位移
                            
                    Duration 策略
                            Map<TopicPartition, Long> timeToSearch = consumer.partitionsFor(topic).stream()
                                                   .map(info -> new TopicPartition(topic, info.partition()))
                                                   .collect(
                                                                Collectors.toMap(Function.identity(), tp -> System.currentTimeMillis())
                                                           );
                            
                            for (Map.Entry<TopicPartition, OffsetAndTimestamp> entry : consumer.offsetsForTimes(timeToSearch).entrySet()) 
                            {
                                consumer.seek(entry.getKey(), entry.getValue().offset());
                            }
                            
            (2) 命令字形式
                    Earliest 策略
                        直接指定 –to-earliest
                        > bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group 
                                                       --reset-offsets --all-topics --to-earliest –execute
                                                       
                    Latest 策略
                        直接指定 –to-latest
                        > bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group 
                                                       --reset-offsets --all-topics --to-latest --execute
                            
                    Current 策略
                        直接指定 –to-current
                        > bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group 
                                                       --reset-offsets --all-topics --to-current --execute
                                                       
                    Specified-Offset 策略
                        直接指定 –to-offset
                        > bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group
                                                       --reset-offsets --all-topics --to-offset <offset> --execute
                                                       
                    Shift-By-N 策略
                        直接指定 –shift-by N
                        > bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group 
                                                       --reset-offsets --shift-by <offset_N> --execute
                                                       
                    DateTime 策略
                        直接指定 –to-datetime
                        > bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group 
                                                       --reset-offsets --to-datetime 2019-06-20T20:00:00.000 --execute
                                                       
                    Duration 策略
                        直接指定 –by-duratio
                        bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group 
                                                     --reset-offsets --by-duration PT0H30M0S --execute
                            
```

### 常见工具脚本
```shell
    1. 概念
    2. 其他脚本
        1. kafka-server-start 和 kafka-server-stop 脚本
                        启动/停止 kafka 服务
                        
        2. Kafka Connect 组件
                用于实现 Kafka 与外部世界系统之间的数据传输
                 connect-standalone 脚本: 支持单节点的 Standalone 模式
                 connect-distributed 脚本: 支持多节点的 Distributed 模式
                 
        3. kafka-acls 脚本
                设置 Kafka 权限, 比如设置哪些用户可以访问 Kafka 的哪些主题之类的权限
                
        4. kafka-broker-api-versions 脚本
                验证不同 Kafka 版本之间服务器和客户端的适配性
                > bin/kafka-broker-api-versions.sh --bootstrap-server localhost:9092
                结果:
                localhost:9092( id:0 rack: null -> (
                        Produce(0): 0 to 7 [usable: 7],
                        ....
                        )
                        
                        (0) : 代表消息的序号
                        0 to 7 : 代表该 kafka 2.2 支持 produce 请求版本 0 ~ 7, 而这个客户端执行,支持最新的 7 版本
                        
                在 0.10.2.0 之前, Kafka 是单向兼容的, 即高版本的 Broker 能够处理低版本 Client 发送的请求, 
                但是低版本的 Broker 不能处理高版本的 Client 发送的请求
                自 0.10.2.0 版本开始, Kafka 正式支持双向兼容, 即低版本的 Broker 也能处理高版本 Client 的请求
                
        5. kafka-dump-log 脚本
                查看 Kafka 消息文件的内容, 包括消息的各种元数据信息, 甚至是消息体本身
                
        6. kafka-log-dirs 脚本
                查询各个 Broker 上的各个日志路径的磁盘占用情况
                
        7. kafka-preferred-replica-election 脚本
                执行 Preferred Leader 选举, 可以为指定的主题执行 “换 Leader” 的操作
                
        8. kafka-reassign-partitions 脚本
                执行分区副本迁移以及副本文件路径迁移
                
    3. kafka-console-producer 脚本    
            生成消息, 使用控制台来向 Kafka 的指定主题发送消息
            > bin/kafka-console-producer.sh --broker-list kafka-host:port --topic test-topic 
                                            --request-required-acks -1 
                                            --producer-property compression.type=lz4
    4. kafka-console-consumer 脚本
            > bin/kafka-console-consumer.sh --bootstrap-server kafka-host:port --topic test-topic
                                           --group test-group --from-beginning
                                           --consumer-property enable.auto.commit=false
                                           
              需要指定 group 信息, 如果没有指定, 每次运行 Console Consumer, 它都会自动生成一个新的消费者组来消费. 会导致集群中有
              大量的以 console-consumer 开头的消费者组
              
              from-beginning :  Consumer 端参数 auto.offset.reset 设置成 earliest, 从头开始消费主题. 如果不指定, 会默认从
              最新位移读取消息.如果此时没有任何新消息, 该命令的输出为空
              
              在命令中禁掉自动提交位移, 让 Console Consumer 脚本提交位移是没有意义的, 因为只是用它做一些简单的测试
              
    5. kafka-producer-perf-test 脚本
            生产者性能脚本
            
            向指定主题发送了 1 千万条消息, 每条消息大小是 1KB
            > bin/kafka-producer-perf-test.sh --topic test-topic --num-records 10000000 --throughput -1 
                                              --record-size 1024 
                                              --producer-props bootstrap.servers=kafka-host:port acks=-1 
                                                               linger.ms=2000 compression.type=lz4
                                                               
            结果:
                    2175479 records sent, 435095.8 records/sec (424.90 MB/sec), 131.1 ms avg latency, 681.0 ms max latency.
                    4190124 records sent, 838024.8 records/sec (818.38 MB/sec), 4.4 ms avg latency, 73.0 ms max latency.
                    10000000 records sent, 737463.126844 records/sec (720.18 MB/sec), 31.81 ms avg latency, 
                                                                                      681.00 ms max latency, 
                                                                                      4 ms 50th, 126 ms 95th, 
                                                                                      604 ms 99th, 672 ms 99.9th.
                    99th 值是 604ms, 这表明测试生产者生产的消息中, 有 99% 消息的延时都在 604ms 以内
                    
    6. kafka-consumer-perf-test脚本
    
            > bin/kafka-consumer-perf-test.sh --broker-list kafka-host:port --messages 10000000 --topic test-topic
            
            结果:
                start.time,                      end.time,          data.consumed.in.MB,       MB.sec,   
                2019-06-26 15:24:18:138, 2019-06-26 15:24:23:805,       9765.6202,            1723.2434,
                                                       
                data.consumed.in.nMsg,   nMsg.sec,    rebalance.time.ms,    fetch.time.ms, fetch.MB.sec,   fetch.nMsg.sec
                       10000000,       1764602.0822,          16,                 5651,      1728.1225,     1769598.3012    
                          
    7. 查看某个主题消息总数
            > bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list kafka-host:port --time -2 --topic test-topic
            结果:
                test-topic:0:0
                test-topic:1:0
                
            > bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list kafka-host:port --time -1 --topic test-topic
            结果:
                test-topic:0:5500000
                test-topic:1:5500000
                
            使用 Kafka 提供的工具类 GetOffsetShell 来计算给定主题特定分区当前的最早位移和最新位移, 将两者的差值累加起来, 就能得到
            该主题当前总的消息数. 对于本例来说, test-topic 总的消息数为 5500000 + 5500000, 等于 1100 万条
            
    8.  kafka-dump-log 脚本(查看消息日志文件数据)
            > bin/kafka-dump-log.sh --files ../data_dir/kafka_1/test-topic-1/00000000000000000000.log 
            结果:
                Dumping ../data_dir/kafka_1/test-topic-1/00000000000000000000.log
                Starting offset: 0
                baseOffset: 0 lastOffset: 14 count: 15 baseSequence: -1 lastSequence: -1 producerId: -1 
                producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 
                CreateTime: 1561597044933 size: 1237 magic: 2 compresscodec: LZ4 crc: 646766737 isvalid: true
                
                
                baseOffset: 15 lastOffset: 29 count: 15 baseSequence: -1 lastSequence: -1 producerId: -1 
                producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 1237 
                CreateTime: 1561597044934 size: 1237 magic: 2 compresscodec: LZ4 crc: 3751986433 isvalid: true
                
            查看每一条具体消息  显示指定 --deep-iteration参数
            > bin/kafka-dump-log.sh --files ../data_dir/kafka_1/test-topic-1/00000000000000000000.log 
                                    --deep-iteration --print-data-log
                                    
    9. kafka-consumer-groups 脚本(查看消费者组位移)
            > bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group test-group
            
            
```

### AdminClient
```shell
    1. 概念
    2. 功能
            (1). 主题管理：包括主题的创建、删除和查询
            (2). 权限管理：包括具体权限的配置与删除
            (3). 配置参数管理：包括 Kafka 各种资源的参数设置, 详情查询. Kafka 资源, 主要有 Broker, 主题, 用户, Client-id 等
            (4). 副本日志管理：包括副本底层日志路径的变更和详情查询
            (5). 分区管理：即创建额外的主题分区
            (6). 消息删除：即删除指定位移之前的分区消息。
            (7). Delegation Token 管理：包括 Delegation Token 的创建、更新、过期和详情查询
            (8). 消费者组管理：包括消费者组的查询、位移查询和删除。
            (9). Preferred 领导者选举：推选指定主题分区的 Preferred Broker 为领导者
            
    3. AdminClient 工作原理
            AdminClient 是一个双线程的设计：前端主线程和后端 I/O 线程. 前端线程负责将用户要执行的操作转换成对应的请求,其中包括
       第一个任务 构建对应的请求对象, 如果要创建主题, 就创建 CreateTopicsRequest 对象, 如果是查询消费者组位移, 
       就创建 OffsetFetchReques 对象. 第二个任务 指定响应的回调逻辑, 如从 Broker 端接收到 CreateTopicsResponse 之后要执行的
       动作,  然后再将请求发送到后端 I/O 线程的队列中；而后端 I/O 线程从队列中读取相应的请求, 然后发送到对应的 Broker 节点上, 之
       后把执行结果保存起来, 以便等待前端线程的获取
       
       问题排查:
                出现执行结果没有返回, 或则是 hang 住了, 可能的原因是后端 I/O 线程出现问题, 可以通过 jstack 查看 AdminClient 程序
                
    4. AdminClient 实例
            (1) 创建主题
                    String newTopicName = "test-topic";
                    try (AdminClient client = AdminClient.create(props)) {
                             NewTopic newTopic = new NewTopic(newTopicName, 10, (short) 3);
                             CreateTopicsResult result = client.createTopics(Arrays.asList(newTopic));
                             result.all().get(10, TimeUnit.SECONDS);
                    }
                    
            (2) 查询消费者组位移
                    String groupID = "test-group";
                    try (AdminClient client = AdminClient.create(props)) {
                             ListConsumerGroupOffsetsResult result = client.listConsumerGroupOffsets(groupID);
                             Map<TopicPartition, OffsetAndMetadata> offsets = 
                                      result.partitionsToOffsetAndMetadata().get(10, TimeUnit.SECONDS);
                             System.out.println(offsets);
                    }
                    
            (3) 获取 Broker 磁盘占用
                    try (AdminClient client = AdminClient.create(props)) {
                             DescribeLogDirsResult ret = client.describeLogDirs(Collections.singletonList(targetBrokerId)); // 指定 Broker id
                             long size = 0L;
                             for (Map<String, DescribeLogDirsResponse.LogDirInfo> logDirInfoMap : ret.all().get().values()) {
                                      size += logDirInfoMap.values().stream().map(logDirInfo -> logDirInfo.replicaInfos).flatMap(
                                               topicPartitionReplicaInfoMap ->
                                               topicPartitionReplicaInfoMap.values().stream().map(replicaInfo -> replicaInfo.size))
                                               .mapToLong(Long::longValue).sum();
                             }
                             System.out.println(size);
                    }
```

### 认证机制
```shell
    1. 概念
        (1) 认证也是鉴权(authentication), 认证的主要目的是确认当前声称为某种身份的用户确实是所声称的用户
        (2) 认证要解决的是证明你是谁的问题, 授权要解决的则是能做什么的问题
        (3) ssl 做信道加密比较多, 使用 SSL 来做通信加密, 使用 SASL 来做 Kafka 的认证实现
        (4) PLAIN 在这里是一种认证机制, 而 PLAINTEXT 说的是未使用 SSL 时的明文传输
    2. 认证机制
            (1) SSL 
                    概念: 
                    引入版本: 0.9
                    适用场景: 一般测试场景
                    
            (2) SASL/GSSAPI
                    引入版本: 0.9
                    适用场景: 本身已经实现的 Kerberos 认证(使用 Active Directory)的场景
                    
            (3) SASL/PLAIN
                    引入版本: 0.10.2
                    适用场景: 中小型公司的 kafka 集群
                    概念: 是一个简单的用户名 / 密码认证机制, 通常与 SSL 加密搭配使用, 它不能动态地增减认证用户, 必须重启 Kafka 集
                          群才能令变更生效. 因为所有认证用户信息全部保存在静态文件中, 只能重启 Broker, 
                          才能重新加载变更后的静态文件
                    
            (4) SASL/SCRAM
                    引入版本: 0.10.2
                    适用场景: 中小型公司的 kafka 集群, 支持认证用户的动态增减
                    概念: 解决动态增减认证用户, 需要重启集群, 通过将认证用户信息保存在 ZooKeeper 的方式
                    
            (5) SASL/OAUTHBEARER
                    引入版本: 2.0
                    适用场景: OAuth 2.0 框架的场景
                    概念: OAuth 是一个开发标准, 允许用户授权第三方应用访问该用户在某网站上的资源, 而无需将用户名和密码提供给第三方应用.
                          Kafka 不提倡单纯使用 OAUTHBEARER,  因为它生成的不安全的 JSON Web Token, 必须配以 SSL 加密才能用在生产环境
                    
            (6) Delegation Token
                    引入版本: 1.1
                    适用场景: 适用与 Kerberos 认证出现 TGT 分发性能瓶颈的场景
                    概念: 它是一种轻量级的认证机制, 目的是补充现有的 SASL 或 SSL 认证. 如果要使用 Delegation Token, 需要先配置好
                         SASL 认证, 然后再利用 Kafka 提供的 API 去获取对应的 Delegation Token. Broker 和客户端在
                         做认证的时候, 可以直接使用这个 token, 不用每次都去 KDC 获取对应的 ticket(Kerberos 认证)或
                         传输 Keystore 文件(SSL 认证)
                         
    3. SASL/SCRAM-SHA-256 配置实例
            第一步: 创建用户
                    创建 3 个用户, admin 用户用于实现 Broker 间通信, writer 用户用于生产消息, reader 用户用于消费消息
                    
                    创建 admin
                    >  bin/kafka-configs.sh --zookeeper localhost:2181 --alter 
                                            --add-config 'SCRAM-SHA-256=[password=admin],SCRAM-SHA-512=[password=admin]' 
                                            --entity-type users --entity-name admin
                                            
                    创建 writer
                    > bin/kafka-configs.sh --zookeeper localhost:2181 --alter 
                                           --add-config 'SCRAM-SHA-256=[password=writer],SCRAM-SHA-512=[password=writer]'
                                           --entity-type users --entity-name writer
                                           
                    创建 reader
                    > bin/kafka-configs.sh --zookeeper localhost:2181 --alter 
                                           --add-config 'SCRAM-SHA-256=[password=reader],SCRAM-SHA-512=[password=reader]' 
                                           --entity-type users --entity-name reader
                                           
                    查看创建的用户信息
                    > bin/kafka-configs.sh --zookeeper localhost:2181 --describe --entity-type users  --entity-name writer
                    
            第二步: 创建 JAAS 文件
                        (1) 为每台单独的物理 Broker 机器都创建一份 JAAS 文件
                        
                            使用 admin 用户实现 Broker 之间的通信
                            内容如下:
                                KafkaServer {
                                org.apache.kafka.common.security.scram.ScramLoginModule required
                                username="admin"
                                password="admin";
                                };
                                
                                注意: 不能有空格键
                                
                        (2) 配置 Broker 的 server.properties 文件
                        
                                 // 开启 SCRAM 认证机制, 并启用 SHA-256 算法
                                sasl.enabled.mechanisms=SCRAM-SHA-256
                                // 为 Broker 间通信也开启 SCRAM 认证, 同样使用 SHA-256 算法
                                sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256
                                // Broker 间通信不配置 SSL
                                security.inter.broker.protocol=SASL_PLAINTEXT
                                // Broker 间通信不配置 SSL
                                listeners=SASL_PLAINTEXT://localhost:9092
                                
            第三步: 启动 Broker
                    > KAFKA_OPTS=-Djava.security.auth.login.config=<your_path>/kafka-broker.jaas bin/kafka-server-start.sh config/server1.properties
                    
            第四步: 发消息
                        (1) 编写 producer.conf 的配置文件
                                security.protocol=SASL_PLAINTEXT
                                sasl.mechanism=SCRAM-SHA-256
                                sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="writer" password="writer";
                                
                        (2) 运行 Console Producer 程序
                                > bin/kafka-console-producer.sh --broker-list localhost:9092,localhost:9093 
                                                                --topic test
                                                                --producer.config <your_path>/producer.conf
                                                                
            第五步: 消费消息
                        (1)  编写 consumer.conf 的配置文件
                                security.protocol=SASL_PLAINTEXT
                                sasl.mechanism=SCRAM-SHA-256
                                sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="reader" password="reader";
                                
                        (2) 运行 Console Consumer 程序
                                > bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093 
                                                                --topic test --from-beginning 
                                                                --consumer.config <your_path>/consumer.conf 
                                                                
            第六步: 动态增减用户
                        (1) 删除 writer 用户, 新增 new_writer 用户
                            > bin/kafka-configs.sh --zookeeper localhost:2181 --alter 
                                                   --delete-config 'SCRAM-SHA-256' --entity-type users --entity-name writer
                            
                            > bin/kafka-configs.sh --zookeeper localhost:2181 --alter
                                                   --delete-config 'SCRAM-SHA-512' --entity-type users --entity-name writer 
                                                   
                            >  bin/kafka-configs.sh --zookeeper localhost:2181 --alter 
                                                    --add-config 'SCRAM-SHA-256=[iterations=8192,password=new_writer]'
                                                    --entity-type users --entity-name new_writer
                                                    
                        (2) 这时候需要修改原先的 producer.conf 配置文件, 将 username 修改为 new_writer
                            
```

### 授权机制
```shell
    1. 概念
            (1)
             
    2. 权限模型
            第一种 : ACL(常用, kafka 使用的)
                        Access-Control List, 访问控制列表, 表示用户与权限的直接映射关系. 规定了什么用户对什么资源有什么样的访问权限.
                    
                        
            第二种: RBAC(常用)
                        Role-Based Access Control, 基于角色的权限控制, 支持对用户进行分组
                        
            第三种: ABAC
                        Attribute-Based Access Control, 基于属性的权限控制
                        
            第四种: PBAC
                        Policy-Based Access Control, 基于策略的权限控制
                        
    3. 开启 ACL
            在 server.properties 配置文件中
                authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
                
            其中 authorizer.class.name 参数指定了 ACL 授权机制的实现类
            
    4. 配置超级用户
            在 server.properties 配置文件中
                super.users=User:superuser1;User:superuser2
                
                 allow.everyone.if.no.acl.found=true  // 所有用户都可以访问任何 ACL 的资源, 不常用
                  
    5. kafka-acls 脚本
            进行授权配置,为用户 Alice 增加集群级别的所有权限,如下
                > bin/kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 
                                 --add --allow-principal User:Alice --operation All --topic '*' --cluster
                                 
                  All 表示所有操作, topic 中的 * 则表示所有主题, 指定 --cluster 则说明为 Alice 设置的是集群权限
                  
                >  bin/kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 
                                  --add --allow-principal User:'*' --allow-host '*' 
                                  --deny-principal User:BadUser --deny-host 10.205.96.119 
                                  --operation Read --topic test-topic
                                  
                  User 后面的星号表示所有用户, allow-host 后面的星号则表示所有 IP 地址. 这个命令的
                  允许所有的用户使用任意的 IP 地址读取名为 test-topic 的主题数据, 同时也禁止
                  BadUser 用户和 10.205.96.119 的 IP 地址访问 test-topic 下的消息
                  
    6. 授权机制可以单独使用
           (1) 往往认证机制和授权机制配合使用, 不过也可以只使用授权机制, 这样只能对某个 ip 进行指定
           
                禁止 127.0.0.1 上的 Producer 向 test 主题发送数据
                > bin/kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 
                                    --add --deny-principal User:* --deny-host 127.0.0.1 
                                    --operation Write --topic test
                                    
    7.  SSL + ACL 配置实例
            第一步: ssl 的配置(https://blog.csdn.net/qq_18522601/article/details/99834726),  SHELL 脚本在 Broker 上运行
                   产生了 server.keystore.jks, server.truststore.jks,  client.keystore.jks 和 client.truststore.jks
                   把以 server 开头的两个文件, 拷贝到集群中的所有 Broker 机器上, 把以 client 开头的两个文件,拷贝到所有要连接
                   Kafka 集群的客户端应用程序机器上.
                   
            第二步: 配置每个 Broker 的 server.properties 文件
                        listeners=SSL://localhost:9093
                        ssl.truststore.location=/Users/huxi/testenv/certificates/server.truststore.jks
                        ssl.truststore.password=test1234
                        ssl.keystore.location=/Users/huxi/testenv/certificates/server.keystore.jks
                        ssl.keystore.password=test1234
                        security.inter.broker.protocol=SSL
                        ssl.client.auth=required
                        ssl.key.password=test1234
                        
            第三步: 配置客户端的 ssl
                        client-ssl.config 配置文件
                            security.protocol=SSL
                            ssl.truststore.location=/Users/huxi/testenv/certificates/client.truststore.jks
                            ssl.truststore.password=test1234
                            ssl.keystore.location=/Users/huxi/testenv/certificates/server.keystore.jks
                            ssl.keystore.password=test1234
                            ssl.key.password=test1234
                            ssl.endpoint.identification.algorithm=
                            
                        一定要加上最后一行, 自 Kafka 2.0 版本开始, 它默认会验证服务器端的主机名是否匹配 Broker 端证书里的主机名.
                        要禁掉此功能的话, 一定要将该参数设置为空字符串
                        
            第四步: 验证
                        >bin/kafka-console-producer.sh --broker-list localhost:9093 --topic test
                         　　　　　　　　　　　　　　　　　 --producer.config client-ssl.config
                         
    8. ACL 配置实践
            (1) 开启 ACL, authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
            (2) 采用白名单机制, 没有设置的用户禁止访问资源, 不要设置 allow.everyone.if.no.acl.found=true
            (3) 
                        
```

### kafka 监控
```shell
    1. 概念
    2. 主机监控
            (1) 主机机器负载（Load）, top 命令的 load average 值分别代表过去 1 分钟、过去 5 分钟和过去 15 分钟的 Load 平均值,
                值低于 cpu 核为正常.
            (2) CPU 使用率, top 中的是考虑了多核进去, 如果是每个 cpu, 平均个数.
            
    3. JVM 监控
            (1) 需要知道 Broker 端 JVM 进程的 Minor GC 和 Full GC(对程序堆进行垃圾回收) 的发生频率和时长,
                长时间的停顿会令 Broker 端抛出各种超时异常
            (2) 设置 Broker 堆的大小就是在程序 full gc 后存活的对象大小 * 1.5, 可以通过 GC 日志来看, 如下：
                    2019-07-30T09:13:03.809+0800: 552.982: [GC cleanup 827M->645M(1024M), 0.0019078 secs]
                监控 kafkaServergc.log 日志, 一旦频繁发生 Full GC, 可以开启 G1 的 -XX:+PrintAdaptiveSizePolicy 开关,
                看哪个线程发生 Full GC
            (3) 活跃对象大小, 设定堆大小的重要依据, 调优 JVM 各个代的堆大小
            (4) 应用线程总数, 了解 Broker 进程对 CPU 的使用情况
            
    4. 集群监控
            (1) 查看 Broker 进程是否启动, 端口是否建立
            (2) 查看 Broker 端关键日志
                     Broker 端服务器日志 server.log, 控制器日志 controller.log, 主题分区状态变更日志 state-change.log
            (3) 查看 Broker 端关键线程的运行状态
                    第一个: Log Compaction 线程, 以 kafka-log-cleaner-thread 开头, 做日志 Compaction(压缩), 一旦它挂掉了, 
                          所有 Compaction 操作都会中断, 会出现 Kafka 内部的位移主题所占用的磁盘空间越来越大等现象.
                          
                    第二个: 副本拉取消息的线程, 以 ReplicaFetcherThread 开头, 执行 follower 副本从 leader 副本拉取消息进行同步,
                           线程异常后会导致 Follower 副本的 Lag 会越来越大
                           
                    可以通过使用 jstack 命令, 或则其他的监控框架
                    
            (4) 查看 Broker 端的关键 JMX 指标
                    a. BytesIn/BytesOut, Broker 端每秒入站和出站字节数, 最好不要接近自己的网络带宽
                    b. NetworkProcessorAvgIdlePercent：网络线程池线程平均的空闲比例.确保这个 JMX 值长期大于 30%. 
                       小于这个值说明网络线程池非常繁忙, 需要通过增加网络线程数或将负载转移给其他服务器的方式
                    c. RequestHandlerAvgIdlePercent：即 I/O 线程池线程平均的空闲比例. 该值长期小于 30%, 需要调整
                       I/O 线程池的数量
                    d. UnderReplicatedPartitions：未充分备份的分区数, 即与 leader 副本未同步的 follower 的数量, 太多就要
                    　　引起重视.
                    e. SRShrink/ISRExpand：即 ISR 收缩和扩容的频次指标. 出现 ISR 中副本频繁进出的情形, 那么这组值一定是很高
                       要诊断下副本频繁进出 ISR 的原因, 并采取适当的措施
                    f. ActiveControllerCount：当前处于激活状态的控制器的数量. 正常情况下, Controller 所在 Broker 上的这个 JMX 
                       指标值是 1, 其他 Broker 上的这个值是 0. 如果存在多台 Broker 上该值都是 1 的情况, 要查看网络连通性, 
                       可能集群出现脑裂。脑裂问题是非常严重的分布式故障, Kafka 目前依托 ZooKeeper 来防止脑裂, 一旦出现脑裂,
                       Kafka 是无法保证正常工作
                       
            (5) 监控 Kafka 客户端
                    a. 查看客户端与 broker 之间的网络往返时间(RTT), 通过 ping
                    b. 对于生产者, 监控以 kafka-producer-network-thread 开头的线程, 这个线程主要发送消息.
                       request-latency 的 JMX 指标, 即消息生产请求的延时. 
                    c. 对于消费者, 监控以 kafka-coordinator-heartbeat-thread　开头的线程, 这个线程事关与 Rebalance.
                        records-lag 和 records-lead JMX 指标, 反映消费进度.
                        如果是 consumer group, join rate 和  sync rate JMX 指标 反映 Rebalance 频繁程度.
                                  
```

### kafka 调优
```shell
    1. 概念
        (1) 高吞吐量、低延时是我们调优 Kafka 集群的主要目标
    2. 优化漏斗
            (1) 优化漏斗是一个调优过程中的分层漏斗, 层级越靠上, 其调优的效果越明显
            (2)
                 第一层：应用程序层. 优化 Kafka 客户端应用程序代码. 如使用合理的数据结构、缓存计算开销大的运算结果, 或是复用构造
                        成本高的对象实例等. 这一层的优化效果最为明显, 通常也是比较简单的
                 第二层: 框架层. 指合理设置 Kafka 集群的各种参数. 直接修改 Kafka 源码进行调优并不容易, 但可以恰当地配置关键参数的值
                 第三层: JVM 层, Kafka Broker 进程是普通的 JVM 进程, 各种对 JVM 的优化在这里也是适用的
                 第四层: 操作系统层, 对操作系统层的优化很重要, 但效果往往不如想象得那么好. 与应用程序层的优化效果相比,它是有很大差距的
                 
    3. 操作系统层
            (1) > mount -o noatime 
                  禁止 atime(文件最后被访问的时间)更新, 因为记录 atime 需要操作系统访问 inode 资源, 并写入.
            (2) 文件系统, 尽量选择 ext4 或 XFS
            (3) swap 空间的设置,  swappiness 设置成一个很小的值, 比如 1～ 10 之间, 防止逻辑内存大于物理内存时, 进程直接被 kill
                                可以执行 sudo sysctl vm.swappiness=N 来临时设置该值. 如果要永久生效, 可以修改
                                 /etc/sysctl.conf 文件,  增加 vm.swappiness=N, 然后重启机器即可
            (4) ulimit -n, 如果设置太小, 会出现 Too Many File Open 的错误
            (5) vm.max_map_count, 设置太小, 在一个主题数超多的 Broker 机器上, 会出现 OutOfMemoryError：Map failed 的问题.
                建议设置为 65536, 方法是修改 /etc/sysctl.conf 文件, 增加 vm.max_map_count=655360, 保存之后, 执行
                sysctl -p 命令使它生效.
            (6) 页缓存大小, 最小值要容纳一个日志段的大小
    4. JVM 层
            (1) 设置堆大小
                    一般将 JVM 堆大小设置成 6～8GB,精确的设置, 查看 GC log(kafkaServergc.log 日志), full gc 后
                存活的对象大小 * 1.5, Full GC 没有被执行过, 可以手动运行 jmap -histo:live < pid > 就能人为触发 Full GC
            (2) GC 收集器的选择
                    建议使用 G1 收集器, 如果经常出现 Full GC, 配置 JVM 参数 -XX:+PrintAdaptiveSizePolicy, 看是哪个线程导致
                Full GC.
                    G1 容易出现大对象的问题, "too many humongous allocations", 存在至少占用半个区域（Region）大小的对象,
                增加堆大小外, 还可以适当地增加区域大小, 设置方法是增加 JVM 启动参数 -XX:+G1HeapRegionSize=N
                
    5. 框架层(Broker)
            (1) 尽力保持客户端版本和 Broker 端版本一致,不一致的客户端和 Broker 之间会发送消息转化.
     
    6. 应用程序层
            (1) 不要频繁地创建 Producer 和 Consumer 对象实例. 构造这些对象的开销很大, 尽量复用它们
            (2) 用完及时关闭. 这些对象底层会创建很多物理资源, 如 Socket 连接, ByteBuffer 缓冲区等.不及时关闭的话, 会造成资源泄露
            (3) 合理利用多线程来改善性能. Kafka 的 Java Producer 是线程安全的, 可以在多个线程中共享同一个实例；
                而 Java Consumer 不是线程安全的
                
    7. 性能指标调优
            (1) 吞吐量调优
                    a. kafka producer 发送消息, 延迟是 2ms ,再等待 8ms, 缓存了 1000 条消息, 再一次性发送.
                    b. Broker 端
                            1. 适当增加 num.replica.fetchers 参数值, 但不超过 cpu 核数,  Follower 副本用多少个线程来拉取消息,
                               默认使用 1 个线程
                            2. 调优 GC 参数避免经常性的 Full GC
                    c. Producer 端
                            1. 适当增加 batch.size 参数值, 由默认的 16KB 增加到 512 KB, 增加消息批次的大小
                            2. 适当增加 lingr.ms 的参数值, 如 10 ~ 100, 增加批次缓存时间
                            3. 设置 compression.type = lz4 或则 zstd, 适配好的压缩算法, 减少网络 I/O 传输量
                            4. 设置 acks = 0, 1
                            5. 设置 retries = 0
                            6. 如果多线程共享同一个 Producer 实例, 增加 buffer.memory 的参数值, 解决出现 
                               TimeoutException：Failed to allocate memory within the configured
                               max blocking time
                    d. Consumer 端
                            1. 采用多 consumer 进程或则线程同时消费数据
                            2. 增加 fetch.min.bytes 参数值, 设置为 1 KB, 默认是 1 字节, 表示只要 Kafka Broker 端积攒了 1 字节
                               的数据, 就可以返回给 Consumer 端
                               
            (2) 延迟调优
                    a. Broker 端
                            1. 适当增加 num.replica.fetchers 参数值, 但不超过 cpu 核数
                            
                    b. Producer 端
                            1. 设置 lingr.ms = 0; 消息尽快发送出去, 停留的时间设置为 0
                            2. 不启动压缩, compression.type = none
                            3. 设置 acks = 1
                            
                    c. Consumer 端
                            1. 设置 fetch.min.bytes = 1
```

### kafka 集群集成监控
```shell
    1. 概念
    2. JMXTool 工具
            (1) JMXTool 工具能够实时查看 Kafka JMX 指标.
            (2) 使用方法
                    查看使用文档
                    > bin/kafka-run-class.sh kafka.tools.JmxTool 
                    
                    查询 Broker 端每秒入站的流量, 通过 BytesInPerSec 的 JMX 指标
                    > bin/kafka-run-class.sh kafka.tools.JmxTool 
                            --object-name kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec 
                            --jmx-url service:jmx:rmi:///jndi/rmi://:9997/jmxrmi
                            --date-format "YYYY-MM-dd HH:mm:ss"
                            --attributes FifteenMinuteRate
                            --reporting-interval 5000
                            
                    参数:
                        --attributes : 指定要查询的 JMX 属性名称, 以逗号分隔的 csv 格式
                        --date-format: 指定显示的日期格式
                        --jmx-url: 指定要连接的 JMX 接口, 默认格式是 service:jmx:rmi:<端口号>/jmxrmi, 如果是在其他主机上运行的
                        　　　　　　 需要带上连接的主机名.
                        --object-name: 指定要查询的 JMX MBean 名称
                        --reporting-interval : 指定实时查询的时间间隔, 默认是每 2 秒一次
                        
    3. kafka Manager
            (1) 用于管理 和监控 Kafka 集群
            (2) 
                第一步: ./sbt clean dist   // 通过 sbt 工具进行编译 kafka Manager
                第二步: 在 Kafka Manager 的 target/universal 目录下找到生成的 zip 文件, 把它解压, 然后修改里面的
                       conf/application.conf 文件中的 kafka-manager.zkhosts 项, 让它指向你环境中的 ZooKeeper 地址,如：
                            kafka-manager.zkhosts="localhost:2181"
                            
                第三步: 运行　kafka Manager
                            > bin/kafka-manager -Dconfig.file=conf/application.conf -Dhttp.port=8080
                            
            (3) 如果只是想　kafka Manager 进行纯监控,不进行设置, 可以修改 config 下的 application.conf 文件, 
                删除 application.features 中的值,如想禁掉 Preferred Leader 选举功能, 可以删除对应
                 KMPreferredReplicaElectionFeature 项. 删除后重启 Kafka Manager, 再次进入到主界面, 
                 Preferred Leader Election 菜单项已经没有
                 
    4. JMXTrans + InfluxDB + Grafana
            
```
