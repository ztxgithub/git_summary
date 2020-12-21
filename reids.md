# reids

## 基础知识
```shell
  1. RTT: Round Trip Time,即客户端从发送请求到接受到服务端响应的时间.
  2. 数据库的并发量是只支持每秒千级, Redis 的并发量是要支持每秒万级并发量
```

## 概要
```shell
    基础篇 1 ~ 10, 
    1. 学习 Redis , 掌握两大维度(应用维度和系统维度), 三大主线(高性能, 高可靠, 高扩展)
            a. 应用维度, 有缓存应用场景(key 过期机制和淘汰机制), 集群应用场景,  用于分布式锁场景, 还有其他的场景, 例如秒杀.
            b. 系统维度,主要包含这 3 大主线
                    第一, 高性能, 有线程模型(处理层), 数据结构(内存层), AOF持久化(存储层), epoll 网络架构(网络层)
                    第二, 高可靠, 主从复制, 哨兵机制
                    第三, 高扩展, 数据分片, 负载均衡.
                 
    2. Redis 问题分析
            直接原因
            (1) 主从库原因, 跟 AOF 重写机制, RDB, AOF, 哨兵机制
            (2) 性能原因, 跟 AOF 重写机制, RDB, 数据结构, 异步机制
            (3) 内存原因, 数据结构, 淘汰机制
            (4) 缓存问题, 淘汰机制
            (5) 切片集群, 数据分布
            
            现象
            (1) 阻塞, 抖动, bigKey 跟性能有关系
            (2) bigKey, CPU 占用飙升跟内存有关系
            (3) 污染, 雪崩, 穿透 跟 缓存有关系
            (4) 秒杀, 数据热点 跟 切片集群有关系
            (5) 数据丢失, 主从不一致 跟主从库 
            
    3. 学习一门技术, 先建立"系统观", 了解总体架构和关键模块, 然后再深入到具体的技术点.
```
## 基础
### 基本架构-键值数据库
```shell
    1. 对于如何使用 Redis, 其到底能做什么, 不能做什么, 需要先搞懂它的数据模型和操作接口.
    2. 数据模型: 对于键值对 key - value , key 类型一般为 string 类型, Redis 支持 value 类型很多,有 String 类型, 列表, 集合, 哈希表等
                而 Memcached 支持 value 类型只能是 string 类型.
    3. 数据操作:
                set/delete/get/scan, 其中 scan 代表根据 key 的范围, 放回对应 value 值.
                
    4. 键值对存储方式,
            1. 保存在内存中, 访问效率高, 访问速度在百 ns 级别, 但掉电数据丢失
            2. 保存在磁盘中, 数据持久化. 访问速度在几 ms 级别,其访问速度慢.
            
            选用的方式要根据实际业务场景, Reids 和 Memcached 都是内存键值数据库, 其 key-value 键值是保存在内存中, 而持久化则是通过
            RDB 和 AOF 来进行的.
            
    5. 基本组件
            (1) 访问框架
                    一般有两种方式, 
                        第一种键值数据库对外提供动态链接库, 应用程序调用动态链接库进行键值数据操作(增删改查). 例如 RocksDB.
                        第二种方式是通过网络框架形式提供键值对操作, 其中网络框架包括 Socket Server 和协议解析, 
                                   例如 Redis 和 Memcached 
                                   
            (2) 索引
                    根据 key 找到对应 value 的存储位置, 像内存键值数据库(例如 Redis, Memcached) 通常采用哈希表作为索引, 
                原因取决与内存的高效随机访问.
                
            (3) 操作模块
                    对于 GET/SCAN, 只需要更具 value 的存储位置返回 value 值.  PUT 和 DELETE 两种操作除了新写入和删除键值外,
                还涉及到分配和释放内存.由于键值数据库的键值对大小通常不一致, 所以一旦保存的键值对数据规模过大, glibc 的分配器会造成
                内存分配问题, 导致严重的内存碎片问题.
                
            (4) 存储模块
                    通过存文件的形式保存在磁盘中,关键是何时进行更新到磁盘中.
                
```
### 键和值的映射关系
```shell
        1. 键和值之间通过哈希表进行映射关联. 需要搞清楚值的类型有很多,可以有字符串,列表,哈希, 有序集合,集合. 而且通过哈希计算出哈希值
           对应的节点包含 key 和对应 value 类型的指针, 可能不同的 key 计算的哈希值相同, 存在哈希冲突, 解决哈希冲突方法是链地址法.
        
        2. 哈希表操作变慢, 可能原因是存在哈希冲突, 哈希值对应链表过长, 或则在进行 rehash 过程　    
```

### Redis 单线程
```shell
    1. Redis 单线程是指网络 IO 和键值对读写由一个线程完成, 而持久化, 异步删除, 集群数据同步则是额外线程执行.
    2. 采用 Redis 单线程的原因是多线程对于临界资源的操作, 需要进行互斥, 随着线程数的增多, 各个线程越容易阻塞, 同时多线程不方便排查
    　　问题和调试.
    3. Redis 虽然采用单线程,但处理速度快的原因, 第一：大部分操作都是在内存中完成, 同时采用高效的数据结构, 例如哈希, 跳表.
       第二: 网络上采用了多路 IO 复用机制, 
    
    4. 网络多路 IO 
            a. accept(), rev() 函数默认情况下会阻塞等待, 这样在一单线程中 accept()/rev() 等待某一个客户端连接(迟迟未成功), 则
            　　其他客户端也无法连接. 所以需要设置这些套接字为非阻塞, 同时配合 IO 多路复用机制, epoll/select, 这样一个线程可以管理
                多个网络 IO
                
    5. 主线程阻塞原因
            (1) 进行持久化时主线程会 fork 子进程进行 RDB 持久化, 而 fork 操作会阻塞主线程, fork 时长跟数据量有关, 数据量越大
            　　 fork 需要的时间越久. 
            
    6.　子进程(主进程 fork 出来)
            (1) bsave 子进程(用于生成 RDB 文件)
            (2) bgrewriteaof(AOF 重写)
            (3) 主从无盘复制子进程(数据同步)
```

## 高可靠
### 数据同步
```shell
    1. Redis 的可靠性
            第一: 数据尽量少丢失, 通过 RDB 和 AOF 备份恢复
            第二: 服务尽量少中断, 通过主备切换, 其中需要主服务和备服务进行数据同步.哨兵机制
            
    2. Redis 主从库模式
            主从库采用读写分离的方式, 避免复杂的同步,协商, 如果是非集中式的节点, 则需要考虑到同一个临界资源的加锁, 节点间协商等问题.
            读操作: 主库, 从库都可以接收. 
            写操作: 只能是主库提供写接口, 先在主库中执行写操作, 之后将写操作同步给从库. 
            
    3. 主从库进行第一次同步过程
            前提: 确定两个 Redis 实例的主从关系, 在从服务上运行 > replicaof 主服务IP 主服务Port 
            第一次同步的三阶段:
            
                第一阶段: 建立连接, 协商同步
                    a. 从实例与主实例建立连接, 从库向主库发送请求命令, psync + [主库的 runID] + [复制进度 offset], 因为是第一次
                       连接,从实例不知道连接主实例的 runID(唯一标识), 所以 runID 设置为 "?", offset 设置为 "-1", 代表第一次复制.
                       
                    b. 主实例收到 psync 命令后, 向从实例发送 FULLRESYNC + [主库的 runID] + [复制进度 offset], 其中 FULLRESYNC
                       表示第一次复制采用的全量复制, 从实例收到这些信息,进行保存.
                       
                第二阶段: 主实例同步全量数据给从实例
                    首先主实例先执行 bsave 命令, 生成 RDB 文件, 再将文件发送给从实例.
                    
                第三阶段: 主实例发送新写命令给从实例
                    在主实例生成 RDB 后, 传输给从实例, 从实例在加载过程中, 主实例接受来自写请求, 先在主实例中执行, 将命令保存在
                    replication buffer 中, 当主实例完成 RDB 文件发送后, 就会把此时 replication buffer 中的修
                    改操作发给从库, 从库执行这些操作, 达到主从一致.
                    
                    replication buffer: 可以通过 client_buffer 配置项来控制这个 buffer 大小. 同时主库会为每一个从库
                    　　　　　　　　　　　　建立一个主从同步的客户端, 每个客户端都有各自的 replication buffer
                    
                    
    4. 主从复制压力
            问题: 如果存在一主多从, 那么主实例忙于不断 fork 子进程生成 RDB 文件和传输 RDB 文件给从实例, 不断 fork 导致主线程阻塞, 
                  使得主线程处理请求慢, 同时占用主线程的网络带宽.
                  
            解决方式:
                  采用 主-从-从 模式, 通过这种级联方式分担主库的数据同步压力, 可以手动选择一个从库1(内存配置较高), 再在其他从库上运行
            replicaof 从库1_IP 从库1_Port, 这样其他的从库就可以跟从库1建立主从关系, 以后由从库1对其他的从库进行数据同步.
            
    5. 主从网络断开恢复
            主实例采用 repl_backlog_buffer 环型缓冲区来进行网络断开恢复后的增量命令的同步. 主库对于将写入操作记录保存到  
       repl_backlog_buffer 环型缓冲区中, 通过 master_repl_offset 偏移量进行记录. 而每个从库自身也保存 slave_repl_offset 偏移量
       用来记录从  repl_backlog_buffer 环型缓冲区同步操作记录, 主从库的连接恢复之后, 从库会给主库发送 psync 命令, 并把自己当前的
       slave_repl_offset 发给主库, 主库只把 master_repl_offset 和 slave_repl_offset 之间的命令操作同步给从库就行.
       
            需要注意:
                repl_backlog_buffer 对应的存储大小一定要合适通过设置 repl_backlog_size = 
                (主库写入命令速度(每秒几条操作) * 操作大小 - 主从库间网络传输命令速度 * 操作大小) * 2, 乘以 2 是为了考虑到突发大请求.
                
                如果配置太小,会导致从库复制的速度赶不上主库, 则会导致全量的复制.
                
                repl_backlog_buffer 是主库中所有从库共享的, 那么从库要进行主从网络断开恢复时, 需要将自己的复制进度(slave_repl_offset)
                发给主库, 主库再根据　master_repl_offset 和 slave_repl_offset 之间的数据进行传输.
                
    6. 数据同步模式
            (1) 全量复制, 第一次连接同步时候, 采用 主-从-从 模式缓解主库压力
            (2) 基于长连接的命令传播, 主从库正常运行后的常规同步阶段
            (3) 增量复制, 应用与主从库断开连接后恢复.
            
    7. 其他
            (1) 一个实例的 Redis 最大十几 GB 左右.
                    
```

### 主从同步问题
```shell
    1. 概念
            (1) Redis 删除过期时间的数据, 同时采用惰性删除(只有当访问时数据过期被删除)和定时删除(每隔 100ms 删除选一小批进行判断删除)
            (2) Redis 3.2 之前, 从库不会判断过期时间, 之间返回.
                Redis 3.2 之后, 从库会判断过期时间, 过期了返回空值. 
    2. 主从数据不一致
            (1) 主从数据不一致是指客户端在从库上读取的值与主库上最新的值不一致.
            (2) 原因: 1. 异步同步, 主从同步是后台异步进行的, 当客户端向主库请求写操作时, 主库会更新本地的操作,并把命令增量同步给从库, 
                        但不会等到从库执行完命令才返回结果给客户端, 而是主库执行完后就返回结果.
                      
                          从库滞后执行同步命令原因:
                                第一: 由于主从库间网络延迟, 导致从库接受到同步命令存在延时.
                                第二: 从库及时收到主库的同步命令时, 正在处理复杂度高的命令(集合操作命令), 这个同步命令就会给阻塞, 直到正在
                                     执行的命令被处理完.
                                     
                      2.脑裂, 客户端分别往 2 个主库发送写数据.
                                 
            (3) 解决方案
                    a. 保证主从库间的网络连接状况良好, 例如主从库部署在同一机房, 或则避免把网络通信密集的应用（例如数据分析应用）和 
                       Redis 主从库部署在一起
                       
                    b. 开发一个外部程序用来监控主从库间的复制进度, 如果某个从库的复制进度差大于设定的阈值, 则可以让客户端不再从这个从库
                       读取数据.
                            每次进行周期性运行这个流程
                                  第一步: 在主库中读取 master_rpl_offset.
                                  第二步: 遍历登记所有从库的 slave_rpl_offset, 
                                         计算复制进度 diff =  master_rpl_offset - slave_rpl_offset, 如果复制进度 diff 大于
                                         设定的阈值, 则不从这个从库中读取数据.
                                         
    3. 读取过期数据
            (1) 读取过期数据: 在主库上实际已经删除了这个数据, 但是在从库上还是能读到这个过期的数据.
            (2) 原因:
                       a. 可能使用了 Redis 3.2 之前的版本, 客户端读了从库的过期时间的数据.
                       b. 客户端向主库使用了 expire + 延迟时间, 此时主库执行了命令, 但同步给从库时有延迟(原因是主从之间进行全量同步), 
                          那么主库过期时间点和从库过期时间点不一致.
                          
            (3) 解决方案
                       a. 使用 Redis 3.2 及以上版本
                       b. 使用 EXPIREAT/PEXPIREAT 命令设置过期时间点, 避免从库上的数据过期时间滞后, 同时让主从库用相同的 NTP 服务器
                          进行同步.
                          
    4. 不合理配置项导致的服务异常
            (1) protected-mode 配置项
                    protected-mode 表示是否限制哨兵实例被其他服务器访问, 部署哨兵实例集群时, protected-mode 要配置 no, 不然哨兵
                实例间就无法通信.同时用 bind 配置项来约束那几个 ip 的哨兵实例进行通信, 来确保安全性
                        protected-mode no
                        bind 192.168.10.3 192.168.10.4 192.168.10.5
                        
            (2) cluster-node-timeout 配置项
                    设置 Redis Cluster 中实例响应心跳消息的超时时间, 应该要设置大点, 如 10 ~ 20 秒
                        
```

### 哨兵机制
```shell
    1. 哨兵机制主要用于主库故障的情况, 进行主从自动转化, 解决三个问题
            第一个问题: 如何确定主库已经出故障了
            第二个问题: 选举(选择哪个从库作为主库)
            第三个问题: 选出来的主库怎么通知其他从库和客户端
            
    2. 哨兵进程的三个功能, 监控, 选主, 通知.
    3. 监控
            (1) 周期性向主从库进行 ping 命令, 如果没有得到响应,则认为 "下线". 其中对于主库的状态判断有 "主观下线" 和 "客观下线".
                一般有多个哨兵实例, 组成哨兵集群, 减少误判的存在. 当对于某一个哨兵判断出主库离线了, 则为 "主观下线", 只有当大部分
                哨兵实例都认为主库下线了, 则认定为 "客观下线".
                
            (2) 哨兵的误判, 可能由于网络延迟, 网络拥塞, 哨兵实例自身的网络连接问题. 误判导致从库自动切换为主库,以及从库中的选主,
                通知到各个从库, 从库与新的主库建立连接, 进行数据同步等会造成额外的开销.
                
    4. 选主
            (1) 哨兵通过筛选(去掉离线的,不稳定的) + 打分(一定的规则), 选出最有的从库升级为主库.
            (2) 筛选
                    先是否在线, 再以外离线的次数(通过超时时间)
            (3) 打分
                    概要
                        依次通过三轮, 从库优先级、从库复制进度以及从库 ID 号, 只要在某一轮中, 有从库得分最高, 那么它就是主库.得分相同
                        进行下一轮
                        
                    
                    第一轮：优先级最高的从库得分最高
                                可以给从库设置 slave-priority 配置项
                                
                    第二轮：和旧主库同步程度最接近的从库得分高
                                根据每个从库中保存的  slave_repl_offset, 哪个离旧主库的 master_repl_offset 最接近.
                                
                    第三轮：ID 号小的从库得分高
                                每个实例都会有一个 ID
                                
```
### 哨兵集群
```shell
    1. 哨兵实例相互发现
            (1) 哨兵实例间相互通信(主库的订阅主题)
                    哨兵实例启动时只需要连接到主库(不需要直到集群中的各个哨兵实例), 再通过主库中的 Pub/Sub 机制,
                而这里频道对应就是 topic , 主库中 “__sentinel__:hello” 的频道, 就是用于各个哨兵实例的配置信息通信.
                
    2.  哨兵实例获取从库的 ip 和　port 
             通过哨兵向主库发送 INFO 命令, 主库给哨兵实例返回从库列表, 哨兵再根据从库列表, 依次与从库建立连接, 
         检测从库的状态.
         
    3.  客户端事件通知(基于 pub/sub)
            哨兵实例是特定模式下的 Redis 实例, 也可以提供 pub/sub 机制, 客户端可以从哨兵的不同的频道订阅消息.其中
            
         主库下线事件(相关的频道)
                 +sdown(实例进入"主观下线"状态)
                 -sdown(实例退出"主观下线"状态)
                 +odown(实例进入"客观下线"状态)
                 -odown(实例退出"客观下线"状态)
                 
         从库重新配置主库事件(相关的频道)
                +slave-reconf-sent(哨兵发送 SLAVEOF 命令重新配置从库)
                +slave-reconf-inprog(从库配置新主库, 但未进行同步操作)
                +slave-reconf-done(从库配置新主库, 并且和主库完成了同步)
                
         新主库切换事件(相关的频道)
                +switch-master(主库地址变化), 订阅到这个消息, 可以从里面解析出新的主库的 ip 和 port
                
         具体流程:
            客户端读取配置文件获取哨兵的地址和端口, 再向哨兵订阅相关的事件.这样不仅可以在主从切换后得到新主库的连接信息, 
         还可以监控到主从库切换过程中发生的各个重要事件.
         
    4. 仲裁出哪个哨兵实例进行主从写好操作
            1. 首先先确定主库是 "客观下线", 当某个哨兵实例发现主库在线异常了("主观下线"), 这时向其他哨兵实例请求命令
              (is-master-downby-addr )判断它们是否也检测到主库下线,如果赞成票超过了哨兵配置文件中的 quorum 值, 则该哨兵实例就认定
               主库为"主观下线"
               
            2. 之后再选择哪个哨兵实例作为 Leader 哨兵, 用于选出新的主库和通知集群中的从库及客户端. 首先每个哨兵自己标注为主库"客观下线",
               发起 Leader 选举,  自己先给自己投上一票, 每个哨兵实例在这一轮主从切换中只能投一张赞成票, 之后其他的过来都投反对票. 
               
               成为 Leader 哨兵,需要满足两个条件
                    第一, 拿到的赞助票数需要是总数的一半以上.
                    第二, 拿到的票数还需要大于等于哨兵配置文件中的 quorum 值
                    
                    
               如果没有不满足条件, 则等待一段时间重新, 重新进行筛选, 需要注哨兵实例的个数, 至少要配置 3 个实例
               
               
    5. 经验
            (1) 同一个集群中所有的哨兵实例的配置要一致, 尤其是主观下线的判断值 down-after-milliseconds
```

## 高扩展
### 切片集群
```shell
    1. 概念
            (1) 一般一个　Redis 实例保存 5 GB ~ 10 GB , 不能影响性能, 持久化时不会阻塞主线程太久.
            (2) 单个 Redis 实例无法满足大数据量, 只能通过分片的形式.
            
    2. 保存更多的数据
            (1) 纵向扩展(scale up), 通过升级单个 Redis 实例的资源配置, 包括内存, 磁盘, CPU．
                    好处: 实施简单, 直接
                    缺点: 
                        第一: 使用 RDB 进行数据持久化时, 会导致 fork 子进程需要的数据量大, 主线程阻塞情况.
                        第二: 纵向扩展会受到硬件和成本的限制.
            (2) 横向扩展(scale out), 横向增加 Redis 实例的个数, 进行数据分片
                    这会涉及到两个问题, 第一个数据进行分片, 如何分布到各个实例上. 第二个客户端怎么确定要访问数据在哪个 Redis 实例上. 
    3. 建立索引(Redis Cluster 方案)
            (1) Redis 切片集群中共有 16384(2^14) 个哈希槽, 键值对映射到哈希槽, 先根据键值对的 key, 进行　CRC16 计算, 得到后 % 16384.
            (2) 哈希槽映射到 Redis 实例
                    方法一: 自动分配, 调用  cluster create, 会自动将各个 Redis 实例间建立连接, 再自动将 16384 槽平均分配到
                     　　　　N 个实例中, 这个适用于每个 Redis 实例的资源配置差不多.
                     
                     方法二: 手动建立连接. 可以指定某个 Redis 实例分配多少个哈希槽. 先使用 cluster meet 命令手动将各个 Redis 实例
                     　　　　建立连接, 再通过使用 cluster addslots 命令, 指定每个实例分配哈希槽的数量.
                       
                            注意: 手动方式, 必须将 16384(2^14) 个哈希槽分配完.
                            
   　4. 客户端确定索引哪个 Redis 实例
            (1) 在刚开始 Redis 分片集群建立时, 每个 Redis 实例可以知道自己分配了那些哈希槽, 通过 pub/sub, 向各个实例发送自己的
            　　　哈希槽信息, 这样就完成了哈希槽信息的扩散, 每个实例就有所有哈希槽的映射关系. 当客户端连接到某个 Redis 实例后,
                 客户端会收到哈希槽的信息以及哈希槽对应 Redis 实例的信息.
                 
            (2) 在实例和哈希槽关系发生变化时(第一种是集群新增/减少 Redis 实例, 第二种是负载均衡, 将哈希槽重新在各个实例分配一次), 
                这时各个 Redis 实例已经更新了, 但客户端的缓存还没有更新.这时提供重定向机制.
                
                重定向机制:
                    a. 情况一: 各个 Redis 实例都已经完成哈希槽的部署, 当客户端根据原先的缓存向一个实例发送数据, 但这个实例中没有对应的
                    　　　　　　哈希槽, 则将这个哈希槽对应的 Redis 实例发送给客户端(通过　MOVED 命令响应结果, 结果中包含新实例的地址)
                               
                               GET hello:key  // 客户端
                               (error) MOVED 13320 172.16.19.5:6379  // Redis 实例返回
                               
                              客户端收到 MOVED 响应后, 会向新的 Redis 实例发送请求, 并且更新本地缓存.
                              
                    b. 情况二:  各个 Redis 实例数据正在迁移中, 那么客户向老的 Redis 实例请求, 会收到 ASK 的响应结果
                                
                                GET hello:key
                                (error) ASK 13320 172.16.19.5:6379
                                
                                这时客户端先向新的实例发送 ASKING 命令, 询问是否可以操作, 之后在发送操作命令, 这种情况下并不会更新
                                客户端本地哈希槽分配缓存, 下一次还是会先去原来的 Redis 实例请求.
```

### Codis 集群
```shell
    1. 概念
    2. Codis 集群框架
            (1) 组成
                    a. codis server: 二次开发的 Redis 实例(处理具体的数据读写请求), 增加支持数据迁移操作功能
                    b. codis proxy： 接收客户端请求, 并把请求转发给 codis server
                    c. Zookeeper 集群：用于保存集群元数据, 例如数据索引位置信息和 codis proxy 信息
                    d. codis dashboard 和 codis fe: 共同组成集群管理工具, 
                            codis dashboard 负责执行集群管理工作, 包括新增删除 codis server 和　codis proxy, 同时进行数据迁移.
                            codis fe 负责提供 dashboard 的 Web 操作界面, 便于直接在 web 上进行集群管理操作.
                            
            (2) 过程
                    第一步: 使用 codis dashboard 设置 codis server 和 codis proxy 的访问地址, 将 codis server 和 codis proxy
                           建立关系.
                    第二步: 客户端直接和 codis proxy 建立连接.
                    第三步: codis proxy 接收到请求, 会查询请求数据和 codis server 的映射关系, 并把请求转发给相应的 codis server
                           进行处理, 当 codis server 处理完请求后, 会把结果返回给 codis proxy, proxy 再把数据返回给客户端
                           
    3 关键技术原理
            (1) 数据分布
                    第一步: Codis 集群一共有 1024 个 Slot, 编号依次是 0 到 1023, 可以手动分配给 codis server, 也可以让 
                    　　　　codis　dashboard 进行自动分配, 如 dashboard 把 1024 个 Slot 在所有 server 上均分
                    第二步: 客户端要读写数据时, 使用 CRC32 计算数据 key 的哈希值, 并将哈希值对 1024 取模, 而取模后的值, 则对应 
                           Slot 的编号. 再根据第一步分配的 Slot 和 server 对应关系, 就知道数据保存在哪个 server 上.
                           
                    数据路由表:  slot 和 codis server 的映射关系, 在 codis dashboard 上分配好路由表后, dashboard 会把路由表
                               发送给 codis proxy, codis-proxy 会把路由表缓存在本地, 当它接收到客户端请求后，直接查询本地的路由表,
                               就可以完成正确的请求转发. 同时也会将路由表信息保存在 Zookeeper 中
                               
                    数据分布在 codis 和 Redis Cluster 区别
                        Codis 中的路由表是通过 codis dashboard 分配和修改的, 当路由表数据发生改变时,  codis dashboard 会把修改后
                        的路由表信息发送给 codis proxy, 和　Zookeeper.
                        
                         Redis Cluster 中, 数据路由表是通过每个实例相互间的通信传递, 最后会在每个实例上保存一份. 当数据路由信息
                         发生变化时, 就需要在所有实例间通过网络消息进行传递. 当实例数量较多时, 同步时间长且消耗较多的网络资源.
                         
            (2) 集群扩容
                    Codis 集群扩容包含 codis server 的增加和 codis proxy 的增加
                    
                    codis server 增加的流程
                        第一步: 启动新的 codis server, 将它加入集群
                        第二步: 把部分数据迁移到新的 server
                                    a. 在源 server 上, Codis 从要迁移的 Slot 中随机选择一个数据, 发送给目的 server
                                    b. 目的 server 收到数据后, 会给源 server 返回确认消息. 这时源 server 会在本地
                                       将刚才迁移的数据删除.
                                    c. Codis 不断重复单个数据的迁移过程, 直到要迁移的 Slot 中的数据全部迁移完成
                                    
                        单数据迁移模式
                                同步迁移: 数据从从源 server 发送给目的 server 的过程中, 源 server 是阻塞的, 无法处理新的请求操作. 
                                         其中涉及到数据在源 server 序列化、网络传输、在目的 server 反序列化, 以及在源 server 删除.
                                         如果是 bigkey 的迁移, 会阻塞很久时间.
                                         
                                异步迁移(Codis 使用)
                                        当源 server 把数据发送给目的 server 后, 就可以处理其他请求操作, 当收到目的 server 的 ack
                                        响应消息后, 源 server 把迁移的数据删除, 同时在迁移的过程中, 迁移的数据在源数据中是只读的, 
                                        当一个 bigkeys 都迁移完, 才删除源 server 的 bigkeys
                                        
                                        
                    codis proxy 增加的流程
                        直接启动 proxy, 再通过 codis dashboard 把 proxy 加入集群, 同时 proxy 的信息也保存在 Zookeeper 上. 这样
                        客户端就可以从 Zookeeper 上读取 proxy 访问列表.
                        
            (3) 集群的可靠性
                    codis server
                         　Codis 集群中, 部署 server group(group 里通常都是一主一从) 和哨兵集群，　实现 codis server 的主从切换，
                         　提升集群可靠性
                         
                    codis proxy 和 Zookeeper
                            首先 Zookeeper 集群使用多个实例来保存数据, 只要有超过半数的 Zookeeper 实例可以正常工作, Zookeeper 
                            集群就可以提供服务, 保证了可靠性.proxy 发生故障, 只需要重启就可以了, proxy 重新通过 codis dashboard 
                            从 Zookeeper 集群上获取路由表.
     
    4. 方案
            (1) 从稳定性和成熟度来看, Codis 使用 proxy 的无状态设计和 Zookeeper 自身的稳定性, Codis 较好.
            (2) 从业务应用客户端兼容性来看, 连接单实例的客户端可以直接连接 codis proxy, 而原本连接单实例的客户端要想连接 Redis Cluster ,
                需要开发新功能
            (3) 从使用 Redis 新命令和新特性, Codis server 是基于开源的 Redis 3.2.8 开发,不支持 Redis 后续的开源版本中的新增命令和
                数据类型BITOP、BLPOP, 以及事务性的命令 multi 和 exec, 想使用开源 Redis 版本的新特性, Redis Cluster 较好.
            (4) 从数据迁移性能维度来看, Codis 能支持异步迁移, 异步迁移对集群处理正常请求的性能影响要比使用同步迁移的小, 使用 Codis 较好
                                        
```

## 性能和内存(16~22)
### 概念
```shell
    1. 影响 Redis 性能的因素
            (1) Redis 内部的阻塞式操作
            (2) CPU 核和 NUMA 架构的影响
            (3) Redis 关键系统配置
            (4) Redis 内存碎片
            (5) Redis 缓冲区
```

### Redis 内部的阻塞式操作
```shell

    阻塞式操作
        1. 和客户端交互
                (1) 网络 IO, 采用多路 IO 复用(epoll), 不会导致 Redis 阻塞.
                (2) 键值对的增删改查操作, 复杂度高(O(N))的增删改查会阻塞 Redis, 例如集合全量查询操作 hgetall, smembers, 以及聚合统计
                　　操作, 例如交集, 并集, 差集.
                    
                    集合删除操作也会存在阻塞, 应用程序删除键值对, 释放内存, 但操作系统需要将这个内存块插入到空闲内存块的链表中, 以便进行管理
                    和分配, 如果一下子删除大量的键值对, 会使得空闲链表的操作时间增加, 造成阻塞. Hash 集合 100 万元素(每个元素 8 Bytes),
                    需要 1 秒左右, 而 Redis 是微妙级的响应.
                    
                    第一个阻塞点: 集合全量查询和聚合操作
                    第二个阻塞点: bigkey 的删除操作
                    第三个阻塞点: 清空数据库(flushdb 和 flushall)
                    
        2. 和磁盘交互
                (1) bsave(生成 RDB 快照文件), brewriteaof(AOF 文件重写), 这些写到磁盘使用子进程, 不会阻塞到主线程.
                (2) 一个同步写磁盘的操作的耗时大约是 1～2ms.
                
                    第四个阻塞点: AOF 日志同步写.
                    
        3. 主从数据同步
                (1) 对于从库来讲, 在接受到主库的 RDB 文件时, 要先使用 flushdb 清空自身的数据库(会阻塞从库的主线程)
                (2) 对于从库来讲,
                
                         第五个阻塞点: 从库在数据同步时加载 RDB 文件会阻塞从库的主线程.
                         
    解决方案(异步执行)
        1. 采用异步执行策略需要是非关键路劲, 关键路劲是请求侧需要放回结果的数据, 例如读操作就是关键路劲. 这样 
           第一个阻塞点: 集合全量查询和聚合操作 和 第五个阻塞点: 从库在数据同步时加载 RDB 文件会阻塞从库的主线程. 这样关键路劲不能用
           异步执行. 其他的阻塞点可以线程异步执行的方式.
           
        2. 主线程通过系统函数 pthread_create 创建 AOF 日志写操子线程, 键值对删除子线程(Redis 4.0后), 文件关闭子线程. 主线程收到
        　　这些命令封装为任务加到对应的任务队列, 对应的线程从任务队列中取并执行.
        
            键值对删除用 UNLINK 命令
            清空数据库加上 async
                > FLUSHDB ASYNC
                > FLUSHALL AYSNC
                
                
        3. 无法进行异常执行的解决方案
                (1) 对于 Redis 4.0 之前无法异步删除 bigkey 键值对, 可以先用 SCAN 命令读取一部分数据, 在进行删除, 可以避
                    一次性删除大量 key 给主线程带来的阻塞. 例如　Hash 类型的 bigkey 删除, 可以使用 HSCAN 命令, 每次从 Hash 集合中
                    获取一部分键值对(例如 200 个), 再使用 HDEL 删除这些键值对, 这样就可以把删除压力分摊到多次操作中,
                    每次删除操作的耗时就不会太长, 也就不会阻塞主线程了
             
                (2) 集合全量查询和聚合操作：可以使用 SCAN 命令，分批读取数据, 再在客户端进行聚合计算
                (3) 从库加载 RDB 文件：把主库的数据量大小控制在 2~4GB 左右, 以保证 RDB 文件能以较快的速度加载

```

### CPU 核和 NUMA 架构的影响
```shell
    1. 概念
            (1) 一个 CPU 处理器会有多个物理核, 每个物理核都有各自私有的缓存(L1, L2 大小在 KB级别, 访问速度在 10 纳秒), 多个物理核之间
            　　 会共享 L3(容量在几 MB ~ 几十MB), 一个物理核可以分为 2 个逻辑核, 而程序是运行在逻辑核上, 而一个服务器会有
                多个 CPU 处理器(CPU Socket), 它们之间通过总线连接.
            (2) 在多 CPU 架构上, 应用程序可以在不同的处理器上运行, 例如, Redis 可以先在 CPU 1 上运行一段时间, 
                然后再被调度到 CPU 2 上运行
                
            (3) 如果应用程序先在一个 CPU Socket 上运行, 并且把数据保存到内存, 然后被调度到另一个 Socket 上运行, 应用程序再进行内存
                访问时，就需要访问之前 Socket 上连接的内存, 这种访问属于远端内存访问. 和访问 Socket 直接连接的内存相比, 远端内存访问
                会增加应用程序的延迟, 这个就是 NUMA 架构(非统一内存访问架构, Non-Uniform Memory Access)
                
            (4)  99% 尾延迟: 99% 尾延迟代表一个值, 是将所有请求的处理延迟从小到大排个序, 99% 的请求延迟小于的值. 例如有 1000 个请求,
                            假设按请求延迟从小到大排序后, 第 991 个请求的延迟实测值是 1ms, 而前 990 个请求的延迟都小于 1ms,
                            这里的 99% 尾延迟就是 1ms
                            
    2. CPU 多核对 Redis 性能的影响
            (1) 每一个 Redis 被调度不同的 CPU 运行,　一些请求就会受到运行时信息(栈指针, 寄存器)、指令和数据重新加载过程的影响,
                这就会导致某些请求的延迟明显高于其他请求
                
            (2) 采取解决方案: 将 Redis 实例和某个特定 CPU 逻辑核进行绑定减少上下文切换,避免较高的尾延迟
                    > taskset -c 0 ./redis-server  // -c 表示绑定到 0 号核上
                    
    3. 多 CPU 的 NUMA 架构
            (1) 网络中断程序是从网卡中读取数据, 存到 CPU SOCKET 下的内存中, 并通过 epoll 去通知 Redis 取数据, 那么网络中断程序和
            　　Redis 需要绑定到同一 CPU SOCKET 中．
            (2) 注意的是　NUMA 架构下 CPU 核的编号方法, 先给每个 CPU Socket 中每个物理核的第一个逻辑核依次编号,再给每个 CPU Socket
                中的物理核的第二个逻辑核依次编号.
                > lscpu
                
                NUMA node0 CPU(s): 0-5,12-17    // 一个 CPU SOCKET 有 6 个物理核
                NUMA node1 CPU(s): 6-11,18-23
                
    4. 风险
            (1) 把 Redis 绑定到一个逻辑核上, 会导致子进程, 后台线程和 Redis 主线程争夺 CPU 资源, 这样会导致 Redis 主线程阻塞, 
            　　外部请求延迟增加.
    5. 解决方案
            (1)  解决方案一：　一个 Redis 实例绑定同一个物理核(绑定 2 个逻辑核)
                    taskset -c 0,12 ./redis-server　　 // 0,12 是代表同一个物理核
                    
            (2) 优化 Redis 源码(Redis 6.0 后, 可以支持 CPU 核绑定的配置操作, 之前的只能修改源代码了)
                    通过代码把子进程和后台线程绑到不同的 CPU 逻辑核上, 这样网络中断程序和　Redis 主线程就可各自在同一个 CPU SOCKET
                下运行.
                
                    通用做法:
                        数据结构及函数：
                            cpu_set_t 数据结构：是一个位图，每一位用来表示服务器上的一个 CPU 逻辑核
                            CPU_ZERO 函数：以 cpu_set_t 为输入参数，把位图中所有的位设置为 0
                            CPU_SET 函数：以 CPU 逻辑核编号和 cpu_set_t 位图为参数，　把位图中和输入的逻辑核编号对应的位设置为 1
                            sched_setaffinity 函数：进程 / 线程 ID 号和 cpu_set_t 为传入参数, 检查 cpu_set_t 中哪一位为 1, 
                                                   就把输入的 ID 号所代表的进程 / 线程绑在对应的逻辑核上
                                                   
                    过程：
                        后台线程
                        
                            //线程函数
                            void worker(int bind_cpu){
                                cpu_set_t cpuset; //创建位图变量
                                CPU_ZERO(&cpu_set); //位图变量所有位设置0
                                CPU_SET(bind_cpu, &cpuset); //根据输入的 bind_cpu 编号，把位图对应为设置为1
                                
                                //把程序绑定在 cpu_set_t 结构位图, 0 : 表示指定的是当前进程
                                sched_setaffinity(0, sizeof(cpuset), &cpuset); 
                                //实际线程函数工作
                            }
                             
                            int main(){
                                pthread_t pthread1
                                //把创建的 pthread1 绑在编号为3的逻辑核上
                                pthread_create(&pthread1, NULL, (void *)worker, 3);
                            }
                        
                           Redis 线程绑定操作：　在 bio.c 文件中的 bioProcessBackgroundJobs 函数中创建了后台线程. 
                                            bioProcessBackgroundJobs 函数类似于刚刚的例子中的 worker 函数，在这个函数中实现绑核
                                            四步操作
                                            
                                            
                        子进程
                            int main(){
                            
                                //用fork创建一个子进程
                                pid_t p = fork();
                                
                                if(p < 0){
                                    printf(" fork error\n");
                                }
                                else if(!p) //子进程代码部分
                                {
                                    cpu_set_t cpuset; //创建位图变量
                                    CPU_ZERO(&cpu_set); //位图变量所有位设置0
                                    CPU_SET(3, &cpuset); //把位图的第3位设置为1
                                    sched_setaffinity(0, sizeof(cpuset), &cpuset); //把程序绑定在3号逻辑核
                                    //实际子进程工作
                                    exit(0);
                                }.
                                ..
                            }
                            
                            
                            Redis 子进程
                                生成 RDB　子进程, rdb.c 文件：rdbSaveBackground 函数
                                AOF 重写子进程, aof.c 文件：rewriteAppendOnlyFileBackground 函数
                                        
                                        
                       
```

### 系统排查 Redis 变慢
```shell
    1. 问题现象发现
            (1) 查看 Redis 响应延迟跟基准性能比起来是否变慢. 
                    a. 获取基准性能的响应延迟方法
                            从 2.8.7 版本开始, redis-cli 命令提供了–intrinsic-latency 选项, 可以用来监测和统计测试期间内的
                        最大延迟, 测试时长可以用–intrinsic-latency 选项的参数来指定, 一般设置为 120 秒
                            > ./redis-cli --intrinsic-latency 120
                              Max latency so far: 17 microseconds.
                              Max latency so far: 44 microseconds.
                              Max latency so far: 94 microseconds.
                              
                    b. 将 Redis 运行时的延迟和基准性能比较, 如果运行时的延迟是基准性能 2 倍以上, 则认定是 Redis 变慢. 
                       如果还要考虑网络因素的话, 可以用 iPerf　这样的工具
                         
    2. 原因排查
            (1) 从 Redis 自身操作特性
                    a. 慢查询命令
                           原理: 主要是对于不同的操作命令, Redis 执行的速度不同, 例如　Value 类型为 String 时, GET/SET 操作对应
                           　　　时间复杂度为 O(1), 而 Value 类型为 Set, SMEMBERS 操作时间复杂度为 O(N)
                           检测方法: 可以通过 Redis 日志, 或者是 latency monitor 工具, 查询变慢的请求, 根据请求对应的具体命令以及
                                    官方文档, 确认下是否采用了复杂度高的慢查询命令
                                    
                           解决方案:
                                    解决方案一: 用其他高效命令代替, 如果需要返回一个 SET 中的所有成员时, 不要使用 SMEMBERS 命令,
                                              而是要使用 SSCAN 多次迭代返回, 避免一次返回大量数据，造成线程阻塞
                                    解决方案二: 当需要执行排序、交集、并集操作时，可以在客户端完成，而不要用 SORT、 SUNION、SINTER 
                                               这些命令, 以免拖慢 Redis 实例
                                               
                                    不要使用 KEYS 命令, 是遍历存储键值对进行匹配的.
                                    
                    b. 过期 key 删除操作
                            原理: 在 Redis 4.0 以前, 删除键值对没有采用后台线程, 而是在 Redis 主线程中操作,大量 bigKeys 会导致
                            　　　Redis 主线程阻塞.
                            
                                 过期 key 删除的算法(每 100 ms 执行)
                                    1. 采样 ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP(默认是 20) 个的 key, 并将其中过期的 key 
                                       全部删除
                                    2. 如果超过 25% 的 key 过期了, 则重复删除的过程, 直到过期 key 的比例降至 25% 以下
                                    
                                 如果第二条被执行, 就会在不断进行删除 key , Redis 就会阻塞(Redis 4.0 以前), 这个现象是频繁使用
                                 带有相同时间参数的 EXPIREAT 命令设置过期 key
                                 
                            解决方案: 
                                   如果一批 key 的确是同时过期, 可以在 EXPIREAT 和 EXPIRE 的过期时间参数上, 加上一个一定大小范围
                                   内的随机数，这样，既保证了 key 在一个邻近时间范围内被删除, 又避免了同时过期造成的压力
                                   
                                   
            (2) 文件系统: AOF 模式
                    1. 概念
                            a. 写到磁盘上, 依赖与文件系统的 write 和 fsync 两个系统调用. write 只是把日志记录写到内核缓冲区,
                               就可以返回,不需要等待日志实际写回到磁盘上, 而 fsync 需要把日志记录写回到磁盘后才能返回,时间要长
                               
                            b. AOF 写回策略及对应的系统调用执行情况
                                    no : 只调用 write 系统函数, 由后台操作系统本身周期性将日志写回到磁盘
                                    everysec: 由 Redis 显式每秒调用 fsync 系统函数, 将日志写回到磁盘, 这个操作通过后台线程去进行
                                    　　　　　　每秒 fsync 到磁盘
                                    always: 每次执行一个操作,　都由 Redis 显式调用 fsync 系统函数将日志写回到磁盘, 如果还是用
                                    　　　　　后台线程, 那么 Redis 无法及时地知道每个操作是否已经完成了, 所以还是在 Redis 主线程进行．
                                    
                    2. 风险点
                           当Redis 主线程再次把新接收的操作记录写回磁盘时会监控 fsync 系统调用的情况, 如果发现上一次的 fsync 还没有
                           执行完, 主线程会阻塞. 而如果 AOF 重写子进程在同时进行时, 会对占用大量磁盘 IO, 那么后台 AOF 子进程的
                           fsync 也会被阻塞, 导致 Redis 阻塞. AOF 重写子进程进行是避免 AOF 日志文件不断增大, 需要生成新的小容量 
                           AOF 文件.
                           
                    3. 排查过程
                            看看　Redis appendfsync 配置项 appendfsync no, appendfsync everysec, appendfsync always, 
                        确认业务方对数据可靠性的要求, 不一定都要设置为 appendfsync always, 例如场景是缓存的, 还可以设置配置项
                        no-appendfsync-on-rewrite yes　代表在进行 AOF 重写子进程时, 不调用后台线程进行 fsync 操作, 写命令只需要
                        写到内存即可.
                        
            (3) 操作系统: 
                    a. swap
                        原因:
                            1. Redis 实例自身使用大量的内存, 导致物理机器的可用内存不足
                            2. 和 Redis 实例在同一机器上的进程进行大量的文件读写操作, 会占用大量的内存, 导致分配给 Redis 实例
                            　　的内存量变少, 进而触发 Redis 实例中 swap 
                            
                        参看 Redis 是否发生 swap
                        
                            第一步. $ redis-cli info | grep process_id
                                    结果: process_id: 5332
                                    
                            第二步 $ cat /proc/5332/smaps | egrep '^(Swap|Size)'
                                    结果:
                                        Size: 4 kB
                                        Swap: 4 kB
                                        Size: 4 kB
                                        Swap: 0 kB
                                        Size: 462044 kB
                                        Swap: 462008 kB
                                    
                                    Size 表示的是 Redis 实例所用的一块内存大小, Swap 则表示这块 Size 大小的内存区域有多少已经被
                                    换出到磁盘上
                        解决方案
                            1. 增加机器内存
                            2. 如果是 Redis 切片集群, 则考虑增加 Redis 实例个数, 减少每个 Redis 实例占用的内存量.
                            
                    b. 内存大页(一页 2MB)
                        原理:
                            虽然内存大页机制, 在 Redis 初始化时, 可以减少内存分配的次数, 但在 Redis 数据持久化时, 此时外部的
                            请求需要修改数据, 因为采用了写时复制, 这时候会申请内存进行数据拷贝, 如果采用内存大页则会申请 2MB, 而
                            修改的数据只是几个字节, 常规的页是 4KB.
                            
                        参看系统是否执行内存大页
                            > cat /sys/kernel/mm/transparent_hugepage/enabled
                               结果 always madvise [never]  // 代笔 never 禁用
                               
                        解决方案
                            禁用内存大页 > echo never  > /sys/kernel/mm/transparent_hugepage/enabled
                            
                            
    3. 实践
            (1) 慢查询命令排查
                    第一种排查: 通过慢查询日志来分析, 需要先设置好 2 个参数, slowlog-log-slower-than 和 slowlog-max-len
                                slowlog-log-slower-than: 慢查询日志记录了操作大于多少微秒的命令
                                slowlog-max-len: 记录的慢查询日志条数上限, 默认是 128, 建议使用 1000, 超过上限后会进行覆盖.
                                
                                使用 slowlog get 命令获取最近一条慢查询的日志信息
                                    SLOWLOG GET 1
                                    1) 1) (integer) 33 //每条日志的唯一ID编号
                                       2) (integer) 1600990583 //命令执行时的时间戳
                                       3) (integer) 20906 //命令执行的时长，单位是微秒
                                       4) 1) "keys" //具体的执行命令和参数
                                          2) "abc*"
                                       5) "127.0.0.1:54793" //客户端的IP和端口号
                                       6) "" //客户端的名称，此处为空
                                       
                                       
                    第二种排查: latency monitor 监控工具(2.8.13)
                                    设置 latency-monitor-threshold , 即命令超过多少微妙就被监控
                                        config set latency-monitor-threshold 1000
                                
                                    使用  latency latest 命令查询最新超过阈值的情况
                                        latency latest
                                        1) 1) "command"
                                           2) (integer) 1600991500 //命令执行的时间戳
                                           3) (integer) 2500 //最近的超过阈值的延迟
                                           4) (integer) 10100 //最大的超过阈值的延迟
                                           
                                           
            (2) 排查 bigkeys 
                    
                    第一种:
                    使用 redis-cli 自带 --bigkeys 选项
                        ./redis-cli --bigkeys
                            -------- summary -------
                            Sampled 32 keys in the keyspace!
                            Total key length in bytes is 184 (avg len 5.75)
                            
                            //统计每种数据类型中元素个数最多的 bigkey
                            Biggest list found 'product1' has 8 items
                            Biggest hash found 'dtemp' has 5 fields
                            Biggest string found 'page2' has 28 bytes
                            Biggest stream found 'mqstream' has 4 entries
                            Biggest set found 'userid' has 5 members
                            Biggest zset found 'device:temperature' has 6 members
                            
                            //统计每种数据类型的总键值个数，占所有键值个数的比例，以及平均大小
                            4 lists with 15 items (12.50% of keys, avg size 3.75)
                            5 hashs with 14 fields (15.62% of keys, avg size 2.80)
                            10 strings with 68 bytes (31.25% of keys, avg size 6.80)
                            1 streams with 4 entries (03.12% of keys, avg size 4.00)
                            7 sets with 19 members (21.88% of keys, avg size 2.71)
                            5 zsets with 17 members (15.62% of keys, avg size 3.40)
                            
                        缺点: 只能统计出每种数据类型个数最多的情况, 无法得到大小排在前 N 位的 bigkey. 同时对于集合类型, 这个
                              只能统计集合元素个数, 而不是实际占用的内存量.一个集合中的元素个数多, 并不一定占用的内存就多
                              
                              ./redis-cli --bigkeys 会对 Redis 实例的性能产生影响, 如果数据量大的话, 运行完会阻塞主线程, 干扰
                              正常的业务操作, 解决方案是 在 Redis 实例业务压力的低峰阶段进行扫描查询, 同时使用 -i 参数控制扫描间隔,
                              即每扫描 100 次暂停 100 ms
                              
                              ./redis-cli --bigkeys -i 0.1
                              
                              
                    第二种: 自己开发对应的统计程序
                                使用 scan 命令对数据进行扫描, 然后用 type 命令获取返回的每一个 key 的类型, 对每个类型进行处理
                                 如果是 string 类型, 使用 strlen 获取字符串长度
                                 如果是集合类型
                                    1. 如果事先直到集合元素的大小, 则集合占用内存 = 集合个数 * 集合元素大小
                                            List 类型：LLEN 命令；
                                            Hash 类型：HLEN 命令；
                                            Set 类型：SCARD 命令；
                                            Sorted Set 类型：ZCARD 命
                                            
                                    2. 如果不知到元素的大小, 用 MEMORY USAGE 命令查询每个键值对占用的内存空间.
                                            例如：
                                                MEMORY USAGE user:info
                                                (integer) 315663239
                              
                  
                            
```

### Redis 内存碎片
```shell
    1. 概念
            (1) 当数据被删除时, Redis 释放的空间不会里面归还给操作系统, 而是由内存分配器统一管理, 此时这部分删除内存还是归在这个
                Redis 进程中.
                
            (2) Redis 中可利用的内存有大量不连续的内存空间(内存碎片), 这导致 Redis 实际能够保存的数据减少, 并且降低 Redis 运行机器的
                成本回报率(例如正好租用了一台 16GB 内存的云主机运行 Redis, 但是却只保存了 8GB 的数据, 租用这台云主机的成本回报率
                也会降低一半)
                
    2. 内存碎片原因
            (1) 操作系统内存分配机制(内因)
                    内存分配器分配策略一般按固定大小分配内存, Redis 可以使用 libc、jemalloc、tcmalloc 多种内存分配器来分配内存,
                    默认使用 jemalloc, jemalloc 可以安装一系列的大小划分内存空间, 例如 8 字节、16 字节.... 2KB、4KB、8KB 等
                    当程序申请的内存最接近某个固定值 时, jemalloc 会给它分配相应大小的空间, 这样就存在内存碎片的风险, 因为 Redis
                    删除键操作以及键值对的大小不一样.
                    
            (2) 键值对大小不一样和删改操作(外因)
            
    3. 判断是否有内存碎片
            (1) 在 redis-cli 运行
                    INFO memory
                    # Memory
                    used_memory:1073741736
                    used_memory_human:1024.00M   // Reids 保存数据申请的内存
                    used_memory_rss:1997159792
                    used_memory_rss_human:1.86G  // 操作系统实际分配给 Redis 的物理内存空间, 里面就包含了碎片
                    …mem_fragmentation_ratio:1.86
                    
                    看 mem_fragmentation_ratio 的指标, mem_fragmentation_ratio =  used_memory_rss / used_memory
                    
                    mem_fragmentation_ratio 大于 1 但小于 1.5 : 属于正常情况
                    mem_fragmentation_ratio 大于 1.5: 内存碎片率超过 50 %, 需要采取措施来减低内存碎片率.
                    mem_fragmentation_ratio < 1: 说明实际的物理内存不够用, 触发了 swap 内存数据置换.
                    
                    
    4. 清理内存碎片
            (1) Redis 4.0-rc3 版本以后提供内存碎片自动清理. 主要是通过数据拷贝迁移, 将不连续的空闲的内存区变为连续的空闲的内存区. 
                同时进行自动内存清理也是有代价的, 在数据迁移拷贝时, Redis 主线程需要阻塞, 同时数据整理和拷贝需要顺序, 这样会大大
                减低 Redis 处理请求的性能.
                
            (2) 解决方案: 通过设置参数, 来控制碎片清理的开始和结束时机, 以及占用的 CPU 比例, 从而减少碎片清理对 Redis 本身请求处理的
                         性能影响
                         
                         // 开启自动内存碎片清理
                         config set activedefrag yes
                         
                         // active-defrag-ignore-bytes 和 active-defrag-threshold-lower 同时满足才进行清理
                         active-defrag-ignore-bytes 100mb：表示内存碎片的字节数达到 100MB 时, 开始清理；
                         active-defrag-threshold-lower 10：表示内存碎片空间占操作系统分配给 Redis 的总空间比例达到 10% 时, 开始清理
                         
                         // CPU 占比参数
                         active-defrag-cycle-min 25： 表示自动清理过程所用 CPU 时间的比例不低于 25%, 保证清理能正常开展
                         active-defrag-cycle-max 75：表示自动清理过程所用 CPU 时间的比例不高于 75%, 一旦超过，就停止清理, 
                                                     从而避免在清理时, 大量的内存拷贝阻塞 Redis, 导致响应延迟升高
                                                     
            (3) 技巧
                    在实践过程中遇到 Redis 性能变慢, 记得查看日志是否正在进行碎片清理. 如果 Redis 的确正在清理碎片, 调小 
                    active-defrag-cycle-max 的值，以减轻对正常请求处理的影响
                         
```

### Redis 缓冲区
```shell
    1. 概念
            (1) 缓冲区的应用场景
                    场景一: 是用来暂存客户端发送的命令数据, 或则是服务器端返回给客户端的数据结果
                    场景二: 主从节点间进行数据同步时, 用来暂存主节点接收的写命令和数据
                    
            (2) Redis 服务器端允许为每个客户端最多暂存 1GB 的命令和数据
                    
    2. 客户端的输入输出缓冲区
            (1) Redis 给每个连接的客户端都设置了一个输入缓冲区和输出缓冲区, 输入缓冲区暂存的是客户端的请求命令, Redis 从输入缓冲区
                取命令进行处理, 再将结果暂存到输出缓冲区.
                
            (2) 客户端的输入缓冲区溢出原因
                    a. 写入 bigkey, 如一下子写入了多个百万级别的集合类型数据
                    b. Redis 处理请求慢, 主线程出现了间歇性阻塞
                    
                输入输出缓冲区溢出的后果
                    某个客户端输入输出缓冲区溢出, 会导致 Redis 把客户端的连接关闭, 使得业务程序无法存取数据.
                    
                    当多个客户端连接占用的内存总量, 超过了 Redis 的 maxmemory 配置项时（例如 4GB）, 就会触发 Redis 进行数据淘汰.
                    一旦数据被淘汰出 Redis,要访问这部分数据, 就需要去后端数据库读取, 这就降低了业务应用的访问性能
                    
                输入缓冲区查看
                    127.0.0.1:6379> CLIENT list
                    id=6 addr=127.0.0.1:58808 fd=8 name= age=14327 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 
                    qbuf-free=32742 obl=0 oll=0 omem=0 events=r cmd=client user=default
                    
                    id=6 addr=127.0.0.1:58808 : 客户端连接的信息
                    cmd: 表示客户端最新执行的命令, 例如 client
                    qbuf: 表示输入缓冲区已经使用的大小, 例如 26 Bytes
                    qbuf-free: 表示输入缓冲区尚未使用的大小, 例如 32742 Bytes
                    
                解决方案
                    a. 没法调整 Redis 为客户端的输入缓冲区的大小.
                    b. 控制好输入缓冲区溢出的原因.
                    
            (3) 输出缓冲区
                    输出缓冲区构成
                        输出缓冲区包括两部分：一部分是一个大小为 16KB 的固定缓冲空间, 用来暂存 OK 响应和出错信息；
                                          另一部分是一个可以动态增加的缓冲空间, 用来暂存大小可变的响应结果
                                          
                    输出缓冲区溢出原因
                           第一: 服务器端返回 bigkey 的大量结果
                           第二: 执行 MONITOR 命令, 这个命令会持续将 Redis 执行命令存到输出缓冲区, 在生成环境建议不使用.
                           第三: 输出缓冲区大小设置得不合理. 设置输出缓冲区要考虑客户端的类型.
                           
                                如果客户端的类型是普通的, 可以对输出缓冲区不进行限制, 因为这类型的客户端是阻塞读取请求结果的, 
                                只有等待收到请求的结果, 才会发送下一个请求命令.
                                                                   1    2 3 4
                                    > client-output-buffer-limit normal 0 0 0
                                    
                                    
                                如果客户端的类型是订阅的, 一旦订阅的 Redis 频道有消息, 服务器端都会通过输出缓冲区把消息发给客户端,
                                如果频道消息较多的话, 也会占用较多的输出缓冲区空间
                                                                   1    2   3   4
                                     client-output-buffer-limit pubsub 8mb 2mb 60
                                     
                                     第 2 个 参数:设置的是缓冲区大小限制
                                     第 3 个 参数和第 4 个参数分别表示缓冲区持续写入量限制和持续写入时间限制
                                     
                                    pubsub 表示客户端类型是订阅；
                                    8mb 表示输出缓冲区的大小上限为 8MB, 一旦实际占用的缓冲区大小要超过 8MB, 服务器端就会直接关闭
                                        客户端的连接； 
                                    2mb 和 60 表示, 如果连续 60 秒内对输出缓冲区的写入量超过 2MB 的话, 服务器端也会关闭客户端连接
                                    
                                    
    3. 主从集群缓冲区
            (1) 概念
                    复制缓冲区: 指在主从数据同步时, 进行全量复制, 在主数据向从数据传输 RDB 的同时, 主数据得接受客户端的请求命令, 
                              这时需要保存在复制缓冲区, 等 RDB 文件传输完成后, 再发送给从节点去执行, 主节点上会为每个从节点都维护
                              一个复制缓冲区. 
                              
                    复制积压缓冲区: 指的是网络恢复时的增量数据同步的缓冲区, 所有主从网络断开恢复时用的复制积压缓冲区是同一个.
                              
                    
                    
            (2) 复制缓冲区
                    复制缓冲区溢出原因:
                            在主从全量复制时, 从节点接受 RDB 文件较慢时, 而主节点接收到大量的写命令, 写命令在复制缓冲区中就会越积越多.
                            最终导致溢出
                            
                    复制缓冲区溢出后果:
                            导致主从连接断开, 使得主从数据同步失败.
                            
                    解决方案:
                            1. 控制主节点的键值对数据量, 使得主从数据同步快, 一般控制在 2~4 GB
                            2. 设置  client-output-buffer-limit 配置项, 来控制合理的复制缓冲区大小. 
                            
                                    > config set client-output-buffer-limit slave 512mb 128mb 60
                                    
                                    slave 参数表明该配置项是针对复制缓冲区的。
                                    512mb 代表将缓冲区大小的上限设置为 512MB；
                                    128mb 和 60 代表的设置是: 如果连续 60 秒内的写入量超过 128MB 的话, 也会触发缓冲区溢出
                            3. 控制从节点的数量, 来避免主节点中复制缓冲区占用过多内存的问题
                            
            (3) 复制积压缓冲区
                    复制积压缓冲区溢出原因:
                        网络一直断开, 环型的复制积压缓冲区旧的操作数据会被新的操作数据覆盖, 导致主从节点间重新开始执行全量复制.
                        
                    解决方案:
                        设置 repl_backlog_size 配置项                                   
```

## 缓存(23~27)
### 缓存概念
```shell
    1. 缓存一些列内容
            工作原理(如何工作)
            替换策略(缓存满了, 该怎么办)
            异常处理(缓存一致性, 缓存穿透, 缓存雪崩, 缓存击穿)
            扩展机制
            
    2. 访问速度
            (1) cpu 访问速度是几十 ns, 32 MB
            (2) 内存访问速度是几百 ns, 32 GB
            (3) 磁盘访问速度是 5 ms, 4 TB 
            
    3. 缓存的特征
            (1) 在分层系统中，数据暂存在快速子系统中有助于加速访问.
            (2) 缓存容量有限, 缓存写满时, 数据需要被淘汰
            
    4. 缓存是否命中分类
            (1) 缓存命中：在 Redis 中有对应的数据
            (2) 缓存缺失: 在 Redis 中没有对应的数据, 需要从后端的数据库(MySQL)读取数据, 再进行缓存更新.
            
    5. 旁路缓存
            (1) 旁路缓存是指读取缓存, 读取数据库和更新缓存的操作都需要在应用程序中来完成, 例如:
                    读取缓存: 当应用程序需要读取数据时, 在代码中显式调用 Redis 的 GET 操作接口进行查询
                    读取数据库: 缓存缺失, 应用程序需要再和数据库连接, 从数据库中读取数据
                    更新缓存: 在应用程序中显式地调用 SET 操作接口, 把更新的数据写入缓存.
                    
    6. 缓存的类型
            (1) 只读缓存(Redis 支持)
                    内容:
                        不修改 Redis 中的数据, 而是直接修改后端的数据库, 再删除 Redis 中的键值对. 当再次访问时, 缓存中没有对应的数据,
                        触发缓存缺失, 此时从数据库中读取对应的键值对, 并写入 Redis.
                        
                    好处:
                        所有最新的数据都在数据库中, 保证数据的可靠.
                        
            (2) 读写缓存
                    内容:
                        除了读请求会发送到缓存进行处理, 同时在缓存中也处理写请求, 此时缓存中保存着最新数据.数据更新到数据库的时机分
                        2 种, 一种是同步直写, 一种是异步写回.
                        
                    写回策略
                        a. 同步直写(Redis 支持), 写请求发给缓存的同时, 也同步发给后端数据库进行处理, 等到缓存和数据库都写完数据, 
                        　　才给客户端返回.
                                     优点: 同步更新到后端的数据库(MySQL), 保证数据可靠, 缓存宕机或发生故障, 最新的数据仍然保存在
                                           数据库中
                                     缺点: 降低缓存的访问性能, 在进行写操作需要等到后端数据库操作完才返回, 增加缓存的响应延迟.
                                     
                        b. 异步写回(Redis 不支持), 优先考虑响应延迟, 所有写请求都直接在缓存中处理, 直接返回给客户端.等到数据要被
                        　　缓存中淘汰出来时, 
                                    在写回到后端数据库.
                                    优点: 响应快
                                    缺点: 当缓存宕机时, 还没来得及同步到后端数据库的数据容易丢失．
                                    
                                    
            (3) 场景
                    对写请求进行加速, 选择读写缓存, 例如商品大促, 商品的库存信息会一直被修改
                    
                    写请求很少, 或只需要提升读请求的响应速度, 选择只读缓存, 例如 在短视频 App 的场景中, 虽然视频的属性有很多,一般
                    确定后, 修改并不频繁               
```

### 替换策略
```shell
    1. 概念
            (1) 缓存数据的淘汰机制包含 2 步, 第一：根据一定的策略, 筛选出不常用的数据. 
                                          第二: 将这些数据从缓存中删除, 为新来的数据腾出空间
                                          
            (2) 长尾效应: 即八二原理, 20 % 的数据贡献了 80% 的访问, 例如热门的电商商品.
            (3) 重尾效应: 20% 的数据可能贡献不了 80% 的访问, 而剩余的 80% 数据反而贡献了更多的访问量, 例如视频播放网站, 每个用户
            　　　　　　　　都有自己个性化的需求.
            
            (4) LRU 算法: Least Recently Used, 最近最少被访问原则. 
            　　　　　　　　基本实现原理是通过链表, 表尾是最不常使用的数据, 表头是最新访问的数据, 当要在链表中访问某个数据时, 将其移动到
            　　　　　　　　表头, 淘汰的时候只要删除表尾的节点, 再把新的数据查到表头. 这种做法会带来很多链表移动操作, 很耗时, 影响 Redis
                          的性能.
                          
                          
                LRU 算法 Redis 实现
                        Redis 每个数据(键值对数据结构 RedisObjec) 有个 lru 字段,　记录了最新被访问的时间戳, 在开始要进行缓存数据淘汰
                时, 随机选出 N 个数据, 作为候选集合, 把这 N 个数据中选出 lru 字段最小的数据进行淘汰. 
                
            (5) 缓存中的干净数据是指缓存中的数据与后端数据库是一致的. 脏数据是指缓存中的数据有被修改, 还没同步到后端数据库.
            　　　　　　　　　
                                          
    2. 设置缓存的大小
            (1) 建议把缓存容量设置为总数据量的 15% 到 30%, 兼顾访问性能和内存空间开销
                    > CONFIG SET maxmemory 4gb
                    
    3. 淘汰策略
            (1) 不进行数据淘汰
                    a. noeviction : 一旦缓存数据写满, 再有写请求来时, Redis 不再提供服务, 而是直接返回错误
                    
            (2) 进行数据淘汰
                    a. 淘汰范围是设置了过期时间的键值对
                            volatile-ttl 策略: 对设置过期时间的键值对进行淘汰, 越早过期的越先被删除.
                            volatile-random 策略: 在设置过期时间的键值对中, 进行随机删除
                            volatile-lru 策略: 使用 LRU 算法筛选设置了过期时间的键值对
                            volatile-lfu 策略:  使用 LFU 算法选择设置了过期时间的键值对
                    b. 淘汰范围是所有键值对
                            allkeys-random 策略: 从所有键值对中随机选择并删除数据
                            allkeys-lru 策略 : 使用 LRU 算法从所有键值对中筛选出来并删除
                            allkeys-lfu 策略 : 使用 LFU 算法从所有键值对中筛选出来并删除
                            
            (3) 选择
                    a. 优先选择 allkeys-lru 策略, 应用场景是业务数据中有明显的冷热数据区分
                    b. 如果业务应用中的数据访问频率相差不大，没有明显的冷热数据区分, 那么使用 allkeys-random
                    c. 如果业务中有置顶的需求, 比如置顶新闻, 置顶视频等, 可以使用 volatile-lru 策略, 不给这些置顶数据设置过期时间,
                       这样置顶的数据不会被删除, 其他数据在设置过期时间　根据 LRU 规则进行筛选
                       
    4. 被淘汰的数据处理
            原理上被淘汰的数据如果是脏数据应该还要写回到后端数据库. 当时 Redis 缓存并没有,都是直接从缓存中删除, 所以在使用 Redis 缓存时,
            如果数据被修改了, 需要在数据修改时就将它写回数据库
            
```

### 缓存异常
```shell
    1. 概念
            (1) 数据的一致性, 有两种情况：
                    第一种: 缓存里有数据, 则缓存中数据与后端的数据库数据是一致的.
                    第二种: 缓存中没有数据, 则数据库中的值是最新的.
                    
            (2) 缓存雪崩,　缓存击穿, 缓存穿透会导致大量的请求积压到数据库层, 使得数据库宕机或是故障 
            (3) 布隆过滤器
                   a. 使用布隆过滤器可以快速判断某个数据是否存在(也可以判断数据是否在数据库中),　先将数据通过布隆过滤器进行标记
                            第一步: 使用 N 个哈希函数, 分别计算这个数据的哈希值, 得到 N 个哈希值
                            第二步: 把这 N 个哈希值对 bit 数组的长度取模, 得到每个哈希值在数组中的对应位置
                            第三步: 把对应位置的 bit 位都设置为 1
                            
                   b. 查看某个数据是否在数据库, 则先对数据进行 N 个哈希函数, 对各个哈希值取数组长度模, 再查看各个 bit 是否为 1, 
                      只要一个不为 1, 则数据不存在.
                    
                    
    2. 缓存的不一致情况
           a. 新增数据不存在缓存不一致情况, 新增数据只需要将数据写到数据库中就完成了. 如果要访问时, 产生缓存缺失, 这时会将数据从数据库中
           　　更新到缓存中. 
           b. 删除/修改操作会造成数据不一致情况
                    如果是先删除缓存值, 再更新数据库. 就会存在删除缓存成功, 但是更新数据库失败, 则请求再次访问缓存时, 导致缓存缺失, 从
                    　　　　　　　　　　　　　　　　　　数据库中读出旧值.
                    
                    如果是先更新数据库值, 在删除缓存值. 就会出现更新数据库成功, 但删除缓存失败, 那么数据库里的值是新值, 但缓存中值还是
                    　　　　　　　　　　　　　　　　　　　旧值, 请求访问时缓存命中, 就得到了旧值.
                    
                    
           原因: 更新数据库操作和删除缓存不是一个原子操作, 不再同一个事务中. 可能存在其中某一个操作失败, 或则多个线程
           　　　并发操作缓存(访问或则更新)
           
       解决方案
            1. 针对其中某一个操作失败的情况, 可以使用重试机制, 把要删除的缓存值或者是要更新的数据库值暂存到消息队列中(kafak 消息队列),
               当应用没有成功地删除缓存值或者是更新数据库值时, 可以从消息队列中重新读取这些值, 然后再次进行删除或更新.如果成功地删除或
               更新, 就把这些值从消息队列中去除, 以免重复操作, 这就保证数据库和缓存的数据一致。否则还需要再次进行重试。
               如果重试超过的一定次数，还是没有成功，就需要向业务层发送报错信息
               
            2. 对于多线程访问缓存时
                    第一种方案: 先更新数据库值, 再删除缓存值.  
                                    这样存在小概率的问题是当线程 A 已经更新数据库值了, 此时线程 B 访问缓存, 这个时候就会读到旧值, 
                                    不过等线程 A 删除缓存值后, 后续就能触发缓存缺失导致读到新值.
                               
                    第二种方案: 如果先删除缓存值, 再更新数据库.
                                    会存在一种情况线程 A 已经删除了缓存值, 接下来要更新数据库时, 线程 B 要访问该值, 但已经删除了, 
                                    触发了缓存缺失又重新把旧值读到缓存中, 此时线程 A 更新完数据库, 但缓存中还是旧值.
                                    
                                    解决方法是延迟双删, 等线程 A 更新完数据库后, sleep 一段时间(等待线程 B 读数据库, 并把值写到缓存后)，
                                   　再进行删除.
                                   
       应用
            Redis作为只读缓存时, 还有优先使用先更新数据库再删除缓存的方法, 原因:
                第一: 先删除缓存值再更新数据库, 有可能导致请求因缓存缺失而访问数据库, 给数据库带来压力
                第二: 如果业务应用中读取数据库和写缓存的时间不好估算, 延迟双删中的等待时间就不好设置
                
    3. 缓存雪崩
            (1) 缓存雪崩是指大量的应用请求无法 Redis 缓存中进行处理, 应用将大量请求发送到数据库层, 导致数据库层的压力激增.
            (2) 原因及解决方案
                    a. 缓存中有大量数据同时过期, 导致大量请求无法得到处理, 造成缓存缺失, 大量的数据向后端数据库请求.
                        
                       解决方案:
                            第一种: 在缓存侧, 避免对大量数据设置同一个过期时间, 可以给这些数据的过期时间增加较小的
                            　　　　随机数(例如 1 ~ 3 秒)
                            
                    b. Redis 缓存实例发生故障宕机, 导致无法处理请求, 一下子积压到数据层. 
                         解决方案:
                            第一种：是在业务系统中实现服务熔断或请求限流机制, 当检测到 Redis 缓存实例宕机, 数据库所在机器的负载压力
                                     突然增加(例如每秒请求数激增), 发生了缓存雪崩.
                                     
                            第二种: 事前预防, 通过主从方式构建 Redis 缓存集群, Redis 缓存的主节点故障宕机, 从节点自动切换为主节点,
                                            继续提供缓存服务.
                                            
    4. 缓存击穿
            (1) 缓存击穿是指某个热点数据过期失效, 导致访问该数据的大量请求，一下子都发送到后端数据库, 使得数据库压力激增
            (2) 解决方案: 对热点数据不进行过期时间设置.
            
    5. 缓存穿透
            (1) 缓存穿透是指要访问的数据既不在 Redis 缓存中, 也不在数据库, 这样流程是先在缓存访问, 没有缓存缺失, 再从数据库查找, 也没有
            　　 做无用功.
            (2) 原因
                    a. 业务层误操作：缓存中的数据和数据库中的数据被误删除
                    b. 恶意攻击：专门访问数据库中没有的数据
            (3) 解决方案
                    第一种: 一旦发生缓存穿透, 则在 Redis 中缓存一个空值或是和业务层协商确定的缺省值（例如，库存的缺省值可以设为 0）
                    第二种: 使用布隆过滤器快速判断数据是否存在, 避免从数据库中查询数据是否存在, 减轻数据库压力. 在把数据写入数据库时，
                    　　　　使用布隆过滤器做个标记. 当缓存缺失后, 先通过布隆过滤器查询下, 不用再去数据库中查询
                    第三种: 在请求入口的前端进行请求检测, 把不合法的请求筛选掉(请求参数不合理, 请求参数是非法值, 请求字段不存在)
                                    
```

### 缓存污染
```shell
    1. 概念
            (1) 缓存污染是缓存中存在某些数据, 被应用层访问的次数很少, 但还继续占用缓存空间.
            
    2. 缓存污染的后果
            当无效的数据占慢缓存, 再往缓存中写入新数据时, 就需要先把这些数据逐步淘汰出缓存, 这就会引入额外的操作时间开销,
            进而会影响应用的性能
            
    3. 解决方案
           事先就把不常使用的数据从缓存中淘汰.
           
           第一种方案： LRU 策略(更关注数据的时效性)
                            RedisObject.lru 记录最新被访问的时间戳, 淘汰掉最小的值, 但是它无法解决扫描式单次查询操作, 即应用对
           　　　　　　 大量数据进行一次读取, 这样大量的数据 lru 字段都会被更新, 也在很长一段时间内这些数据不被淘汰, 这样还是无法解决
                       缓存污染
                       
                     
                       
           第二种方案: LFU 策略(更关注数据的访问频次)
                        (1) LFU 策略是在 LRU 策略继承上增加了数据被访问的次数, 当使用 LFU 策略筛选淘汰数据时, 首先会根据数据的访问
                            次数进行筛选, 把访问次数最低的数据淘汰出缓存. 如果两个数据的访问次数相同, LFU 策略再比较这两个数据
                            的访问时效性, 把距离上一次访问时间更久的数据淘汰出缓存
                            
                        (2) Redis LFU 策略实现
                                Redis 在实现 LFU 策略时, 只是把原来 24bit 大小的 lru 字段, 又进一步拆分成了两部分
                                     ldt 值：lru 字段的前 16 bit, 表示数据的访问时间戳；
                                     counter 值：lru 字段的后 8bit, 表示数据的访问次数
                                  
                        (3) LFU 细节:     
                                a. counter 最大只能存 255, Redis 并没有采用数据被访问一次, 对应的 counter++, 而是采用更优化的
                                　　计数方式
                                
                                        //每次数据被访问时, 取范围(0, 1) 的随机数 r
                                        double r = (double)rand()/RAND_MAX;
                                        ...
                                        // 用计数器当前的值乘以配置项 lfu_log_factor 再加 1, 再取其倒数
                                        double p = 1.0/(baseval*server.lfu_log_factor+1);
                                        
                                        // 如果 p > 随机数 r, 则 counter++
                                        if (r < p) counter++;
                                        
                                     通过配置 lfu_log_factor 越大, 实际数据访问次数越多, counter 才能达到 255, 例如
                                     当 lfu_log_factor 为 1 时, 实际访问次数为 100K 后, counter 值就达到 255 
                                     当 lfu_log_factor 为 100 时, 当实际访问次数为 10M 时, counter 值才达到 255
                                     
                                     推荐使用 lfu_log_factor 为 10, 百、千、十万级别的访问次数对应的 counter 值有明显的区分,
                                     百级别访问对应的 counter 是 10 , 千级别访问对应的 counter 是 18, 
                                     十万级别的访问对应的 counter 是 142
                                     
                                b. 如果有些数据在短时间内被大量访问后就不会再被访问, 这样 couter 会很大, 会被留在缓存中, 占用空间,
                                   这时采用 counter 值的衰减机制, 使用衰减因子配置项 lfu_decay_time 来控制访问次数的衰减,计算
                                   当前时间和数据最近一次访问时间的差值(以分钟为单位) 再把这个差值除以 lfu_decay_time 值, 
                                   其结果就是数据 counter 要衰减的值
                                        // 衰减值
                                        decay_count = (当前时间戳 - RedisObject.ldt)/60/lfu_decay_time
                                        
                        (4) 业务应用中有短时高频访问的数据, 建议采用 volatile-lfu 策略, 根据这些数据的访问时限设置它们的过期时间
                                            
```

## Redis 锁(29~30)
### 概念
```shell
    1. 保证并发访问方式, 第一种是加锁, 第二种是原子操作
    2. 加锁操作
            加锁是在访问 Redis 缓存时, 客户端需要先加锁, 访问成功后再解锁.
            缺点:
                (1) 频繁的加锁解锁, 减低系统的并发访问性能.
                (2) Redis 客户端要加锁时, 需要用到分布式锁, 而分布式锁实现复杂, 需要用额外的存储系统来提供加解锁操作, 同时还要考虑
                    获取锁的客户端异常了, 锁能够及时释放.
```

### 原子操作
```shell
    1. Redis 的两种原子操作方法, 实现各个客户端对临界区代码的互斥使用, 即临界区代码内是一个事务, 要么都执行, 要么都不执行.
            第一种是 单命令操作, 把多个操作在 Redis 中实现成一个操作
            第二种是 把多个操作写到一个 Lua 脚本中, 以原子性方式执行单个 Lua 脚本
            
    2. 单命令操作
            (1) 在实际应用中, 数据修改时可能包含多个操作, 至少包括读数据、数据增减、写回数据三个操作, Redis 提供 INCR/DECR 命令, 
                把这三个操作转变为一个原子操作了
                
                例如: 客户端可以使用 decr id 直接完成对商品 id 的库存值减 1 操作, 多个客户端执行也会进行互斥.
                
                问题: 单命令操作是 Redis 内置的命令, 只支持简单的增减数据, 不支持一些列复杂的操作.
                
                
    3. Lua 脚本
            (1) 适用与一系列复杂的操作, 整个 Lua 脚本作为一个整体执行(EVAL 命令来执行脚本), 在执行的过程中不会被其他命令打断, 
                保证了 Lua 脚本内的所有操作都能执行成功, 不被打断. 
                
            (2)  Lua 脚本主要是为了防止同一个客户端多个线程同时执行一些列操作, 可以采用 lua 脚本, Redis 也会依次串行执行脚本
                 代码, 避免并发操作带来的数据错误
                 
                 例如对于 社交网络中的每分钟点赞次数限制, 单个客户端每分钟最多只能点赞 20 下, 需要 incr count 和 当点赞为 1 时设置
                 一分钟超时, 两个是要原子进行的.
                 
                    local current
                    current = redis.call("incr",KEYS[1])
                    if tonumber(current) == 1 then
                    redis.call("expire",KEYS[1],60)
                    end
                 
                 执行 lua 脚本
                 > redis-cli --eval lua.script keys , args
                 
            (3) 把很多操作都放在 Lua 脚本中原子执行, 会导致 Redis 执行脚本的时间增加, 也会降低 Redis 的并发性能
                所以在编写 Lua 脚本时, 只将并发控制的操作写入脚本中      
```

### Redis 分布式锁
```shell
    1. 概念
            (1) Redis 本身可以被多个客户端共享访问, 正好就是一个共享存储系统, 就可以用来保存分布式锁
            (2) 单机锁和分布式锁的区别
                    a. 单机锁就是代表一个变量的值, 变量值为 0 时, 表示没有线程获取锁. 所以一个线程调用加锁操作, 先检查锁变量值是否为 0.
                       如果是 0, 就把锁的变量值设置为 1, 表示获取到锁.
                       如果不是 0, 就返回错误信息, 表示加锁失败, 已经有别的线程获取到锁了
                       
                    b. 分布式锁也是相同的判断逻辑, 只不过锁变量需要由一个共享存储系统(Redis)来维护, 
                       加锁操作变成了读取, 判断,设置共享存储系统中的锁变量值. 
                       释放锁的操作就变成设置共享存储系统中的锁变量值.
                       
            (3) Redis 在实现分布式锁时要求
                    第一: 分布式锁的加锁和释放锁的过程, 涉及多个操作. 在实现分布式锁时, 保证这些锁操作的原子性；
                    第二: 保证共享存储系统的锁变量数据可靠性, 不能使得 Redis 宕机导致锁变量值异常或则缺失.
                    
    2. 基于单个 Redis 节点实现分布式锁
            (1) 将 Redis 中键保存为锁的变量名, Redis 中的值保存为锁的值. 客户端 A 和 C 同时请求加锁, 因为 Redis 使用单线程处理请求,
                所以，即使客户端 A 和 C 同时把加锁请求发给 Redis, Redis 也会串行处理它们的请求
                
            (2) 保障锁操作的原子性
                    第一种方式: 使用 setnx 命令加锁, del 命令解锁. 
                                    del 命令解锁就是将锁变量删除, 而 setnx 命令则是当这个值不存在时, 进行创建并赋值, 存在不操作.
                                    
                               风险:
                                    1. 如果客户端加锁成功, 但自己出现异常, 无法解锁, 导致其他客户端无法申请到锁. 解决方案是设置
                                       锁的过期时间.
                                       
                                    2. 存在客户端 A 执行 SETNX 命令加锁后, 客户端 B 可以执行 DEL 命令释放锁, 客户端 A 的锁就被
                                       误释放。如果客户端 C 正好也在申请加锁，就可以成功获得锁，进而开始操作共享数据.解决方案是
                                       需要能区分不同客户端的锁操作, 可以通过每个客户端在申请锁时, 对锁变量设置唯一值代表是当前操作的
                                       客户端, 在释放锁操作时, 客户端需要判断，当前锁变量的值是否和自己的唯一标识相等, 
                                       只有在相等的情况下, 才能释放锁
                                       
                               实现:
                                    // 加锁, unique_value作为客户端唯一性的标识, 过期时间为 10s
                                    SET lock_key unique_value NX PX 10000
                                    
                                    lua 脚本
                                        // 释放锁 比较unique_value是否相等，避免误释放, 读取锁, 判断值, 操作锁都是原子性需要用 lua 脚本
                                        // KEYS[1]表示 lock_key, ARGV[1]是当前客户端的唯一标识
                                        if redis.call("get",KEYS[1]) == ARGV[1] then
                                            return redis.call("del",KEYS[1])
                                        else
                                            return 0
                                        end
                                        
                                    > redis-cli --eval unlock.script lock_key , unique_value
                                   
            (3) 如果单实例的 Redis 宕机了, 则分布式锁就无法使用了.
                                    
    3. 基于多个 Redis 节点实现高可靠的分布式锁
            (1) 分布式锁算法 Redlock
                    基本思路是让客户端和多个独立的 Redis 实例依次请求加锁, 如果客户端能够和半数以上的实例成功地完成加锁操作, 
                则代表客户端申请锁成功.
                
            (2) 实现
                    第一步: 客户端获取当前时间
                    第二步: 客户端按顺序依次向 N 个 Redis 实例执行加锁操作, 对每个 Redis 实例加锁操作和在单实例上执行的加锁操作一样,
                           使用 SET 命令(SET lock_key unique_value NX PX 10000), 同时如果某个 Redis 实例故障, 导致加锁失败, 
                           则要进行超时时间进行重试加锁, 还不成功就进行下一个 Redis 实例加锁.加锁操作的超时时间需要远远地小于锁的有效时
                            间, 一般也就是设置为几十毫秒.
                    第三步: 确定最终加锁是否成功
                                需要满足两个条件:
                                    条件一：在 Redis 集群中, 客户端成功获取锁超过半数；
                                    条件二：客户端获取锁的总耗时没有超过锁的有效时间
                                    
                                满足条件后, 需要重新计算这把锁的有效时间(锁的过期时间 - 客户端获取锁的总耗时), 如果小于共享数据的操作
                                的操作时间, 则也进行释放锁.
                                
                            释放锁: 执行释放锁的 Lua 脚本就可以了
            
```

## 集群(31~37)
### 事务机制
```shell
    1. 概念
            (1) DISCARD 命令: 只是用来主动放弃事务执行, 将暂存的命令队列清空. 
            (2) WATCH 机制
                    主要适用于事务过程中, 在事务开始时对某一个或则多个键进行监控, 当事务调用 EXEC 命令执行时, WATCH 机制会先检查
                    监控的键是否被其它客户端修改(其他客户端没有进行事务操作,只是单纯 set 命令), 如果修改了则放弃这个事务.
    2. Redis 实现事务
            第一步: 使用一个命令来表示一个事务开始, MULTI 命令
            第二步: 客户端将一系列操作(增删改查)发送到 Redis, Redis 只是将这些命令暂存到命令队列中, 不执行
            第三步: 客户端向服务器端发送提交事务的命令, 让服务器最终执行一系列命令, EXEC
            
    3. Redis 事务属性
            (1) 原子性
                    a. 第一种情况(可以保证原子性): 在执行 EXEC 命令前, 客户端发送的操作命令本身就有错误(命令不存在或则语法错误), 
                       此时该不正常的命令入命令队列时, Redis 会检测到错误, 即使之后有正确的命令入命令队列, 等到执行 EXEC 时,
                       Redis 不会执行命令队列中任何一个命令,  直接返回事务失败.
                       
                    b. 第二种情况(不保证原子性), 对于命令和操作的数据类型不匹配(例如 lpop 命令对 String 类型数据进行操作), 
                       这种 Redis 检测不出来, 只有到真正执行时才报错, 但还是会把正确的命令执行完. 在这种情况下, 事务的原子性就无法
                       得到保证. 
                        
                       Redis 没有提供回滚功能, 将已经修改的数据恢复到事务执行前的状态.
                       
                    c. EXEC 命令执行时实例故障, 如果开启了 AOF 日志, 可以保证原子性. 使用 redis-check-aof 工具检查 AOF 日志文件, 
                       把已完成的事务操作从 AOF 中删除, 这样使用 AOF 恢复实例时, 可以保证恢复到事务前的状态.
                       
            (2) 一致性
                    a. 命令入队时就报错, 事务没有执行, 具备一致性
                    b. 命令入队时没报错, 实际执行时报错, 有错误的命令不会被执行, 正确的命令可以正常执行, 也不会改变数据库的一致性
                    c. EXEC 命令执行时实例发生故障, 也能保证一致性.
                            
            (3) 隔离性
                    a. 并发操作在 EXEC 命令前执行, 隔离性的保证要使用 WATCH 机制来实现, 没有使用 WATCH 机制则隔离性无法保证.
                    b.  并发操作在 EXEC 命令后执行, 隔离性可以保证.
                    
            (4) 持久性
                    事务的持久性属性没办法做到实时同步,根据持久化的策略.
                    
    4. Redis 的事务机制可以保证一致性和隔离性，但是无法保证持久性

```
### 脑裂
```shell
    1. 概念
        (1) 脑裂: 主从集群中同时存在两个主库, 使得客户端无法往哪一个主库进行写数据, 从而导致数据丢失.
        (2) 脑裂是发生在主从切换的过程中．
        (3) 主库发生假故障原因
                a. 和主库部署在同一台服务器上的其他程序临时占用了大量资源(例如 CPU 资源),使得主库资源受限, 无法及时恢复哨兵心跳.
                b. 主库自身遇到了阻塞的情况, 例如, 处理 bigkey 或是发生内存 swap, 使得短时间内无法响应心跳
        
    2. 脑裂原因
            (1) 原主库假故障, 哨兵实例集群发现原主库假故障(可能服务器 cpu 爆满, 无法给哨兵心跳响应), 判定了客观下线, 
                在进行主从切换过程中, 原主库又正常了, 可以接受客户端的请求.
                
    3. 脑裂后果
            会导致数据丢失, 原主库在主从切换时还在接受客户端的请求, 原主库有新的请求, 再新的主库产生后, 哨兵实例会通知原主库 执行 
            slave of 命令, 和新主库重新进行全量同步, 此时原主库需要清空本地的数据, 加载新主库发送的 RDB 文件, 这样就导致数据丢失
            
    4. 解决方案
            使用  min-slaves-to-write 和 min-slaves-max-lag 配置项
            min-slaves-to-write：　设置主库能进行数据同步的最少从库数量；
            min-slaves-max-lag： 设置主从库间进行数据复制时, 从库给主库发送 ACK 消息的最大延迟（以秒为单位）
            
            即主库连接的从库中至少有 write 个从库, 和主库进行数据复制时的 ACK 消息延迟不能超过 min-slaves-max-lag 秒, 如果有一条
            不满足, 那么主库就不接受客户端的请求．
            
            例如: 
            将 min-slaves-to-write 设置为 1, 把 min-slaves-max-lag 设置为 12s, 把哨兵的 down-after-milliseconds 设置为 10s,
            主库因为某些原因卡住了 15s, 导致哨兵判断主库客观下线，开始进行主从切换. 因为原主库卡住了 15s, 没有一个从库能和原
            主库在 12s 内进行数据复制, 原主库也无法接收客户端请求了
            
            正常配置：
                假设从库有 K 个, 可以将 min-slaves-to-write 设置为 K/2+1（如果 K 等于 1, 就设为 1）,
                将 min-slaves-max-lag 设置为十几秒(例如 10 ~ 20s)
```

### 秒杀
```shell
    1. 秒杀场景的负载特征
            (1) 第一个特征: 瞬时并发量高. 需要用 Redis 先拦截大部分请求, 避免直接将请求发送给数据库. 
            (2) 第二个特征: 读多写少, 而且读的操作都是简单的查询操作.
            
    2. 秒杀阶段
            (1) 秒杀前, 用户会不断刷新商品详情页, 这时候不需要 Redis , 需要将商品详情页的页面元素静态化, 使用 CDN 或则浏览器把静态化
            　　 　　　　元素缓存起来, 这样秒杀前就不需要到达服务端.
            (2) 秒杀活动开始, 进行库存查验, 库存扣减和订单处理三个阶段. 
                            
                            库存查验, 库存扣减不适合在数据库里操作,应该在 Redis 中处理, 
                　　　　　　　并且库存查验, 库存扣减是原子性的操作, 如果它们放在数据库中, 对数据库的压力会很大, 同时需要额外的开销, 需要
                　　　　　　　保证 Redis 的库存量和数据库的库存量是一致的, 还有下单量超过实际库存量, 因为数据库的处理速度较慢, 不能及时
                            更新库存余量, 会导致大量库存查验的请求读取到旧的库存值, 并进行下单, 就会出现下单数量大于实际的库存量,
                            导致出现超售
                            
                            订单处理是需要在数据库里操作的, 订单处理会涉及支付、商品出库、物流等多个关联操作, 涉及到数据库中的多张表,
                            要保证处理的事务性, 需要在数据库中完成.
                            
    3. 基于分布式锁来支撑秒杀场景
            先让客户端向 Redis 申请分布式锁, 只有拿到锁的客户端才能执行库存查验和库存扣减, 大量的秒杀请求就会在争夺分布
            式锁时被过滤掉, 并且库存查验和扣减也不用使用原子操作, 客户端已经互斥拿到分布式锁了
            
    4. 建议
            可以使用切片集群中的不同 Redis 实例来保存分布式锁和商品库存信息, 这样客户端先在一个 Redis 实例中申请分布式锁, 成功后再向
            另一个 Redis 实例中进行商品查询, 扣减, 处理等操作.
                            
                            
```

### 运维
```shell
    1. info 命令(最基本的监控命令)
            (1) 实例本身配置信息 : info server
            (2) 客户端统计信息：　info client
            (3) 通用统计信息： info stat (重要)
                    命令的执行情况(命令使用的 CPU 资源)
            (4) 数据库整体统计信息 key 的数量等信息: info keyspace
            (5) 不同类型命令的调用统计信息: info commandstats (重要)
                    比如命令的执行次数和执行时间
            (6) CPU 使用情况: info cpu (重要)
                    CPU 资源使用情况
            (7) 内存使用情况：　info memory (重要)
                    内存使用情况
            (8) RDB, AOF 运行情况: info persistence (重要)
            (9) 主从复制的运行情况: info replication (重要)
            (10) 切片集群的运行情况: info cluster
            (11) 扩展模块信息: info modules
            
    2. 数据迁移工具 Redis-shake
    3. 匹配源 Redis 和 目的 Redis 是否一致: Redis-full-check 
    4. 集群管理工具 CacheCloud
            实现了主从集群、哨兵集群和 Redis Cluster 的自动部署和管理, Redis 的监控状态.
              
                    
```

## 特点
```shell
  1. redis 相对于其他高级编程语言采用相同的数据结构要更少内存。
  2. redis 有着其他其他数据库的功能，复制，集群，高可用
  3. 复制和持久性
        Redis 采用的是主副本架构，并支持异步复制，用户执行此类复制时可以将数据复制到多个副本服务器。它提供高效读取性能
        （因为请求可以在多个服务器间进行拆分）和恢复功能（主服务器发生中断时）。为了实现持久性，Redis 支持时间点备份
        （将 Redis 数据集复制到磁盘）
  4. 高可用性和可扩展性
        Redis 在单个主节点或群集拓扑中提供主副本架构，可以构建高度可用的解决方案，从而提供一致的性能和可靠性。
        如果需要调整群集大小，提供了横向扩展、纵向缩减或扩展。
```

## 运行
```shell
  1. 编译安装后，在 redis-unstable/src 目录下，  ./redis-server (redis 服务)  ./redis-cli(交互客户端)
```

## 使用
```shell
  1. 允许对每一个 key 进行过期(expires) 操作
  2. 对 reids 命令操作作为一个事务(里面包含多条命令)
  3. 块插入操作应对大数据量
  4. redis stream (redis 5)
  5. 管理调试 Redis-cli

```

## 技巧
```shell
  1. 像 list, streams, sets, sorted sets , hashes 这些有多个元素组成的数据类型(data types), 在加入第一个元素时
     不需要单独创建一个空的集合，在加的时候自动创建。在删除最后一个元素时也不需要单独删除空的集合，
     redis 会自动删除(stream 例外)
  2. 整数数组和压缩列表作为底层数据结构, 省内存空间, 因为底层是申请了连续的空间, 集合中的元素连续存放, 这样不需要额外指针进行串联, 避免
  　　额外指针的开销.
  3. Redis 基本 IO 模型中, bigKey, 全量返回这些操作都是潜在的性能瓶颈.
  4. AOF 重写过程存在风险, 
        风险 1 AOF 重写过程需要主线程创建 bgrewriteaof 子进程, 这需要子进程拷贝父进程的页表, 如果 Redis 存储键值对
  　　         数据过大, 则会导致主线程阻塞. 　
        风险 2 主线程在 AOF 重写过程中, 接受来自客户端的写请求, 这时主线程需要重新申请这个键值对的内存, 如果是 bigkey 类型的化, 
               主线程会因为申请大空间而面临阻塞风险. 
               
  5. 主从复制使用 RDB 文件而不使用 AOF 原因
        1. RDB 文件恢复的效率要比 AOF 高.
        2. RDB 数据传输要比 AOF 高
        
  6. 主从切换过程中, 客户端能否正常进行请求操作
        主从集群一般采取读写分离, 所以读请求可以成功, 主库故障了可以分配到从库. 但写请求没法成功进行.
        
  7. 哨兵实例数据越多, 会使得下线的误判率越低, 但同时需要更多数量哨兵实例回复, 这样导致时间变大, 使得请求数量堆积, 对于高响应的
  　　请求会超时报警.
  
  8. Redis 通过 SCAN 命令, 根据执行时设定的数量参数, 返回指定数量的数据, 替代 keys 命令(keys 命令是同时返回所有匹配的数据)
     例如: 从 user 这个 Hash 集合中返回 key 前缀以 103 开头的 100 个键值对
                >  HSCAN user 0 match "103*" 100
                
  9. 在和 Redis 实例交互时, 应用程序中的客户端使用缓冲区的好处
            第一: 可以在客户端控制发送速率, 避免把过多的请求一下子全部发到 Redis 实例, 导致实例因压力过大而性能下降.
            第二: 在 Redis 主从集群时, 主从节点进行故障切换是需要一定时间的, 主节点无法服务外来请求. 客户端的缓冲区可以暂存请求，
                 仍然可以正常接收业务应用的请求, 这就可以避免直接给应用返回无法服务的错误
    
```

## 数据结构(11~15)

### 概要
```shell
    1. 一个指针占 8 个字节.
```
### 字符串对象
```shell
    1. 应用场景为单值键值对, 一对一的, 如果考虑内存精简, 查询速度不需要很快, 如果用 key(string)-value(string) , 都用字符串类型的化, 
       则需要一个 dictEntry 结构(32 Bytes), 2 个 redisObject 类型(16 + 16), 一个 key-value 需要 64 Byte, 
       那么可以考虑 key(string) - value(hset)
```

### 集合统计模式
```shell
    1. 聚合统计
            (1) 聚合统计是指统计多个集合元素的聚合结果, 包括叫交集统计(统计多个集合的共有元素), 差集统计(和两个集合相比, 统计其中一个
                集合独有的元素), 并集统计(统计多个集合的所有元素)
            (2) 聚合统计可以使用 set 对象, 应用的场景是 APP 每天新增的用户, 以及留存的用户.
                流程：
                        a.  首先保存每天的用户登陆 ID 的集合, 集合名为 user:id:日期, value 为 set 对象(里面是 user_id),
                            还有一个是记录登陆过的所有用户 ID 的集合(集合名为 user:id)
                            
                        b. 集合名为 user:id:日期集合先赋值, 再和 user:id 集合进行差集, 得出今天的新增用户
                               SDIFFSTORE user:new user:id:20200804 user:id
                               
                        c. 再紧接着更新下 user:id 集合
                               SUNIONSTORE user:id user:id user:id:20200803
                               
                        d. 计算留存则是今天和昨天进行交集统计
                                SINTERSTORE user:id:rem user:id:20200803 user:id:20200804
                                
            (3) 风险:
                    聚合统计复杂度高, 数据量大时会导致 Redis 阻塞, 采用解决方案是用从库进行专门聚合计算, 或则是数据读到客户端, 
                客户端进行聚合计算.
                
    2. 排序统计
            (1) 对于需要展示最新列表, 排行榜, 其数据更新频繁或则需要分页显示, 则用 sort set 对象.因为它是通过权重(score)进行排序和
            　　获取数据的
            
    3. 二值状态统计(bitmap 对象, 扩展数据类型)
            (1) 对于集合元素的取值只有两种状态 0 和 1, 例如用户的签到, 商品的有无. 可以使用 bitmap 对象.
            (2) 方法:
                        a. getbit/setbit, 起始地址从 0 开始
                               > SETBIT uid:sign:3000:202008 2 1
                               > GETBIT uid:sign:3000:202008 2
                        b. BITCOUNT  统计 1 的个数
                               > BITCOUNT uid:sign:3000:202008
                        c. bitop 操作, 各个 bitmap 进行与, 或, 异或操作.
                               > bitop and resmap bm1 bm2 bm3
                               
    4. 基数统计(HyperLogLog 对象, 扩展数据对象)
            (1) 基数统计是统计一个集合中不重复的元素个数.
            (2) 如果数据量小并且想要精确的, 可以用 set/hash 对象, 
                    set 对象, 使用 SCARD 命令统计个数.
                    hash 对象,  使用 HLEN 命令统计个数.
                    
            (3) 如果是数据量大, 并且允许统计存在误差, 则使用 HyperLogLog 对象, 节省内存空间
                    > PFADD page1:uv user1 user2 user3 user4 user5  // 添加
                    >  PFCOUNT page1:uv  // 统计
```

### GEO(扩展数据类型)
```shell
    1. 概念
            (1) LBS(Location-Based Service)基于位置信息服务, 访问数据一组经纬度信息, 同时可以查询相邻的经纬度范围.
            (2) 场景是叫车服务, 一辆车有 id, 以及一组经纬度, 用 hash 保存是无序的, 不适用与范围查找.所以 GEO 数据类型
            　　是以 sort set 为底层数据结构
                
                GEO 的权重(score) 是一个浮点型的数值, 通过 GeoHash 编码的方式将一组经纬度转化为二进制的编码值, 其中
                确定要区域进行 N 次二等分, 经度范围 [-180,180], 纬度范围是 [-90, 90], 这时查看待转化的经度纬度落在哪个分区里, 如果是
                落在左分区, 就用 0 表示, 落在有分区用 1 表示.
                
                例如：　先做第一次二分区操作, 把经度区间[-180,180]分成了左分区[-180,0) 和右分区 [0,180]
                       经度值 116.37 是属于右分区[0,180], 用 1 表示第一次二分区后的编码值.
                       
                       这样经纬度通过 N 次二等分, 就能得出各自编码值, 例如 116.37，39.86）的各自编码值是 11010 和 10111,
                       再组合成最终编码值(从左到右) 第 0 位是经度的第 0 位 1, 第 1 位是纬度的第 0 位 1, 第 2 位是经度的第 1 位 1,
                       第 3 位是纬度的第 1 位 0, 最终的编码是　1110011101
            
            (2) GEO 操作
                    a. GEOADD 命令：用于把一组经纬度信息和相对应的一个 ID 记录到 GEO 类型集合
                            将车辆 id 33, 经纬度(116.034579, 39.030452) 加入到　 cars:locations　集合中,
                            > GEOADD cars:locations 116.034579 39.030452 33
                    b. GEORADIUS 命令：会根据输入的经纬度位置, 查找以这个经纬度为中心的一定范围内的其他元素
                            > GEORADIUS cars:locations 116.054579 39.030452 5 km ASC COUNT 10
                            查找以这个经纬度为中心的 5 公里内的车辆信息
                    
```

### 自定义数据类型
```shell
    1. 第一步: 定义新数据类型的底层结构
            新建 newtype.h 头文件
            
                struct NewTypeNode {
                    long value;
                    struct NewTypeNode *next;
                };
                
                struct NewTypeObject {
                    struct NewTypeNode *head;
                    size_t len;
                }NewTypeObject;
                
       第二步: 在 redisObject.type 属性中, 新增 type
                    在 server.h 文件中, 宏定义新增 NewTypeObject 类型
                    　
                    # define OBJ_NEWTYPE 7
                    
       第三步: 编写新类底层结构的创建和释放函数
                    在 object.c 文件中, 新增创建函数 createNewTypeObject
                    
                        robj *createNewTypeObject(void){
                        
                            NewTypeObject *h = newtypeNew();
                            robj *o = createObject(OBJ_NEWTYPE, h);
                            return o;
                        }
                        
                    在底层实现文件 t_newtype.c
                    
                        NewTypeObject *newtypeNew(void){
                            NewTypeObject *n = zmalloc(sizeof(*n));
                            n->head = NULL;
                            n->len = 0;
                            return n;
                        }
                        
       第四步: 编写新底层数据类型对应的操作
                    在 t_newtype.c 文件中, 实现 ntinsertCommand 函数
                    
                        void ntinsertCommand(client *c){
                         //基于客户端传递的参数，实现在NewTypeObject链表头插入元素
                        }
                        
                    在 server.h 文件中, 声明函数
                        void ntinsertCommand(client *c)
                        
                    
                    在 server.c 文件中, 将命令和函数管理起来
                        struct redisCommand redisCommandTable[] = {
                            ...
                            {"ntinsert",ntinsertCommand,2,"m",...}
                        }
                        
       第五步:
            还希望新的数据类型能被持久化保存, 需要在 Redis 的 RDB 和 AOF 模块中增加对新数据类型进行持久化保存的代码
                    
                    
```

### 时间序列数据
```shell
    1. 概念
            (1) 时间序列数据类型物联网数据, 某个时刻上报的检测值,状态等数据.
            (2) 时间序列数据的特点, 对于写,是新增记录, 要求复杂度低, 尽量不要阻塞.
                读操作是查询模式多, 可以进行单条记录查询, 某个时间范围的数据查询, 聚合计算, 某个时间内的平均值, 最大值, 最小值, 求和等等
                
    2. 基于 Hash 和 Sorted Set 保存时间序列数据(针对读模式多)
            (1) 用 Hash 集合满足时间序列数据的单键查询需求, 但是其不支持范围查找, Hash 的 key 为时间戳, value 就是对应的监测值
                    
                    > HGET device:temperature 202008030905
                        "25.1"
                        
            (2) 用 Sort Set 来保存时间序列数据, 时间戳作为权重值(score), 时间戳上的数据作为 Sort Set 本身的元素.
                使用 ZRANGEBYSCORE 命令,查询这个最大时间戳和最小时间戳时间范围内的温度值
                
                > ZRANGEBYSCORE device:temperatureSortSet 202008030907 202008030910
                    1) "25.9"
                    2) "24.9"
                    3) "25.3"
                    
            (3) 保证数据写入 Hash 和 Sort Set 集合是原子性操作(通过 Multi 和 Exec)
                    MULTI 命令：表示一系列原子性操作的开始.收到这个命令后，再收到的命令需要放到一个内部队列中,后续一起执行，保证原子性。
                    EXEC 命令：表示一系列原子性操作的结束. Redis 开始执行刚才放到内部队列中的所有命令操作
                    
                    127.0.0.1:6379> MULTI
                    OK
                    
                    127.0.0.1:6379> HSET device:temperature 202008030911 26.8
                    QUEUED
                    
                    127.0.0.1:6379> ZADD device:temperature 202008030911 26.8
                    QUEUED
                    
                    127.0.0.1:6379> EXEC
                    1) (integer) 1
                    2) (integer) 1
                    
            (4) 风险(对时间序列数据进行聚合操作)
                    a. Sort Set 只能进行范围查找, 无法进行聚合操作, 这时只能将某一时间段内的数据回传给客户端, 由客户端进行聚合操作.
                       但这样会导致大量的数据在 Redis 实例和客户端间频繁传输, 影响到其他的命令的请求.
                       
    3. 基于 RedisTimeSeries 模块保存时间序列数据
            (1) RedisTimeSeries 模块针对时间序列数据提供了数据类型和访问接口, 在 Redis 中可以进行聚合操作, 减少大量数据回传给客户端.
            (2) RedisTimeSeries 模块使用, 因为它不需要内置的功能模块, 需要编译成动态链接库, 在加载 
                    > loadmodule redistimeseries.so
            (3) 用 TS.CREATE 命令创建一个时间序列数据集合
                    > TS.CREATE device:temperature RETENTION 600000 LABELS device_id 1
                    创建一个 key 为 device:temperature、数据有效期为 600s 的时间序列数据集合, 设置标签属性表明这个数据集合中
                    记录的是属于设备 ID 号为 1 的数据
                    
            (4) 用 TS.ADD 命令插入数据, 用 TS.GET 命令读取最新数据
                    > TS.ADD device:temperature 1596416700 25.1
                      1596416700
                    
                    > TS.GET device:temperature
                      25.1
                      
            (5) 用 TS.MGET 命令按标签过滤查询的数据集合
                    > TS.MGET FILTER device_id!=2
                    
            (6) 聚合计算
                    用 TS.RANGE 命令指定要查询的数据的时间范围，　同时用 AGGREGATION 参数指定要执行的聚合计算类型, 
                计算类型有 求均值(avg)、求最大 / 最小值 (max/min), 求和(sum)
                
                 在某个范围内每 180s 聚合计算
                > TS.RANGE device:temperature 1596416700 1596417120 AGGREGATION avg 180000
                
                  1) 1) (integer) 1596416700
                  2) "25.6"
                  2) 1) (integer) 1596416880
                  2) "25.8"
                  3) 1) (integer) 1596417060
                  2) "26.1"
                  
            (7) 缺点
                    RedisTimeSeries底层数据结构使用链表, 它的范围查询的复杂度是 O(N) 级别的, 它的 TS.GET 查询只能返回最新的数据,
                    没有办法像第一种方案的 Hash 类型一样, 可以返回任一时间点的数据
                    
    4. 应用场景
            (1) 部署的网络带宽高, Redis 实例内存大, 采用 Hash 和 Sort Set 组合的形式
            (2) 部署环境网络, 内存资源有限, 并且数据量大, 聚合计算频繁, 需要按数据集合属性查询, 则采用 RedisTimeSeries.
                    
```

### Redis 消息队列方案
```shell
    1. 消息队列的需求
            (1) 消息保序, 生产者发送的数据包的顺序和消费者接受数据包的顺序要一致.
            (2) 处理重复消息. 对于网络阻塞情况导致消息重传, 要能够处理重复的消息.
            (3) 消息的可靠性保证. 当消费者重启, 能够重新读取消息再次进行处理, 防止消息的漏处理.
            
    2. 基于 List 的消息队列解决方案
            (1) 消息保序: 对于 list 保序的需求,生产者进行 lpush, 消费者进行 rpop, 同时为了防止消费者不断 while 循环进行 rpop 操作, 
                         消耗CPU 资源, 可以新建一个线程通过 brpop 命令阻塞等待消息的获取.
            (2) 处理重复消息: 生产者需要在发送消息时加上全局的消息 ID 在 lpush 到 list 中, 生产者能够记录已经处理过的消息 ID
                            > LPUSH mq "101030001:stock:5"   // 其中消息 ID 为 101030001
            (3) 消息的可靠性: 通过 brpoplpush 命令, 使得 Redis 再把取出来的数据放到备份队列中, 如果程序正常处理, 则将数据从备份队列
            　　　　　　　　　　删除. 如果消费者异常重启了, 则从备份队列中读取.
            
            (4) 缺点: 当生产者速度要远大于消费者处理速度时, Redis 内存会不断增大, 当 List 类型不支持消费组(多个消费程度)
            
    3. 基于 Streams 的消息队列解决方案(Redis 5.0)
            (1) Streams 是 Redis 专门为消息队列设计的数据类型, 提供丰富的消息队列操作命令
            (2) XADD：插入消息, 保证有序, 可以自动生成全局唯一 ID
            
                   // mqstream 表示队列名, * 表示让  Redis 为插入的数据自动生成一个全局唯一的 ID, 也可以自己加全局唯一的 ID
                   // key 为 "repo", value 为 "5"
                    > XADD mqstream * repo 5   
                    结果
                        "1599203861727-0"
                        
            (3) XREAD：用于读取消息, 可以按消息 ID 读取数据
                    > XREAD block 10000 streams mqstream $
                    
                    block 10000 : 阻塞 10 秒, 如果还没有数据, 则返回空值.
                    $: 表示获取最新的数据
                    
                       也可以用消息ID, 代表从 ID 号为 1599203861727-0 的消息开始, 读取后续的所有消息.    
                    > XREAD BLOCK 100 STREAMS mqstream 1599203861727-0
                    
            (4) XGROUP: 创建消费组
                XREADGROUP: 让消费组内的消费者读取消息
                
                // 为消息队列 mqstream 创建消费组 group1
                // 0 : 代表从头开始消费, $ : 代表从尾部开始消费, 只接受新消息  
                > XGROUP create mqstream group1 0    
                
                // 让消费组 group1 中的消费者 consumer1 从消息队列  mqstream 读取所有未被消费的消息, ">" 表示从第一条尚未被消费的消息开始读取
                > XREADGROUP group group1 consumer1 streams mqstream >
                
                
                // 让消费组 group2 中的消费者 consumer2 从消息队列  mqstream 读取 9 条未被消费的消息
                > XREADGROUP group group2 consumer2 count 9 streams mqstream >
                
            (5) 
                为了保证消费者在发生故障或宕机再次重启后, 仍然可以读取未处理完的消息, Streams 会自动使用内部队列(PENDING List)
                保存消费组里每个消费者读取的消息, 直到消费者使用 XACK 命令通知 Streams“消息已经处理完成”．　如果消费者没有成功处理消
                息, 它就不会给 Streams 发送 XACK 命令, 消息仍然会留存. 消费者可以在重启后, 用 XPENDING 命令查看已读取、但尚未确认
                处理完成的消息.
                    
                    // 查看消息队列 mqstream, 消费组 group2 中已读取, 但未完成的消息
                    XPENDING mqstream group2
                    1) (integer) 3
                    2) "1599203861727-0"
                    3) "1599274925823-0"
                    4) 1) 1) "consumer1"
                          2) "1"
                       2) 1) "consumer2"
                          2) "1"
                       3) 1) "consumer3"
                          2) "1"
                          
                          
                    // 查看消息队列 mqstream, 消费组 group2 , 消费者 consumer2 已读取, 但未完成的消息
                    XPENDING mqstream group2 - + 10 consumer2
                    1) 1) "1599274912765-0"
                       2) "consumer2"
                       3) (integer) 513336
                       4) (integer) 1
                       
                       
                一旦消息 1599274912765-0 被 consumer2 处理, consumer2 就可以使用 XACK 命令通知 Streams, 然后这条消息就会被删除.
                当我们再使用 XPENDING 命令查看时, 就可以看到, consumer2 已经没有已读取、但尚未确认处理的消息了
                
                XACK mqstream group2 1599274912765-0
                (integer) 1
                
                XPENDING mqstream group2 - + 10 consumer2
                (empty list or set)
```

### SDS(simple dynamic string, 简单动态字符串)
```shell
    1. redis string 采用的是简单动态的字符串.如果 value 是一个 const 字符串,则使用 c风格的字符串,如果 value 
       之后有修改,则采用 SDS 数据类型
    2. SDS 还被用作缓冲区(buffer), 例如 AOF 模块中的 AOF 缓冲区, 客户端状态中的输入缓存区
    3. SDS 数据结构
            struct sdshdr{
                int len;  // 有效的字节数,不包含 '\0'
                int free;  // buf 数组中未使用的字节数
                char buf[];
            }
    4. SDS 与 C 字符串的区别
            (1) SDS 获取字符串长度的时间复杂度为 0(1) 取 sdshdr.len, 而 c 字符串则是 O(N) 
            (2) 防止缓冲区溢出, sds 对外的 api 接口中 sdscat() 函数实现了长度的判断.
            (3) sdshdr.len 和 sdshdr.free 的作用避免重复分配内存影响性能. 空间预分配用于优化 SDS 的字符串增加
                操作,
                空间预分配的策略
                      a. 如果增加后的 len 小于 1MB, 则让其 free 与 len 的大小一致, 故需要分配 free + len + 1 内存大小空间
                      b. 如果　sdshdr.len 的大小超过 1MB, 则每次内存分配时 free 都申请 1MB
                缩短字符串操作可以将 sdshdr.free 空间增加,以便后续的使用.
            (4) 二进制安全(binary-safe), 而与 c 字符串而言是以 '\0'结尾, SDS 不仅可以保存文本数据,还可以保存任意格式的
                二进制数据,里面保存的字节大小通过 sdshdr.len 获取的.
               
```

### list/双向链表(adlist.h)
```shell
    1.  integers 列表键的底层实现是一个链表,链表中每个节点都保存一个整数值.发布与订阅,慢查询,监视器等功能也用到了链表, 
        Redis 服务器本身使用链表保存多个客户端的状态信息,使用链表来构建客户端输出缓冲区
    2. 
        typedef struct listNode {
            struct listNode *prev;
            struct listNode *next;
            void *value;
        } listNode;
                
        typedef struct list {
            listNode *head;  // 表头结点
            listNode *tail;  // 表尾结点
            void *(*dup)(void *ptr);  // 节点值复制函数
            void (*free)(void *ptr);  // 节点值释放函数
            int (*match)(void *ptr, void *key);  // 节点值对比函数
            unsigned long len;  // 节点个数
        } list;
        
        每个链表节点有一个 listNode 结构表示,每个节点都有一个指向前置节点和后置节点的指针, Redis 的链表实现是双向链表.
        每个链表使用 list 结构体来表示, 该结构体带有表头结点指针,表尾结点指针,节点个数, 通过为链表设置不同类型特定函数,
        Reids 链表可以用于保存各种不同类型的值.
          
```

### dict/哈希表(字典, dict.h)
```shell
    1. 字典在 Redis 中应用广泛, Redis 的数据库就是使用字典进行实现的.对数据库的增,删,改,查也是在字典的基础上.
       例如: SET msg "hello world"
       这个键为 "msg", 值为 "hello world"
    2. Redis dict 的实现方式,采用哈希表进行实现,一个哈希表里包含多个哈希结点,每个哈希结点对应于字典中的键值对.
    　　每个 dict 带有 2 个哈希表， 1 个是日常是使用， 1 个是 rehash 时使用
    3. 结构体
        typedef struct dictEntry {
            void *key;  // 8 Bytes
            union {
                void *val;  // 保存 value 类型的指针,
                uint64_t u64;
                int64_t s64;
                double d;
            } v;  // 8 Bytes
            // 通过 hash 算法后得到相同 index 值，　进行 hash 冲突的解决(链地址法)，　因为　dictEntry　节点没有
            // 指向链表尾部的指针，为了新增节点的效率，只需要将新节点添加到链表的表头位置
            struct dictEntry *next;    // 8 Bytes
        } dictEntry;
        
        因为 Redis 采用 jemalloc 内存分配库, 一次性分配的内存是接近于 2 幂次方, 减少频繁分配的次数. 例如 dictEntry 理论是
        24 Byte就可以了, 但实际上要分配 32(2^5) 字节
        
        typedef struct dictType {
            uint64_t (*hashFunction)(const void *key);
            void *(*keyDup)(void *privdata, const void *key);
            void *(*valDup)(void *privdata, const void *obj);
            int (*keyCompare)(void *privdata, const void *key1, const void *key2);
            void (*keyDestructor)(void *privdata, void *key);
            void (*valDestructor)(void *privdata, void *obj);
        } dictType;
        
        /* This is our hash table structure. Every dictionary has two of this as we
         * implement incremental rehashing, for the old to the new table. */
        typedef struct dictht {
            dictEntry **table;  // dictEntry * 数组, 数据元素就是一个哈希桶, 桶内是 key 进行哈希计算后的冲突
            unsigned long size;  // 哈希表键值对的大小
            unsigned long sizemask;  // 通过哈希值和 sizemask 计算出索引值,可能会导致 hash 冲突.
            unsigned long used;  // 已经使用的键值对的数量
        } dictht;
        
        typedef struct dict {
            // 用于操作特定类型键值对的函数, Ridis 为用途不同的字典设置不同类型的特定函数
            dictType *type;  
            // 保存需要传给那些类型特定函数的可选参数
            void *privdata;
            dictht ht[2];  // 定义为 2 个元素的数组,用于 rehash
            long rehashidx; /* rehashing not in progress if rehashidx == -1 */
            unsigned long iterators; /* number of iterators currently running */
        } dict;
        
        一般情况下,字典只使用 ht[0] 哈希表, ht[1] 哈希表只会在对 ht[0] 进行 rehash 时使用. rehashidx 记录了 rehash 的进度
        如果没有进行 rehash ,则 rehashidx 置为 -1.
    4. 哈希算法,将一个新的键值对添加到字典表中,需要将利用 key 使用哈希算法(dict->type->hashFunction(key))
    5. 处理键冲突主要是用链地址法.
    6. rehash, 主要为了是哈希表的负载因子在合理的范围内，防止相同索引对应下哈希节点过多.
    
        渐近式 rehash(常用)
                    如果哈希表的键值对很多，则一次性将 ht[0] 移到 ht[1] 中，会导致这个过程中 dict 服务失效，所以采用渐进式的 rehash
             在渐进式 rehash 过程中, 同时使用 ht[0] 和 ht[1] 的哈希表, 保存有 dict.rehashidx 值(即在 ht[0] 进行　rehash 的下标值),
             即第一次请求, 从哈希表 1 (ht[0])中的第一个索引位置开始(第一个索引下标), 顺带着将这个索引位置上的所有 entries(冲突节点) 
             rehash 到哈希表 2 中. 到第二次请求时, 将 ht[0] 中的第二个下标下所有冲突节点 rehash 到哈希表 2 中.
                  
    
    
        　 一次性rehash 步骤:(只适用于键值对少的，不常用)
                第一步: 为 dict 结构体中的 ht[1] 哈希表分配空间, 取决于执行的操作(扩展操作还是收缩操作)和原先 ht[0] 已使用的
                　　　　键值对的大小(ht[0].used)
                            1. 当 rehash 是扩展操作时，新分配的 ht[1] 的大小，先计算出 ht[0].used * 2, 再根据这个值得出第一个
                            　　大于等 2 ^ n, 例如　ht[0].used * 2　为 8 , 则恰好是第一个大于等于 2 ^ 3, 所以新分配的 ht[1]
                                为 8
                            2. 当 rehash 是收缩操作时, 新分配的 ht[1] 的大小，先计算出 ht[0].used, 再根据这个值得出第一个
                               大于等 2 ^ n
                第二步: 在将 ht[0] 中的键值对进行重新哈希计算，将键值对放置到 ht[1] 指定的位置
                第三步: 先将原来老的 ht[0] 释放，再将 ht[1] 设置为 ht[0], ht[1] 创建一个新的哈希表
                
                
                
                
        (1) rehash 的条件
                a. 第一种情况, 装载因子(load factor) >= 1, 并且哈希表允许被 rehash(没有进行 RDB 生成, AOF 重写), 这种情况
                　　　　　　　　代表每个哈希桶内只有一个键值对, 往后就冲突, 需要链地址法解决冲突, 这样查询会比较慢.
                b. 第二种情况, 装载因子(load factor) >= 5, 就得马上进行 rehash, 已经极大影响查询效率.
                
        (2) 渐近式 rehash 时机(已经达到 rehash 的条件)
                a. 有外部的客户端的请求, 同步进行渐进式 rehash
                b. 即使没有外部请求, Redis 有定时任务, 按照一定频率(100ms/次), 会检测是否满足 rehash, 满足则进行渐进式 rehash, 
                   每次执行不超过 1 ms.
            
        
```

### 跳跃表(skiplist, server.h/zskiplistNode server.h/zskiplist)
```shell
    1. 跳跃表是一种有序数据结构，通过在每个节点中维持多个指向其他节点的指针，其中第一层节点 1(value: 1), 第一层节点 2(value: 8)
       那么第一层节点 1 除了指向同一层的下一个节点 2, 还指向下一层节点 3 , 下一层的范围是在 1 ~ 8,　即增加了多级索引, 时间复杂度为
       O(logN)
    2. 有序集合键底层实现之一就是用跳跃表．适用于有序集合中的元素数量多的情况． Redis 还有一种应用情况是在集群结点中用作内部数据结构．
    3. 
        typedef struct zskiplistNode {
            sds ele;
            // 分值，节点按各自保存的分值从小到大排列，如果分值大小相同，则根据 sds els 的大小排序，小的排在前面
            double score;
            // 后退指针，指向位于当前节点的前一个节点，从表尾向表头遍历使用, 每个节点只有一个后退指针，每次只能后退前一个节点
            struct zskiplistNode *backward;
            struct zskiplistLevel {
                // 前进指针，每一层都有指向表尾方向的前进指针，从表头向表尾遍历使用,一次可以跳过多个节点，
                struct zskiplistNode *forward;
                // 跨度, 前进指针 forward 与当前指针的距离，　指向 NULL 的所有前进指针的跨度都为 0,跨度实际上是计算排位(rank)
                // 在查找某个节点的过程中，将沿途访问过所有层的跨度累计起来，得到的结果就是目标节点在跳跃表的排位.
                unsigned long span;
            } level[];
            // 每创建一个跳跃表节点，随机生成一个 1 ~ 32 值作为 level 数组的大小(层的高度), level[0] 代表第一层
        } zskiplistNode;
        
        typedef struct zskiplist {
            // header:表头节点, tail 表尾节点
            struct zskiplistNode *header, *tail;
            // 跳跃表的长度，跳跃表目前包含节点的数量(不包含表头节点)
            unsigned long length;
            // 目前跳跃表内，层数最大的节点的层数(表头节点不计算在内)
            int level;
        } zskiplist;
        
        表头节点和其他节点的构造是一样，不过表头节点的后退指针(backward), 分值(score),ele 这些属性不会被使用，只显示
        表头节点的各个层．
```

### 整数集合/整数数组(intset.h)
```shell
    1. 整数集合为 intSet, 是集合键(set zadd)的底层实现之一，如果集合键都是有整数来保存，并且整数集个数少
    2. typedef struct intset {
            // 编码方式,有 INTSET_ENC_INT16, INTSET_ENC_INT32, INTSET_ENC_INT64
           uint32_t encoding;
           // 集合包含的元素的数量, 也是 contents 数组的逻辑长度(不一定等于 sizeof(contents))
           uint32_t length;
           // 保存元素的数组，各个元素的大小按值从小到大排序，虽然 contents 申明为 int8_t 数组，当并不保存任何 int8_t
           // 类型的值， contents 数组的元素类型真正取决于 encodeing 的值, 如果　encodeing　为　INTSET_ENC_INT16，
           // 则逻辑元素实际占用了 2 个 contens int8_t 的大小
           int8_t contents[];
       } intset;
    3. 保存的 int8_t contents 逻辑元素的数据类型要比现在的大，则进行升级整数集合，第一步：根据新元素的类型，扩张整数集合
    　　底层数组的空间大小，并为新元素分配空间．第二步: 将底层数组现有的所有元素都转换成与新元素相同的类型(因为新元素的占用空间大)，
    　　并将转换后的元素放置在正确的位子上，要确保底层数据的有序性．第三步：将新加的元素加入到底层数组中(注意保持有序性)
    
    4. 升级的好处；
            (1) 数组中采用同一个数据类型，读取的效率要高，提高灵活性，因为整数集合是通过自动升级底层数组来适应新元素，该底层数组
            　　保存数据类型单位是元素中最大数据类型
            (2) 自动升级底层数据，可以避免预先申请为 int64 大小的单位空间(考虑到日后会有  int16, int32, int64)
    5. 不支持降级，即使　int64_t 类型元素删除了，整数集合的编码仍然会支持 INTSET_ENC_INT64, 底层数组仍然是 int64 类型
```

### 压缩列表(ziplist)

```shell
    1. 列表键和哈希键底层实现之一就是压缩列表(ziplist) 结构体，列表键使用压缩列表(ziplist)实现的场景，当列表键只包含少量的列表项，
       并且每个列表项要么是小整数值，要么是短的字符串．　哈希键使用压缩列表(ziplist)实现的场景，一个哈希键包含少量的键值对，
       并且每个键值对的键和值要么是小整数值，要么是短的字符串
       例如:
            127.0.0.1:6379> HMSET profile "name" "jack" "age" 28 "job" "teacher"
            OK
            127.0.0.1:6379> OBJECT encoding profile
            "ziplist"
    2. 压缩列表的构成
            zlbytes(4 bytes) + zltail(4 bytes) + zllen(2 bytes) + entry1+ ...... entryN + zlend(1 bytes)
            
            zlbytes: 代表压缩列表占用内存字节数(包括 zlbytes, zlend)
            zltail: 代表从　zlbytes　地址开始，到　entryN(表尾节点) 的字节数，这样通过 zltail(偏移量)就能定位到 entryN 的位置
            zllen: 如果值小于 2^16(65535), 则代表节点 entry 数量，如果 zllen 值为 65535, 则真实的节点个数只能通过遍历来获取．
            entry: 节点
            zlend：　值为 0xff, 代表压缩列表结束符
            
            (1) entry 节点的组成
                    previous_entry_length + encoding + content
                    
                    a. previous_entry_length　代表前一个节点的长度， 如果前一个节点的长度小于 254　byte, 则 
                       previous_entry_length 值用一个字节保存(因为 255-> 0xff, 是压缩列表结束符(zlend)),
                       如果前一个节点的长度大于 254　bytes, 则 previous_entry_length 值用五个字节保存，其中第一个字节设置为 0xfe, 
                       之后的四个字节则用于保存前 一节点的长度．　previous_entry_length 作用是方便将压缩列表从表尾向表头进行遍历．
                       
                    b. encoding
                        记录保存数据的类型和长度
                        
                    c. content
                        content 可以是字节数组或则整数，其有 encoding 决定的．例如: encoding 值为 00001011(前面 2 位 00代表
                        保存单位是字节， 001011 则是字节个数 11)
                        
                        previous_entry_length + encoding +     content
                            .......             00001011    "hello world"
                            
    3. 压缩列表是一种为节约内存而开发的顺序型数据结构, 如果要查找定位第一个元素和最后一个元素, 可以通过表头三个字段进行定位, 复杂度 O(1),
       如果是其他的元素, 则只能逐个查找, 复杂度 O(N)
                    
```

## 对象

### 概要
```shell
    1. Redis 并没有直接使用自身定义的数据结构(简单动态字符串 sds, 双向链表, 字典(哈希表)，整数集合(整数数组)，压缩列表)，而是基于这些数据结构创建
    　　一个对象系统，这个系统包含字符串对象，列表对象，哈希对象，集合对象以及有序集合对象．
    2. Redis 使用对象来表示数据库中的键和值，而键一般都是字符串对象. Redis 中的每个对象都是由一个 redisObject 结构体表示
        typedef struct redisObject {
            //类型
            unsigned type:4;
            
            // 编码
            unsigned encoding:4;
            
            // 指向底层实现数据结构的指针
            void *ptr;
            
            //...
            
        }
        (1) type(类型): 
                        REDIS_STRING(字符串对象)
                                (0) REDIS_ENCODING_INT, REDIS_ENCODING_EMBSTR, REDIS_ENCODING_RAW
                                (1) value 中的 string 可以是一个二进制数据，比如是 .jpg 格式内容, 不过最大是 512 MB
                                (2) 如果字符串对象保存是一个整数，则 redisObject.encoding 设置为 REDIS_ENCODING_INT, 
                                    redisObject.ptr  == long
                                (3) 如果字符串对象保存一个字符串值，并且字符串长度小于 32 字节，那么　redisObject.encoding 
                                    设置为 REDIS_ENCODING_EMBSTR, mbstr 编码是专门适合短字符的编码方式, 它跟 raw 编码产生的效果
                                    是相同的, 只不过 embstr 编码是调用一次内存分配函数创建 redisObject 结构和 sdshdr 结构, 
                                    而 raw 编码则要分两次调用内存分配函数，比如先调用一次内存分配函数创建 redisObject 结构, 
                                    再调用一次内存分配函数创建　sdshdr 结构. 使用 embstr 编码的字符串对象减低内存分配和释放的频率，
                                    同时需要的数据结构分配在连续的内存空间，更好得利用缓存
                                    
                                (4) 如果字符串对象保存一个字符串值，并且字符串长度大于 32 字节, 　redisObject.encoding 
                                    设置为 REDIS_ENCODING_RAW, SDS 的数据量较大, Redis 不再把 redisObject 结构和 SDS
                                    结构一块申请, 会给 SDS 分配独立的空间, redisObject.ptr 指向 sdshdr 结构.
                                
                                (5) 将 long double 类型值保存到 reids 中，是使用字符串对象，并且以 embstr 编码形式保存
                                (6) 字符串对象内编码之间的转化，对字符串对象在进行一些命令会导致编码的转化，例如 int 编码的字符串对象
                                    进行 append　命令， 则编码就会变为 raw 编码，因为 append 命令针对一个字符串值．
                                    因为 embstr 编码的字符串对象不具备任何操作接口，所以 embstr 编码的字符串对象是只读的，
                                    如果要对 embstr 编码的字符串对象进行操作(例如 append 操作)，则需要将 embstr 编码转化为 raw 
                                    编码，所以 embstr 编码的字符串对象在执行修改命令之后，总会变成 raw 编码的字符串对象．
                                    
                        REDIS_LIST(列表对象)
                                redis list 实现是通过链表形式，其主要是针对高效的数据插入. 如果想要高效的查找, 则可以选择
                                sorted sets 数据对象, list 的应用场景一般是多线程的消息的存取.例如社区网络服务中保存最新的消息等.
                                一用户发布图片时, 则将该图片 lpush 到 ist_speci 中, 当有用户访问图片首页时, 
                                则用 lrang 0 9 取最新的 10 条记录
                                (1) 当列表对象保存字符串元素长度小于 64 字节, 并且保存的元素个数小于 512 个, 则采用 ziplist
                                    编码(压缩编码). 否则采用双向链表．所以在使用的过程中，很有可能列表对象的底层实现由压缩列表转化
                                    为双向链表.
                                    
                                    
                        REDIS_HASH(哈希对象), 
                                哈希对象的编码有 ziplist(压缩列表)和 dict(字典), 
                                (1) ziplist(压缩列表), 对于哈希对象中的键值对插入顺序，先是将键的压缩列表节点(entry1)插入到压缩列表表尾，
                                    再将键对应的值的压缩列表节点(entry2)插入到缩列表表尾. 压缩列表的键值对的压缩是按照列表尾方向插入．
                                (2) dict(字典)
                                    
                                当哈希对象中的键值对的键和值保存字符串长度小于 64 字节, 并且保存的键值对个数小于 512 个, 
                                则采用 ziplist 编码(压缩编码).否则 dict(字典)
                                
        　　　　　　　　　REDIS_SET(集合对象)
                                代表无序的 strings , 用 sets 的实际例子是一个新闻的标签。
                                无序集合对象底层实现是整数集(intset)和字典(dict), dict 实现是每个键是一个字符串对象, 
                                而键所对应的值设置为 NULL. 采用 intset 编码条件是集合对象保存的元素都是整数值并且保存的元素的个数
                                不能超过 512 个, 否则采用 dict(字典)来实现．
                                
                       REDIS_ZSET(有序集合对象)
                                 代表有序的 set, 其类似于 hash 与 set 的综合, 其每个元素对应于 float value. 其内部的排序的实现方式
                                 是如果 A.score > B.score ,那么 A > B. 如果 A.score ==  B.score, 则再查看 A 和 B 的 
                                 string 按字母顺序排列.
                                        
                                 因为更新完 score 后, sorted set 会自动更新, 因为这个特性，所以很适合排行榜的场景。
                                        
                                 sorted sets(有序集合对象)的底层实现编码有 ziplist(压缩列表)和 skiplist(跳表)
                                    (1) ziplist(压缩列表),每个集合元素使用两个紧挨一起的压缩列表节点保存，第一个压缩列表节点(entry1)
                                        保存元素的成员, 第二个压缩列表节点(entry2)保存元素对应的 score . 压缩列表内的集合元素按照
                                         score 的大小排序, score 越小的集合元素排在压缩列表表头.
                                        
                                    (2) skiplist(跳表) 
                                            skiplist 编码使用 zset 结构体作为底层实现.
                                            typedef struct zset {
                                                zskiplist *zsl;
                                                dict *dict;
                                            } zset;
                                            
                                            zsl 跳跃表按分值从小到大保存,每个跳跃表节点保存一个集合元素,zskiplistNode.ele 保存元素, 
                                            zskiplistNode.score 则保存元素的分值,对有序集合进行范围操作(ZRANGE) 都是通过
                                            跳跃表的 api 进行实现.
                                            
                                            zset 中的 dict 为有序集合创建从成员到分值的映射.字典的键保存元素,字典的值保存元素 score,
                                            通过 O(1) 复杂度就可以根据元素取到 score.
                                            
                                            zskiplist 和 dict 通过指针都共用相同的元素和 score. zskiplist 保证较快定位到范围查找.
                                            dict 保证较快取得某一个值.
                                            
                                    (3) 有序集合使用 ziplist 编码的条件是有序集合保存元素的数量小于 128 个, 
                                        其元素成员的长度小于 64 字节.否则使用 skiplist
                                
        
           因为键值对中，键永远都是字符串对象，所以 "字符串键"　指的是这个数据库键对应的值为字符串对象．
           "列表键" 指的是这个数据库键对应的值为列表对象．通过 type 命令，对键值对中键进行操作，返回的不是键的
           对象类型(永远是字符串对象)，而是该键对应的值的对象类型．
           
           127.0.0.1:6379> set msg "hello world"
           OK
           127.0.0.1:6379> type msg
           string
           127.0.0.1:6379> rpush str_list 1 2 3 4 5
           (integer) 5
           127.0.0.1:6379> type str_list
           list
           
        (2)  encoding(编码)：编码的值决定了 ptr 指向哪种数据结构，
               REDIS_ENCODING_INT: long 类型的整数
               REDIS_ENCODING_EMBSTR: 使用 embstr 编码的简单动态字符串实现的字符串对象.　使用 sdshdr 结构体　
               REDIS_ENCODING_RAW: 简单动态字符串，　使用 sdshdr 结构体
               REDIS_ENCODING_HT：　字典(哈希表)
               REDIS_ENCODING_LINKEDLIST: 双向链表
               REDIS_ENCODING_ZIPLIST： 压缩列表
               REDIS_ENCODING_INTSET: 整数集合
               REDIS_ENCODING_SKIPLIST：　跳跃表
        
        (3) 一个对象可能有多种数据结构实现，例如:
                字符串对象
                列表对象：压缩列表或则双向链表实现
                哈希对象: 字典(哈希表)或则压缩列表实现
                无序集合对象: 整数集合，字典(哈希表)
                有序集合对象: 压缩列表，跳跃表实现
                
            使用 object encoding 命令查看数据库键的值对象的编码
            127.0.0.1:6379> set msg_long "long long long long long long long long long "
            OK
            127.0.0.1:6379> OBJECT encoding msg_long
            "raw"
            127.0.0.1:6379> SADD nums 1 3 5
            (integer) 3
            127.0.0.1:6379> OBJECT encoding nums  // 只包含整数，则使用整数集合
            "intset"
            127.0.0.1:6379> sadd nums "strings"
            (integer) 1
            127.0.0.1:6379> OBJECT encoding nums　　// 如果再加上字符串, 则使用字典数据结构(REDIS_ENCODING_HT)
            "hashtable"
            
        (4) 这个　redisObject　结构体的实现方式，可以通过 encoding 的取值以及 void *ptr, 可以随着不同的使用场景动态
        　　 改变底层实现的数据结构，更为灵活．例如在列表对象包含的元素较少时， Redis 采用压缩列表作为列表对象的底层实现，
        　　　因为压缩列表比双向链表更省空间，同时在元素少的情况下，在内存中以连续块保存的压缩列表比双向链表更快载入缓冲中．
            　但当存储的元素越来越多，则更注重功能性，所以双向链表则更适合保存大量的元素．
               
           
```
### data types

```shell
  1. key - value 中 value 不仅仅只有 string 类型还可以保持其他的数据类型.其中一种是 sorted sets, 一个有顺序的唯一的集合,
     其内部的原理是 string_value 绑定了一个 float 型的值(score), 其内部操作可以获取 top 10 或则 bottom 10
  2. key 技巧
      (1) key 最好不用太长(例如 1024 字节), 即占用了内存, 在 key 的比较又耗时。如果真的有 key 特别长的情况，
          也最好可以用 hash 算法(SHA1) 来得到固定的长度的值，这个要考虑业务场景(会有冲突的情况)
      (2) key 也不用刻意变短，例如 "user:1000:followers" 要比 "u1000flw" 合适
      (3) 一个 key 大小最大只能是 512 MB
       
       　　       
  4. redis list(列表对象)
        
  5. sets(集合对象)
        
  6. sorted sets(有序集合对象)
  
  7. 哈希对象
         
  8. bitmaps
        bitmaps 的一个优势是节省空间。实际应用，要统计自网址公开以来，每一次用户访问进行 setbit 操作，位索引为 : 
        当前时间戳 - 公开网址的时间戳 / (3600 * 24) , key 为用户 id, 则可以通过 bitcount 得到访问的天数.
        对于数据分片，最好避免使用大的 key . 常用的方法是每个 key 保存 M bit, 其 key 的名字为 bit-number(总数)/ M,
        在 key 中索引序号为 bit-number(总数) % M

```

### 对象操作
```shell
    1. 类型检测,某一特定的命令只针对某一对象类型进行操作,例如 set 对字符串对象有效, llen 对列表对象有效.实现的原理是对
       redisObject.type 进行判断.
    2. 内存回收
            C 语言本身不具备自动内存回收功能,需要在对象结构体定义一个字段 redisObject.refcount (引用计数)实现内存的回收
            机制.在创建一个新对象时,引用计数值初始化为 1, 被一个新的程序使用时,引用计数加 1(incrRefCount), 当对象不再
            被一个程序使用时(decrRefCount),引用计数值减一,如果计数值为 0, 则释放资源.
    3. 对象共享,有了引用计数的实现,那么如果键 A 和键 B 对应的值的对象是一样的,则可以将该值对象进行共享,减少内存的使用.
       查看键对应的值的引用计数命令 object refcount
    4. 对象的空转时长. redisObject.lru:22 记录了对象最后一次被命令程序访问的时间, 而命令 object idletime 键的结果是
       空转时长(当前时间戳减去对象的 lru 的时间戳),单位是秒,即开始计数没有被访问的时间.
       
       
```

### 命令
```shell
      1. set 命令会覆盖写(默认情况下), NX 选项代表 set 时候不存在该 key 成功。 XX 代表 set 时候存在该 key 是设置成功
      2. redis 的命令 incr 命令会将 string 先变为 int , 再加 1 ,后存为 string
              > set counter 100
              OK
              > incr counter
              (integer) 101
          同时 incr 操作是原子的, 不同的客户端同时操作也会确保原子性, 如果原先的 key 值为 10,2 个客户端同时 incr, 
          那么读出来只会是 12.
      3. GETSET 命令先设置某个值,再返回原来的值。其场景是统计一个小时的访问量并复位。在一个小时内有访问加 1 ,当一个小时
         到时，GETSET 0, 返回 old_value
      4. MSET/MGET 批量的设置和获取可以减少吞吐量
              > mset a 10 b 20 c 30
                 OK
              > mget a b c
              1) "10"
              2) "20"
              3) "30"
      5. exists 命令判断 key 是否存在
              > set mykey hello
              OK
              > exists mykey
              (integer) 1
              > del mykey
              (integer) 1
              > exists mykey
              (integer) 0
      6. 设置 key 的生存时间
        (1) expire 命令，单位是秒
                > expire key 5
                  (integer) 1
        (2) 在 set 命令的 ex 选项(option)
                  > set key 100 ex 10
                  OK
                  > ttl key   (ttl 还剩多少 expire 时间)
                  (integer) 9
        (3) 如果过期时间要以【毫秒】为单位, 则用【pexpire】 和 【pttl】命令
      7. list 操作
            (1) rpush 命令将元素插到 list tail 中。lpush 命令则是将元素插入 list read 中.
            (2) lrang 命令(lrang first_index last_index)， 查看范围 first_index ~ last_index，从 0 开始
                如果 last_index 为 -1,则代表最后一个元素. -2 代表倒数第二个元素.
                实例:
                            > rpush mylist A
                            (integer) 1
                            > rpush mylist B
                            (integer) 2
                            > lpush mylist first
                            (integer) 3
                            > lrange mylist 0 -1
                            1) "first"
                            2) "A"
                            3) "B"
            (3) 可以一次性插入多个元素, rpush mylist 1 2 3 4 5 "foo bar"
            (4) 可以进行 rpop 和 lpop 操作, 这两个操作在 list 为空时里面返回 nil
                    > rpush mylist a b c
                    (integer) 3
                    > rpop mylist
                    "c"
                    > rpop mylist
                    "b"
            (5) brpop 和 blpop 则是使消费者(consumer) 能够阻塞等待。
            (6) 显示输出一个上限的 list，rpush 可以随便 rpush 任何多个元素到 list 中，但你可以用 ltrim 来
                约束要 lrang 的范围，同时注意当重新 rpush/lpush 后，原先 ltrim 就失效了
                    > rpush mylist 1 2 3 4 5
                    (integer) 5
                    > ltrim mylist 0 2
                    OK
                    > lrange mylist 0 -1
                    1) "1"
                    2) "2"
                    3) "3"
             (6) llen 命令返回 list 的元素个数 
             
      8. redis hashes
            (1) hset key field value , 代表一个 hashes 中一个 key 可以有多个 field, 以及各自的 field_value.
            (2) > hmset user:1000 username antirez birthyear 1977 verified 1
                OK
                > hget user:1000 username
                "antirez"
            (3) hincrby 命令对 key 中的某个 field 字段进行操作
                > hincrby user:1000 birthyear 10
                (integer) 1987
            (4) hlen 查看键值对的数量
                > hlen user:1000
                
            (5) HSCAN key cursor [MATCH pattern] [COUNT count]
                    cursor: 下标, 从 0 开始
                    MATCH: 匹配的模式, 正则表达式, 例如,  HSCAN myhash 0 match "run*"
                    COUNT: 输出的个数, 例如, HSCAN myhash 0 COUNT 10 , 输出 10 
      9. sets
            (1) set 是无序保存的
                    > sadd myset 1 2 3
                    (integer) 3
                    > smembers myset
                    1. 3
                    2. 1
                    3. 2       
                    从结果来看， set 是无序保存的
            (2) smembers 命令，打印所有的 set 的元素，但每次输出结果顺序不一样.
            (3) sismember 命令(sismember myset 3)，查看元素是否存在
            (4) sinter 命令得出各个 sets 的交集
            (5) spop 
            (6) sunionstore : 对 set_src 复制到 set_dst ( sunionstore set_dst set_src)
            (7) scard 命令: set 集合的个数. 
            (8) srandmember 命令: srandmember key [count], 随机从 set 选取元素
      10. sorted sets
            (1)  zadd key score member  
                      > zadd hackers 1940 "Alan Kay"
                      (integer) 1
                      > zadd hackers 1957 "Sophie Wilson"
                      (integer) 1
                      > zadd hackers 1953 "Richard Stallman"
                      (integer) 1
            (2) 输出
                  > zrange hackers 0 -1 (正序输出)
                      1) "Alan Turing"
                      2) "Hedy Lamarr"
                      3) "Claude Shannon"
                  
                  > zrevrange hackers 0 -1 (逆序输出)
                       (1)  "Claude Shannon"
                       (2)  "Hedy Lamarr"
                       (3)  "Alan Turing"
                  >  zrange hackers 0 -1 withscores (正序访问，也输出 score)
                        1) "Alan Turing"
                        2) "1912"
                        3) "Hedy Lamarr"
                        4) "1914"
            (3) 对 range 的操作
                  a. zrangebyscore key min max [WITHSCORES] [LIMIT offset count]
                  实例: 
                        > zrangebyscore hackers -inf 1950    (min:-inf 为负无穷)
                        1) "Alan Turing"
                        2) "Hedy Lamarr"
                        3) "Claude Shannon"
            (4) zremrangebyscore : 通过 score 范围删除元素
                      > zremrangebyscore hackers 1940 1960
                        (integer) 4
            (5) zrank : 获取这个元素的在 sorted set 的位置(index 值从 0 开始)， zrank key member
                      > zrank hackers "Hedy Lamarr"
                        (integer) 0
            (6) zrevrank : 逆序获取这个元素的在 sorted set 的位置，就是最后一个元素是为 0 , 往前推
            (7) 如果要对某个元素的 score 进行更新, 则直接使用 zadd 已有的 member 就能直接更新了
            
        11. bitmaps
              (1)  setbit 命令:  setbit key offset value, 其中如果 offset 的值超过保存的 string 的长度，则自动扩展
                                 
              (2)  getbit 命令: getbit key offset， 如果 offset 的值超过保存的 string 的长度,则返回 0
              (3)  bitop 命令: bitop operation destkey key , 其中 oper 有  AND, OR, XOR and NOT.
              (4)  bitcount 命令: bitcount key [start end], 统计 key 中有 1 的值总共有多少位
              (5)  bitpos 命令: bitpos key bit_value [start] [end] : 得到第一个值为 bit_value(1 或则 0) 的索引位置
        
```

## 单机数据库
### 数据库
```shell
    1. struct redisServer{
            // 服务器的数据库数量
            int dbnum;
            // 数组,保存服务器中的所有数据库
            redisDb *db;
        }
        
    2. 切换数据库
        127.0.0.1:6379> SELECT 1
        OK
        127.0.0.1:6379[1]> set msg_1 1
        OK
        127.0.0.1:6379[1]> get msg_1
        "1"
        127.0.0.1:6379[1]> SELECT 0
        OK
        127.0.0.1:6379> get msg_1
        (nil)
        
        命令 select 操作是进行目标数据库的切换, 默认 Redis 服务器有 16 个数据库, redisServer.db[0] ~ redisServer.db[15]
        select 0 起始对应于服务器内部的 reidisDb 结构体
        typedef struct redisClient {
            // 记录客户端当前正在使用的数据库
            redisDb *db;
        }
        
    3. 数据库键空间
             typedef struct redisDb {
                  // 数据库键空间,保存数据库中所有键值对
                  dict *dict;
             }
        (1) 添加新键, 要添加一个新的键值对到数据库中,实际上就是将一个新的键值对添加到键空间字典了,其中键为字符串对象,而值
            则为任意 Redis 对象.
        (2) 删除键,根据指定的键删除对应的键值对.
                redis> del book
        (3) 更新键
        (4) 其他键空间操作, FLUSHDB 对应清空数据库(删除键空间所有键值对) randomkey 命令 对应随机返回某个键
            dbsize 用于返回数据库数量.
        (5) 维护
                redis 的状态
                127.0.0.1:6379> INFO stats
                # Stats
                total_connections_received:2
                total_commands_processed:66
                instantaneous_ops_per_sec:0
                total_net_input_bytes:2837
                total_net_output_bytes:37238
                instantaneous_input_kbps:0.00
                instantaneous_output_kbps:0.00
                rejected_connections:0
                sync_full:0
                sync_partial_ok:0
                sync_partial_err:0
                expired_keys:0
                expired_stale_perc:0.00
                expired_time_cap_reached_count:0
                evicted_keys:0
                keyspace_hits:6
                keyspace_misses:2
                pubsub_channels:0
                pubsub_patterns:0
                latest_fork_usec:435
                migrate_cached_sockets:0
                slave_expires_tracked_keys:0
                active_defrag_hits:0
                active_defrag_misses:0
                active_defrag_key_hits:0
                active_defrag_key_misses:0
                
                过期时间是在读取时进行判断,服务器在读取一个键时对该键进行过期判断,如果已经过期了,则删除这个过期键,再进行接下来的操作.
                
    4. 设置键的生存时间或则过期时间
            (1) 设置键的生存时间: 通过 expire 命令(秒) 或则 pexpired 命令(毫秒)来设置 key 的存活时间,存活时间一到,该键删除.
                127.0.0.1:6379> set key1 1
                OK
                127.0.0.1:6379> expire key1 5  // 设置存活秒数
                (integer) 1
                
            (2) 设置键的过期时间: 通过 expireat 命令(时间戳,秒)或则 pexpiredat 命令(时间戳,毫秒)设置 key 在某一个时间点后
                过期删除.
                
                [time 命令]获取当前的时间戳
                [ttl 命令](秒) 和 pttl 命令(秒): 返回该键的剩余存活时间
                
                127.0.0.1:6379> time  // 获取当前的时间戳
                1) "1566976518"
                2) "541749"
                127.0.0.1:6379> expireat key1 1566976530
                (integer) 1
            
            (3) [expire 命令], [pexpired 命令], [expireat 命令] 这三个命令都可以转化为 [pexpiredat 命令](时间戳,毫秒)
                [persist 命令]移除过期时间,即对 key 不进行过期时间的设置
                redis> persist key_1
            (4) 
                typedef struct redisDb {
                      // 过期字典, 保存键的过期时间, 过期字典的键是一个指针指向键空间的某个键, 过期字典的值则是以毫秒为单位的
                      // 时间戳
                      dict *expires;
                 }
                 
                过期时间的判断具体实现是根据现在的时间戳与 redisDb.expires 中对应的过期时间戳进行比较
    5. 过期时键删除策略
            第一种方法: 定时删除: 在设置键的过期时间时,同时创建一个定时器,当定时器检测到过期时间到达时,删除对应的键.
                       缺点: 需要占用大量的 CPU 资源来判断键是否过期,优点: 较快将过期键从过期 dict 中删除,减少内存的
                       使用,创建一个定时器需要 Redis 服务器中的时间事件(具体的实现采用无序链表, 查找一个事件的时间复杂度为
                       O(N),不高效), 所以不推荐使用.
            第二种方法: 惰性删除: 在每次从键空间取键时进行判断,检测是否过期,如果过期则将键删除.
                       优点是对 CPU 友好,只对要处理的键进行判读.缺点是占用内存,如果一些键很长时间没有被访问,则该键一直不会
                       被删除,很占用内存.
            第三种方法: 定期删除:每隔一段时间,程序检测数据库中键的过期时间,将过期的键删除
                       定期删除策略的难点是在删除操作执行的时长和频率.
                       
            Redis 过期删除策略是配合使用惰性删除和定期删除,惰性删除具体实现 db.c/expireIfNedded 函数实现. 定期删除由 
            redis.c/activeExpireCycle 函数实现, 每当 Redis 的服务器周期性操作 redis.c/serverCron 函数执行，
            activeExpireCycle 函数会被调用，分多次遍历服务器中的各个数据库，从数据库中的 expires 字典随机抽取一部分
            键进行过期时间的判断，过期了就删除．
                
    6. RDB 文件
    
        1. 表示 Redis 内存快照, 记录某一时刻的数据. 
        2. Redis 是全量数据快照, 内存中的所有数据进行备份, 这些通过 bgsave 子进程来备份, 防止阻塞主线程, 其中主线程 fork 
           bsave 子进程时, 会进行主进程页表的复制(页表中是保存键值对指针的地址), 那么子进程有 fork 那个时刻的内存所有键值对的数据, 
           主进程和 bsave 子进程会共用键值对的物理地址, 有 2 份不同的页表, 但页表里面的值都指向相同键值对的物理地址. 写时复制是指
           当主进程接受写操作时, 会重新申请一个物理地址进行写入, 而主进程的页表中的映射改为新的物理地址.
        3. 快照过程中数据能修改吗?
                可以同时进行修改, 不过是通过操作系统提供的写时复制(Copy-On-Write, COW), 主线程正常接受请求, 子进程 bgsave 进行
           对应的备份, 如果主线程需要对某个值进行修改, 则先对这个值进行复制, bgsave 子进程再把原来的值副本写到 
           RDB 文件中(代表 t 时刻的状态), 主线程可以直接修改内存中原来的数据
           
        4. 快照的频率不好把控, 频率太低, 则两次快照间如果宕机了, 有比较多的数据会丢失, 如果频率太高了, 则会阻塞主线程, 原因是主线程
           fork 出子进程需要资源,时间, 而且频率太高, 上一个子进程还没有快照完, 下一个子进程就开始执行了.
           
        5. 解决方案(Redis 4.0)
                混合使用 AOF 日志和内存快照, 内存快照以一定的频率执行, 在两次快照之, 使用 AOF 日志记录这期间的所有命令操作.
           这样就不需要频繁内存快照, AOF 日志也只需要记录两个快照间的所有操作.
           
        6. 内存快照优点
                可以快速的恢复数据库, 把 RDB 文件直接读入内存, 这就避免了 AOF 需要顺序、逐一重新执行操作命令带来的低效性能问题.
                
        7. AOF 和 RDB 选择
                第一: 要求数据不能丢失, 选择内存快照和 AOF 混合使用
                第二: 允许分钟级别的数据丢失, 可以只使用 RDB
                第三: 如果只用 AOF, 则优先使用 everysec 的配置选项，因为它在可靠性和性能之间取了一个平衡
            
               
       在执行 save 命令是将创建一个 RDB 文件，这时程序对数据库的键进行检查，已过期的键不会保存在 RDB 文件中． 
       加载 RDB 文件，在 Redis 服务器启动时, 如果是主服务器家加载 RDB 文件，则需要对文件中的键进行过期检测，如果过期
       则不加入到内存数据库．如果是从服务器则将 RDB 文件中所有数据进行载入不管有没有过期，因为在 Redis 同步时会将从服务器
       清空，全量同步.
            (1) save 命令和 bgsave 命令用于生成 RDB 文件(rdb.c/rdbSave 函数)
                    save 命令会阻塞 Redis 服务器进程(不能处理任何命令), 直到 RDB 文件创建完成. 
                    
                    bgsave 命令则创建出一个子进程, 由子进程进行 RDB 文件的生成, Redis 服务器进程继续执行, 在 bgsave 命令执行期间,
                         客户端发送 save/bgsave 请求会被服务器拒绝,避免同时调用 rdbSave()函数. 这个是 Redis 默认的配置.
                         
            (2) 由于 AOF 文件更新频率通常要比 RDB 文件高(当过期键被惰性删除或则定期删除),所以服务器会优先使用 AOF 文件来
            　　还原数据库状态(内存中的 redis 数据).只有在 AOF 持久化处于关闭状态,服务器才会使用 RDB 还原数据库状态.
                加载 RDB 由 rdb.c/rdbLoad 函数完成.
            (3) bgrewriteaof 命令和 bgsave 命令不能同时执行. 如果 bgsave 正在执行,客户端发送 bgrewriteaof 命令会被延迟到
                bgsave 命令执行完毕后才执行. 如果 bgrewriteaof 正在执行, 客户端发送 bgsave 命令会被拒绝.
            (4) 间隔保存
                    用户通过设置服务器配置 save 选项, 让服务器每隔一段时间自动执行 bgsave 命令.
                    redis> save 900 5     服务器在 900 秒内,超过 5 次对数据库进行修改,就会调用 bgsave 命令
                    struct redisServer {
                    
                        // 记录了上一次执行 save/bgsave 后, 服务器对数据库状态进行多少次修改操作(写入,删除,更新)
                        long long dirty;
                        
                        // 上一次执行 save/bgsave 的时间戳
                        time_t lastsave;
                        ...
                        // 记录 RDB 保存配置选项的数组
                        struct saveparam *saveparams;
                        
                    }
                    
                    struct saveparam {
                        // 秒数 900 
                        time_t seconds;
                        // 修改数 5
                        int changes;
                    }
                    
                    向一个集合键增加三个新元素: sadd database Redis MongoDB MariaDB, 则 dirty 数增加 3
                    Redis 服务器周期性操作函数 serverCron 函数默认每隔 100 ms 执行一次,对服务器进行维护,其中就有
                    检测 save 选项设置保存配置是否满足条件, 如果满足则执行 bgsave 命令, 并将 redisServer.dirty 置为 0,
                    更新 redisServer.lastsave
            (5) RDB 文件结构
                    "REDIS" + db_version(4 字节,字符串整数) + databases + EOF(1 字节) + check_sum(8 字节)
                    
                    databases: 包含数据库状态,其保存的键值对信息
                               databases 结构如下:
                                    SELECTDB(常量,1 字节) + db_number + key_value_pairs
                                    SELECTDB: 当读到这个常量时,接下来要读入一个数据库序号(db_number, 从 0 开始), 因大小不同,
                                    　　　　　 长度可以是 1 字节, 2 字节, 5 字节.拿到 db_number 后,在调用 select 命令切换到对应的
                                              数据库.
                                    key_value_pairs: 保存数据库中的所有键值对，如果键值对中设置了过期时间，则过期时间一起保存.
                                           
                                                     
                    check_sum: 8 字节无符号整数, 一个校验和对 REDIS, db_version, databases, EOF 四部分的内容进行计算.服务器在
                               加载 RDB 文件时,将各个部分进行校验和在跟 check_num 比较看数据是否损坏.
            
    7. AOF(append only File)
           基本概念
                (1) Redis 采用 AOF 方式(先执行命令再记录日志), 
                        优点:
                             第一: 避免 AOF 文件中出现记录错误命令的情况, 因为记录日志命令操作不会进行语法检查, 要确保这边记录是否正确,
                                  可以先执行, 执行成功后在记录.
                             第二: 不会阻塞当前的写操作, 因为是先执行命令后记录日志.
                             
                        风险:
                            第一: 如果已经执行完命令, 在写到 AOF 文件之前宕机了, 那么重启后无法用日志进行恢复.
                            第二: AOF 写到磁盘操作慢会阻塞其他的操作, 因为 AOF 写日志是在主线程中执行的.
                            
                            主要还是考虑 AOF 写回到磁盘的时机.
                            
                (2) AOF 文件格式
                        *3           // 当前命令有 3 个部分
                        $3           // 数字表示后面命令/键/值一共有多少个字节 
                        set
                        $7
                        testkey
                        $9
                        testvalue
                        
           AOF 三种写回策略(对应于 AOF 配置项 appendfsync)
                (1) 第一种 Always
                        同步写回: 每个写命令执行完后, 立马同步将日志写回磁盘
                        优点: 可靠性高, 数据基本不丢失
                        缺点: 每次都要同步更新到磁盘, 主线程的性能影响较大.
                        
                (2) 第二种 Everysec
                        每秒写回：每个写命令执行完, 只是先把日志写到 AOF 文件的内存缓冲区, 每隔一秒把缓冲区中的内容写入磁盘.
                        优点: 性能适中
                        缺点: 宕机 1 秒内数据丢失
                        
                (3) 第三种 No
                        依托于操作系统控制的写回: 每次写命令执行完, 只是将日志写到 AOF 文件的内存缓冲区, 再由操作系统决定何时将缓冲区
                        内容写回磁盘
                        优点: 性能好
                        缺点: 宕机丢失的数据较多.
                        
                        
           AOF 文件存在问题
                (1) 文件大小受文件系统限制, 无法保存过大的文件
                (2) 如果文件太大, 之后再往里面追加命令记录的话, 效率也会变低
                (3) 发生宕机时, AOF 中记录的命令要一个个被重新执行, 用于故障恢复，如果日志文件太大, 整个恢复过程就会非常缓慢, 
                    影响 Redis 的正常使用, 所以 AOF 文件不适用于宕机时的数据恢复.
                    
                    
           AOF 重写机制
                (1) 原因是: AOF 是对操作命令不断的追加, 可能对同一个键进行依次的操作, 但我们关心的是同一个键最新的状态. 可以对 AOF
                           文件进行整合, 重写.
                           
                           AOF 日志是在主线程中进行的, 而 AOF 重写机制是由子进程 bgrewriteaof(主线程 fork 出来的)
                           
                (2) 过程(一个拷贝, 两处日志)
                        每次进行 AOF 重写时, 都会 fork 出 bgrewriteaof 子进程, 同时将内存中键值对数据拷贝给子进程, 主线程还是
                     照常处理新的操作, 更新到 AOF 日志, 而另外一个子进程的 AOF 重写日志也在进行, 这个新的操作也会写到重写日志的
                     缓冲区, 同时直接根据数据库里数据的最新状态, 生成这些数据的插入命令, 再将 AOF 重写日志内容替换到 AOF 日志中.
                
                            
           其他
                (1) 只有当过期键被惰性删除或则定期删除后，才向 AOF 文件追加一条 DEL 命令．
                (2) 执行 bgrewriteaof 命令进行 AOF 重写的过程, 这时程序对数据库的键进行检查，已过期的键不会保存在 AOF 文件中
    8. 当 Redis 运行在复制模式下(主从复制)，从服务器的过期键删除动作由主服务器控制，即当客户端在从服务器中获取过期的键时，还是会
    　　放回键值并且不删除，但当客户端在主服务器中获取过期的键时，删除过期的键，并且向从服务器发送 del key 命令，将从服务器上的
    　　键删除, 这样保证了数据的一致性.
    9. 数据库通知
            让客户端通过订阅模式，来获得数据库中键的变化以及数据库中命令的执行情况．
            键空间通知(key-space notification) : 关注指定的键执行了什么命令　__keyspace@<dbid>__:<key>
            键事件通知(key-event notification): 关注指定的命令(例如: del)有哪些键被执行． __keyspace@<dbid>__:<event>
            发送通知的实现 notifyKeyspaceEvent(type, event, key, dbid) 函数：　如果给定的通知类型 type 不是服务器允许
            发送的通知类型，那么函数会直接返回．否则检测是否为发送键空间通知，如果是则将通知发送给频道 __keyspace@<dbid>__:<key>,
            内容为键所发生的事件<event>,. 如果是键事件通知, 则将通知发送给频道 __keyevent@<dbid>__:<event>, 内容为发生
            事件的键<key>.
       
```

## 编程

### pipelining(流水线)
```shell
  1. 使用 pipelining 将多条命令同时发出，提高 redis 的查询。
  2. Redis 作为 tcp server 使用了 client-server 模式，client 请求等待 redis server 的响应。Redis 服务的处理能力要远远大于
     网络传输的速度，如果都是依次请求响应，再请求等待响应，那么效率不是很高，会影响性能。
  3. pipelining 技术则是不需要等待上一次服务器的响应，一次性发送多条命令，最后一起接受多条响应数据。需要注意一点，
     如果客户端使用 pipelining 技术发送多条命令时， redis 服务端会占用额外的内存来排队组织回复消息.
  4. 效率的提升不仅仅是 RTT 的缩小，对于 redis 服务端而言调用 read()/write 系统函数进行 socket IO 操作，
     使得程序不断在用户态到内核态中切换很耗时间。一次性从 read() 中读取多个请求，且一次性将多个响应信息写到 write() 中，
     极大提高效率.
  5. 在基准测试中，用死循环来测试 redis 的性能问题是不明智的
     例如:
            FOR-ONE-SECOND:
                Redis.SET("foo","bar")
            END
     即使是在 loopback 的环境中，把网络延迟排查了，但需要考虑内核调度.这个时候 benchmark 测试程序向 redis　发送
     命令，这时需要等待 redis server 的响应,此时内核将会调度 cpu 给 redis server 进行运行.
```

### 延迟监控
```shell
  1. 延迟监控组成
        (1) 延迟钩子(搜集不同的延迟代码路径)
        (2) 通过不同事件将延迟峰值分隔为时间序列记录
        (3) 上报引擎从时间序列中取 raw data
        (4) 分析引擎提供可行性报告(主要通过检测值)
  2. events and time series
        (1) events: 不同的监测代码，例如 command event 主要监视 slow commands 的延迟峰值. 而延迟峰值则是该 event 超过配置的
                    延迟门限值
        (2) 一个 time series 关联一个监测事件(monitor event), 其原理如下:
                a. 当延迟峰值发生时，会被记录在合适时间序列(time series)
                b. 每个时间序列(time series) 有 160 元素构成
                c. 每个元素是对(pair)形式存在，(时间戳, 事件被执行的毫秒数)
                d. 在一秒内相同事件产生延迟峰值背会合并，因为用户可能设置很小的门限值，那么至少 180 秒都会有延迟峰值(latency spikes)
  3. 启动延迟监控(latency monitoring)
        (1) ./redis-cli
            127.0.0.1:6379> CONFIG SET latency-monitor-threshold 10    定义 10 毫秒阈值，event 超过 10 毫秒会被记录.
  4. 延迟监控命令
        (1) latency latest 命令
            记录最近被记录的 events, 输出 4 个部分, 第一部分事件名字(event name), 第二部分最新事件记录时间戳(超过阈值)，
            第三部分最新事件记录(超过阈值)它的毫秒数，第四部分这类事件持续最长时间(毫秒数) 
        例如：
            127.0.0.1:6379> CONFIG SET latency-monitor-threshold 10   // 设置 10 毫秒被记录
            127.0.0.1:6379> debug sleep 2     // command event 持续 2 秒
            OK
            (2.00s)
            127.0.0.1:6379> debug sleep .25   //command event 持续 0.25
            127.0.0.1:6379> LATENCY latest
            1) 1) "command"            // event 名
               2) (integer) 1563263917  // 时间戳
               3) (integer) 250   // 最近一次超过阈值的持续时间(毫秒)
               4) (integer) 2000  // 最长一次超过阈值的持续时间(毫秒)
        (2) latency history event-name
            输出 event-name 的超过阈值的记录 pair<时间戳, 持续的毫秒数>, 最多返回 160 条记录。
        例如:
            127.0.0.1:6379> latency history command
            1) 1) (integer) 1563263902
               2) (integer) 1000
            2) 1) (integer) 1563263908
               2) (integer) 2000
        (3) latency reset [event-name]
            重置了 event 超过阈值的记录信息(清空记录),如果不指明 event-name , 则将所有的 events 记录清空。
            latency reset 返回是成功删除的event 数量.
        (4) latency graph event-name
            较直观的反映出被记录下来的趋势分布，其中垂直标签(向下读)，代表这一列运行 latency graph 命令时多久前执行的。
        例如: 
            127.0.0.1:6379> debug sleep .5
            127.0.0.1:6379> debug sleep .5
            127.0.0.1:6379> debug sleep .1
            127.0.0.1:6379> debug sleep .5
            127.0.0.1:6379> debug sleep .5
            127.0.0.1:6379> debug sleep .3
            127.0.0.1:6379> LATENCY graph command
            command - high 500 ms, low 100 ms (all time high 500 ms)
            --------------------------------------------------------------------------------
            ## ## 
            || ||_
            || |||
            ||_|||

            988777
            mmmmmm
            
            其中第一列是在 9 minute 前执行的， 最后一列是在 7 minute 前执行的
        (5) latency doctor
            分析报告
        例如:
            127.0.0.1:6379> LATENCY doctor
            Dave, I have observed latency spikes in this Redis instance. You don't mind talking about it, do you Dave?

            1. command: 6 latency spikes (average 400ms, mean deviation 133ms, period 202.67 sec). 
            Worst all time event 500ms.

            I have a few advices for you:

              - Check your Slow Log to understand what are the commands you are running which are too slow to execute. 
              - Please check http://redis.io/commands/slowlog for more information.
              - Deleting, expiring or evicting (because of maxmemory policy) large objects is a blocking operation.
              -  If you have very large objects that are often deleted, expired, or evicted, try to fragment those 
                 objects into multiple smaller objects.
```

## 文档资料
```shell
  1. redis 内存优化  https://redis.io/topics/memory-optimization
  2. 分布式
        (1) 将数据分区在不同的 redis 中 https://redis.io/topics/partitioning
        (2) redis 分布式锁 https://redis.io/topics/distlock
        (3) redis 复制： https://redis.io/topics/replication
        (4) 高可用 : https://redis.io/topics/sentinel
        (5) 吞吐量检测: https://redis.io/topics/latency-monitor
        (6) 基准测试: https://redis.io/topics/benchmarks
        (7) redis 集群: https://redis.io/topics/cluster-tutorial
                        https://redis.io/topics/cluster-spec
  3. 索引
        (1) 创建二级索引 https://redis.io/topics/indexes
  4. 规格书(specifications)
        (1) 设计草案: https://redis.io/topics/rdd
        (2) redis 通信协议: https://redis.io/topics/protocol
        (3) Redis RDB format : https://github.com/sripathikrishnan/redis-rdb-tools/wiki/Redis-RDB-Dump-File-Format
            
```



## 编程接口(hiredis.h)

```shell
    1. redisContext *redisConnect(const char *ip, int port);
       描述:
            redis 连接
       返回值:
            结果
```

