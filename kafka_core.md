# kafka 源码

## 概念
```shell
    1. 资料
            (1) https://blog.csdn.net/lianggx3/category_10406229.html
            
```
## Scala 编程
```shell
    1. 定义变量和函数
            (1) 变量
                    val 等同于 Java 中的 final 变量, 一旦被初始化, 就不能再被重新赋值
                    var 是非 final 变量, 可以重复被赋值
                        var a:Long = 1L, 
                        var a = 1L;  // 也可以不跟数据类型, 这个要具体的值自动匹配了
                        a:Long  // 变量名后面可以跟 : + 数据类型 
                        
            (2) 函数
                    a.
                        def max(x: Int, y: Int): Int = {
                            if (x > y) x
                            else y
                        }
                        
                        max 是函数名, 括号中的 x 和 y 是函数输入参数, 它们都是 Int 类型的值. 结尾的“Int =”组合表示 max 函数
                        返回一个整数, 没有使用 return 关键字, 而是直接写了 x 或 y. 在 Scala 中, 函数体具体代码块最后一行的
                        值将被作为函数结果返回.
                        
                    b. Scala 的 Unit 类似于 Java 的 void
                    
    2. 元组(Tuple)
            (1) 是个容器, 一旦被创建, 就不能再被更改. 元组中的数据可以是不同数据类型的
                    val a = (1, 2.3, "hello", List(1,2,3))
                    a._1 // 访问元组的第一个元素
                        
    3. 循环
            (1) 命令式编程
                val list = List(1, 2, 3, 4, 5)
                for(elem <- list)
                    println(elem)
                    
            (2) 函数式编程
                 list.foreach(e => println(e))
                 
    3. case 类
            (1) Case 类适合用来表示不可变数据, case 类自动地为所有类字段定义 Getter 方法.
                   对比与 java 写法:
                        public final class Point {
                            private int x;
                            private int y;
                            public Point(int x, int y) {
                            this.x = x;
                            this.y = y;
                            }
                            // setter methods......
                            // getter methods......
                        }
                        
                   scala 的写法:
                   
                        case class Point(x:Int, y: Int) // 默认写法, 不能修改x和y
                        case class Point(var x: Int, var y: Int) // 支持修改x和y
                        
                        Scala 会自动地帮你创建出 x 和 y 的 Getter 方法
                        
    4. 模式匹配
            (1) 
                def describe(x: Any) = x match {
                    case 1 => "one"
                    case false => "False"
                    case "hi" => "hello, world!"
                    case Nil => "the empty list"
                    case e: IOException => "this is an IOException"
                    case s: String if s.length > 10 => "a long string"
                    case _ => "something else"
                }
                
                 x 是 Any 类型, 所有 Object 类型, case _”的写法相当于 default
                 
    5. Option 对象
            (1) 更好地规避 NullPointerException 异常的
            (2) Option[Any] 中括号里面的 Any 就是上面说到的 Any 类型, 可以是任何类型. 如果值存在的话, 使用 Some(x) 来获取值
                或给值赋值, 否则就使用 None 来表示
                
            (3) 
                val keywords = Map("scala" -> "option", "java" -> "optional")  // 创建 map 对象
                keywords.get("java") , // 有值返回 "optional" 值. Option[String] = Some(optional)
                keywords.get("C")  //  获取 key 值为 C 的 value 值. 由于不存在故返回 None, Option[String] = None
                    
```

## 日志模块
### 日志段(LogSegment)
```shell
    1. 概念
            (1) 状态管理机和副本管理器都是以日志为基础的
    2. kafka 日志结构
            (1) kafka 日志对象由多个日志段对象组成, 每个日志段对象会在磁盘上创建一组文件, 包括消息日志文件(.log), 
                位移索引文件(.index), 时间戳索引文件(.timeindex), 已中止(Aborted)事务的索引文件(.txnindex), 该日志段中的
                文件名是该日志段的起始位移值(Base Offset), 即该日志段中所存的第一条消息的位移值
                
            (2) 一个 Kafka 主题有很多分区, 每个分区就对应一个 Log 对象, 在物理磁盘上则对应于一个子目录. 比如创建一个双分区的主题
                test-topic, 那么 Kafka 在磁盘上会创建两个子目录：test-topic-0 和 test-topic-1, 就是两个 Log 对象.每个子目录下
                存在多组日志段, 也就是多组.log, .index, .timeindex 文件组合
                
    3. 日志段类声明
            class LogSegment private[log] 
                (
                    val log: FileRecords,
                    val lazyOffsetIndex: LazyIndex[OffsetIndex],
                    val lazyTimeIndex: LazyIndex[TimeIndex],
                    val txnIndex: TransactionIndex,
                    val baseOffset: Long,
                    val indexIntervalBytes: Int,
                    val rollJitterMs: Long,
                    val time: Time
                ) extends Logging { … }
                
            参数:
                log: 实际保存 Kafka 消息的对象(FileRecords), 跟消息日志文件有关
                lazyOffsetIndex: 延迟初始化的原理, 降低了初始化时间成本, 跟位移索引文件有关
                lazyTimeIndex: 延迟初始化的原理, 降低了初始化时间成本, 跟时间戳索引文件有关
                txnIndex: 跟已中止事务索引文件有关
                baseOffset: 每个日志段对象保存自己的起始位移, 在磁盘上看到的文件名就是 baseOffset 的值.每个 LogSegment 对象实例
                            一旦被创建, 起始位移就固定了, 不能再被更改
                indexIntervalBytes: 控制日志段对象新增索引项的频率, 默认情况下日志段至少新写入 4KB 的消息数据才会新增一条索引项
                rollJitterMs: 日志段对象新增倒计时的“扰动值", 因为日志段对象新增倒计是全局配置, 所以存在未来某一个时间点出现大量的
                              日志段对象同时被创建, 造成磁盘 I/O 压力很大, 有 rollJitterMs 值的干扰, 每个新增日志段在创建
                              时会彼此岔开一小段时间, 可以缓解物理磁盘的 I/O 负载瓶颈
                time: 用于统计计时的一个实现类
                
    4. append 方法
            def append(
                        largestOffset: Long,  // 消息批次中消息的最大位移值
                        largestTimestamp: Long,  // 消息批次中消息的最大时间戳
                        shallowOffsetOfMaxTimestamp: Long, // 最大时间戳对应消息的位移
                        records: MemoryRecords  // 真正要写入的消息集合
                        
                       )
                       
            第一步:
                     判断该日志段是否为空, 如果是空 Kafka 记录要写入消息集合的最大时间戳, 并将其作为后面新增日志段倒计时的依据.
            第二步:
                    确保传入参数的最大位移值是合法, 判断的标准是它与日志段起始位移的差值是否在一个日志段最大数据量内. 如果越界了,
                    考虑升级下 kafka 版本(已知问题)
            第三步:
                    调用 log.append() , 将内存中的消息写入到操作系统的页缓存
            第四步:
                    更新日志段的最大时间戳以及最大时间戳所属消息的位移值属性. 每个日志段都要保存当前最大时间戳信息和所属消息的位移信息.
                     Broker 端提供定期删除日志就是根据最大时间戳信息, 同时保存了时间戳索引项(最大时间戳信息与所属位移的信息)
            第五步:
                    更新索引项和写入的字节数, 日志段每写入 4KB 数据就写入一个索引项. 同时清空已写入字节数, 用来下次重新累积计算
                    
    5. read 方法
            def read(
                        startOffset: Long,  // 要读取的第一条消息的位移
                        maxSize: Int,  // 能读取的最大字节数
                        maxPosition: Long = size,  // 能读到的最大文件位置, 即以字节为索引单位.
                        minOneMessage: Boolean = false  // 是否允许在消息体过大时至少返回第一条消息
                    )
                    
                  minOneMessage 为 true 时, 出现消息体字节数超过 maxSize , read 方法依然能返回至少一条消息. 引入这个参数主要是
                  为了确保不出现消费饿死的情况
                  
            第一步:
                    根据 startOffset 位移值, 得到读取的起始文件位置 (startPosition)
                    
            第二步:
                    再根据 maxSize 和 maxPosition 参数共同计算要读取的总字节数. 例如 maxSize=100, maxPosition=300,  
                    startPosition=250, 那么 read 方法只能读取 50 字节, 因为 maxPosition - startPosition = 50, 和 maxSize
                    比起来取较小的.
                    
            第三步:
                    读取消息, 调用 FileRecords 的 slice 方法
                    
    6. recover 方法
            恢复日志段: Broker 在启动时会从磁盘上加载所有日志段信息到内存中, 并创建相应的 LogSegment 对象实例
            
            第一步:
                    清空所有索引文件
                    
            第二步: 
                    校验日志段中所有消息集合, 6.1 校验消息集合, 该消息必须要符合 Kafka 定义的二进制格式, 该集合中最后一条消息的位移值
                    不能越界, 即它与日志段起始位移的差值必须是一个正整数值. 6.2 保存最大时间戳以及所属消息的位移值, 根据这两项, 更新
                    索引项, 不断累加当前已读取的消息字节数, 更新事务型 Producer 的状态以及 Leader Epoch 缓存

            第三步:
                    执行消息日志索引文件截断. Kafka 会将日志段当前总字节数和刚刚累加的已读取字节数进行比较, 如
                    果发现前者比后者大, 说明日志段写入了一些非法消息, 需要执行截断操作, 将日志段大小调整回合法的数值,同时调整索引文件
                    相应的大小
         
```

### 日志对象(Log)
```shell
    1. 概念
            (1) 日志定义了很多管理日志段的操作.
            (2) 定义同名的 Class 和 Object, 属于 Scala 中的伴生对象用法
            (3) Kafka 日志文件固定是 20 位的长度
    2. 日志模块下的类
            (1) LogAppendInfo
                    LogAppendInfo(C): 保存一组待写入消息的各种元数据信息. 如, 这组消息中第一条消息的位移值, 最后一条消息的
                                      位移值, 这组消息中最大的消息时间戳等等.
                    LogAppendInfo(O): 为其对应伴生类的工厂方法类, 里面定义一些工厂方法, 用于创建特定的 LogAppendInfo 实例
                    
                    
                    case class LogAppendInfo(var firstOffset: Option[Long],
                                             var lastOffset: Long, // 消息集合最后一条消息的位移值
                                             var maxTimestamp: Long, // 消息集合最大消息时间戳
                                             var offsetOfMaxTimestamp: Long, // 消息集合最大消息时间戳所属消息的位移值
                                             var logAppendTime: Long, // 写入消息时间戳
                                             var logStartOffset: Long, // 消息集合首条消息的位移值
                                             // 消息转换统计类，里面记录了执行了格式转换的消息数等数据
                                             var recordConversionStats: RecordConversionStats,
                                             
                                             // 消息集合中消息使用的压缩器（Compressor）类型，比如是Snappy还是LZ4
                                             sourceCodec: CompressionCodec, 
                                             targetCodec: CompressionCodec, // 写入消息时需要使用的压缩器类型
                                             shallowCount: Int, // 消息批次数，每个消息批次下可能包含多条消息
                                             validBytes: Int, // 写入消息总字节数
                                             offsetsMonotonic: Boolean, // 消息位移值是否是顺序增加的
                                             // 首个消息批次中最后一条消息的位移, 在 0.11 以后指向 消息集合的最后一条消息
                                             lastOffsetOfFirstBatch: Long, 
                                             recordErrors: Seq[RecordError] = List(), // 写入消息时出现的异常列表
                                             errorMessage: String = null) {  // 错误码
                                             ......
                                             }
                    
            (2) Log
                    Log(C): Log 源码中最核心的代码.
                    Log(O): Log 伴生类的工厂方法, 定义了很多常量以及一些辅助方法
                    
            (3) RollParams
                    RollParams（C）： 用于控制日志段是否切分(Roll)的数据结构
                    RollParams（O）： RollParams 伴生类的工厂方法
                    
            (4) LogMetricNames：定义 Log 对象的监控指标
            (5) LogOffsetSnapshot：封装分区所有位移元数据的容器类
            (6) LogReadInfo：封装读取日志返回的数据及其元数据
            (7) CompletedTxn：记录已完成事务的元数据, 主要用于构建事务索引
            
    3. Log Class & Object
            (1) 伴生对象多用于保存静态变量和静态方法(比如静态工厂方法等), Log Object 如下:
                    object Log {
                        val LogFileSuffix = ".log" 
                        val IndexFileSuffix = ".index"  // 位移索引文件.
                        val TimeIndexFileSuffix = ".timeindex"
                        val ProducerSnapshotFileSuffix = ".snapshot"   // 幂等型或事务型 Producer 快照文件
                        val TxnIndexFileSuffix = ".txnindex"
                        // 删除日志端的中间文件, 删除日志段文件是异步操作, Broker 端会把日志段文件从.log 后缀修改为.deleted 后缀
                        val DeletedFileSuffix = ".deleted"  
                        val CleanedFileSuffix = ".cleaned"  //  Compaction 操作的产物
                        val SwapFileSuffix = ".swap"  //  Compaction 操作的产物
                        val CleanShutdownFile = ".kafka_cleanshutdown"
                        val DeleteDirSuffix = "-delete"  // 应用于文件夹, 当删除一个主题时, 其下的分区文件夹会被加上这个后缀
                        val FutureDirSuffix = "-future"  // 是用于变更主题分区文件夹地址
                        ……
                    }
                    
            (2) Log 类
                    class Log(
                                @volatile var dir: File,   // 重要, 这个日志所在的文件夹路径, 即主题分区的路径
                                @volatile var config: LogConfig,
                                @volatile var logStartOffset: Long,  // 重要, 日志的当前最早位移, 对外提供可见最早一条消息
                                @volatile var recoveryPoint: Long,
                                scheduler: Scheduler,
                                brokerTopicStats: BrokerTopicStats,
                                val time: Time,
                                val maxProducerIdExpirationMs: Int,
                                val producerIdExpirationCheckIntervalMs: Int,
                                val topicPartition: TopicPartition,
                                val producerStateManager: ProducerStateManager,
                                logDirFailureChannel: LogDirFailureChannel
                                ……
                                
                                // 下一条待插入消息的偏移值, LEO
                                @volatile private var nextOffsetMetadata: LogOffsetMetadata = _
                                // 高水位
                                @volatile private var highWatermarkMetadata: LogOffsetMetadata = ....
                                
                                // 用 Map 的数据结构来保存分区日志下所有的日志段信息, Key 值是日志段的起始位移值, Value 则是日志段对象本身
                                private val segments: ConcurrentNavigableMap[java.lang.Long, LogSegment] =
                                
                                // Leader Epoch 用来判断出现 Failure 时是否执行日志截断操作  
                                // Leader Epoch Cache 是一个缓存类数据，里面保存了分区 Leader 的 Epoch 值与对应位移值的映射关系
                                @volatile var leaderEpochCache: Option[LeaderEpochFileCache] = None
                                
                             )
                    
                    初始化 log
                        1. 创建分区日志路劲
                        2. Leader Epoch Cache 
                                (1) 创建 Leader Epoch 检查点文件.
                                (2) 生成 Leader Epoch Cache 对象.
                                
                        3. 加载所有日志段对象
                                (1) 遍历分区日志下所有文件, 删除上次 Failure 遗留下来各种临时文件
                                   (包括 .cleaned, .swap, .delete 文件)
                                (2) 清空已有日志段集合, 并重新加载日志段文件
                                (3) 处理第一步返回的有效 .swap 文件集合
                                (4) recoverLog 操作, 恢复日志段对象, 然后返回恢复之后的分区日志 LEO 值
                          
                        5. 更新 nextOffsetMetadata 和 logStartOffset
                        6. 更新 Leader Epoch Cache, 清除无效数据.
                        
    4. Log 对象常见操作
            (1) 高水位管理操作
                    a. 高水位值的初始值是 Log Start Offset 值, 每个 Log 对象都会维护一个 Log Start Offset 值
                    b. 高水位对象 LogOffsetMetadata 类型
                            LogOffsetMetadata(
                                                messageOffset: Long,
                                                segmentBaseOffset: Long = Log.UnknownOffset,
                                                relativePositionInSegment: Int = LogOffsetMetadata.UnknownFilePosition
                                             ) 
                                参数:
                                        messageOffset : 消息位移值, 对应就是高水位的值
                                        segmentBaseOffset: 日志段的起始地址(该消息位移值所处的日志段), 可以用来判断两条消息是否
                                                           处于同一个日志段的
                                        relativePositionInSegment: 保存该位移值所在日志段的物理磁盘位置
                                        
                    c. 获取和设置高水位, 主要就是操作 LogOffsetMetadata.messageOffset 的值
                    d. 更新高水位值
                            有 2 个方法, updateHighWatermark 方法和 maybeIncrementHighWatermark 方法, 
                       updateHighWatermark 方法, 主要用在 Follower 副本从 Leader 副本获取到消息后更新高水位值, 一旦拿到新的消息,
                       就必须要更新高水位值；
                       maybeIncrementHighWatermark 方法, 主要是用来更新 Leader 副本的高水位值. Leader 副本高水位值的更新是
                       有条件的——某些情况下会更新高水位值, 某些情况下可能不会, Producer 端向 Leader 副本写入消息时, 分区的高水位值
                       就可能不需要更新——因为它可能需要等待其他 Follower 副本同步的进度.
            (2) 日志段管理
                    a. val segments: ConcurrentNavigableMap[java.lang.Long, LogSegment] = new
                       通过 ConcurrentNavigableMap 将日志段进行管理, 第一 ConcurrentNavigableMap 结构本身具备线程安全, 
                       同时 key 可以排序,  Key 为每个日志段的起始位移, 根据所有日志段的起始位移值对它们进行排序和比较, 同时还
                       能快速地找到与给定位移值相近的前后两个日志段
                       
                    b. 日志段对象添加, 简单加入到 map 中
                    c. 日志段对象的删除, 得考虑到日志留存策略(基于时间维度的, 基于空间纬度的, 基于 Log Start Offset 维度),
                       deleteOldSegments() 方法会判断是否开启删除规则, 如果开启, 那么会分别调用：
                        deleteRetentionMsBreachedSegments() 删除 segment 的时间戳超过了设置时间的日志段；
                        deleteRetentionSizeBreachedSegments() 删除日志段空间超过设置空间大小的日志段. 
                        同时无论是否开启删除规则, 都会删除在 log start offset 之前的日志段
                    d. 替换, 用新的日志段替换旧的日志段.
                   
            (3) 关键位移值管理
                    a. 对于 Log Start Offset 和 LEO 等位移的管理
                    b. LEO 对象被更新的时机
                            第一: Log 对象初始化时, 要创建一个 LEO 对象, 并对其进行初始化.
                            第二: 写入新消息时, LEO 不对向后移动
                            第三: Log 对象发生日志切分(Log Roll), 发生在当前日志段对象已满的时候, 发生日志切分时, Log 对象切换
                                 Active Segment, 这时 LEO 和段大小都要被更新.
                            第四: 日志截断(Log Truncation) 的时候
                    c. Log Start Offset 更新时机
                            第一: Log 对象初始化时, 一般是将第一个日志段的起始位移值赋值给它
                            第二: 日志截断时, 一旦日志中的部分消息被删除, 可能会导致 Log Start Offset 发生变化
                            第三: Follower 副本同步时, 一旦 Leader 副本的 Log 对象的 Log Start Offset 值发生变化,
                                  Follower 副本也需要尝试去更新该值.
                            第四: 删除日志段
                            第五: 删除消息时, 删除消息通过后移 Log Start Offset 的值来实现.
            (4) 读写操作
                    a. 主要是读写日志.
                    b. append 写
                            第一步: 分析并验证待写入的消息集合, 并返回校验结果(LogAppendInfo)
                            第二步: 消息格式规整, 即删除无效格式或则无效字节.　如果需要分配位移值, 使用当前 LEO 值作为待写入消息集合
                                   中第一条消息的位移值, 验证消息, 确保消息大小不超限
                            第三步: 更新 Leader Epoch 缓存.
                            第四步: 确保消息大小不超限
                            第五步: 执行日志切分, 如果当前的日志段满了, 或则日志段中的第一个消息的 maxTime 已经过期, 或则 index 
                                   索引满了, 则进行日志切分
                            第六步: 验证事务状态
                            第七步: 执行真正的消息写入操作, 调用日志段对象的 append 方法实现
                            第八步: 更新 LEO 对象
                            第九步: 更新事务状态
                            第十步: 返回写入结果
                            
                    c. read 读取
                            1. 
                                def read(
                                            startOffset: Long,  // 从 Log 对象的哪个位移值开始读消息
                                            maxLength: Int,  // 最多能读取多少字节
                                            // 设置读取隔离级别, 主要控制能够读取的最大位移值, 多用于 Kafka 事务
                                            isolation: FetchIsolation,
                                            // 即是否允许至少读一条消息. 如果消息很大, 超过 maxLength, read 方法永远不会返回
                                            // 任何消息. 但如果设置该参数为 true, read 方法就保证至少能够返回一条消息
                                            minOneMessage: Boolean
                                         ): FetchDataInfo = {
                                            ......
                                            }
```

### 索引对象
```shell
    1. 概念
    2. 索引文件组织架构
            a. AbstractIndex.scala：定义最顶层的抽象类, 封装所有索引类型的公共操作
            b. LazyIndex.scala: 定义 AbstractIndex 的一个包装类, 实现索引项延迟加载, 主要是提高性能
            c. OffsetIndex.scala: 定义位移索引, 保存 < 相对位移值, 文件磁盘物理位置 >对
            d. TimeIndex.scala: 定义时间戳索引, 保存 < 时间戳, 相对位移值 > 对
            e. TransactionIndex.scala: 定义事务索引, 为已中止事务(Aborted Transcation)保存重要的元数据信息. 只有启用 Kafka 
                                       事务后, 这个索引才有可能出现.
                                       
    3. AbstractIndex
            a. 
                abstract class AbstractIndex(
                                  // 每个索引对象在磁盘上都对应一个索引文件, 1.1 以后允许迁移底层的日志路径, 所以设置为 volatile 
                                  @volatile var file: File, 
                                  
                                  // 索引对象对应日志段对象的起始位移值, 正常情况下 日志文件和索引文件都是成组出现
                                  val baseOffset: Long, 
                                  
                                  // 控制索引文件的最大长度, 默认情况下 segment.index.bytes(10MB) 参数传入
                                  val maxIndexSize: Int = -1, 
                                  val writable: Boolean  // 是否支持可写
                                ) 
                                
                                
                              
                AbstractIndex 中有 protected def entrySize: Int // 抽象方法 entrySize 来表示不同索引项的大小
                  对于 OffsetIndex(位移索引) 而言, 该值就是 8,  这里的位移值是相对于 baseOffset 的位移值, 同时代表每个日志段最多为
                  2^32, 即 4GB. 
                  对于 TimeIndex(时间戳索引) 而言, 该值是 12, 同理这里的位移值也是相对位移.
                  
            b. 通过内存映射文件创建索引文件, 这种方式是在虚拟内存中(实际上是页缓存), 流程步骤如下:
                    第一步: 创建索引文件
                    第二步: 以读写方式或只读方式打开索引文件
                    第三步: 设置索引文件长度(预设的索引文件大小不能太小)
                    第四步: 更新索引长度字段
                    第五步: 创建 MappedByteBuffer 对象
                    第六步: 如果是新创建的索引文件, 将 MappedByteBuffer 对象的当前位置置成 0, 如果索引文件已存在, 
                           将 MappedByteBuffer 对象的当前位置设置成最后一个索引项所在的位置 
                           
            c. 写入索引项
                    第一步: 确定索引文件未写满
                    第二步: 如果索引文件为空或则要写入的位移大于当前已写入索引项的位移(_lastOffset), 则向索引对象 mmap 写入
                    　　　　相对位移值和物理位置偏移, 更新统计信息(当前索引项数量 _entries 和当前已写入索引项的位移_lastOffset)．
                    第三步: 执行校验. 写入的索引项格式必须符合要求, 索引项个数 * 单个索引项占用字节数 == 当前文件物理大小
                    
            d. 索引查找
                    主要是通过二分查找形式进行查找, 但由于 kafka 的索引是通过追加文件的形式, 同时这些索引项部分是缓存中内存中, 原先
                    需要查找 page 12 中的索引值, 按照常规的二分法, 需要 先通过 page 0, 6, 9, 11, 12. 如果此后随着索引的增加, 
                    page 12 会占满, 则新的索引项在 page 13 填充, 此时常规二分法需要 Page #0, 7, 10, 12 和 13. page 7 和 19
                    缓存数据可能会缺页, 重新从磁盘中读取, 拉低效率.
                    
                    改进方法：　设置数据的热区和冷区, 保证经常被访问的各个 page　是固定的, 查询最热那部分数据所遍历的 Page 永远是
                              固定的, 大概率在页缓存中, 从而避免无意义的 Page Fault 
                              
    4. 位移索引(OffsetIndex)
            (1) 概念
                    a. 位移索引项, 
                            key 值为消息的相对位移(4 字节), 采用相对位移是因为每个 OffsetIndex 对象在创建时, 都已经保存对应
                            　　　　日志段对象的起始位移, 只需要保存与起始位移的差值就可以了.
                            value 值(4 字节)为保存该消息的日志段文件中该消息第一个字节的物理文件位置
                            
                    b. OffsetIndex(位移索引)被用来快速定位消息所在的物理文件位置
     
    5. 时间戳索引(TimeIndex)
            (1) 概念
                    a. 时间戳索引项, <时间戳(8 字节), 相对位移(4 字节)>
                    
    6. 实践
            (1) 先使用 TimeIndex 获取指定时间戳对应的消息位移值, 然后再利用 OffsetIndex 去定位该位移值所在的物理文件位置
            (2) 不要随意更改索引文件的文件名和里面的内容.
            
```

## 请求处理模块

### Request/Response 机制
```shell
    1. 概念
    2. Request/Response 机制
          (1)  
            class Request(
                            /* processor 是 Processor 线程的序号, 即这个请求是由哪个 Processor 线程接收处理的, 
                             * 保存 Processor 线程序号原因是当 Request 被后面的 I/O 线程处理完成后, 还得依靠 Processor 线程
                             * 发送 Response 给请求发送方, Processor 线程仅仅是网络接收线程, 不会执行真正的 Request 请求
                             * 处理逻辑( I/O 线程负责)
                            val processor: Int, 
        
                            val context: RequestContext, // context 是用来标识请求上下文信息(RequestContext)
                            
                            // 记录 Request 对象被创建的时间(单位为纳秒), 用于各种时间统计指标的计算
                            val startTimeNanos: Long, 
                            memoryPool: MemoryPool, // 一个非阻塞式的内存缓冲区, 避免 Request 对象无限使用内存
                            
                            /*  Request 发送方必须按照 Kafka RPC 协议规定的格式向该缓冲区写入字节, 否则会抛出 
                             *　InvalidRequestException 异常
                            @volatile private var buffer: ByteBuffer, // 真正保存 Request 对象内容的字节缓冲区
                            
                            /* metrics 是 Request 相关的各种监控指标的一个管理类, 构建一个 Map,封装所有的请求 JMX 指标
                            metrics: RequestChannel.Metrics 
                           ) extends BaseRequest {
                                ......
                             }
                             
                            
            参数:
                    processor : Broker 端参数 num.network.threads 控制 Broker 每个监听器上创建的 Processor 线程数
                    
                    context : 
                              public class RequestContext implements AuthorizableRequestContext {
                                // Request 头部数据, 主要是对用户不可见的元数据信息, 如 Request 类型, Request API 版本, clientId等
                                public final RequestHeader header;
                                
                                // Request 发送方的 TCP 连接串标识, 由 Kafka 根据一定规则定义, 用于表示 TCP 连接
                                public final String connectionId; 
                                
                                public final InetAddress clientAddress; // Request 发送方 IP 地址
                                public final KafkaPrincipal principal;  // Kafka用户认证类, 用于认证授权
                                // 监听器名称, 可以是预定义的监听器(如 PLAINTEXT), 也可自定义
                                public final ListenerName listenerName; 
                                
                                // 安全协议类型, 目前支持4种：PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL
                                public final SecurityProtocol securityProtocol; 
                                public final ClientInformation clientInformation; // 用户自定义的一些连接方信息
                                
                                // 从给定的 ByteBuffer 中提取出 Request 和对应的 Size 值
                                public RequestAndSize parseRequest(ByteBuffer buffer) {
                               
                                }
                                // 其他Getter方法
                                ......
                            }
                            
            
            
          (2) Response
                Response: 定义 Response 的抽象基类. 每个 Response 对象都包含对应的 Request 对象, 其中 onComplete 方法, 用来实现
                          Response 被处理后需要执行的回调逻辑
                SendResponse: 保存返回结果的 Response 子类, 里面的 onCompletionCallback, 指定处理完成之后的回调逻辑
                NoResponse: 有些 Request 处理完成后无需单独执行额外的回调逻辑, NoResponse 就是为这类 Response 准备
                CloseConnectionResponse：用于出错后需要关闭 TCP 连接的场景, 返回 CloseConnectionResponse 给 Request 发送方,
                                        显式地通知它关闭连接
                                        
                abstract class Response(val request: Request) {
                   // request
                  locally {
                    val nowNs = Time.SYSTEM.nanoseconds
                    request.responseCompleteTimeNanos = nowNs
                    if (request.apiLocalCompleteTimeNanos == -1L)
                      request.apiLocalCompleteTimeNanos = nowNs
                  }
                  
                  def processor: Int = request.processor
                  def responseString: Option[String] = Some("")
                  def onComplete: Option[Send => Unit] = None
                  // 覆写
                  override def toString: String
                }
                
           
          (3) RequestChannel 
                    是传输 Request/Response 的通道
                    
                class RequestChannel(val queueSize: Int, // Request 队列的最大长度
                                     val metricNamePrefix : String,
                                     time: Time) extends KafkaMetricsGroup {
                                     
                          import RequestChannel._
                          val metrics = new RequestChannel.Metrics
                          
                          /* 用阻塞队列保存 Broker 接收到的各类请求, queueSize 默认为 500(queued.max.requests), 
                           * 每个 RequestChannel 上的队列长度是 500
                          private val requestQueue = new ArrayBlockingQueue[BaseRequest](queueSize)
                          
                          // Processor 线程池, <线程序号, 线程对象>
                          private val processors = new ConcurrentHashMap[Int, Processor]()
                          
                          val requestQueueSizeMetricName = metricNamePrefix.concat(RequestQueueSizeMetric)
                          val responseQueueSizeMetricName = metricNamePrefix.concat(ResponseQueueSizeMetric)
                  }
                  
                KafkaMetricsGroup 封装许多实用的指标监控方法, 如 newGauge 方法用于创建数值型的监控指标
                RequestChannel 的作用
                    1. 管理 Processor
                            Kafka Broker 端所有网络线程 processors 都是在　RequestChannel 中维护的, 每当 Broker 启动时, 
                       都会调用  addProcessor 方法, 向 RequestChannel 对象添加 num.network.threads 个 Processor 线程, 
                       同时这个 num.network.threads　参数允许在 broker 允许时动态修改
                       
                    2. 处理 Request 和 Response
                            即发送 Request, 将 Request 对象放到 Request 队列中, 接受 Request, 即从队列中取出 Request 对象.
                       发送 Response, 根据　Response.processor, 从网络线程池中找到对应的线程, 将该 Response 压入到对应
                       Processor 线程的 Response 队列中
                       
                    3. 监控指标( Request 和 Response 的性能)
                    
                            object RequestMetrics {
                                // 每秒处理的 Request 数, 用来评估 Broker 的繁忙状态
                                val RequestsPerSec = "RequestsPerSec"
                                
                                /* 计算 Request 在 Request 队列中的平均等候时间(毫秒), 如果 Request 在队列的等待时间过长, 
                                 * 需要增加后端 I/O 线程的数量加快队列中 Request 的拿取速度
                                val RequestQueueTimeMs = "RequestQueueTimeMs"
                                
                                /* 计算 Request 实际被处理的时间(毫秒)
                                 *
                                val LocalTimeMs = "LocalTimeMs"
                                
                                /* 等待其他 Broker 完成指定逻辑的时间, 例如 producer 端的 ack=all 设置. 
                                val RemoteTimeMs = "RemoteTimeMs"
                   
                                /* 计算 Request 被处理的完整流程时间
                                val TotalTimeMs = "TotalTimeMs"
                          
                            }
```

### NIO 通信机制
```shell
    1. 概念
            (1) kafka 网络通信主要由 SocketServer 组件和 KafkaRequestHandlerPool 构成.
            (2) SocketServer 组件包含了 Acceptor 线程, Processor 线程和 RequestChannel 等对象
                实现 Reactor 模式, 处理外部多个 Clients(包含 Producer, Consumer 或其他 Broker)的并
                发请求, 并负责将处理结果封装进 Response 中, 返还给 Clients. 
            (3) KafkaRequestHandlerPool 组件就是 I/O 线程池, 用于执行实际的请求处理逻辑.
            (4) 如果　Clients 与 Broker 的通信网络延迟很大（比如 RTT>10ms）, 可以调大控制缓冲区大小的参数(sendBufferSize 和
                recvBufferSize)
    2. SocketServer 组件构成
            (1) AbstractServerThread 类: Acceptor 线程和 Processor 线程的抽象基类, 定义公有方法, 如 shutdown(关闭线程)
            (2) Acceptor 线程类: 接收和创建外部 TCP 连接的线程, 每个 SocketServer 实例只会创建一个 Acceptor 线程, 创建连接,
                                并将接收到的 Request 传递给下游的 Processor 线程处理
            (3) Processor 线程类:　这一个 Processor 线程处理单个 TCP 连接上所有请求的线程, 每个 SocketServer 实
                例默认创建 num.network.threads 个　Processor 线程. Processor 线程负责将接收到的 Request 添加到,
                Processor 线程是在 Acceptor 线程中管理和维护的
                RequestChannel 的 Request 队列上, 同时还负责将 Response 返还给 Request 发送方
            (4) Processor 伴生对象类：定义了一些与 Processor 线程相关的常见监控指标和常量等, 如 Processor 线程空闲率等
            (5) SocketServer 类: 对以上所有组件的管理和操作, 如创建和关闭 Acceptor, Processor 线程等
            (6) SocketServer 伴生对象类: 定义一些有用的常量, 明确 SocketServer 组件中的哪些参数是允许动态修改的
            
    3. Acceptor 线程
            (1) Acceptor 线程相当于 Reactor 模式的 Dispatcher.
                   class Acceptor( 
                                    val endPoint: EndPoint,
                                    val sendBufferSize: Int, 
                                    val recvBufferSize: Int,
                                    brokerId: Int, // broker 节点 id
                                    connectionQuotas: ConnectionQuotas, 
                                    metricPrefix: String) extends AbstractServerThread(connectionQuotas) 
                                   with KafkaMetricsGroup {
                                   
                              /* 创建底层的NIO Selector对象
                               * Selector对象负责执行底层实际 I/O 操作, 如监听连接创建请求、读写请求等
                              private val nioSelector = NSelector.open()
                              
                              /* Broker 端创建对应的 ServerSocketChannel 实例, 再把该 serverChannel 向 Selector 对象注册
                              val serverChannel = openServerSocket(endPoint.host, endPoint.port)
                              
                              // 创建 Processor 线程池, Processor 线程数组, 同时该类中进行对 Processor 线程池的管理
                              private val processors = new ArrayBuffer[Processor]()
                              private val processorsStarted = new AtomicBoolean
                              private val blockedPercentMeter = newMeter(s"${metricPrefix}AcceptorBlockedPercent",
                                "blocked time", TimeUnit.NANOSECONDS, Map(ListenerMetricTag -> endPoint.listenerName.value))
                            }
                    
                   参数:
                            endPoint: 定义 Kafka Broker 连接信息, 如 PLAINTEXT://localhost:9092. Acceptor 需要用到 
                                      endPoint 包含的主机名和端口信息创建来 Server Socket
                                      
                            sendBufferSize：　SocketOptions 的 SO_SNDBUF, 用于设置出站(Outbound)网络 I/O 的底层缓冲区大小.
                                             该值默认是 Broker 端参数 socket.send.buffer.bytes 的值, 即 100KB
                                      
                            recvBufferSize:  SocketOptions 的 SO_RCVBUF, 即用于设置入站(Inbound)网络 I/O 的底层缓冲区大小.
                                             该值默认是 Broker 端参数 socket.receive.buffer.bytes 的值, 即 100KB
                                                         
            (2) run 方法
                    第一步: Acceptor 线程启动
                    第二步: 循环地轮询准备就绪的 I/O 事件(处理 tcp 的连接), 调用 accept 方法创建 Socket 连接, 同时通过轮询方法
                    　　　　从 Processor 线程池中指定 processor 线程, 将这个 socket 连接加入到 Socket processor 线程中.
                    
                    
    4. Processor 线程
            (1) 
                 class Processor(
                                    val id: Int,
                                    time: Time,
                                    maxRequestSize: Int,
                                    requestChannel: RequestChannel, // Processor 与 Handler 线程之间传递请求数据的队列 
                                    connectionQuotas: ConnectionQuotas,
                                    connectionsMaxIdleMs: Long,
                                    failedAuthenticationDelayMs: Int,
                                    listenerName: ListenerName,
                                    securityProtocol: SecurityProtocol,
                                    config: KafkaConfig,
                                    metrics: Metrics,
                                    credentialProvider: CredentialProvider,
                                    memoryPool: MemoryPool,
                                    logContext: LogContext,
                                    connectionQueueSize: Int = ConnectionQueueSize
                                 ) 
                               extends AbstractServerThread(connectionQuotas) with KafkaMetricsGroup {
                              /* 创建的新连接信息( SocketChannel 对象) 记录分配给当前 Processor 的待处理的 SocketChannel 对象 
                               * 每当 Processor 线程接收新的连接请求时, 都会将对应的 SocketChannel 放入这个队列, 保存的是要
                               * 创建的新连接信息
                              private val newConnections = new ArrayBlockingQueue[SocketChannel](connectionQueueSize)
                              
                              
                              /* 这是一个临时 Response 队列, 当 Processor 线程将 Response 返还给 Request 发送方之后, 还要将
                               * Response 放入这个临时队列.因为有些 Response 回调逻辑要在 Response 被发送回发送方之后才能执行
                              private val inflightResponses = mutable.Map[String, RequestChannel.Response]()
                              
                              /* 每个 Processor 线程都会维护自己的 Response 队列
                               private val responseQueue = new LinkedBlockingDeque[RequestChannel.Response]()
                               }
                               
                              
            (2) run 方法
                    override def run(): Unit = {
                        // 标识当前线程启动完成
                        startupComplete()
                        try {
                          while (isRunning) {
                            try {
                              /*  1. 创建新监听事件, 从 newConnections 获取 SocketChannel 对象注册到 selector 中进行该连接
                               *     事件监听, 注册 OP_READ 事件
                              configureNewConnections()
                              
                            
                              /* 2. 处理待处理 Response 对象(从　responseQueue　队列中取出), 发送 Response 给 Request 发送方, 
                              　　　　并且将 Response 放入临时 Response 队列(inflightResponses 队列)
                              processNewResponses()
                              
                              /* 3. 执行实际 I/O 操作
                                    发送缓存的响应对象给客户端, 执行 NIO poll, 获取对应 SocketChannel 上准备就绪的I/O操作
                              *     真正执行 I/O 动作的方法是这里的 poll 方法, poll 方法的核心代码就 selector.poll(pollTimeout)
                              *     在底层, 实际调用的是 Java NIO Selector 的 select 方法去执行那些准备就绪的 I/O 操作,
                              *     不管是接收 Request, 还是发送 Response。
                              poll()
                              
                              /* 4. 将接收到的 Request 放入 Request 队列
                               *    遍历处理 poll 操作放置在 Selector 的读数据, 封装请求信息为　Request 对象, 并记录请求队列中
                               　　　等待 Handler 线程处理, 同时标记当前 Selector 暂时不再接收新的请求
                               *    将接收到的 Request 放入 Request 队列
                              processCompletedReceives()
                              
                              /* 5. 为已成功发送 Response 对象调用回调逻辑
                               *    为临时 Response 队列中的 Response 执行回调逻辑
                               *    遍历底层 SocketChannel 已发送的 Response, 将其从 inflightResponses 集合中移除, 对其执行
                               　　　Response 的回调函数, 并标记当前 Selector 可以继续读取数据
                              processCompletedSends()
                              
                              /* 6. 处理因发送失败而导致的连接断开
                               *    从底层 Selector 中获取那些已经断开的连接, 之后把它们从 inflightResponses 中移除掉,
                                   同时更新它们的配额数据(减少对应 IP 上的连接数)
                              processDisconnected()
                     
                              /* 关闭超过配额限制部分的连接(在 TCP 连接中找出最近未被使用的, 即在最近一段时间内, 没有任何 
                                  Request 经由这个连接被发送到 Processor 线程 )
                              closeExcessConnections()
                            } catch {
                              case e: Throwable => processException("Processor got uncaught exception.", e)
                            }
                          }
                        } finally {
                          // 关闭底层资源
                          debug(s"Closing selector - processor $id")
                          CoreUtils.swallow(closeAll(), this, Level.ERROR)
                          shutdownComplete()
                        }
                      }
                      
            (3) 队列
                    1. newConnections
                            保存将要连接的 SocketChannel 对象, 固定上限是 20, 每当 Processor 线程接收新的连接请求时, 将对应的 
                       SocketChannel 放入这个队列. 后面在创建连接时(调用 configureNewConnections), 就从该队列中取出
                       SocketChannel, 然后注册新的连接
                       
                    2. inflightResponses
                            这是一个临时 Response 队列. 当 Processor 线程将 Response 返还给 Request 发送方之后, 还要将 
                       Response 放入这个临时队列, 因为有些 Response 回调逻辑要在 Response 被发送回发送方之后, 才能执行, 主要
                       用于暂存.
                       
                    3. responseQueue
                            每个 Processor 线程都会维护自己的 Response 队列, 保存着需要被返还给发送方的所有 Response 对象
                            
            (4) 方法
                    1.  configureNewConnections() 可以知道　Processor 线程都维护了一个 Selector 类实例, SocketChannel(连接)
                     　　注册到该 Selector 中, 进行事件监听(读事件)
                    2. processCompletedReceives() 
                            接收和处理 Request 的逻辑, Processor 从底层 Socket 通道读取已接收到的网络请求, 然后转换成 Request 
                       实例, 并将其放入到 Request 队列, requestChannel.sendRequest(req) 就是将 Request 放入 Request 队列
                   
```

### 请求优先级
```shell
    1. 概念
            (1) Data plane(数据类请求) : 例如, PRODUCE 和 FETCH 请求
            (2) Control plane(控制类请求): 例如, LeaderAndIsr, StopReplica 和 UpdateMetadata
            (3) 监听器(listen)
                    通过建立多组监听器来区分数据类请求和控制类请求的处理逻辑, listeners 和 advertised.listeners 进行配置监听器,
                格式为: 监听器名称://主机名：端口, 例如: PLAINTEXT://kafka-host:9092
                    listenerName：监听器名字, 预定义的名称有 PLAINTEXT, SSL, SASL_PLAINTEXT 和 SASL_SSL. 允许自定义其他监听器
                                  名称, 比如 CONTROLLER, INTERNAL 等
                    
                    目前监听器使用安全协议有 PLAINTEXT, SSL, SASL_PLAINTEXT 和 SASL_SSL
                    
            (4) 在 SocketServer 组件启动之后, 才启动 Processor 和 Acceptor 线程
                    
    2. 
        class SocketServer(val config: KafkaConfig,
                           val metrics: Metrics,
                           val time: Time,
                           val credentialProvider: CredentialProvider)
          extends Logging with KafkaMetricsGroup 
          with BrokerReconfigurable {
              
              /* SocketServer 的请求队列长度, 由 Broker 端参数 queued.max.requests　决定, 默认是 500
              private val maxQueuedRequests = config.queuedMaxRequests
              ........
              // data-plane
              private val dataPlaneProcessors = new ConcurrentHashMap[Int, Processor]()
              
              /* 处理数据类请求的 Acceptor 线程池, 每套监听器对应一个 Acceptor 线程
              private[network] val dataPlaneAcceptors = new ConcurrentHashMap[EndPoint, Acceptor]()
              val dataPlaneRequestChannel = new RequestChannel(maxQueuedRequests, DataPlaneMetricPrefix, time)
              
              /* control-plane 用于处理控制类请求的 Processor 线程, 目前定义了专属的 Processor 线程而非线程池来处理控制类请求
               * Control plane 的配套资源只有 1 个 Acceptor 线程 + 1 个 Processor 线程 + 1 个的请求队列(大小为 20)
               * 开启 Control plane 设置, 其 Processor 线程只有 1 个, Acceptor 线程也是 1 个. 对应的 RequestChannel 里面的
               * 请求队列长度被硬编码成 20, 而不是一个可配置的值. 即控制类请求的数量应该远远小于数据类请求, 不需要为它创建线程池和
               * 很长的请求队列
              private var controlPlaneProcessorOpt : Option[Processor] = None
              private[network] var controlPlaneAcceptorOpt : Option[Acceptor] = None
              
              /* 处理控制类请求专属的 RequestChannel 对象
              val controlPlaneRequestChannelOpt: Option[RequestChannel] = config.controlPlaneListenerName.map(_ =>
                new RequestChannel(20, ControlPlaneMetricPrefix, time)
              
        }
        
         Acceptor 线程池：保存 SocketServer,  为每个监听器定义的 Acceptor 线程, 负责分发该监听器上的入站连接建立请求
         Processor 线程池：即网络线程池, 负责将请求放入到请求队列中.
         RequestChannel: 承载请求队列的请求处理通道.
         
    3. 创建 Data plane 所需资源
            遍历配置的所有监听器, 然后为每个监听器执行以下操作：
                (1) 初始化该监听器对应的最大连接数计数器
                (2) 为该监听器创建 Acceptor 线程
                (3) 创建 Processor 线程池(线程个数由 num.network.threads 决定)
                (4) 将 <监听器, Acceptor 线程> 加入到 Acceptor 线程池统一管理
                
            例如, 配置 listeners=PLAINTEXT://localhost:9092, SSL://localhost:9093,  会为 PLAINTEXT 和 SSL 这两套监听器
                分别创建一个 Acceptor 线程和一个 Processor 线程池, 前提是这是 Data plane
                
    4. 创建 Control plane 所需资源
            第一步: 如果配置了用于 Control plane 监听器, 初始化该监听器对应的最大连接数计数器
            第二步: 创建 Acceptor 线程
            第三步: 创建 Processor 线程
            第四步: 将 Processor 线程添加到专属 RequestChannel, 将 Processor 线程添加到 Acceptor 下的 Processor 线程池.
            
        注意: 只能有 1 套监听器用于 Control plane, 而不能像 Data plane 那样可以配置多套监听器
        在 broker 端通过 control.plane.listener.name 来指定哪个监听器用于 Control plane, 默认是空的, 不启动请求优先级区分机制.
        例如:
            listener.security.protocol.map=CONTROLLER:PLAINTEXT,INTERNAL:PLAINTEXT,EXTERNA
            listeners=CONTROLLER://192.1.1.8:9091,INTERNAL://192.1.1.8:9092,EXTERNAL://10.11.1.8:9092
            control.plane.listener.name=CONTROLLER
        
    5. kafka 可以为数据类请求和控制类请求分别设置各自的监听器, 各自处理对应的请求.
            
```

### 请求处理流程
```shell
    1. 概念
    2. KafkaRequestHandler 文件组成
            (1) KafkaRequestHandler: 请求处理线程类. 每个请求处理线程实例, 负责从 SocketServer 的 RequestChannel 的请求队列
                                     获取请求对象, 并处理.
                KafkaRequestHandlerPool：请求处理线程池, 负责创建, 维护, 管理和销毁请求处理线程.
                BrokerTopicMetrics: Broker 端主题相关监控指标的管理类
                BrokerTopicStats(C):  Broker 端主题相关监控指标的管理操作
                BrokerTopicStats(O): BrokerTopicStats 的伴生对象类, Broker 端主题相关的监控指标, 如常见的 MessagesInPerSec
                                     和 MessagesOutPerSec 等
                                        
    3. KafkaRequestHandler 类
            class KafkaRequestHandler(
                                        /* I/O 线程序号, 即请求处理线程的序号
                                        id: Int, 
                                        brokerId: Int, // 所在 Broker 序号, 即 broker.id
                                        val aggregateIdleMeter: Meter,
                                        val totalHandlerThreads: AtomicInteger, // I/O 线程池大小
                                        /* 请求处理通道, Kafka 在构造 KafkaRequestHandler 实例时, 必须关联 SocketServer 组件
                                         * 的 RequestChannel 实例, 这个就是要获取请求的地方(请求保存在 RequestChannel 的请求队列)
                                        val requestChannel: RequestChannel, 
                     apis: KafkaApis, // KafkaApis　类，　用于真正实现请求处理逻辑的类
                                        time: Time) extends Runnable with Logging {
               }
               
            
            run 方法: 循环处理请求, 从 RequestChannel 中拿到请求, 更新一些相关的统计指标, 如果从请求队列中获取的是普通请求, 先更新
                     请求移出队列的时间戳, 再交由 KafkaApis 的 handle 方法执行实际的请求处理逻辑代码, 请求处理完成, 释放缓冲区资源.
                     进入到下一轮循环　
                     
    4. KafkaRequestHandlerPool 类
        class KafkaRequestHandlerPool(
                                        val brokerId: Int, // 所在 Broker 序号, 即 broker.id
                                        
                                        /* SocketServer 的请求处理通道管理的请求队列被所有 I/O 线程所共享
                                        val requestChannel: RequestChannel, 
                                        val apis: KafkaApis, // api：KafkaApis类，实际请求处理逻辑类
                                        time: Time,
                                        numThreads: Int, //  num.io.thread, 运行后可以动态调整　
                                        requestHandlerAvgIdleMetricName: String,
                                        logAndThreadNamePrefix : String) extends Logging with 
                                        KafkaMetricsGroup {
                                        }
                                        
    5. 全流程
            第一步: Clients 或其他 Broker 发送请求给 Acceptor 线程. Acceptor 线程通过调用 accept 方法, 创建对应的 
                   SocketChannel, 然后将该 Channel 实例传给 assignNewConnection 方法, 等待 Processor 线程将该 Socket 连接
                   请求, 放入到它维护的待处理连接队列中. 后续 Processor 线程的 run 方法会不断地从该队列中取出这些 Socket 连接请求,
                   然后监听对应的 Socket 事件.
                   
            第二步: Processor 线程处理请求, 并放入请求队列(从底层 I/O 获取到发送数据, 将其转换成 Request 对象实例, 
                   并最终添加到请求队列的过程)
                   
            第三步: I/O 线程处理请求, 从请求队列中获取 Request 实例, 然后交由 KafkaApis 的 handle 方法, 执行真正的请求处理逻辑.
            
            第四步: KafkaRequestHandler 线程将 Response 放入 Processor 线程的 Response 队列
            
            第五步: Processor 线程发送 Response 给 Request 发送方
                    
```

### kafkaApis
```shell
    1. 概念
            (1) kafkaApis 包含实际的业务逻辑, 例如 创建 Topic 实现, Consumer 提交位移实现等等.
            (2) 在处理不同类型的 RPC 请求时, KafkaApis 会用到不同的组件(ReplicaManager, GroupCoordinator 等等.)
    2. 
        class KafkaApis(val requestChannel: RequestChannel, // SocketServer 组件的请求通道对象, 请求队列提供者
                        val replicaManager: ReplicaManager, // 副本管理器, 管理集群所有副本的状态转换
                        val adminManager: AdminManager, // topic、分区配置等管理器
                        val groupCoordinator: GroupCoordinator, // 消费者组协调器组件, 用于维护消费者组的管理.
                        val txnCoordinator: TransactionCoordinator, // 事务管理器组件, 实现 Kafka 事务功能.
                        val controller: KafkaController, // kafka 集群控制器组件, 管理与保存元数据
                        val zkClient: KafkaZkClient, // // ZooKeeper客户端程序, Kafka 依赖于该类实现与 ZooKeeper 交互
                        val brokerId: Int,   // broker.id参数值
                        val config: KafkaConfig, // Kafka 配置类, 提供 Broker 端参数的定义和保存.
                        val metadataCache: MetadataCache, // 元数据缓存类, 保存和更新集群 Broker 间原数据缓存.
                        val metrics: Metrics,
                        val authorizer: Option[Authorizer],
                        val quotas: QuotaManagers, // 配额管理器组件, 负责客户端, 副本等对象的配额管理.
                        val fetchManager: FetchManager,
                        brokerTopicStats: BrokerTopicStats,
                        val clusterId: String,
                        time: Time,
                        val tokenManager: DelegationTokenManager) extends Logging {
         }
         
    3. handle 处理
        def handle(request: RequestChannel.Request): Unit = {
          try {
            /* 根据请求头部信息中的 apiKey 字段判断属于哪类请求, 然后调用响应的 handleXXXX 方法
             * 如果新增 RPC 协议类型, 则：
             * 1. 添加新的 apiKey 标识新请求类型
             * 2. 添加新的 case 分支
             * 3. 添加对应的 handlexxx 方法   
            request.header.apiKey match {
              case ApiKeys.PRODUCE => handleProduceRequest(request)
              case ApiKeys.FETCH => handleFetchRequest(request)
              case ApiKeys.LIST_OFFSETS => handleListOffsetRequest(request)
              case ApiKeys.METADATA => handleTopicMetadataRequest(request)
              ．．．．．．．
                }
              } catch {
                // 如果是严重错误，则抛出异常
                case e: FatalExitError => throw e
                // 普通异常的话，记录下错误日志
                case e: Throwable => handleError(request, e)
              } finally {
                // 记录一下请求本地完成时间，即Broker处理完该请求的时间
                if (request.apiLocalCompleteTimeNanos < 0)
                  request.apiLocalCompleteTimeNanos = time.nanoseconds
              }
            }
            
    4. 
        sendResponse(RequestChannel.Response): 最底层的 Response 发送方法, 调用 SocketServer 组件中 RequestChannel 的
        sendResponse 方法, 会把待发送的 Response 对象添加到对应 Processor 线程的 Response 队列上, 然后交由 Processor
        线程完成网络间的数据传输
```

## Controller 模块
```shell
    1. 概念
    2. Controller 模块构成
            (1) ControllerContext: 保存 Controller 元数据的容器
            (2) ControllerChannelManager：　Controller　向 Broker 发送请求所用的通道管理器.
            (3) ControllerEventManager: 定义各种 Controller 事件以及这些事件被处理的代码.
            (4) 领导者选举: 负责执行和维护集群上各种领导者选举的实现逻辑
            (5) 其他功能: 更新集群原数据, 删除主题.
```

### 集群元数据
```shell
    1. 概念
            (1) 集群 Broker 会与 Controller 建立通信, 并获取集群中的最新元数据.
    2. 
        class ControllerContext {
          val stats = new ControllerStats // Controller 统计信息类
          var offlinePartitionCount = 0 // 离线分区计数器
          var shuttingDownBrokerIds: mutable.Set[Int] = mutable.Set.empty // 关闭中 Broker 的 Id 列表
          private var liveBrokers: Set[Broker] = Set.empty // 当前运行中 Broker 对象列表
          private var liveBrokerEpochs: Map[Int, Long] = Map.empty // 运行中 Broker Epoch 列表
          var epoch: Int = KafkaController.InitialControllerEpoch // Controller 当前 Epoch 值
          
          /* Controller 对应 ZooKeeper 节点的 Epoch 值, 使用 epochZkVersion 来判断和防止 Zombie Controller, 
          var epochZkVersion: Int = KafkaController.InitialControllerEpochZkVersion 
   
         
          var allTopics: Set[String] = Set.empty
          val partitionAssignments = mutable.Map.empty[String, mutable.Map[Int, ReplicaAssignment]] // 主题分区的副本列表
          // 主题分区的 Leader/ISR 副本信息
          val partitionLeadershipInfo = mutable.Map.empty[TopicPartition, LeaderIsrAndControllerEpoch] 
          val partitionsBeingReassigned = mutable.Set.empty[TopicPartition] // 正处于副本重分配过程的主题分区列表
          val partitionStates = mutable.Map.empty[TopicPartition, PartitionState] // 主题分区状态列表
          val replicaStates = mutable.Map.empty[PartitionAndReplica, ReplicaState] // 主题分区的副本状态列表
          val replicasOnOfflineDirs: mutable.Map[Int, Set[TopicPartition]] = mutable.Map.empty // 不可用磁盘路径上的副本列表
         
          val topicsToBeDeleted = mutable.Set.empty[String] // 待删除主题列表
         
          val topicsWithDeletionStarted = mutable.Set.empty[String] // 已开启删除的主题列表
          val topicsIneligibleForDeletion = mutable.Set.empty[String] // 暂时无法执行删除的主题列表
        }
        
    3. ControllerStats
            /* 统计指标：UncleanLeaderElectionsPerSec 和所有 Controller 事件状态的执行速率与时间
            private[controller] class ControllerStats extends KafkaMetricsGroup {
              /* 统计每秒发生的 Unclean Leader选举次数
              val uncleanLeaderElectionRate = newMeter("UncleanLeaderElectionsPerSec", "elections", TimeUnit.SECONDS)
             
              val rateAndTimeMetrics: Map[ControllerState, KafkaTimer] = ControllerState.values.flatMap { state =>
                /* Controller 事件通用的统计速率指标(毫秒), 由于 Controller 事件有很多种, 对应的速率监控指标也有很多,有一些 
                 * Controller 事件是需要额外关注,  IsrChangeNotification 事件是标志 ISR 列表变更的事件, 如果这个事件经常出现,
                 * 说明副本的 ISR 列表经常发生变化, 是不正常情况
                state.rateAndTimeMetricName.map { metricName =>
                  state -> new KafkaTimer(newTimer(metricName, TimeUnit.MILLISECONDS, TimeUnit.SECONDS))
                }
              }.toMap
            }
            
            
    4. offlinePartitionCount
            统计集群中所有离线或处于不可用状态的主题分区数量, 具体实现: ControllerContext 中的 updatePartitionStateMetrics 方法
      根据给定主题分区的当前状态和目标状态, 来判断该分区是否是离线状态的分区. 如果是, 则累加 offlinePartitionCount 字段的值,
      否则递减该值
      
    5. shuttingDownBrokerIds
            保存所有正在关闭中的 Broker ID 列表, 根据这个字段来判断 Broker 当前是否已关闭, 处于关闭状态的 Broker 是不适合
       执行某些操作的(如分区重分配（Reassignment）以及主题删除)
       
    6. liveBrokers
            保存当前所有运行中的 Broker 对象, 每当新增或移除已有 Broker 时, ZooKeeper 就会更新其保存的 Broker 数据,从而引发
        Controller 修改元数据, 调用 updateBrokerMetadata 方法来增减 Broker 列表 中的对象
        
    7. liveBrokerEpochs
            保存所有运行中 Broker 的 Epoch 信息. 
            
    8. epoch & epochZkVersion(Controller 侧的数据)
            epoch 是 ZooKeeper 中 /controller_epoch 节点的值, 代表 Controller 在整个 Kafka 集群的版本号. 
            epochZkVersion 是 /controller_epoch 节点的 dataVersion 值, 使用 epochZkVersion 判断和防止 
       Zombie Broker(一个非常老的 Broker 被选举成为 Controller)
       
    9. allTopics
            保存集群上所有的主题名称, 每当有主题的增减, Controller 就要更新该字段的值
            
    10. partitionAssignments
            保存所有主题分区的副本分配情况, 
            
```

### Controller 与 Broker 通信
```shell
    1. 概念   
            (1) Controller 目前只会向 Broker 发送三类请求, 分别是 LeaderAndIsrRequest,  StopReplicaRequest 和 
                UpdateMetadataReques
            (2) LeaderAndIsrRequest: 告诉 Broker 相关主题各个分区的 Leader 副本位于哪台 Broker 上, ISR 中的副本都在哪些 
                                     Broker 上, 应该被赋予最高优先级
            (3) StopReplicaRequest: 对于接受该命令的 broker 停止对副本操作, 使用场景是分区副本迁移和删除主题.
            (4) UpdateMetadataRequest： 该请求会更新 Broker 上的元数据缓存, 集群上的所有元数据变更, 都发生在 Controller 端,
                                        再经由这个请求广播给集群上的所有 Broker
    2. LeaderAndIsrRequest, StopReplicaRequest, UpdateMetadataRequest 都继承 AbstractControlRequest 
            
            public abstract class AbstractControlRequest extends AbstractRequest {
             
                public static final long UNKNOWN_BROKER_EPOCH = -1L;
             
                public static abstract class Builder<T extends AbstractRequest> extends AbstractRequest.Builder<T> {
                    protected final int controllerId; // Controller 所在的 Broker ID。
                    protected final int controllerEpoch; // Controller 的版本信息, 用于隔离 Zombie Controller 
                    protected final long brokerEpoch; // 目标 Broker 的 Epoch, 用于隔离 Zombie Broker
             
                    protected Builder(ApiKeys api, short version, int controllerId, int controllerEpoch, long brokerEpoch) {
                        super(api, version);
                        this.controllerId = controllerId;
                        this.controllerEpoch = controllerEpoch;
                        this.brokerEpoch = brokerEpoch;
                    }
                }
            }
                
    3. RequestSendThread(怎么发)
            Controller 事件处理线程, 负责向这个队列写入待发送的请求, 而 RequestSendThread 的线程负责执行真正的请求发送.
       Controller 会为集群中的每个 Broker 都创建一个对应的 RequestSendThread 线程
       
            class RequestSendThread(
                                        val controllerId: Int, // Controller 所在 Broker 的 Id
                                        val controllerContext: ControllerContext, // Controller 元数据信息
                                        val queue: BlockingQueue[QueueItem], // 请求阻塞队列
                                        val networkClient: NetworkClient, // 用于执行发送的网络 I/O 类
                                        val brokerNode: Node, // 目标 Broker 节点
                                        val config: KafkaConfig, // Kafka 配置信息
                                        val time: Time,
                                        val requestRateAndQueueTimeMetrics: Timer,
                                        val stateChangeLogger: StateChangeLogger,
                                        name: String)
                                            extends ShutdownableThread(name = name) {
                                            }
                                            
            (1) doWork 业务逻辑是从阻塞队列中取出待发送的请求, 把它发送出去, 之后阻塞等待 Response 的返回, 当接收到 Response 之后,
                调用 callback 执行请求处理完成后的回调逻辑. 
                
    4. ControllerChannelManager
            (1) 作用
                    a. 管理 Controller 与集群 Broker 之间的连接, 并为每个 Broker 创建 RequestSendThread 线程实例
                    b. 将要发送的请求放入到指定 Broker 的阻塞队列中, 等待 RequestSendThread 线程进行处理
                    
            (2) ControllerChannelManager 类重要的数据结构 brokerStateInfo<broker_id, ControllerBrokerStateInfo>
                  ControllerBrokerStateInfo(
                                                networkClient: NetworkClient,
                                                /* 目标 Broker 节点对象, 里面封装目标 Broker 的连接信息, 比如主机名,端口号等
                                                brokerNode: Node, 
                                                /* 请求消息阻塞队列, Controller 为每个目标 Broker 都创建一个消息队列
                                                messageQueue: BlockingQueue[QueueItem],
                                                
                                                /* Controller 使用这个线程给目标 Broker 发送请求
                                                requestSendThread: RequestSendThread, 
                                                queueSizeGauge: Gauge[Int],
                                                requestRateAndTimeMetrics: Timer,
                                                reconfigurableChannelBuilder: Option[Reconfigurable]
                                            )
                                            
            (3) ControllerChannelManager 的方法
                    startup 方法： Controller 组件在启动时, 会调用 ControllerChannelManager 的 startup 方法. 该方法会从
                                  元数据信息中找到集群的 Broker 列表, 然后依次为它们调用 addBroker 方法, 把它们加到 
                                  brokerStateInfo 变量中, 最后再依次启动 brokerStateInfo 中的 RequestSendThread 线程.
                    shutdown 方法：关闭所有 RequestSendThread 线程, 并清空必要的资源.
                    sendRequest 方法：发送请求, 实际上就是把请求对象提交到请求队列.
                    addBroker 方法：添加目标 Broker 到 brokerStateInfo 数据结构中, 并创建必要的配套资源, 如请求队列,
                                   RequestSendThread 线程对象等.最后, RequestSendThread 启动线程.
                    removeBroker 方法：从 brokerStateInfo 移除目标 Broker 的相关数据
                    
                每当集群中扩容新的 Broker 时, Controller 就会调用 addBroker 方法为新 Broker 增加新的 RequestSendThread 线程
            
```

### Controller 单线程事件处理器
```shell
    1. 概念
            (1) Controller 单线程事件处理 Controller 元数据信息, 不用担心多线程访问导致的安全问题
            (2) Controller 端有多个线程向事件队列写入不同种类的事件, 有 ZooKeeper 端注册的 Watcher 线程, 
                KafkaRequestHandler 线程, Kafka 定时任务线程. ControllerEventThread 的线程专门负责处理队列中的事件
                
    2. Controller 单线程事件处理器组成
          
            (2) ControllerEvent: Controller 事件, 事件队列中被处理的对象
            (3) ControllerEventManager：事件处理器, 用于创建和管理 ControllerEventThread, 包含 ControllerEventProcessor,
                                        Controller 端的事件处理器接口
            (4) ControllerEventThread：专属的事件处理线程, 处理不同种类的 ControllEvent. 这个类是 ControllerEventManager 类
                                       内部定义的线程类
                                       
    3. ControllerEventProcessor
            定义了处理 Controller 事件 2 种方式, 第一种普通处理(最常用), 第二种抢占式
            
    4. ControllerEvent
            每个 ControllerEvent 都定义了一个状态 ControllerState, 多个 ControllerEvent 可能归属于相同的 ControllerState,
       例如, TopicChange 和 PartitionModifications 这 2 个 ControllerState 都属于 TopicChange(ControllerState), 都与 
       Topic 的变更有关
       
    5. ControllerEventManager
            构造了一个阻塞队列, 同时配以专属的事件处理线程, 实现了对各类 ControllerEvent 的处理
            (1) ControllerEventProcessor
                    事件处理器接口
                    
            (2) QueuedEvent
                    事件队列上的事件对象, 每个 QueuedEvent 对象实例都带一个 ControllerEvent. 定义 process(处理事件), 
                preempt(以抢占方式处理事件) 和 awaitProcessing(等待事件处理)
                
            (3) ControllerEventThread(消费事件)
                    阻塞等待从事件队列中取出 QueuedEvent, 如果是 ShutdownEventThread 事件, 则关闭这个线程.如果是其他的事件,就调用
                QueuedEvent 的 process 方法执行对应的处理逻辑, 同时计算事件被处理的速率
                
    6. 其他方法
            (1) put 是把指定 ControllerEvent 插入到事件队列, 而 clearAndPut 则是先执行具有高优先级的抢占式事件, 之后清空队列
                所有事件, 最后再插入指定的事
            
```

### Controller 选举
```shell
    1. 概念
            (1) 在一个 Kafka 集群中, 同一个时间内, 只有一个 broker 被选举为 Controller
            (2) /controller 节点内容:
                    brokerid : 0, 代表序号为 0 的 broker 是集群中的 Controller
                    ephemeralOwner 字段不为 0x0, 代表是临时节点(一旦该节点与 zookeeper 断开连接, 则该节点会被删除)
            (3) 集群上所有的 Broker 都在实时监听 ZooKeeper 上的 /controller 节点, 监听该节点是否存在, 监听到不存在, 尝试创建
                 /controller 节点, 创建成功则该 Broker 为新的 Controller. 同时监听该节点数据是否发送变更, 一旦发现该节点的内容发生
                 变化, Broker 也会立即启动新一轮的 Controller 选举
    2. 组成(KafkaController)
            (1) 选举触发器(ElectionTrigger) : 主题分区副本的选举, 即为哪些分区选择 Leader 副本
            (2) KafkaController Object： KafkaController 伴生对象, 定义一些常量和回调函数类型
            (3) ControllerEvent：定义 Controller 事件类型
            (4) ZooKeeper 监听器: 监听 ZooKeeper 中各个节点的变更
            (5) KafkaController Class：定义 KafkaController 类以及实际的处理逻辑
            
    3. KafkaController 类
            (1) 原生字段定义
        
                    class KafkaController(
                                            /* Kafka 配置信息, 获得到 Broker 端所有参数的值
                                            val config: KafkaConfig,
                                            
                                            /* ZooKeeper客户端, Controller 与 ZooKeeper 的所有交互均通过该属性完成
                                            zkClient: KafkaZkClient,  
                                            
                                            time: Time,  // 提供时间服务(如获取当前时间)的工具类
                                            metrics: Metrics,  // 实现指标监控服务(如创建监控指标)的工具类
                                            
                                            /* Broker 节点信息, 包括主机名, 端口号, 所用监听器等
                                            initialBrokerInfo: BrokerInfo,
                                            
                                            /* Broker Epoch值, 用于隔离老 Controller 发送的请求
                                            initialBrokerEpoch: Long,
                                            
                                            /* 实现 Delegation token 管理的工具类, Delegation token是一种轻量级的认证
                                            tokenManager: DelegationTokenManager,
                                            
                                            /* Controller 端事件处理线程名字前缀
                                            threadNamePrefix: Option[String] = None
                                          )
                                        extends ControllerEventProcessor with Logging with KafkaMetricsGroup {
                                        ......
                                        }
                                        
            (2) 辅助字段定义
                     // 集群元数据类, 保存集群所有元数据
                      val controllerContext = new ControllerContext
                      
                      // Controller 端通道管理器类, 负责 Controller 向 Broker 发送请求
                      var controllerChannelManager = new ControllerChannelManager(controllerContext, config, time, metrics,
                                                                                   stateChangeLogger, threadNamePrefix)
            
                      // 线程调度器, 当前唯一负责定期执行分区重平衡 Leader 选举
                      private[controller] val kafkaScheduler = new KafkaScheduler(1)
                     
                      // Controller 事件管理器, 负责管理事件处理线程
                      private[controller] val eventManager = new ControllerEventManager(config.brokerId, this, time,
                                                                            controllerContext.stats.rateAndTimeMetrics)
                                                                            
                      // 副本状态机, 负责副本状态转换
                      val replicaStateMachine: ReplicaStateMachine = new ZkReplicaStateMachine(config, stateChangeLogger, controllerContext, zkClient,
                        new ControllerBrokerRequestBatch(config, controllerChannelManager, eventManager, controllerContext, stateChangeLogger))
                        
                      // 分区状态机, 负责分区状态转换
                      val partitionStateMachine: PartitionStateMachine = new ZkPartitionStateMachine(config, stateChangeLogger, controllerContext, zkClient,
                        new ControllerBrokerRequestBatch(config, controllerChannelManager, eventManager, controllerContext, stateChangeLogger))
                        
                      // 主题删除管理器, 负责删除主题及日志
                      val topicDeletionManager = new TopicDeletionManager(config, controllerContext, replicaStateMachine,
                        partitionStateMachine, new ControllerDeletionClient(this, zkClient))
                        
            (3) 各类 ZooKeeper 监听器字段
            
                       // Controller 节点 ZooKeeper 监听器, 监听 /controller 节点变更, 包括节点创建, 删除, 数据变更.
                      private val controllerChangeHandler = new ControllerChangeHandler(eventManager)
                      
                      // Broker 数量 ZooKeeper 监听器, 监听 Broker 的数量变化(Zookeeper 下 /brokers/ids 的数量)
                      private val brokerChangeHandler = new BrokerChangeHandler(eventManager)
                      
                      // Broker 信息变更 ZooKeeper 监听器集合, 监听 Broker 的数据变更, 如 Broker 的配置信息发生的变化
                      private val brokerModificationsHandlers: mutable.Map[Int, BrokerModificationsHandler] = mutable.Map.empty
                      
                      // 主题数量 ZooKeeper 监听器, 监控主题数量变更,  /brokers/topics 节点是否新增主题信息.
                      private val topicChangeHandler = new TopicChangeHandler(eventManager)
                      
                      // 主题删除 ZooKeeper 监听器, 监听主题删除节点 /admin/delete_topics 的子节点数量变更
                      private val topicDeletionHandler = new TopicDeletionHandler(eventManager)
                      
                      // 主题分区变更 ZooKeeper 监听器, 监控主题分区数据变更,如新增加副本, 分区更换 Leader 副本
                      private val partitionModificationsHandlers: mutable.Map[String, PartitionModificationsHandler] = mutable.Map.empty
                      
                      // 分区副本重分配任务 ZooKeeper 监听器, 一旦发现新提交的任务, 就为目标分区执行副本重分配
                      private val partitionReassignmentHandler = new PartitionReassignmentHandler(eventManager)
                      
                      // Preferred Leader 选举 ZooKeeper 监听器, 一旦发现新提交的任务, 就为目标主题执行 Preferred Leader 选举
                      private val preferredReplicaElectionHandler = new PreferredReplicaElectionHandler(eventManager)
                      
                      /* ISR 副本集合变更 ZooKeeper 监听器, 一旦被触发, 就需要获取 ISR 发生变更的分区列表, 
                       * 然后更新 Controller 端对应的 Leader 和 ISR 缓存元数据.
                      private val isrChangeNotificationHandler = new IsrChangeNotificationHandler(eventManager)
                      
                      /* 日志路径变更 ZooKeeper 监听器, 一旦被触发, 需要获取受影响的 Broker 列表, 然后处理这些 Broker 上
                       * 失效的日志路径。
                      private val logDirEventNotificationHandler = new LogDirEventNotificationHandler(eventManager)
                      
                      
                        
                      
            (4) 统计字段定义
                    / 当前 Controller 所在 Broker Id
                      @volatile private var activeControllerId = -1
                      
                      // 离线分区总数
                      @volatile private var offlinePartitionCount = 0
                      
                      // 满足 Preferred Leader 选举条件的总分区数
                      @volatile private var preferredReplicaImbalanceCount = 0
                      
                      // 总主题数
                      @volatile private var globalTopicCount = 0
                      
                      // 总主题分区数
                      @volatile private var globalPartitionCount = 0
                      
                      // 待删除主题数
                      @volatile private var topicsToDeleteCount = 0
                      
                      //待删除副本数
                      @volatile private var replicasToDeleteCount = 0
                      
                      // 暂时无法删除的主题数
                      @volatile private var ineligibleTopicsToDeleteCount = 0
                      
                      // 暂时无法删除的副本数
                      @volatile private var ineligibleReplicasToDeleteCount = 0
                      
                      
    4. Controller 选举流程
            (1) 触发流程
                    场景一: 集群首次启动
                                当 Broker 重启时会调用 startup 方法, 会注册 ZooKeeper 状态变更监听器, 用于监听 Broker 与 
                           ZooKeeper 之间的会话是否过期, 写入 Startup 事件到事件队列, 在启动 ControllerEventThread 线程, 
                           开始处理事件队列中的 Startup 事件, 之后 KafkaController.process() 再进行 Startup 事件的处理, 
                           其中包括 Controller 的选举.
                           
                    场景二: /controller 节点消失
                                所有检测到 /controller 节点消失的 Broker(通过 zookeeper 监听方式), 都会立即调用 elect 方法
                           执行竞选逻辑.
                           
                    场景三: /controller 节点数据变更
                                这代表 controller 要易主了, 如果 Broker 之前是 Controller, 那么该 Broker 需要首先执行
                           卸任操作(清空各种数据结构的值, 取消 ZooKeeper 监听器, 关闭各种状态机以及管理器), 然后再尝试竞选； 
                           如果 Broker 之前不是 Controller, 直接去竞选新 Controller
                           
            (2) 选举 Controller
                    第一步: 查看是否 Controller 被选举出来, 通过获取 ControllerId, 如果不为 -1, 则 Controller 已经被选举出来,
                           直接返回.
                           
                    第二步: 创建 /controller 节点.
                    第三步: 如果创建成功, 执行选举成功后的动作,  注册各类 ZooKeeper 监听器, 删除日志路径变更和 ISR 副本变更通知事件,
                           启动 Controller 通道管理器, 以及启动副本状态机和分区状态机
```

### Controller 功能
```shell
    1. 概念
    2. 集群成员管理
            (1) 成员数量管理
                    主要是新增成员和移除现有成员.
                    原理: 每个 broker 启动时, 会在 Zookeeper 的 /brokers/ids 节点下创建对应的 broker.id 临时节点, 该节点的内容
                         包括 Broker 配置的主机名, 端口号以及所用 listen 的信息. Broker 正常关闭或意外退出时, ZooKeeper 上对应的
                         临时节点会自动消失. 
                         
                         当监听到 /brokers/ids 下的子节点数目发生变化, 调用 BrokerChangeHandler 处理方法. 向 Controller 事件队
                         列写入一个 BrokerChange 事件. Controller 端定义的所有 Handler 的处理逻辑, 都是向事件队列写入相应的
                         ControllerEvent, 真正的事件处理逻辑位于 KafkaController 类的 process 方法中.
                         
                    流程:
                            第一步: 前提条件是该 Broker 是 Controller, 从 ZooKeeper 中获取集群 Broker 列表(最新的运行中 Broker 列表)
                            第二步: 获取 Controller 当前保存的 Broker 列表
                            第三步: 比较两个列表, 获取新增 Broker 列表, 待移除 Broker 列表, 已重启 Broker 列表和当前运行中的
                                   Broker 列表. 已重启 Broker 是根据 Epoch 来判断版本号.
                            第四步: 为每个新增 Broker 创建与之连接的通道管理器和底层的请求发送线程(RequestSendThread)
                            第五步: 为每个已重启的 Broker 移除它们现有的配套资源(通道管理器, RequestSendThread等), 并重新添加它们
                            第六步: 为每个待移除 Broker 移除对应的配套资源
                            第七步: 为新增 Broker 执行更新 Controller 元数据和 Broker 启动逻辑, 
                                        Broker 启动逻辑有移除元数据中新增 Broker 对应的副本集合, 给集群现有 Broker 发送元数据
                                        更新请求, 令它们感知到新增 Broker 的到来, 给新增 Broker 发送元数据更新请求, 
                                        令它们同步集群当前的所有分区数据, 将新增 Broker 上的所有副本设置为 Online 状态,即可用状态.
                                        重启之前暂停的副本迁移操作, 重启之前暂停的主题删除操作, 为新增 Broker 注册 
                                        BrokerModificationsHandler 监听器(允许 Controller 监控它们在 ZooKeeper 上的节点的数据变更)
                            第八步: 为已重启 Broker 执行重添加逻辑, 包含更新 ControllerContext, 执行 Broker 重启动逻辑
                            第九步: 为待移除 Broker 执行移除 ControllerContext 和 Broker 终止逻辑, 即删除 Controller 元数据
                                   缓存中与之相关的所有项, 处理这些 Broker 上保存的副本, 注销之前为该 Broker 注册的 
                                   BrokerModificationsHandler 监听器
                                   
            (2) 成员信息管理
                    原理: 通过 ZooKeeper 监听器的方式, 来监听器是 BrokerModificationsHandler, 一旦 Broker 的信息发生变更, 该监
                          听器的 handleDataChange 方法就会被调用, 向事件队列写入 BrokerModifications 事件. 处理这个  
                          BrokerModifications 事件逻辑是获取 ZooKeeper 上的 Broker 数据, 将其与元数据缓存上的数据进行比对.
                          如果发现两者不一致, 就会更新元数据缓存, 同时调用 onBrokerUpdate 方法执行更新逻辑, 即向集群所有 Broker 
                          发送更新元数据信息请求, 把变更信息广播出去
                          
    3. 主题管理
            (1) 主题创建/变更
                    原理: 监听 ZooKeeper 监听器(TopicChangeHandler), 对应于 Zookeeper 下的 /brokers/topic 节点, 一旦该节点下
                         新增主题信息, 该监听器的 handleChildChange 就会被触发, Controller 通过 ControllerEventManager 对象,
                         向事件队列写入 TopicChange 事件, KafkaController 的 process 方法接到该事件后, 调用 processTopicChange
                         方法执行主题创建.从 ZooKeeper 中获取所有主题, 与元数据缓存比对, 找出新增主题列表与已删除主题列表.使用
                         ZooKeeper 中的主题列表更新元数据缓存, 为新增主题注册分区变更监听器, 从 ZooKeeper 中获取新增主题的副本
                         分配情况, 清除元数据缓存中属于已删除主题的缓存项, 为新增主题更新元数据缓存中的副本分配条目, 调整新增主题
                         所有分区以及所属所有副本的运行状态为“上线”状态
                         
            (2) 删除主题
                    原理: 监听 ZooKeeper 监听器(TopicDeletionHandler), 当要删除指定的主题时, 是在 /admin/delete_topics 节点下
                          创建名为待删除主题名的子节点, 一旦监听到该节点被创建, TopicDeletionHandler 的 handleChildChange 方法
                          就会被触发, Controller 会向事件队列写入 TopicDeletion 事件.
                          
                    流程: 
                          第一步: 从 ZooKeeper 的 /admin/delete_topics 下获取子节点列表(待删除主题列表)
                          第二步: 比对 controller 中元数据缓存中的主题列表, 得到待删除主题, 删除 /admin/delete_topics 下对应的
                                 子节点
                          第三步: 检查 Broker 端参数 delete.topic.enable 的值, 为 false, 即不允许删除主题, 只会清除 ZooKeeper 
                                 下的对应子节点, 不会做其他操作. 如果为 true, 遍历待删除主题列表, 将那些正在执行分区迁移的主题暂时
                                 设置成“不可删除”状态.
                          第四步: 可以删除的主题交由 TopicDeletionManager, 由它执行真正的删除逻辑.
                    
```

## 状态机模块
```shell
    1. 有些状态机或则管理器具备相对独立的功能框架, 不依赖于使用方, TopicDeletionManager(主题删除管理器), 
       ReplicaStateMachine(副本状态机)和 PartitionStateMachine(分区状态机).
            TopicDeletionManager：负责对指定 Kafka 主题执行删除操作, 清除待删除主题在集群上的各类“痕迹”。
            ReplicaStateMachine：负责定义 Kafka 副本状态, 合法的状态转换, 以及管理状态之间的转换.
            PartitionStateMachine：负责定义 Kafka 分区状态, 合法的状态转换, 以及管理状态之间的转换
```

### 主题删除管理器
```shell
    1. 概念
    2. 组成:
            class TopicDeletionManager(
                                            /* KafkaConfig 类, 保存 Broker 端参数, 可以获取 Broker 端参数 delete.topic.enable 
                                            config: KafkaConfig, 
                                            controllerContext: ControllerContext, // 集群元数据
                                            replicaStateMachine: ReplicaStateMachine, // 副本状态机, 用于设置副本状态
                                            partitionStateMachine: PartitionStateMachine, // 分区状态机, 用于设置分区状态
                                            
                                            /* 负责实现删除主题以及后续的动作, 比如更新元数据等. ControllerDeletionClient 
                                             * 实现 DeletionClient 接口的类, 实现 deleteTopic, deleteTopicDeletions, 
                                             * mutePartitionModifications 和 sendMetadataUpdate 接口
                                            client: DeletionClient 
                                       ) extends Logging {
                                            this.logIdent = s"[Topic Deletion Manager ${config.brokerId}] "
                                            // 是否允许删除主题, 初始化时被 delete.topic.enable 赋值.
                                            val isDeleteTopicEnabled: Boolean = config.deleteTopicEnable
                                       }
                                       
    3. DeletionClient 接口及其实现
            DeletionClient 这个接口只有一个实现类, 即 ControllerDeletionClient.
            
            deleteTopic 方法: 删除指定主题, 即删除 ZooKeeper 下对应的 /brokers/topics/节点, /config/topics/节点和 
                             /admin/delete_topics/节点
                             
            deleteTopicDeletions 方法:  删除 /admin/delete_topics 下的给定 topic 子节点
            mutePartitionModifications 方法:  屏蔽主题分区数据变更监听器, 实现就是取消 /brokers/topics/<topic> 节点数据变更的监听,
                                             这样当该主题的分区数据发生变更后, 不会触发 Controller 相应的处理逻辑.
                                             目的是避免对相同主题操作之间的相互干扰. 如用户 A 发起主题删除, 而同时用户 B 为这个主题
                                             新增分区. 这两个操作就会相互冲突, 造成逻辑上的混乱以及状态的不一致. 需要在移除主题副本
                                             和分区对象前, 先执行这个方法, 以确保不再响应用户对该主题的其他操作.
                                             
            sendMetadataUpdate 方法:  给集群所有 Broker 发送更新请求, 告诉不要再为已删除主题的分区提供服务. 会给集群中的所有 
                                     Broker 发送更新元数据请求, 告知它们要同步给定分区的状态
                                     
    5. TopicDeletionManager(主题删除管理器)
            (1) resumeDeletions 方法 
                    重启主题删除操作, 主题因为某些事件一时无法完成删除, 比如主题分区正在进行副本重分配等.一旦这些事件完成后, 主题
                重新具备可删除的资格. 这时需要调用 resumeDeletions 重启删除操作, 具体的内容为从元数据缓存中获取要删除的主题列表,
                遍历每个要删除的主题, 查看所有副本的状态. 如果副本状态都是 ReplicaDeletionSuccessful, 就说明该主题已经被成功删除.
                再调用 completeDeleteTopic 方法, 完成后续的操作(清除元数据以及 zk 状态). 对于那些删除操作未开始, 并且暂时无法执行
                删除的主题, 将这类主题加到待重试主题列表中, 用于后续重试. 如果主题是能够被删除的, 就将其加入到待删除列表中. 调用 
                retryDeletionForIneligibleReplicas 方法, 来重试待重试主题列表中的主题删除操作. 对于待删除主题列表中的主题则
                调用 onTopicDeletion 删除. 
                
            (2) completeDeleteTopic 方法
                    第一步：注销分区变更监听器, 防止删除过程中因分区数据变更导致监听器被触发, 引起状态不一致
                    第二步: 获取该主题下处于 ReplicaDeletionSuccessful 状态的所有副本对象, 即所有已经被成功删除的副本对象.
                    第三步: 利用副本状态机将这些副本对象转换成 NonExistentReplica 状态, 等同于在状态机中删除这些副本
                    第四步: 更新元数据缓存中的待删除主题列表和已开始删除的主题列表
                    第五步: 移除 ZooKeeper 上关于该主题的一切记录
                    第六步: 移除元数据缓存中关于该主题的一切记录
                        
            (3) onTopicDeletion 方法
                    找出给定待删除主题列表中那些尚未开启删除操作的所有主题, 获取到这些主题的所有分区对象, 将这些分区的状态依次
                    调整成 OfflinePartition 和 NonExistentPartition(等同于将这些分区从分区状态机中删除), 把这些主题加到
                    “已开启删除操作” 主题列表中, 给集群所有 Broker 发送元数据更新请求, 告知不要再为这些主题处理数据.分区删除操作会
                    执行底层的物理磁盘文件删除动作(通过副本状态机状态转换操作完成)
                                         
```

### 副本状态机
```shell
    1. 概念
            (1) 了解副本状态机有助于排查副本间的状态不一致的问题.副本状态机定义 Kafka 副本的状态集合, 控制这些状态之间的流转规则.
            (2) 每个 Kafka Controller 都定义自己的副本状态机, 但是只有在当前 Controller 实例成为 leader 时才会启动运行名下的状态机.
            (3) 在副本状态转换操作的逻辑中, 通过 Controller 给 Broker 发送请求来实现 Broker 上的副本信息更新
            (4) KafkaController 对象在构建时, 就会初始化一个 ZkReplicaStateMachine 实例, 每个 Broker 都会创建 KafkaController 
               实例, 但并不会启动副本状态机, 只有当 Broker 被选举为 Controller 时, 才会启动创建好的副本状态机和分区状态机
    2. 组成
            (1) ReplicaStateMachine: 副本状态机抽象类, 定义一些常用方法(如 startup, shutdown 等), 以及状态机最重要的处理逻辑方法 
                                     handleStateChanges
            (2) ZkReplicaStateMachine: 副本状态机具体实现类, 重写 handleStateChanges 方法, 实现副本状态之间的状态转换. 目前, 
                                       ZkReplicaStateMachine 是唯一的 ReplicaStateMachine 子类.
            (3) ReplicaState: 副本状态集合, Kafka 目前共定义 7 种副本状态.
            
    3. 副本状态及状态管理
            (1) 
                NewReplica: 副本被创建之后所处的状态(可能的前置状态, NonExistentReplica)
                OnlineReplica: 副本正常提供服务时所处的状态(可能的前置状态, NewReplica, OfflineReplica, ReplicaDeletionIneligible)
                OfflineReplica: 副本服务下线时所处的状态(可能的前置状态, OnlineReplica, NewReplica, ReplicaDeletionIneligible)
                ReplicaDeletionStarted: 副本被删除时所处的状态(可能的前置状态, OfflineReplica)
                ReplicaDeletionSuccessful: 副本被成功删除后所处的状态(可能的前置状态, ReplicaDeletionStarted-删除成功)
                ReplicaDeletionIneligible: 开启副本删除, 但副本暂时无法被删除时所处的状态
                　　　　　　　　　　　　　　　　(可能的前置状态, ReplicaDeletionStarted-删除不成功, OfflineReplica-重试副本删除)
                NonExistentReplica: 副本从副本状态机被移除前所处的状态
                                    (可能的前置状态, ReplicaDeletionSuccessful-副本对象被移出副本状态机)
                
            (2) 当副本对象首次被创建, 会被置于 NewReplica 状态. 经过初始化后, 当副本对象能提供服务, 状态机会将其调整为 OnlineReplica,
                并一直以这个状态持续工作.
                
                如果副本所在的 Broker 关闭或者是因为其他原因不能正常工作, 副本需要从 OnlineReplica 变更为 OfflineReplica, 
                表明副本已处于离线状态
                
                一旦开启删除主题这样的操作, 状态机将副本状态跳转到 ReplicaDeletionStarted, 以表明副本删除已经开启. 如果删除成功, 
                则置为 ReplicaDeletionSuccessful, 如果不满足删除条件(如所在 Broker 处于下线状态, 或则进行分区再分配), 那
                就设置成 ReplicaDeletionIneligible, 以便后面重试
                
                当副本对象被删除后, 其状态会变更为 NonExistentReplica, 副本状态机将移除该副本数据
                
    4. ZkReplicaStateMachine(具体实现类)
            (1) 辅助方法
                    logFailedStateChange: 仅仅是记录一条错误日志, 说明执行一次无效的状态变更.
                    logInvalidTransition： 记录错误日志, 即记录一次非法的状态转换.
                    logSuccessfulTransition：记录一次成功的状态转换操作
                    getTopicPartitionStatesFromZk：从 ZooKeeper 中获取指定分区的状态信息, 包括每个分区的 Leader 副本, 
                                                   ISR 集合等数据
                    doRemoveReplicasFromIsr：把给定的副本对象从给定分区 ISR 中移除
                    removeReplicasFromIsr：调用 doRemoveReplicasFromIsr 方法, 实现将给定的副本对象从给定分区 ISR 中移除的功能.
                    doHandleStateChanges：执行状态变更和转换操作的主力方法.
                    
            (2) handleStateChanges 方法
                    第一步: 调用 doHandleStateChanges 方法执行真正的副本状态转换, 在调用之前, 将所有副本对象按照 Broker 进行分组, 
                           依次执行状态转换操作. 例如副本对象的格式为 < 主题名, 分区号, 副本 Broker ID>,  replicas 为集合
                           （<test, 0, 0>, <test, 0, 1>, <test, 1, 0>, <test, 1, 1>）, 需要整理为 Broker_id 分组.
                           Map(0 -> Set(<test, 0, 0>, <test, 1, 0>)，1 -> Set(<test, 0, 1>, <test, 1, 1>))
                           
                    第二步: 给集群中的相应 Broker 批量发送请求.
                    
            (3) doHandleStateChanges 方法
                    第一步: 获取给定副本对象在 Controller 端元数据缓存中的当前状态, 如果没有保存某个副本对象的状态, 将其初始化为
                           NonExistentReplica 状态
                    第二步: 根据不同 ReplicaState 中定义的合法前置状态集合以及传入的目标状态（targetState）, 将给定的副本对象集合
                           划分成两部分：能合法转换的副本对象集合, 和执行非法状态转换的副本对象集合. 为后者中的每个副本对象记录
                           一条错误日志
                    第三步: 对合法转换的副本对象集合进行状态转换, 例如副本被创建时被转换到 NewReplica状态, 
                           副本正常工作时被转换到 OnlineReplica 状态, 副本停止服务后被转换到 OfflineReplica 状态等等.
                           
                           
            (4) 状态转换处理逻辑
                    场景一: 转换到 NewReplica 状态
                                尝试从元数据缓存中获取这些副本对象的分区信息数据, 包括分区的 Leader 副本在哪个 Broker 上, 
                           ISR 中都有哪些副本等等, 如果找不到对应的分区数据, 就直接把副本状态更新为 NewReplica.否则就需要给
                           该副本所在的 Broker 发送请求, 让它知道该分区的信息. 同时还要给集群所有运行中的 Broker 发送请求, 让它们
                           感知到新副本的加入
                           
                    场景二: 转换到 OnlineReplica 状态
                                第一步: 获取元数据中该副本所属的分区对象, 以及该副本的当前状态
                                第二步: 查看当前状态是否是 NewReplica. 如果是, 则获取分区的副本列表, 并判断该副本是否在当前的副本
                                       列表中, 假如不在, 就记录错误日志, 并更新元数据中的副本列表. 如果状态不是 NewReplica, 说明
                                       是一个已存在的副本对象, 那么获取对应分区的详细数据, 然后向该副本对象所在的 Broker 发送
                                        LeaderAndIsrRequest 请求, 令其同步获知并保存该分区数据.
                                第三步: 将该副本对象状态变更为 OnlineReplica. 那么该副本就处于正常工作状态
                                
                    场景三: 转换到 OfflineReplica 状态
                                第一步: 给所有符合状态转换的副本所在的 Broker, 发送 StopReplicaRequest 请求, 告诉这些 Broker 
                                       停掉其上的对应副本(Kafka 的副本管理器组件 ReplicaManager负责处理这个逻辑).
                                第二步: 根据分区是否保存 Leader 信息, 将副本集合划分成两个子集: 有 Leader 副本集合和无 Leader 
                                       副本集合. 有无 Leader 信息并不仅仅包含 Leader, 还有 ISR 和 controllerEpoch 等数据
                                第三步: 遍历有 Leader 的子集合, 向这些副本所在的 Broker 发送 LeaderAndIsrRequest 请求, 去更新
                                       停止副本操作之后的分区信息, 再把这些分区状态设置为 OfflineReplica
                                第四步: 遍历无 Leader 的子集合,  向这些副本所在的 Broker 发送 UpdateMetadataRequest 请求, 告知
                                       它们更新对应分区的元数据, 然后再把副本状态设置为 OfflineReplica.
```

### 分区状态机
```shell
    1. 概念
            (1) 每个 Broker 启动时, 都会创建对应的分区状态机和副本状态机实例, 但只有 Controller 所在的 Broker 才会启动它们.如果 
                Controller 变更到其他 Broker, 老 Controller 所在的 Broker 会调用这些状态机的 shutdown 方法关闭它们, 新
                Controller 所在的 Broker 会调用状态机的 startup 方法启动它们.
            (2) 每个分区都必须选举出 Leader 才能正常提供服务, 只能为 OnlinePartition 和 OfflinePartition 状态的分区选举 Leader
    2. 组成
            (1) PartitionStateMachine: 分区状态机抽象类. 定义 startup, shutdown 公共方法, 同时给出处理分区状态转换入口方法
                                       handleStateChanges 的签名
            (2) ZkPartitionStateMachine: PartitionStateMachine 唯一的继承子类, 重写了父类的 handleStateChanges 方法, 并配以
                                         私有的 doHandleStateChanges 方法, 共同实现分区状态转换的操作.
            (3) PartitionState 接口及其实现对象: 定义 4 类分区状态, 分别是 NewPartition, OnlinePartition, OfflinePartition 和
                                               NonExistentPartition. 以及它们之间的流转关系
            (4) PartitionLeaderElectionStrategy 接口及其实现对象: 定义 4 类分区 Leader 选举策略. 
            (5) PartitionLeaderElectionAlgorithms: 分区 Leader 选举的算法实现, 4 类选举策略的实现代码.
    3. 分区状态及状态管理
            (1) 
                NewPartition: 分区被创建后被设置成这个状态, 表明它是一个全新的分区对象, 处于这个状态的分区被认为是“未初始化”, 不能
                              选举为 Leader.(合法前置状态, NonExistentPartition)
                OnlinePartition: 分区正式提供服务时所处的状态.
                                        合法前置状态有：　NewPartition - 通过 Broker 启动或则新分区初始化
                                                       OfflinePartition - 分区选举 leader
                                                       
                OfflinePartition: 分区下线后所处的状态
                                        合法前置状态有： NewPartition
                                                       OnlinePartition - broker 下线或则主题被删除
                                                       
                NonExistentPartition：分区被删除, 并且从分区状态机移除后所处的状态
                                        合法前置状态有：　OfflinePartition　- 主题被成功删除
                                        
    4. 分区 Leader 选举的场景及方法
            (1) 触发 leader 选举条件
                OfflinePartitionLeaderElectionStrategy：因为 Leader 副本下线而引发的分区 Leader 选举
                ReassignPartitionLeaderElectionStrategy：因为执行分区副本重分配操作而引发的分区 Leader 选举
                PreferredReplicaPartitionLeaderElectionStrategy：因为执行 Preferred 副本 Leader 选举而引发的分区 Leader 选举
                ControlledShutdownPartitionLeaderElectionStrategy：因为正常关闭 Broker 而引发的分区 Leader 选举
                
            (2) OfflinePartitionLeaderElectionStrategy 选举
                    def offlinePartitionLeaderElection(
                                                        assignment: Seq[Int], // 这是分区的副本列表
                                                        isr: Seq[Int],
                                                        liveReplicas: Set[Int], // 该分区下所有处于存活状态的副本
                                                        uncleanLeaderElectionEnabled: Boolean, // 脏选举
                                                        controllerContext: ControllerContext
                                                        )
                    参数:
                            assignments: 分区的副本列表(AR, Assigned Replicas)
                            isr: 保存分区所有与 Leader 副本保持同步的副本列表,  Leader 副本也在 ISR 中
                            liveReplicas: 保存该分区下所有处于存活状态的副本
                            uncleanLeaderElectionEnabled: 默认是 false, Unclean Leader 选举指在 ISR 列表为空的情况下, Kafka 
                                                          选择一个非 ISR 副本作为新的 Leader,存在丢失数据的风险
                                                          
                    原理: 顺序搜索 AR 列表, 把同时满足副本是存活状态并且, 副本在 ISR 列表中的副本作为新的 leader.如果找到这样的副本,
                         会检查是否开启 Unclean Leader 选举, 如果开启了, 则降低标准, 只要满足上面第一个条件即可. 如果未开启, 
                          则本次 Leader 选举失败, 没有新 Leader 被选出.
                          
    5. 处理分区状态转换的方法
            (1) handleStateChanges 方法
                    handleStateChanges 把 partitions 的状态设置为 targetState, 还可能需要用 leaderElectionStrategy 策略为 
                partitions 选举新的 Leader, 最终将 partitions 的 Leader 信息返回.
                
                    流程：
                            第一步: 调用 doHandleStateChanges 方法执行分区状态转换；
                            第二步: Controller 给相关 Broker 发送请求, 告知它们这些分区的状态变更.
                        
            (2) doHandleStateChanges 方法
                    第一步: 做状态初始化的工作, 不在元数据缓存中的所有分区的状态, 会被初始化为 NonExistentPartition.
                    第二步: 检查哪些分区执行的状态转换不合法, 并为这些分区记录相应的错误日志.
                    第三步: 根据不同目标状态, 进行不同业务逻辑.
                    
            (3) OnlinePartition 目标分区状态的转化
                    第一步: 为 NewPartition 状态的分区做初始化操作,即在 ZooKeeper 中, 创建并写入分区节点数据. 节点的位置
                           是 /brokers/topics/<topic>/partitions/<partition>, 每个节点数据都要包含分区的 Leader 和 ISR 等数据.
                           而 Leader 和 ISR 的确定规则是：选择存活副本列表的第一个副本作为 Leader；选择存活副本列表作为 ISR.
                    第二步：为具备 Leader 选举资格的分区推选 Leader.
                    
            (4) 推选 Leader 方法
                    第一步: 从 ZooKeeper 中获取给定分区的 Leader, ISR 信息, 并将结果封装进名为 validLeaderAndIsrs 的容器中. 具体
                    　　　　实现为批量读取 ZooKeeper 中给定分区的所有 Znode 数据, 分别保存可选举 Leader 分区列表和选举失败分区列表,
                           遍历每个分区的 Znode 节点数据, 如果成功拿到 Znode 节点数据, 节点数据包含 Leader 和 ISR 信息且节
                           点数据的 Controller Epoch 值小于当前 Controller Epoch 值, 则将该分区加入到可选举 Leader 分区列表.
                           如果发现 Zookeeper 中保存的 Controller Epoch 值大于当 Epoch 值, 说明该分区已经被一个更新的 
                           Controller 选举过 Leader, 需要终止本次 Leader 选举, 并将该分区放置到选举失败分区列表
                            
                    第二步: 根据不同的分区情况进行 leader 选举(4 种策略), 选择 Leader 的规则, 就是选择副本集合中首个存活且处于 ISR
                           中的副本作为 Leader, 同时区分出成功选举 Leader 和未选出 Leader 的分区
                           
                    第三步: 更新 ZooKeeper 节点数据, 以及 Controller 端元数据缓存信息. 即将上一步中所有选举失败的分区, 全部加入到
                            Leader 选举失败分区列表. 使用新选举的 Leader 和 ISR 信息, 更新 ZooKeeper 上分区的 Znode 节点数据.
                            对于 ZooKeeper Znode 节点数据更新成功的那些分区, 封装对应的 Leader 和 ISR 信息, 构建 LeaderAndIsr
                            请求, 并将该请求加入到 Controller 待发送请求集合, 等待后续统一发送, 最后方法返回选举结果,包括成功选举
                            并更新 ZooKeeper 节点的分区列表, 选举失败分区列表, 以及 ZooKeeper 节点更新失败的分区列表.
                      
```

## 延迟操作模块
```shell
    1. 概念
            (1) 延时请求(Delayed Operation), 即延迟请求, 指因未满足条件而暂时无法被处理的 Kafka 请求. 如配置 acks=all 的生产者
                发送的请求可能一时无法完成, 因为 Kafka 必须确保 ISR 中的所有副本都要成功响应这次写入.
```

### 延时处理机制(分层时间轮算法)
```shell
    1. 概念
            (1) 分层时间轮的桶 Bucket, 实际上是一个双向列表.
            (2) 在 Kafka 源码中, 时间轮对应 utils.timer 包下的 TimingWheel 类, 每个 Bucket 下的链表对应 TimerTaskList 类,
                链表元素对应 TimerTaskEntry 类, 而每个链表元素里面保存的延时任务对应 TimerTask. 
                TimerTaskEntry 与 TimerTask 是 1 对 1 的关系, TimerTaskList 下包含多个 TimerTaskEntry, TimingWheel 包含
                多个 TimerTaskList
            (3) 能立即完成的请求马上完成, 否则就放入到名为 Purgatory 的缓冲区中,后面 DelayedOperationPurgatory 类的方法会自动地
                处理这些延迟请求
                
    2. TimerTask 类
            (1) 定义
                    
                trait TimerTask extends Runnable {
                  /* 表示这个定时任务的超时时间, 通常是 request.timeout.ms 参数值, 每个 TimerTask 实例关联一个TimerTaskEntry
                   * 就是说每个定时任务需要知道它在哪个 Bucket 链表下的哪个链表元素上
                  val delayMs: Long // timestamp in millisecond
                  
                  /* timerTaskEntry 用于封装当前延时任务并记录到时间格中, 属于延时任务与时间格之间建立关系的桥梁
                  private[this] var timerTaskEntry: TimerTaskEntry = null
                  
                  /* 取消定时任务, 原理就是将关联的 timerTaskEntry 置空
                  def cancel(): Unit = {
                    synchronized {
                      if (timerTaskEntry != null) timerTaskEntry.remove()
                      timerTaskEntry = null
                    }
                  }
                  
                  /* 关联 timerTaskEntry, 原理是给 timerTaskEntry 字段赋值
                  private[timer] def setTimerTaskEntry(entry: TimerTaskEntry): Unit = {
                    synchronized {
                      // if this timerTask is already held by an existing timer task entry,
                      // we will remove such an entry first.
                      if (timerTaskEntry != null && timerTaskEntry != entry)
                        timerTaskEntry.remove()
                 
                      timerTaskEntry = entry
                    }
                  }
                  
                  // 获取关联的 timerTaskEntry 实例
                  private[timer] def getTimerTaskEntry(): TimerTaskEntry = {
                    timerTaskEntry
                  }
                }
                
    3. TimerTaskEntry
            (1) 承载定时任务, 以及如何在链表中实现双向关联, 本质上是一个链表结点, 其字段定义了结点的前置和后置指针, 以及所属的时间格
            (2) 定义
                    private[timer] class TimerTaskEntry(val timerTask: TimerTask,
                                                        /* 定义定时任务的过期时间, 假设有个 PRODUCE 请求在当前时间 1 点钟被发送
                                                         * 到 Broker, 超时时间是 30 秒, 那么该请求必须在 1 点 30 秒之前完成,
                                                         * 否则将被视为超时.
                                                        val expirationMs: Long
                             
                                                       ) extends Ordered[TimerTaskEntry] {
                     
                      @volatile
                      var list: TimerTaskList = null   // 绑定的 Bucket 链表实例
                      var next: TimerTaskEntry = null  // next 指针
                      var prev: TimerTaskEntry = null  // prev 指针
                     
                      /* if this timerTask is already held by an existing timer task entry, setTimerTaskEntry will remove it.
                       * 关联给定的定时任务
                      if (timerTask != null) timerTask.setTimerTaskEntry(this)
                     
                      /* 关联定时任务是否已经被取消了
                      def cancelled: Boolean = {
                        timerTask.getTimerTaskEntry != this
                      }
                      
                      /* 从 Bucket 链表中移除自己, 调用 TimerTaskList 的 remove 方法来做这件事
                      def remove(): Unit = {
                        var currentList = list
                        
                        /* 置空这个动作是在 TimerTaskList 的 remove 中完成的, 而这个方法可能会被其他线程同时调用, 使用 while 循环
                         * 的方式来确保 TimerTaskEntry 的 list 字段确实被置空了. Kafka 才能安全地认为此链表元素被成功移除
                        while (currentList != null) {
                          currentList.remove(this)
                          currentList = list
                        }
                      }
                     
                      override def compare(that: TimerTaskEntry): Int = {
                        this.expirationMs compare that.expirationMs
                      }
                    }
                    
    4. TimerTaskList 
            (1) TimerTaskList 描述时间轮的一格(即时间格), 在实现上采用双向链表实现, 用于封装位于特定时间区间范围内的所有的延时任务.
            (2) 构造函数
                    
                    private[timer] class TimerTaskList(
                                                        // 用于标识当前这个链表中的总定时任务数
                                                        taskCounter: AtomicInteger 
                                                      ) extends Delayed {
                     
                      /* TimerTaskList forms a doubly linked cyclic list using a dummy root entry
                      // root.next points to the head
                      // root.prev points to the tail
                      /** 根结点 */
                      private[this] val root = new TimerTaskEntry(null, -1)
                      root.next = root
                      root.prev = root
                      // 表示这个链表所在 Bucket 的过期时间戳。
                      private[this] val expiration = new AtomicLong(-1L)
                    }
                    
            (3) 
                 /* 代码使用 AtomicLong 的 CAS 方法 getAndSet 原子性地设置过期时间戳, 之后将新过期时间戳和旧值进行比较, 看看是否不同
                    然后返回结果. 目前 Kafka 使用一个 DelayQueue 统一管理所有的 Bucket, 也就是 TimerTaskList 对象. 随着时钟不断
                    向前推进, 原有 Bucket 会不断地过期, 然后失效. 当这些 Bucket 失效后, 会重用这些 Bucket.重用的方式就是重新设置
                    Bucket 的过期时间, 并把它们加回到 DelayQueue 中. 这里进行比较的目的, 就是用来判断这个 Bucket 是否要被插入到 DelayQueue。
                  def setExpiration(expirationMs: Long): Boolean = {
                    expiration.getAndSet(expirationMs) != expirationMs
                  }
                 
                  // Get the bucket's expiration time
                  def getExpiration(): Long = {
                    expiration.get()
                  }
                  
    5. TimingWheel
        (1) 定义
                private[timer] class TimingWheel(
                                                    /* 滴答一次的时长, 类似于手表例子中向前推进一格的时间. 对于秒针而言, 
                                                     * tickMs 就是 1 秒. 分针是 1 分, 时针是 1 小时.
                                                     * 在 Kafka 中, 第 1 层时间轮的 tickMs 被固定为 1 毫秒, 即向前推进一格 
                                                     * Bucket 的时长是 1 毫秒.
                                                    tickMs: Long, 
                                                 
                                                    /* 每一层时间轮上的 Bucket 数量, 第 1 层的 Bucket 数量是 20
                                                    wheelSize: Int, 
                                                    startMs: Long, // 时间轮对象被创建时的起始时间戳
                                                    taskCounter: AtomicInteger, // 这一层时间轮上的总定时任务数
                                                    
                                                    /* 将所有 Bucket 按照过期时间排序的延迟队列, 随着时间不断向前推进, 
                                                     * Kafka 需要依靠这个队列获取那些已过期的 Bucket, 并清除它们
                                                    queue: DelayQueue[TimerTaskList] 
                                               
                                                ) 
                  {
                  /* 这层时间轮总时长, 等于滴答时长乘以 wheelSize. 以第 1 层为例, interval 就是 20 毫秒. 由于下一层时间轮的滴答时长
                   * 就是上一层的总时长, 因此第 2 层的滴答时长就是 20 毫秒, 总时长是 400 毫秒，以此类推。
                  private[this] val interval = tickMs * wheelSize
                  
                  /* 时间轮下的所有 Bucket 对象, 即所有 TimerTaskList 对象
                  private[this] val buckets = Array.tabulate[TimerTaskList](wheelSize) { _ => new TimerTaskList(taskCounter) }
                  
                  /* 当前时间戳, 将它设置成小于当前时间的最大滴答时长的整数倍.
                   * 即假设滴答时长是 20 毫秒, 当前时间戳是 123 毫秒, 那么 currentTime 会被调整为 120 毫秒
                  private[this] var currentTime = startMs - (startMs % tickMs) // rounding down to multiple of tickMs
                 
                  /* 如果第 1 层的 interval 无法容纳定时任务的超时时间, 就创建并配置好第 2 层时间轮, 并再次尝试放入, 依然无法容纳,
                   * 就再创建和配置第 3 层时间轮, 以此类推，直到找到适合容纳该定时任务的第 N 层时间轮。
                  @volatile private[this] var overflowWheel: TimingWheel = null
                }
                
        (2) addOverflowWheel 方法
                每当需要一个新的上层时间轮时, 就会调用 addOverflowWheel 方法. 实现为创建一个新的 TimingWheel 实例, 也就是创建
            上层时间轮. 所用的滴答时长等于下层时间轮总时长, 而每层的轮子数都是相同的. 创建完成之后, 将新创建的实例赋值给 
            overflowWheel 字段
            
        (3) add 方法
                用于往时间轮中添加延时任务, 该方法接收一个 TimerTaskEntry 类型对象, 即对延时任务 TimerTask 的封装.
                第一步: 获取定时任务的过期时间戳(定时任务过期时的时点, 会被触发)
                第二步: 看定时任务是否已被取消. 如果已经被取消, 则无需加入到时间轮中. 如果没有被取消, 就接着看这个定时任务是否已经过期.
                       如果过期了, 自然也不用加入到时间轮中. 如果没有过期, 就看这个定时任务的过期时间是否能够被涵盖在本层时间轮的
                       时间范围内. 如果可以,则进入到下一步
                第三步: 首先计算目标 Bucket 序号, 也就是这个定时任务需要被保存在哪个 TimerTaskList. 如第 1 层的时间轮有 20 个 
                       Bucket, 每个滴答时长是 1 毫秒. 第 2 层时间轮的滴答时长应该就是 20 毫秒, 总时长是 400 毫秒. 第 2 层第 1 个 
                       Bucket 的时间范围应该是[20，40), 第 2 个 Bucket 的时间范围是[40，60）, 依次类推。假设现在有个延
                       时请求的超时时间戳是 237, 那么它就应该被插入到第 11 个 Bucket 中.
                第四步: 如果这个 Bucket 是首次插入定时任务, 要将这个 Bucket 加入到 DelayQueue 中, 方便 Kafka 轻松地获取那些
                       已过期 Bucket, 并删除它们.如果定时任务的过期时间无法被涵盖在本层时间轮中, 那么就按需创建上一层时间戳, 
                       然后在上一层时间轮上完整地执行刚刚所说的所有逻辑
                       
    6. Timer 
            (1) Timer 接口定义管理延迟操作的方法
            (2) 
                
                trait Timer {

                  // 将给定的定时任务插入到时间轮上, 等待后续延迟执行
                  def add(timerTask: TimerTask): Unit
                 
                  // 向前推进时钟, 执行已达过期时间的延迟任务
                  def advanceClock(timeoutMs: Long): Boolean
                 
                  // 获取时间轮上总的定时任务数
                  def size: Int
                 
                  // 关闭定时器
                  def shutdown(): Unit
                }
                
    7. SystemTimer
            (1) 实现延迟操作, SystemTimer 类是 Timer 接口的实现类. 它是一个定时器类, 封装了分层时间轮对象, 为 Purgatory 提供延迟
                请求管理功能, Purgatory 是保存延迟请求的缓冲区.
            (2) 
                class SystemTimer(executorName: String, // Purgatory 的名字
                                  tickMs: Long = 1, // 默认时间格时间为 1 毫秒
                                  wheelSize: Int = 20, // 默认时间格大小为 20
                                  startMs: Long = Time.SYSTEM.hiResClockMs // 该 SystemTimer 定时器启动时间
                                 ) extends Timer {
                 
                  // timeout timer, 单线程的线程池用于异步执行定时任务
                  private[this] val taskExecutor = Executors.newFixedThreadPool(1, new ThreadFactory() {
                    def newThread(runnable: Runnable): Thread =
                      KafkaThread.nonDaemon("executor-"+executorName, runnable)
                  })
                  
                  // 延迟队列保存所有 Bucket, 即所有 TimerTaskList 对象, 只有在 Bucket 过期后, 才能从该队列中获取到.
                  private[this] val delayQueue = new DelayQueue[TimerTaskList]()
                  
                  // 总定时任务数
                  private[this] val taskCounter = new AtomicInteger(0)
                  
                  // 分层时间轮对象
                  private[this] val timingWheel = new TimingWheel(
                    tickMs = tickMs,
                    wheelSize = wheelSize,
                    startMs = startMs,
                    taskCounter = taskCounter,
                    delayQueue
                  )
             
                  // 维护线程安全的读写锁
                  private[this] val readWriteLock = new ReentrantReadWriteLock()
                  private[this] val readLock = readWriteLock.readLock()
                  private[this] val writeLock = readWriteLock.writeLock()
                }
            (3) 
                size 方法: 计算给定 Purgatory 下的总延迟请求数
                shutdown 方法: 关闭线程池
                addTimerTaskEntry 方法: 将给定的 TimerTaskEntry 插入到时间轮中, 有 3 种情况, 第一种情况, 如果该任务既未取消
                                       也未过期, 那么 addTimerTaskEntry 方法将其添加到时间轮. 第二种情况, 如果该任务已取消, 
                                       则该方法什么都不做, 直接返回. 第三种情况, 如果该任务已经过期, 则提交到相应的线程池, 
                                       等待后续执行
                reinsert 方法: 调用 addTimerTaskEntry 重新将定时任务插入回时间轮
                advanceClock 方法: 遍历 delayQueue 中的所有 Bucket, 并将时间轮的时钟依次推进到它们的过期时间点, 令它们过期. 
                                  再将这些 Bucket 下的所有定时任务全部重新插入回时间轮
                                  
    8. DelayedOperation 类
            (1) 是所有 Kafka 延迟请求类的抽象父类, 实现了 TimerTask 特质
            (2) 
                abstract class DelayedOperation(
                                                //  只需要传入一个超时时间, 是客户端发出请求的超时时间(客户端参数 request.timeout.ms)
                                                override val delayMs: Long, 
                                                lockOpt: Option[Lock] = None)
                  extends TimerTask with Logging {
                  
                       // Visible for testing
                      private[server] val lock: Lock = lockOpt.getOrElse(new ReentrantLock)
                                            
                      // 标识该延迟操作是否已经完成
                      private val completed = new AtomicBoolean(false)
                      
                      /* 解决一个问题:
                            当多个线程同时检查某个延迟操作是否满足完成条件时, 如果其中一个线程持有锁(上面的 lock 字段), 然后
                            执行条件检查, 会发现不满足完成条件. 而另一个线程执行检查时却发现条件满足, 但是这个线程又没有拿到锁, 那么
                            该延迟操作将永远不会有再次被检查的机会, 会导致最终超时.
                            
                       * 目的: 确保拿到锁的线程有机会再次检查条件是否已经满足
                      private val tryCompletePending = new AtomicBoolean(false)
                      
                     
                }
            (3)
                forceComplete 方法: 强制完成延迟操作, 不管它是否满足完成条件. 每当操作满足完成条件或已经过期, 就需要调用该方法完成
                                   该操作.
                isCompleted 方法：检查延迟操作是否已经完成.
                onExpiration 方法: 强制完成之后执行的过期逻辑回调方法. 只有真正完成操作的那个线程才可以调用这个方法.
                onComplete 方法: 完成延迟操作所需的处理逻辑. 这个方法只会在 forceComplete 方法中被调用.
                tryComplete 方法：尝试完成延迟操作的顶层方法, 内部会调用 forceComplete 方法
                maybeTryComplete 方法：线程安全版本的 tryComplete 方法. 慢慢地取代 tryComplete, 外部代码调用的都是这个方法.
                                       存在多个线程同时被访问, 
                                       如果拿到锁对象, 就依次执行清空 tryCompletePending 状态, 完成延迟请求, 释放锁以及读取最新
                                       retry 状态的动作. 
                                       如果未拿到锁的线程, 就只能设置 tryCompletePending 状态, 来间接影响 retry 值, 从而给获取
                                       到锁的线程一个重试的机会
                run 方法：调用延迟操作超时后的过期逻辑, 也就是组合调用 forceComplete + onExpiration
                
    9. DelayedOperationPurgatory 类
            (1) 
            (2) 
                    final class DelayedOperationPurgatory[T <: DelayedOperation](
                        purgatoryName: String,
                        timeoutTimer: Timer,  // 定时器
                        brokerId: Int = 0,  // 所在 broker 节点 ID
                        // 控制删除线程移除 Bucket 中的过期延迟请求的频率, 在绝大部分情况下, 都是 1 秒一次
                        purgeInterval: Int = 1000, 
                        reaperEnabled: Boolean = true, // 是否启用后台指针推进器
                    timerEnabled: Boolean = true) extends Logging with KafkaMetricsGroup {
                    ......
                    }
                    
            (3) tryCompleteElseWatch 方法
                    检查操作是否能够完成, 先尝试完成请求, 如果不能, 就把它加入到对应 Key 所在的 WatcherList 中, 等待后面再试
```

## 副本管理模块
### 副本同步的原理
```shell
    1. 概念
            (1) 副本读取状态
                    a. 截断中(Truncating, 副本执行截断操作)
                    b. 获取中(Fetching, 副本被读取)
            (2) 分区读取状态
                    a. 可获取, 代表副本获取线程当前可以读取数据
                    b. 截断中, 表明分区副本正在执行截断操作(比如该副本刚刚成为 Follower 副本)
                    c. 被推迟, 表明副本获取线程获取数据时出现错误, 需要等待一段时间后重试
            (3) 副本读取状态处于获取中, 并不一定表示分区读取状态就是可获取状态. 分区是否能够被获取的条件要比副本严格一些.
            (4) 拉取线程逻辑：循环执行截断操作和获取数据操作, 拉取线程只能拉取处于可读取状态的分区的数据.
            
    2. AbstractFetcherThread 抽象类
            (1) 实现的功能是从 Broker 获取多个分区的消息数据, 至于获取之后如何对这些数据进行处理, 则交由子类来实现.
            (2)
                abstract class AbstractFetcherThread(
                                                        name: String,  // 线程名称
                                                        clientId: String, // Client Id, 用于日志输出
                                                        
                                                        // 数据源 Broker 地址, 指此线程要从哪个 Broker 上读取数据
                                                        val sourceBroker: BrokerEndPoint, 
                                                        
                                                        // 线程处理过程报错的分区集合
                                                        failedPartitions: FailedPartitions,
                                                        
                                                        /* 获取分区数据出错后的等待重试间隔, 默认是 Broker 端参数 
                                                         * replica.fetch.backoff.ms 值
                                                        fetchBackOffMs: Int = 0,
                                                        isInterruptible: Boolean = true)// 线程是否允许被中断
                  extends ShutdownableThread(name, isInterruptible) {
                      // 定义 FetchData 类型表示获取的消息数据
                      type FetchData = FetchResponse.PartitionData[Records]
                      
                      // 定义 EpochData 类型表示 Leader Epoch 数据
                      type EpochData = OffsetsForLeaderEpochRequest.PartitionData
                      
                     
                      private val partitionStates = new PartitionStates[PartitionFetchState]
                }
                
                字段定义:
                        (1) FetchResponse.PartitionData 保存的是 Response 中单个分区数据拉取的各项数据, 包括从该分区的 Leader 
                                                        副本拉取回来的消息, 该分区的高水位值和日志起始位移值等
                                                        
                            public static final class PartitionData<T extends BaseRecords> {
                                    public final Errors error; // 错误码
                                    public final long highWatermark;// 高水位值
                                    public final long lastStableOffset;// 最新 LSO 值, kafka 事务相关
                                    public final long logStartOffset;// 最新 Log Start Offset 值
                                    
                                    // 用于指定可对外提供读服务的 Follower 副本；kafka 2.4 之后支持部分 Follower 副本可以对外提供读服务
                                    public final Optional<Integer> preferredReadReplica; 
                                    
                                    public final List<AbortedTransaction> abortedTransactions; // 该分区对应的已终止事务列表
                                    public final T records; // 消息集合, 最重要的字段
                              }   
                                                                             
            (3) processPartitionData 方法
                    a. 用于处理读取回来的消息集合. processPartitionData 是一个抽象方法, 需要子类(ReplicaFetcherThread)实现
                        它的逻辑
                        
                    b.
                        protected def processPartitionData(
                                                            topicPartition: TopicPartition, // 读取哪个分区的数据
                                                            fetchOffset: Long, // 读取到的最新位移值
                                                            partitionData: FetchData // 读取到的分区消息数据
                                                          ): Option[LogAppendInfo] // 写入已读取消息数据前的元数据
                                                          
            (4)  truncate 方法
                    a. 截取消息, 需要子类实现它的逻辑, 告诉 Kafka 要把指定分区下副本截断到哪个位移值
                    b. 
                        protected def truncate(
                                                topicPartition: TopicPartition, // 要对哪个分区下副本执行截断操作
                                                truncationState: OffsetTruncationState // Offset + 截断状态(一个截断完成与否)
                        ): Unit
                                     
            (5) buildFetch 方法
                    a. 构建 FetchRequest 对象.
                    b. 
                        protected def buildFetch(
                                                /* 一组要读取的分区列表, 分区是否可读取取决于 PartitionFetchState 中的状态
                                                partitionMap: Map[TopicPartition, PartitionFetchState]):
                        /* 封装 FetchRequest.Builder 对象
                        ResultWithPartitions[Option[ReplicaFetch]]
                        
            (6) doWork 方法
                    a. 定义
                            override def doWork(): Unit = {
                                maybeTruncate() // 执行副本截断操作
                                maybeFetch() // 执行消息获取操作
                            }
                    b. AbstractFetcherThread 线程只要一直处于运行状态, 则不断运行副本截断操作, 以及消息获取操作. 不断进行副本截断
                       操作原因是分区的 Leader 可能会随时发生变化, 每当有新 Leader 产生时, Follower 副本就必须主动执行截断操作, 
                       将自己的本地日志裁剪成与 Leader 一模一样的消息序列, 甚至 Leader 副本本身也需要执行截断操作, 将 LEO 调整到
                       分区高水位处.
                       
                    c. maybeTruncate() 方法, 首先分区状态处于截断中状态, 按照分区有无 Epoch 值进行分组, 如果分区存在 Leader Epoch
                       值时, 会将副本的本地日志截断到 Leader Epoch 对应的最新位移值处(truncateToEpochEndOffsets 方法)
                       如果分区不存在对应的 Leader Epoch 记录, 那么依然使用原来的高水位机制, 调用方法 truncateToHighWatermark 
                       将日志调整到高水位值处
                       
                    d.  truncateToHighWatermark 方法
                            先遍历给定的所有分区, 依次为每个分区获取当前的高水位值, 并保存在分区读取状态类中, 调用 doTruncate 方法
                        执行真正的日志截断操作. 等将给定的所有分区都执行对应的操作后, 更新这组分区的分区读取状态
                        
                    e.  maybeFetch 方法
                            从 leader 中获取消息并处理
                            第一步: 为 partitionStates 中的分区构造 FetchRequest 对象, partitionStates 中保存的是要去获取消息
                                   的分区以及对应的状态, 得到的结果是 ReplicaFetch 对象 + 出错的分区, ReplicaFetch 对象是包含
                                   要读取的核心信息(要读取哪个分区, 从哪个位置开始读, 最多读多少字节等) + FetchRequest.Builder 对象
                                   
                            第二步: 处理这组出错分区. 将这组分区加入到有序 Map 末尾等待后续重试, 发现当前没有任何可读取的分区, 
                                   会阻塞等待一段时间.
                            第三步: 发送 FETCH 请求给对应的 Leader 副本, 并处理相应的 Response(processFetchRequest 方法)
                            
                    f. processFetchRequest 方法
                            第一步: 调用 fetchFromLeader 方法给 Leader 发送 FETCH 请求, 并阻塞等待 Response 的返回, 然后
                                   更新 FETCH 请求发送速率的监控指标.
                            第二步: 拿到 Response 后, 从中取出分区的核心信息, 比较要读取的位移值和当前 AbstractFetcherThread 
                                   线程缓存的该分区下一条待读取的位移值是否相等, 以及当前分区是否处于可获取状态. 如果不满足这两个条件,
                                   说明这个 Request 可能是之前很久都未处理的请求, 就不用处理. 
                                   如果满足这两个条件并且 Response 没有错误, 提取 Response 中的 Leader Epoch 值, 交由子类实现
                                   具体的 Response 处理(调用 processPartitionData 方法), 将该分区放置在有序 Map 的末尾以
                                   保证公平性. 如果该 Response 有错误, 调用对应错误的定制化处理逻辑, 然后将出错分区加入到出错分区
                                   列表中.
                            第三步: 调用 handlePartitionsWithErrors 方法, 统一处理上一步处理过程中出现错误的分区
                            
    3. ReplicaFetcherThread 子类
            (1) 定义
                    class ReplicaFetcherThread(
                                               name: String, // 线程名称
                                               
                                               /* 单台 Broker 上, 允许存在多个 ReplicaFetcherThread 线程, Broker 端参数 
                                                * num.replica.fetchers, 决定 Kafka 创建多少个 Follower 拉取线程. Follower 
                                                * 拉取的线程 Id, 即线程的编号
                                               fetcherId: Int,
                                                
                                               // 数据源 Broker 地址, 即分区的 leader 是在哪个节点上
                                               sourceBroker: BrokerEndPoint,
                                               
                                               /* 封装 Broker 端所有的参数信息, ReplicaFetcherThread 类也是通过它来获取
                                                * Broker 端指定参数的值
                                               brokerConfig: KafkaConfig, 
                                               failedPartitions: FailedPartitions, // 处理过程中出现失败的分区
                                               
                                               // 副本管理器.该线程类通过副本管理器来获取分区对象, 副本对象以及它们下面的日志对象
                                               replicaMgr: ReplicaManager, 
                                               metrics: Metrics,
                                               time: Time,
                                               quota: ReplicaQuota, // 用做限流, 用作 Follower 副本拉取速度控制
                                               
                                               /* 这是用于实现同步发送请求的类, 同步发送是指该线程使用它给指定 Broker 发送请求,
                                                * 然后线程处于阻塞状态, 直到接收到 Broker 返回的 Response.
                                               leaderEndpointBlockingSend: Option[BlockingSend] = None 
                                              
                                              )
                    extends AbstractFetcherThread(name = name,
                                                    clientId = name,
                                                    sourceBroker = sourceBroker,
                                                    failedPartitions,
                                                    fetchBackOffMs = brokerConfig.replicaFetchBackoffMs,
                                                    isInterruptible = false) {
                                                    
                              // 副本 Id 就是副本所在 Broker 的 Id
                              private val replicaId = brokerConfig.brokerId
                              private val logContext = new LogContext(s"[ReplicaFetcher replicaId=$replicaId, leaderId=${sourceBroker.id}, " +
                                s"fetcherId=$fetcherId] ")
                              this.logIdent = logContext.logPrefix
                              
                              // 用于执行请求发送的类
                              private val leaderEndpoint = leaderEndpointBlockingSend.getOrElse(
                                new ReplicaFetcherBlockingSend(sourceBroker, brokerConfig, metrics, time, fetcherId,
                                  s"broker-$replicaId-fetcher-$fetcherId", logContext))
                                  
                              // Follower 发送的 FETCH 请求被处理返回前的最长等待时间, Broker 端参数 replica.fetch.wait.max.ms 
                              private val maxWait = brokerConfig.replicaFetchWaitMaxMs
                              
                              // 每个 FETCH Response 返回前必须要累积的最少字节数. Broker 端参数 replica.fetch.min.bytes
                              private val minBytes = brokerConfig.replicaFetchMinBytes
                              
                              // 每个合法 FETCH Response 的最大字节数. Broker 端参数 replica.fetch.response.max.bytes
                              private val maxBytes = brokerConfig.replicaFetchResponseMaxBytes
                              
                              // 单个分区能够获取到的最大字节数.Broker 端参数 replica.fetch.max.bytes
                              private val fetchSize = brokerConfig.replicaFetchMaxBytes
                              
                              private val brokerSupportsLeaderEpochRequest = brokerConfig.interBrokerProtocolVersion >= KAFKA_0_11_0_IV2
                              
                              // 维持某个 Broker 连接上获取会话状态的类
                              private val fetchSessionHandler = new FetchSessionHandler(logContext, sourceBroker.id)
                    }
                    
                    注意:
                            (1) maxWait, minBytes, maxBytes, fetchSize 主要控制 Follower 副本拉取 Leader 副本消息的行为,
                               比如一次请求到底能够获取多少字节的数据, 或者当未达到累积阈值时, FETCH 请求等待多长时间等
                               
            (2) processPartitionData 方法(处理拉取的消息)
                    第一步: 从副本管理器获取指定主题分区对象, 并且获取日志对象, 创建消息集合对象(通过获取到的数据进行转化)
                    第二步: 进行异常判断, 要读取的起始位移值与本地日志 LEO 值是否相等, 不相等为异常(要读取的位移值肯定
                           要本地日志 LEO 相等)
                    第三步: 将消息集合写入到 Follower 副本本地日志.
                    第四步: 更新 Follower 副本的高水位值, 将 FETCH 请求 Response 中包含的高水位值作为新的高水位值, 尝试更新
                           Follower 副本的 Log Start Offset 值, 因为 Leader 的 Log Start Offset 可能发生变化, 比如用户手动
                           执行删除消息的操作等. Follower 副本的日志需要和 Leader 保持一致, 如果 Leader 的 Log Start Offset 值
                           发生变化, Follower 自然也要发生变化, 以保持一致.
                           
            (3) buildFetch 方法(构建拉取消息的请求)
                    遍历每个分区, 将处于可获取状态的分区添加到 builder 后续统一处理, 对于有错误的分区加入到出错分区列表
                   获取日志起始位移值, 构建读取对象 partitionData, 生成 fetchEquestData, 构造 FETCH 请求的 Builder 对象, 返回
                   Builder 对象以及出错分区列表.
                   
            (4) truncate 方法(执行截断日志操作)
                    取到分区对象, 取到分区本地日志, 执行截断操作 
                    
```

## ReplicaManager(副本管理器)
```shell
    1. 概念
            (1) 负责管理和操作集群中 Broker 的副本, 还承担部分的分区管理工作, 比如变更整个分区的副本日志路径等.
            (2) 在线上环境最好不要使用 deleteRecords 删除操作, 这会改变 Log Start Offset, 与 follwer 副本同步线程更新 
                Log Start Offset 冲突.
            (3) 每个 Broker 在启动的时, 都会创建 ReplicaManager 实例. 该实例一旦被创建, 就会对 Leader 副本或 Follower 副本进行管理。
    2. 代码结构
            (1) ReplicaManager 类: 副本管理器的具体实现代码, 里面定义读写副本、删除副本消息的方法以及其他管理方法
            (2) ReplicaManager 对象: ReplicaManager 类的伴生对象, 定义 3 个常量
            (3) HostedPartition 及其实现对象: 表示 Broker 本地保存的分区对象的状态. 有 不存在状态(None), 在线状态(Online)和
                                            离线状态(Offline)
            (4) FetchPartitionData: 定义获取到的分区数据以及重要元数据信息, 如高水位值, Log Start Offset 值等
            (5) LogReadResult: 表示副本管理器从副本本地日志中读取到的消息数据, 相关元数据信息, 如高水位值, Log Start Offset 值等
            (6) LogDeleteRecordsResult: 表示副本管理器执行副本日志删除操作后返回的结果信息
            (7) LogAppendResult：表示副本管理器执行副本日志写入操作后返回的结果信息
            
    3. ReplicaManager 类
            (1) 定义
                    class ReplicaManager(
                                            val config: KafkaConfig, // 配置管理类
                                            metrics: Metrics, // 监控指标类
                                            time: Time,  // 定时器类
                                            val zkClient: KafkaZkClient, // ZooKeeper客户端
                                            scheduler: Scheduler,   // Kafka 调度器
                                            
                                            /* 日志管理器, 负责创建和管理分区的日志对象, 里面定义很多操作日志对象的方法
                                            val logManager: LogManager,
                                            val isShuttingDown: AtomicBoolean, // 是否已经关闭
                                            quotaManagers: QuotaManagers,  // 配额管理器
                                            val brokerTopicStats: BrokerTopicStats, // Broker 主题监控指标类
                                            
                                             /* Broker 元数据缓存, 保存集群上分区的 Leader, ISR 等信息, 每台 Broker 上的
                                              * 元数据缓存, 是从 Controller 端的元数据缓存异步同步过来的
                                            val metadataCache: MetadataCache, 
                                            
                                            /* 失效日志路径的处理器类,  Kafka 提升服务器端高可用性的一个改进, 单块磁盘坏掉, 整个
                                             * Broker 的服务也不会中断,      
                                            logDirFailureChannel: LogDirFailureChannel,
                                            
                                            // 处理延时 PRODUCE 请求的 Purgatory
                                            val delayedProducePurgatory: DelayedOperationPurgatory[DelayedProduce], 
                                            
                                            // 处理延时 FETCH 请求的 Purgatory
                                            val delayedFetchPurgatory: DelayedOperationPurgatory[DelayedFetch],  
                                             
                                            // 处理延时 DELETE_RECORDS 请求的 Purgatory
                                            val delayedDeleteRecordsPurgatory: DelayedOperationPurgatory[DelayedDeleteRecords],  
                                            
                                            // 处理延时 ELECT_LEADERS 请求的 Purgatory
                                            val delayedElectLeaderPurgatory: DelayedOperationPurgatory[DelayedElectLeader], 
                                         threadNamePrefix: Option[String]) 
                                         extends Logging with KafkaMetricsGroup {....}
                                         
                                         
            (2) 自定义字段
                    a. controllerEpoch
                            该字段表示最新一次变更分区 Leader 的 Controller 的 Epoch 值, 默认值为 0, Controller 每发生一次变更,
                            该字段值都会 +1.
                            作用: 隔离过期 Controller 发送的请求, 即老的 Controller 发送的请求不能再被继续处理, 判断新老 Controller
                                 的方法是看请求携带的 controllerEpoch 值, 是否等于这个字段. 如果请求中携带的 controllerEpoch 值
                                 小于该字段值, 则是老的 Controller 发送出来的. ReplicaManager 在处理控制类请求时, 会更新该字段, 
                                 当请求的中携带的 controllerEpoch 值大于 ReplicaManager 中保存的值
                                 
                    b. allPartitions
                            包含 Broker 上保存的所有分区对象数据, 一个 Partition 实例表示定义和管理单个分区, 利用 logManager 帮助
                       它完成对分区底层日志的操作. ReplicaManager 类对于分区的管理, 是通过 Partition 对象完成的
                       
                    c. replicaFetcherManager
                            创建 ReplicaFetcherThread 类实例(帮助 Follower 副本向 Leader 副本拉取消息, 并写入到本地日志中).
                       ReplicaManager 类利用 replicaFetcherManager 字段, 对所有 Fetcher 线程进行管理, 包括线程的创建, 启动, 
                       添加, 停止和移除
```

### 副本控制器读写副本
```shell
    1. 概念
        (1) 无论是读取副本还是写入副本, 都是通过底层的 Partition 对象完成
    2. 副本写入
        (1) 副本写入的场景
                场景一: 生产者向 Leader 副本写入消息(appendRecords 方法)
                场景二: Follower 副本拉取消息后写入副本(直接调用 Partition 对象的方法)
                场景三: 消费者组写入组信息(appendRecords 方法)
                场景四: 事务管理器写入事务信息(包括事务标记, 事务元数据等)--- appendRecords 方法
                
        (2) 定义
                 def appendRecords(
                                        timeout: Long, // 请求处理超时时间, 对于生产者是 request.timeout.ms 参数值
                                        
                                        /* 请求 acks 设置, Kafka 默认使用 -1, 等待其他副本全部写入成功再返回
                                        requiredAcks: Short, 
                                        
                                        /* 是否允许写入内部主题, 对于普通的生产者为 False, 不允许写入内部主题.
                                         * 对于 消费者组 GroupCoordinator 组件, 就是向内部位移主题写入消息(该字段为 true)
                                        internalTopicsAllowed: Boolean, 
                                        
                                        /* 写入方来源: 副本(Replication), coordinator, 客户端(Client)
                                         * Replication 表示写入请求是由 Follower 副本发出, 从 Leader 副本获取到的消息写入到
                                         * follwer 底层的消息日志中. Coordinator 表示这些写入由 Coordinator 发起, 它既可以是
                                         * 管理消费者组的 GroupCooridnator, 也可以是管理事务的 TransactionCoordinator.
                                         * Client 表示本次写入由客户端发起
                                        origin: AppendOrigin,  
                                        
                                        // 待写入消息, 按分区分组的、实际要写入的消息集合
                                        entriesPerPartition: Map[TopicPartition, MemoryRecords], 
                                        
                                        // 写入成功之后, 要调用的回调逻辑函数
                                        responseCallback: Map[TopicPartition, PartitionResponse] => Unit, 
                                        
                                        /* 专门用来保护消费者组操作线程安全的锁对象, 在其他场景中用不到
                                        delayedProduceLock: Option[Lock] = None, 
                                        
                                        /* 消息格式转换操作的回调统计逻辑, 主要用于统计消息格式转换操作过程中的一些数据指标, 比如
                                         * 总共转换多少条消息, 花费多长时间
                                        recordConversionStatsCallback: Map[TopicPartition, RecordConversionStats] => Unit = _ => () 
                                   ): Unit = {...}
                                   
                 appendRecords 方法
                    第一步: 判断 requiredAcks 的取值是否合理(-1, 0, 1), 如果不是这些值则构造特定异常对象, 封装进回调函数.
                    
                    第二步: 调用 appendToLocalLog 方法, 将要写入的消息集合保存到副本的本地日志, 构造 PartitionResponse 对象实例,
                           封装写入结果以及一些重要的元数据信息, 比如本次写入有没有错误(errorMessage), 下一条待写入消息的位移值, 
                           本次写入消息集合首条消息的位移值等等, 再更新消息格式转换的指标数据.需要调用 
                           delayedProduceRequestRequired 方法, 来判断本次写入是否算是成功
                           
                    第三步: 如果还需要等待其他副本同步完成消息写入, 就不能立即返回,创建 DelayedProduce 延时请求对象, 并把该对象交由
                           Purgatory 来管理. DelayedProduce 是生产者端的延时发送请求, 对应的 Purgatory 就是 ReplicaManager 
                           类构造函数中的 delayedProducePurgatory. Purgatory 管理, 是调用 tryCompleteElseWatch 方法尝试
                           完成延时发送请求, 如果暂时无法完成, 就将对象放入到相应的 Purgatory 中, 等待后续处理.
                           
                    第四步:如果无需等待其他副本同步完成消息写入, 则构造响应的 Response, 并调用回调逻辑函数
                    
    3. 副本读取
            (1) 不论是消费者 API, 还是 Follower 副本, 拉取消息都是向 Broker 发送 FETCH 请求, Broker 端接收到该请求后, 调用 
                fetchMessages 方法从底层的 Leader 副本取出消息
            (2) fetchMessages 方法也会延时处理 FETCH 请求, 因为 Broker 端必须要累积足够多的数据后, 才会返回 Response 给请求发送方.
            
            (3) 定义
                    def fetchMessages(
                                            /* 请求处理超时时间, 对于消费者而言, 是 request.timeout.ms 参数值；
                                             * 对于 Follower 副本而言, 是 Broker 端参数 replica.fetch.wait.max.ms 
                                            timeout: Long, 
                                            /* 副本 ID, 对于消费者而言, 该参数值是 -1.
                                             * 对于 Follower 副本而言, 该值就是 Follower 副本所在的 Broker ID
                                            replicaId: Int, 
                                            
                                            /* 对于消费者而言, 对应于 Consumer 端参数 fetch.min.bytes 和 fetch.max.bytes 值；
                                             * 对于 Follower 副本而言, 对应于 Broker 端参数 replica.fetch.min.bytes 和
                                             * replica.fetch.max.bytes 值
                                            fetchMinBytes: Int, // 够获取的最小字节数
                                            fetchMaxBytes: Int, // 够获取的最大字节数
                                            
                                            /* 对能否超过最大字节数做硬限制, 如果 hardMaxBytesLimit=True, 表示读取请求返回的
                                             * 数据字节数绝不允许超过最大字节数
                                            hardMaxBytesLimit: Boolean, 
                                            
                                            /* 规定读取分区的信息, 比如要读取哪些分区, 从这些分区的哪个位移值开始读, 
                                             * 最多可以读多少字节等等
                                            fetchInfos: Seq[(TopicPartition, PartitionData)], 
                                            
                                            /* 这是一个配额控制类, 判断是否需要在读取的过程中做限速控制
                                            quota: ReplicaQuota, 
                                            
                                            /* Response 回调逻辑函数, 当请求被处理完成后, 调用该方法执行收尾逻辑
                                            responseCallback: Seq[(TopicPartition, FetchPartitionData)] => Unit, 
                                            isolationLevel: IsolationLevel,
                                            clientMetadata: Option[ClientMetadata]): 
                                            Unit = {......}
                                            
                                            
                    fetchMessages 方法
                          第一步：　读取本地日志
                                    先判断读取消息的请求方是 Follower 副本, 还是普通的 Consumer, 通过 replicaId
                                   字段, 如果是 replicaId 等于 -1, 则是 Consumer, 如果大于 0, 则代表 Follower 副本. 
                                   对于 Follower 副本, 能读取到 Leader 副本 LEO 值以下的所有消息；对于普通 Consumer 而言, 只能
                                   读取到 Leader 副本高水位值以下的消息. 调用 readFromLog 方法读取本地日志上的消息数据, 并将结果
                                   赋值给 logReadResults 变量
                                   
                          第二步: 根据读取结果构建 Response
                                    根据读取到的结果, 统计可读取的总字节数, 在判断是否立马返回 Reponse, 只要满足其中之一条件就可以了
                                 第一个条件:请求没有设置超时时间, 请求被处理后立即返回. 第二个条件: 未获取到任何数据. 第三个条件:
                                 已累积到足够多数据. 第四个条件: 读取过程中出错. 如果都不满足, 构建 DelayedFetch 对象, 然后把
                                 延时对象交由 delayedFetchPurgatory 后续自动处理.
                                    
```

### ReplicaManager 管理副本
```shell
    1. 概念
    2. 分区及副本管理
            (1) ReplicaManager.allPartitions 定义
                    private val allPartitions = new Pool[TopicPartition, HostedPartition](
                        valueFactory = Some(tp => HostedPartition.Online(Partition(tp, time, this)))
                        
                每个 ReplicaManager 实例都维护所在 Broker 上保存的所有分区对象, 而每个分区对象 Partition 下面又定义了一组副本对象
                Replica. 通过这样的层级关系, 副本管理器实现对于分区的直接管理和对副本对象的间接管理.即 ReplicaManager 通过直接操作
                分区对象来间接管理下属的副本对象.
                
                broker 分区中的 follwer 副本和 leader 副本不会一直不变的, 会动态变更, 这些变更是通过 Controller 给 Broker 发送
                LeaderAndIsrRequest 请求来实现的. 当 Broker 端收到这类请求后, 会调用副本管理器的 becomeLeaderOrFollower 方法来处
                理, 并依次执行 “成为 Leader 副本” 和 “成为 Follower 副本”的逻辑
                
            (2) becomeLeaderOrFollower 方法
                    becomeLeaderOrFollower 方法就是具体处理 LeaderAndIsrRequest 请求的地方, 同时也是副本管理器添加分区的地方, 包含
                了处理 Controller Epoch 事宜, 执行成为 Leader 和 Follower 的逻辑, 构造 Response 三大部分.
                
                    第一部分: 处理 Controller Epoch 事宜
                                第一步: 比较 LeaderAndIsrRequest 携带的 Controller Epoch 值和当前 Controller Epoch
                                       值. 如果发现前者小于后者, 说明 Controller 已经变更到别的 Broker 上, 需要构造一
                                       个 STALE_CONTROLLER_EPOCH 异常并封装进 Response 返回.
                               
                                第二步: 将携带的 Controller Epoch 值更新到当前缓存的 Controller Epoch, 再提取出 
                                       LeaderAndIsrRequest 请求中涉及到的分区, 之后依次遍历这些分区.
                                
                                第三步: 从 ReplicaManager 的 allPartitions 中取出对应的分区对象, 直接将其赋值给 partitionOpt 
                                       字段, 如果是 Offline 状态的分区, 说明该分区副本所在的 Kafka 日志路径出现 I/O 故障
                                       (比如磁盘满), 需要构造对应的 KAFKA_STORAGE_ERROR 异常并封装进 Response, 同时令
                                        partitionOpt 字段为 None； 如果是 None 状态的分区, 则创建新分区对象, 然后将其加入到
                                        allPartitions 中, 进行统一管理, 并赋值给 partitionOpt 字段.
                                        
                                第四步: 检查 partitionOpt 字段的分区的 Leader Epoch.确保请求中携带的 Leader Epoch 值要大于
                                       当前缓存的 Leader Epoch, 否则是过期 Controller 发送的请求, 就直接忽略它, 不做处理
                                       
                    第二部分: 执行 “成为 Leader 副本” 和 “成为 Follower 副本”的逻辑
                                第一步: 确定两个分区集合, 一个是把该 Broker 当成 Leader 的所有分区集合；另一个是把该 Broker 当成
                                 　　　 Follower 的所有分区集合. 主要是看 LeaderAndIsrRequest 请求中分区的 Leader 信息, 
                                       是不是和本 Broker 的 ID 相同. 如果相同, 则表明该 Broker 是这个分区的 Leader；否则
                                       表示当前 Broker 是这个分区的 Follower.
                                       
                                 第二步: 分别为它们调用 makeLeaders 和 makeFollowers 方法, 正式让 Leader 和 Follower 角色生效
                                        对于那些当前 Broker 成为 Follower 副本的主题, 需要移除之前的 Leader 副本监控指标,
                                        以防出现系统资源泄露的问题. 对于那些当前 Broker 成为 Leader 副本的主题, 要移除它们之前的 
                                        Follower 副本监控指标.
                                        
                                 第三步: 如果有分区的本地日志为空, 说明底层的日志路径不可用. 需要标记该分区为 Offline 状态, 即更新 
                                 　　　　allPartitions 中分区的状态, 移除对应分区的监控指标
                                 
                    第三部分: 构造 Response 对象
                                第一步: 启动一个专属线程来执行高水位值持久化, 定期地将 Broker 上所有非 Offline 分区的高水位值
                                       写入检查点文件. 这个线程是个后台线程, 默认每 5 秒执行一次
                                       
                                第二步: 添加日志路径数据迁移线程.  将路径 A 上面的数据搬移到路径 B 上, 用于支持 Kafka 的 JBOD
                                       (Just a Bunch of Disks)
                                       
                                第三步: 关闭空闲副本拉取线程和空闲日志路径数据迁移线程. 判断空闲与否条件是, 分区 Leader/Follower 
                                       角色调整后，是否存在不再使用的拉取线程
                                       
                                第四步: 执行 LeaderAndIsrRequest 请求的回调处理逻辑. 这里的回调逻辑对 Kafka 两个内部主题
                                      (__consumer_offsets 和 __transaction_state)有用, 其他主题不适用, 通常情况下, 
                                       可以无视这里的回调逻辑
                                       
                                第五步: 构造 LeaderAndIsrRequest 请求的 Response 对象, 并进行返回
                                
                                
            (3) makeLeaders 方法
                    a. makeLeaders 方法让当前 Broker 成为给定一组分区的 Leader, 即让当前 Broker 下该分区的副本成为 Leader 副本.
                        大致流程为停掉这些分区对应的获取线程, 更新 Broker 缓存中的分区元数据信息, 将指定分区添加到 Leader 分区集合
                        
                    b. 定义
                           
                            private def makeLeaders(
                                                    controllerId: Int, 　//  Controller 所在 Broker 的 ID
                                                    controllerEpoch: Int, // Controller Epoch 值, Controller 版本号
                                                    
                                                     /* LeaderAndIsrRequest 请求中携带的分区信息, 包括每个分区的 Leader 是谁
                                                      * ISR 都有哪些等数据
                                                    partitionStates: Map[Partition, LeaderAndIsrPartitionState],
                                                    
                                                     /* 请求的 Correlation 字段, 只用于日志调试
                                                    correlationId: Int,
                                                    
                                                    /* 按照主题分区分组的异常错误集合
                                                    responseMap: mutable.Map[TopicPartition, Errors],
                                                    
                                                     /* 操作磁盘上高水位检查点文件的工具类
                                                    highWatermarkCheckpoints: OffsetCheckpoints): 
                                                    Set[Partition] = {
                                                        ......
                                                        }
                                                        
                            makeLeaders 方法接收 6 个参数, 并返回一个分区对象集合, 这个集合就是当前 Broker 是 Leader 的所有分区.
                            
                    c. 具体流程
                            第一步: 将给定的一组分区的状态全部初始化成 Errors.None, 停止为这些分区服务的所有拉取线程, 原因是
                            　　　　该 Broker 的这些分区的是 Leader 副本, 不再是 Follower 副本, 没有必要再使用拉取线程.
                            
                            第二步: 调用 Partition 的 makeLeader 方法, 去更新给定一组分区的 Leader 分区信息, makeLeader 方法
                                   保存分区的 Leader 和 ISR 信息, 同时创建必要的日志对象, 重设远端 Follower 副本(remoteReplicas)
                                   的 LEO 值. ReplicaManager 在处理 FETCH 请求时, 会更新 remoteReplicas 中副本对象的 LEO
                                    值. Leader 副本会将自己更新后的 LEO 值与 remoteReplicas 中副本的 LEO 值进行比较. 来决定是否
                                    更新高水位值
                                    
                            第三步: 如果当前 Broker 成功地成为了该分区的 Leader 副本, 就返回 True, 否则表示处理失败. 如果成功设置
                             　　　　Leader, 把该分区加入到已成功设置 Leader 的分区列表中, 并返回该列表.
                             
            (4) makeFollowers 方法
                    a. makeFollowers 方法遍历 partitionStates 中的所有分区, 然后执行“成为 Follower”的操作, 执行其他动作,
                       主要包括重建 Fetcher 线程, 完成延时请求等
                       
                    b. 流程
                            第一部分: 遍历 partitionStates 中的所有分区, 然后执行“成为 Follower”的操作, 从分区的详细信息中
                                     获取分区的 Leader Broker ID,  去 Broker 元数据缓存中找到 Leader Broker 对象； 如果 
                                     Leader 对象存在, 则执行 Partition 类的 makeFollower 方法将当前 Broker 配置成该分区的
                                     Follower 副本. 如果 makeFollower 方法执行成功, 就说明当前 Broker 被成功配置为指定分区的
                                     Follower 副本, 那么将该分区加入到结果返回集中. 如果 Leader 对象不存在, 依然创建出分区 
                                     Follower 副本的日志对象.
                                     
                                     Partition 类的 makeFollower 方法:
                                        第一步: 更新 Controller Epoch 值；
                                        第二步: 保存副本列表(Assigned Replicas, AR)和清空 ISR；
                                        第三步: 创建日志对象；
                                        第四步: 重设 Leader 副本的 Broker ID
                                        
                            第二部分: 
                                    第一步: 移除现有 Fetcher 线程, 因为 Leader 可能已经更换, 所以要读取的 Broker 以及要读取的
                                           位移值可能发生变化.
                                    第二步: 将当前 Broker 设置为 Follower 副本的分区, 确定 Leader Broker 和起始读取位移值
                                           fetchOffset, 这些信息都在 LeaderAndIsrRequest 中
                                    第三步: 使用 Leader Broker 和 fetchOffset 添加新的 Fetcher 线程.
                                    第四步: 返回将当前 Broker 设置为 Follower 副本的分区列表.
                                    
                                    
    3. ISR 管理
            (1)  maybeShrinkIsr 方法阶段性地查看 ISR 中的副本集合是否需要收缩
                 maybePropagateIsrChanges 方法定期向集群 Broker 传播 ISR 的变更
                 
            (2) maybeShrinkIsr 方法
                    a. 收缩是把 ISR 副本集合中那些与 Leader 差距过大的副本移除的过程. 差距过大, 是 ISR 中 Follower 副本滞后
                       Leader 副本的时间超过 Broker 端参数 replica.lag.time.max.ms 值的 1.5 倍. 1.5 倍 
                       replica.lag.time.max.ms 原因是 ReplicaManager 类的 startup 方法会在被调用时创建一个异步线程, 定时
                       查看是否有 ISR 需要进行收缩, 定时频率是 replicaLagTimeMaxMs 值的一半, 而判断 Follower 副本是否需要被移除
                       ISR 的条件是, 滞后程度是否超过了 replicaLagTimeMaxMs 值, 这样中间差了 1.5 的 replicaLagTimeMaxMs
                       
                    b. 流程
                            遍历该副本管理器上所有分区对象, 依次为这些分区中状态为 Online 的分区执行 Partition 类的 
                            maybeShrinkIsr 方法.
                                第一步: 判断是否需要执行 ISR 收缩, 调用 needShrinkIsr 方法来获取与 Leader 不同步的副本, 
                                       如果存在这样的副本, 说明需要执行 ISR 收缩.
                                       
                                第二步: 再次获取与 Leader 不同步的副本列表, 并把它们从当前 ISR 中剔除, 计算出最新的 ISR 列表
                                
                                第三步: 调用 shrinkIsr 方法去更新 ZooKeeper 上分区的 ISR 数据以及 Broker 上元数据缓存.
                                
                                第四步: 尝试更新 Leader 分区的高水位值. 如果 ISR 收缩后只剩下 Leader 副本一个, 那么高水位值的更新
                                       就不再受那么多限制
                                       
                                第五步: 根据上一步的结果, 来尝试解锁之前不满足条件的延迟操作
                                
            (3) maybePropagateIsrChanges 方法
                    a.  maybePropagateIsrChanges 专门负责创建 ISR 通知事件, 由一个异步线程定期完成的
                    b. 确定 ISR 变更传播的条件,需要同时满足两点, 
                            第一点: 存在尚未被传播的 ISR 变更；
                            第二点: 最近 5 秒没有任何 ISR 变更, 或者自上次 ISR 变更已经有超过 1 分钟的时间
                        
                    c. 一旦满足这两个条件, 会创建 ZooKeeper 相应的 Znode 节点, 清空 isrChangeSet 集合, 最后更新最近 ISR 变更时间戳
                    
```

### broker 元数据缓存
```shell
    1. 概念
        (1)  Broker 上的元数据缓存(MetadataCache) 是 Controller 通过 UpdateMetadataRequest 请求发送给 Broker 的, 即
             Controller 实现了一个异步更新机制, 将最新的集群信息广播给所有 Broker
             
        (2) Broker 需要保存最新的元数据的原因:
                a. Broker 就能够及时响应客户端发送的元数据请求(即处理 Metadata 请求), follwer 副本无法处理读写请求, 但是可以处理
                　　客户端的 Metadata 请求, 去获取集群的元数据信息
                b. Kafka 的一些重要组件会用到这部分数据, 如副本管理器会使用它来获取 Broker 的节点信息, 事务管理器会使用它来
                   获取分区 Leader 副本的信息等等
                   
    2. MetadataCache 类
            (1) MetadataCache 的实例化是在 KafkaServer 类的 startup 方法, 实例化成功就会被 Kafka 的 4 个组件使用, 如下:
                    a. KafkaApis： 是执行 Kafka 各类请求逻辑, 该类大量使用 MetadataCache 中的主题分区和 Broker 数据, 执行主题
                                   相关的判断与比较, 和获取 Broker 信息.
                                   
                    b. AdminManager： Kafka 定义的专门用于管理主题的管理器, 定义很多与主题相关的方法, 会用到 MetadataCache 中的
                                     主题信息和 Broker 数据, 以获取主题和 Broker 列表
                    
                    c. ReplicaManager: 需要获取主题分区和 Broker 数据, 还会更新 MetadataCache
                    
                    d. TransactionCoordinator： 管理 Kafka 事务的协调者组件, 需要用到 MetadataCache 中的主题分区的 Leader 
                                                副本所在的 Broker 数据, 向指定 Broker 发送事务标记
                                                
            (2) 定义
                    class MetadataCache(brokerId: Int) extends Logging {
                      // 保护它写入的锁对象
                      private val partitionMetadataLock = new ReentrantReadWriteLock()
                
                      // 保存了实际的元数据信息
                      @volatile private var metadataSnapshot: MetadataSnapshot = MetadataSnapshot(partitionStates = mutable.AnyRefMap.empty,
                        controllerId = None, aliveBrokers = mutable.LongMap.empty, aliveNodes = mutable.LongMap.empty)
                        
                      .......
                      }
                      
                    
                    case class MetadataSnapshot(
                                                    /*  Map 类型, Key 是主题名称, 
                                                     *  Value 是一个 Map 类型(其 Key 是分区号, 
                                                                             Value 是 UpdateMetadataPartitionState 类型的字段)
                                                    * UpdateMetadataPartitionState 类型是 UpdateMetadataRequest 请求内部所需的数据结构
                                                    partitionStates: mutable.AnyRefMap[String, mutable.LongMap[UpdateMetadataPartitionState]],
                                                    
                                                    controllerId: Option[Int], // Controller 所在 Broker 的 ID
                                                    
                                                    // 当前集群中所有存活着的 Broker 对象列表
                                                    aliveBrokers: mutable.LongMap[Broker], 
                                                    
                                                    /* 一个 Map 的 Map 类型, 其 Key 是 Broker ID 序号, 
                                                     *  Value 是 Map 类型, 其 Key 是 ListenerName(Broker 监听器类型), Value 是 Broker 节点对象
                                                    aliveNodes: mutable.LongMap[collection.Map[ListenerName, Node]]) 
                                                    
                                              
                    static public class UpdateMetadataPartitionState implements Message {
                        private String topicName;     // 主题名称
                        private int partitionIndex;   // 分区号
                        private int controllerEpoch;  // Controller Epoch 值
                        private int leader;           // Leader 副本所在 Broker ID
                        private int leaderEpoch;      // Leader Epoch 值
                        private List<Integer> isr;    // ISR 列表
                        private int zkVersion;        // ZooKeeper 节点 Stat 统计信息中的版本号
                        private List<Integer> replicas;  // 副本列表
                        private List<Integer> offlineReplicas;  // 离线副本列表
                        private List<RawTaggedField> _unknownTaggedFields; // 未知字段列表
                    }
                    
            (3) 判断类方法
                    判断给定主题或主题分区是否包含在元数据缓存中的方法.
                    
            (4) 获取类方法
                    a. getAllTopics 方法, 获取当前集群元数据缓存中的所有主题.
                    b. getAllPartitions 方法, 获取元数据缓存中的分区对象
                    c. getPartitionReplicaEndpoints 方法, 该方法接收主题分区和 ListenerName, 以获取指定监听器类型下该主题分区
                       所有副本的 Broker 节点对象, 并按照 Broker ID 进行分组. 
                       
            (5) 更新类方法
                    a. updateMetadata 方法, 对于各个 broker 来说, 取 UpdateMetadataRequest 请求中的分区数据, 然后更新本地
                       元数据缓存
                            第一部分:
                                    第一步:
                                        创建这两个字段, aliveBrokers 保存存活 Broker 对象, Key 类型是 Broker ID, 而 Value 类型
                                    是 Broker 对象. aliveNodes 保存存活节点对象, Key 类型也是 Broker ID, Value 类型是
                                    < 监听器, 节点对象 > 对
                                    
                                    第二步: 
                                        从 UpdateMetadataRequest 中获取 Controller 所在的 Broker ID, 并赋值给 
                                    controllerIdOpt 字段. 如果集群没有 Controller, 则赋值该字段为 None. 遍历 
                                    UpdateMetadataRequest 请求中的所有存活 Broker 对象, 取出它配置的所有 EndPoint 类型(即
                                    Broker 配置的所有监听器), 遍历它配置的监听器, 并将 < 监听器, Broker 节点对象 > 对保存起来,
                                    再将 Broker 加入到存活 Broker 对象集合和存活节点对象集合
                                    
                            第二部分: 确保集群 Broker 配置相同的监听器, 同时初始化已删除分区数组对象
                            
                                    第一步: 使用存活 Broker 节点对象, 获取当前 Broker 所有的 < 监听器, 节点 > 对, 拿当前 Broker
                                           配置的所有监听器, 如果发现配置的监听器与其他 Broker 有不同, 则记录一条错误日志.
                                           
                                    第二步: 构造一个已删除分区数组, 将其作为方法返回结果, 然后判断 UpdateMetadataRequest 请求
                                           是否携带任何分区信息, 如果没有则构造一个新的 MetadataSnapshot 对象, 使用之前的分区信息
                                           和新的 Broker 列表信息；如果有进入最后一个部分.
                                           
                            第三部分: 提取 UpdateMetadataRequest 请求中的数据, 然后填充元数据缓存
                            
                                    第一步: 备份现有元数据缓存中的分区数据, 到 partitionStates 的局部变量中.
                                    第二步: 获取 UpdateMetadataRequest 请求中携带的所有分区数据, 并遍历每个分区数据, 如果发现
                                           分区处于被删除的过程中, 就将分区从元数据缓存中移除, 并把分区加入到已删除分区数组中,
                                           否则就将分区加入到元数据缓存.
                                    第三步: 使用更新过的分区元数据, 和第一部分计算的存活 Broker 列表及节点列表, 构建最新的元数据
                                           缓存, 然后返回已删除分区列表数组
                                    
       
```

## 消费者组管理模块
```shell
    1. 概念
            (1) 消费者组管理组成
                    a. 消费者组元数据： 主要包括 GroupMetadata 和 MemberMetadata, 这两个类共同定义消费者组的元数据的内容
                    b. 组元数据管理器： 由 GroupMetadataManager 类定义, 提供消费者组的增删改查功能
                    c. __consumer_offsets： Kafka 的内部主题, 有消费者组提交位移记录功能, 还负责保存消费者组的注册记录消息
                    d. GroupCoordinator: 组协调者组件, 提供通用的组成员管理和位移管理
```

### 消费者组元数据
```shell
    1. 概念
            (1) 一个消费者组下可以有多个成员, 所以一个 GroupMetadata 实例会对应于多个 MemberMetadata 实例.
            
    2. MemberMetadata.scala 文件
            (1) 包含类和对象
                    a. MemberSummary 类：组成员概要数据, 提取最核心的元数据信息
                    b. MemberMetadata 伴生对象：仅仅定义一个工具方法, 供上层组件调用.
                    c. MemberMetadata 类: 消费者组成员的元数据. Kafka 为消费者组成员定义很多数据
                    
            (2) MemberSummary 类
                    
                    case class MemberSummary(  
                                                /* 成员ID, 由 Kafka 自动生成, 格式为 "consumer- 组 ID-< 序号 >- "
                                                memberId: String, 
                                                
                                                /* Consumer 端参数 group.instance.id 值, 消费者组静态成员的 ID, 静态成员机制
                                                 * 的引入能规避不必要的消费者组 Rebalance 操作.
                                                groupInstanceId: Option[String],  
                                                
                                                /* 消费者组成员配置的 client.id 参数, 由于 memberId 不能被设置, 
                                                 * 可以用这个字段来区分消费者组下的不同成员
                                                clientId: String,  
                                                
                                                /* 运行消费者程序的主机名, 记录这个客户端是从哪台机器发出的消费请求
                                                clientHost: String,  
                                                
                                                /* 消费者组成员使用的分配策略, 标识消费者组成员分区分配策略的字节数组, 由
                                                 * 消费者端参数 partition.assignment.strategy 值设定, 默认是
                                                 * RangeAssignor 策略, 即按照主题平均分配分区
                                                metadata: Array[Byte], 
                                                
                                                /* 成员订阅分区, 保存分配给该成员的订阅分区. 每个消费者组都要选出一个 Leader 
                                                 * 消费者组成员, 负责给所有成员分配消费方案. Kafka 再将制定好的分配方案序列化成
                                                 * 字节数组, 赋值给 assignment, 分发给各个成员
                                                assignment: Array[Byte] 
                                           ) 
                                           
            (3) MemberMetadata 伴生对象
                    从一组给定的分区分配策略详情中提取出分区分配策略的名称, 用来统计一个消费者组下的成员到底配置多少种分区分配策略.
                    
            (4) MemberMetadata 类(组成员元数据类)
                    
                    private[group] class MemberMetadata
                    (
                        var memberId: String,
                        val groupId: String,
                        val groupInstanceId: Option[String],
                        val clientId: String,
                        val clientHost: String,
                        
                        /* Rebalane 操作超时时间, 即一次 Rebalance 操作必须在这个时间内
                         * 完成, 否则被视为超时, Consumer 端参数 max.poll.interval.ms 的值
                        val rebalanceTimeoutMs: Int, 
                        
                         /* 会话超时时间, 当前消费者组成员依靠心跳机制“保活”, 如果在会话
                          * 超时时间之内未能成功发送心跳, 组成员就被判定成“下线”,
                          * 从而触发新一轮的 Rebalance, Consumer 端参数 session.timeout.ms 值
                        val sessionTimeoutMs: Int,
                        
                        /* 标识的是消费者组被用在哪个场景, "consumer" 作为普通的消费者组使用, 
                         * "connect"供 Kafka Connect 组件中的消费者使用
                        val protocolType: String, 
                        
                        /* 成员配置的多组分区分配策略, Consumer 端参数 partition.assignment.strategy 的类型是 List, 
                         * 可以为消费者组成员设置多组分配策略, 这个字段也是一个 List 类型, 每个元素是一个元组(Tuple).元组的
                         * 第一个元素是策略名称, 第二个元素是序列化后的策略详情
                        var supportedProtocols: List[(String, Array[Byte])]  
                    )
                    
                    扩展字段:
                        assignment: 保存分配给该成员的分区分配方案
                        awaitingJoinCallback：表示组成员是否正在等待加入组
                        awaitingSyncCallback：表示组成员是否正在等待 GroupCoordinator 发送分配方案
                        isLeaving：表示组成员是否发起“退出组”的操作
                        isNew：表示是否是消费者组下的新成员
                        
            (5) GroupMetadata 类(组元素类)
                    a. GroupMetadata 管理的是消费者组而不是消费者组成员级别的元数据
                    b. GroupMetadata.scala 文件由 6 部分构成
                    
                            GroupState 类：定义消费者组的状态空间.有 5 个状态, 分别是 Empty, PreparingRebalance,
                                           CompletingRebalance, Stable 和 Dead. 
                                                Empty 表示当前无成员的消费者组；
                                                PreparingRebalance 表示正在执行加入组操作的消费者组；
                                                CompletingRebalance 表示等待 Leader 成员制定分配方案的消费者组；
                                                Stable 表示已完成 Rebalance 操作可正常工作的消费者组；
                                                Dead 表示当前无成员且元数据信息被删除的消费者组
                                                
                                           一个消费者组从创建到正常工作, 它的状态流转路径是 Empty -> PreparingRebalance -> 
                                           CompletingRebalance -> Stable
                                                
                            GroupMetadata 类：组元数据
                            GroupOverview 类：定义非常简略的消费者组概览信息
                            GroupSummary 类：与 MemberSummary 类类似, 定义消费者组的概要信息
                            CommitRecordMetadataAndOffset 类：保存写入到位移主题中的消息的位移值, 以及其他元数据信息, 
                                                             主要就是保存位移值
                                                             
                    c. GroupOverview 类
                            在命令行运行 kafka-consumer-groups.sh --list 时, Kafka 就会创建 GroupOverview 实例返回给命令行
                            
                            case class GroupOverview
                            (
                                groupId: String, // 组 ID 信息, 即 group.id 参数值
                                protocolType: String, // 消费者组的协议类型
                                state: String // 消费者组的状态
                            )
                            
                    d. GroupSummary 类
                    
                            case class GroupSummary
                            (
                                state: String, // 消费者组状态
                                protocolType: String, // 协议类型
                                protocol: String, // 消费者组选定的分区分配策略
                                members: List[MemberSummary] // 成员元数据
                            ) 
                            
                    e. GroupMetadata 类
                                                        
                            private[group] class GroupMetadata(val groupId: String,  // 组ID
                                                               initialState: GroupState, // 消费者组初始状态
                                                               time: Time) extends Logging 
                           {
                              type JoinCallback = JoinGroupResult => Unit
                             
                              private[group] val lock = new ReentrantLock
                              // 定义了消费者组的状态空间,当前有 5 个状态，
                              private var state: GroupState = initialState
                              
                              /* 记录状态最近一次变更的时间戳, 用于确定位移主题的过期消息, 位移主题中的消息也要遵循 Kafka 的留存策略,
                               * 所有当前时间与该字段的差值超过了留存阈值的消息都被视为“已过期”（Expired）
                              var currentStateTimestamp: Option[Long] = Some(time.milliseconds())
                              
                              var protocolType: Option[String] = None
                              
                              /* 消费组 Generation Id, Generation 等同于消费者组执行过 Rebalance 操作的次数, 
                               * 每次执行 Rebalance , GenerationId 加 1
                              var generationId = 0
                              
                              /* 记录消费者组的 Leader 成员, 可能不存在, 消费者组中 Leader 成员的 Member ID 信息, 当消费者组
                               * 执行 Rebalance , 需要选举一个成员作为 Leader, 负责为所有成员制定分区分配方案. Rebalance 早期阶段
                               * 这个 Leader 可能尚未被选举出来. 这就是 leaderId 字段是 Option 类型的原因
                              private var leaderId: Option[String] = None
                              private var protocol: Option[String] = None
                              
                              // 成员元数据列表信息
                              private val members = new mutable.HashMap[String, MemberMetadata]
                              
                              // Static membership mapping [key: group.instance.id, value: member.id]
                              // 静态成员Id列表
                              private val staticMembers = new mutable.HashMap[String, String]
                              private val pendingMembers = new mutable.HashSet[String]
                              private var numMembersAwaitingJoin = 0
                              
                              /* 分区分配策略支持票数, 是一个 HashMap 类型, Key 是分配策略的名称, Value 是支持的票数
                                每个成员可以选择多个分区分配策略, 如成员 A 选择[“range”, “round-robin”], B 选择 [“range”],
                                C 选择[“round-robin”, “sticky”], 那么这个字段就有 3 项, 分别是：<“range”, 2>, 
                                <“round-robin”, 2> 和 <“sticky”, 1>
                              private val supportedProtocols = new mutable.HashMap[String, Integer]().withDefaultValue(0)
                              
                              /* 保存按照主题分区分组的位移主题消息位移值的 HashMap, Key 是主题分区, 
                               * Value 是 CommitRecordMetadataAndOffset 类型(封装了位移提交消息的位移值), 
                               * 当消费者组成员向 Kafka 提交位移时, 会向这个字段插入对应的记录
                              private val offsets = new mutable.HashMap[TopicPartition, CommitRecordMetadataAndOffset]
                              
                              private val pendingOffsetCommits = new mutable.HashMap[TopicPartition, OffsetAndMetadata]
                              private val pendingTransactionalOffsetCommits = new mutable.HashMap[Long, mutable.Map[TopicPartition, CommitRecordMetadataAndOffset]]()
                              private var receivedTransactionalOffsetCommits = false
                              private var receivedConsumerOffsetCommits = false
                             
                              // 消费者组订阅的主题列表, 用于帮助从 offsets 字段中过滤订阅主题分区的位移值
                              private var subscribedTopics: Option[Set[String]] = None
                             
                              var newMemberAdded: Boolean = false
                            }
                    
```

### 消费者组元数据管理
```shell
    1. 概念
    2. 消费者组状态管理
            (1) transitionTo 方法
                    将消费者组状态变更成给定状态, 确保变更必须是合法的状态转换, 还会更新状态变更的时间戳字段.
                Kafka 有个定时任务, 会定期清除过期的消费者组位移数据, 就是依靠这个时间戳字段, 来判断过期与否.
                
            (2) canRebalance 方法
                    判断消费者组是否可以 Rebalance 操作, 判断依据是当前状态是否是 PreparingRebalance 状态的合法前置状态,, 只有
                Stable, CompletingRebalance 和 Empty 这 3 类状态的消费者组, 才有资格开启 Rebalance
                
            (3) is 和 not 方法
                    分别判断消费者组的状态与给定状态符不符合, 主要被用于执行状态校验. is 方法被大量用于上层调用代码, 用于执行各类
               消费者组管理任务的前置状态校验工作.
               
    3. 成员管理
            (1) 添加成员(add)
                    将成员对象添加到 members 字段, 同时更新其他一些必要的元数据, 如 Leader 成员字段、分区分配策略支持票数等.
                    
                 第一步: 判断 members 字段是否包含已有成员. 如果没有, 说明要添加的成员是该消费者组的第一个成员, 那么令该成员
                        协议类型(protocolType)成为组的 protocolType.
                 
                 第二步: 连续进行三次校验, 分别确保待添加成员的组 ID, protocolType 和组配置一致, 以及该成员选定的分区分配策略与
                        组选定的分区分配策略相匹配. 如果这些校验有任何一个未通过, 就会立即抛出异常
                        
                 第三步: 判断消费者组的 Leader 成员是否已经选出. 如果还没有选出, 就将该成员设置成 Leader 成员.Leader 该成员负责为
                        消费者所有成员制定分区分配方案
                        
                 第四步: 更新消费者组分区分配策略支持票数, 设置成员加入组后的回调逻辑, 更新已加入组的成员数.
                 
            (2) 移除成员(remove)
                     从 members 中移除给定成员, 更新分区分配策略支持票数, 更新已加入组的成员数, 最后判断该成员是否是 Leader 成员,
                 如果是就选择成员列表中尚存的第一个成员作为新的 Leader 成员.
             
            (3) 查询成员
                    has 方法: 判断消费者组是否包含指定成员；
                    get 方法: 获取指定成员对象；
                    size 方法: 统计总成员数
                    
    4. 位移管理
            (1) 管理消费者组的提交位移(Committed Offsets), 主要包括添加和移除位移值.
            (2) 提交位移(消费者组需要向 Coordinator 提交已消费消息的进度), 提交位移在 Coordinator 端保存在内部位移主题中.
            (3) 位移提交消息(Commit Record) 是消费者组成员向内部主题写入符合特定格式的事件消息
            (4) 添加位移值
                    a. initializeOffsets 方法
                            将给定的一组订阅分区提交位移值加到 offsets 字段中, 实现方法: 当消费者组的协调者组件启动时, 
                       会创建一个异步任务, 定期地读取位移主题中相应消费者组的提交位移数据, 并把它们加载到 offsets 字段中
                       
                    b. onOffsetCommitAppend 方法
                            提交位移消息被成功写入后调用该方法, 主要判断的依据是 offsets 字段中是否已包含该主题分区对应的消息值, 
                            或者 offsets 字段中该分区对应的提交位移消息, 在位移主题中的位移值是否小于待写入的位移值, 如果是的话,
                            就把该主题已提交的位移值添加到 offsets 中
                            
                    c. completePendingTxnOffsetCommit 方法
                            完成一个待决事务(Pending Transaction)的位移提交, 待决事务是指正在进行中, 还没有完成的事务.在处理待决
                       事务的过程中, 可能会出现将待决事务中涉及到的分区的位移值添加到 offsets 中的情况
                       
            (5) 移除位移值
                    a. offsets 中订阅分区的已消费位移值也是可以被移除的.如果当前时间与已提交位移消息时间戳的差值, 超过
                       Broker 端参数 offsets.retention.minutes 值, 会将这条记录从 offsets 字段中移除(removeExpiredOffsets 方法)
                       
                    b. getExpiredOffsets 方法
                            从 offsets 字段中过滤出同时满足 3 个条件的所有分区, 
                            第一点: 分区所属主题不在订阅主题列表之内, 当方法传入不为空的主题集合时, 说明该消费者组正在消费中, 
                                   正在消费的主题是不能执行过期位移移除. 
                            第二点: 主题分区已经完成位移提交, 处于提交中状态, 即保存在 pendingOffsetCommits 字段中的分区也是
                                    不能移除的. 
                            第三点:  该主题分区在位移主题中对应消息的存在时间超过阈值, 老版本的 Kafka 消息直接指定过期时间戳, 
                                    只需要判断当前时间是否越过这个过期时间,  新版 Kafka 判断过期与否, 主要是基于消费者组状态, 
                                    如果是 Empty 状态, 过期的判断依据 就是当前时间与组变为 Empty 状态时间的差值, 是否超过 
                                    Broker 端参数 offsets.retention.minutes 值； 如果不是 Empty 状态, 就看当前时间与提交位移
                                    消息中的时间戳差值是否超过了 offsets.retention.minutes 值, 如果超过为已过期, 对应的位移值
                                    需要被移除；如果没有超过, 就不需要移除.
                                    
                            当过滤出同时满足这 3 个条件的分区后, 提取出它们对应的位移值对象并返回.
                            
                    c. removeExpiredOffsets 方法
                            如果消费者组状态是 Empty, 就传入组变更为 Empty 状态的时间, 若该时间没有被记录, 则使用提交位移消息
                            本身的写入时间戳, 来获取过期位移；
                            
                            如果是普通的消费者组类型, 且订阅主题信息已知, 就传入提交位移消息本身的写入时间戳和订阅主题集合共同
                            确定过期位移值；
                            
                            如果 protocolType 为 None, 表示这个消费者组其实是一个 Standalone 消费者, 传入提交位移消息本身的
                            写入时间戳来决定过期位移值；
                            
                            
    5. 分区分配策略管理
            (1)  supportedProtocols 字段的管理, supportedProtocols 字段是分区分配策略的支持票数, 这个票数在添加成员, 移除成员
                       会进行相应的更新. 消费者组每次 Rebalance 后, 要使用哪个分区分配策略, 需要特定的方法来对这些票数进行统计, 
                       把票数最多的那个策略作为新的策略.
                       
            (2) candidateProtocols 方法
                            找出组内所有成员都支持的分区分配策略集, 先获取组内的总成员数, 找出 supportedProtocols 中那些支持票数等
                       于总成员数的分配策略, 并返回它们的名称.支持票数等于总成员数等同于所有成员都支持该策略.
                        
            (3) selectProtocol 方法
                            选出消费者组的分区消费分配策略, 实现方法会判断组内是否有成员, 如果没有任何成员, 无法确定选用哪个策略方法
                       就会抛出异常并退出. 否则调用 candidateProtocols 方法, 获取所有成员都支持的策略集合, 然后让每个成员投票,
                       票数最多的那个策略当选.
                         
```

### 组元数据管理器(组管理器)
```shell
    1. 概念
        (1) 提供管理消费者组: 添加消费者组, 移除组, 查询组这样组级别的基础功能, 以及对内部位移主题的操作方法
        (2) 每个 Broker 在启动时都会创建并维持一个 GroupMetadataManager 实例, 实现管理该 Broker 负责的消费者组.
        (2) 每个消费者组及其位移的数据, 都只会保存在位移主题的一个分区下
    2. 定义
            class GroupMetadataManager(
                                        val brokerId: Int, // 所在 Broker 的 Id
                                        
                                        /* 保存 Broker 间通讯使用的请求版本, Broker 端参数 inter.broker.protocol.version 值,
                                           用来确定位移主题消息格式的版本
                                        interBrokerProtocolVersion: ApiVersion,  
                                        
                                        /* 内部位移主题配置类, 定义与位移管理相关的重要参数, 比如位移主题日志段大小设置,
                                         * 位移主题备份因子, 位移主题分区数配置等
                                        val config: OffsetConfig,  
                                        
                                       /* 副本管理器类, GroupMetadataManager 类使用该字段实现获取分区对象, 日志对象以及
                                        * 写入分区消息
                                       replicaManager: ReplicaManager,
                                       
                                       zkClient: KafkaZkClient,  // ZooKeeper 客户端, 从 ZooKeeper 中获取位移主题的分区数
                                       time: Time) extends Logging with KafkaMetricsGroup {
                                        .....
                                        }
                                        
            其他字段:
                    1. compressionType
                            压缩器类型, Kafka 向位移主题写入消息前, 可以选择对消息执行压缩操作, 是否压缩取决于 Broker 端参数 
                       offsets.topic.compression.codec 值, 默认是不压缩。如果位移主题占用的磁盘空间比较多. 可以考虑启用压缩节省资源
                       
                    2. groupMetadataCache(重要)
                            保存这个 Broker 上 GroupCoordinator 组件管理的所有消费者组元数据, Key 是消费者组名称, Value 是消费者
                       组元数据(GroupMetadata).通过该字段实现对消费者组的添加, 删除和遍历操作
                       
                    3. loadingPartitions
                            位移主题下正在执行加载操作的分区号集合, 这些分区都是位移主题分区, 即 __consumer_offsets 主题下的分区, 
                       加载指读取位移主题消息数据, 填充 GroupMetadataCache 字段的操作.
                       
                    4. ownedPartitions
                            位移主题下完成加载操作的分区号集合, 与 loadingPartitions 类似的, 该字段保存的分区也是位移主题下的分区.
                       但它保存的分区都是已经完成加载操作的分区
                       
                    5. groupMetadataTopicPartitionCount
                            位移主题的分区数, 是 Broker 端参数 offsets.topic.num.partitions 的值, 默认是 50 个分区, 若要
                       修改分区数, 除了变更该参数值之外, 也可以手动创建位移主题, 并指定不同的分区数.
                       
    3. 消费者组元数据管理
            (1) 查询获取消费者组元数据
                    a.  getGroup 方法, 如果该组信息不存在, 就返回 None. 可能该消费者组确实不存在, 或是, 该组对应的 Coordinator 
                                      组件变更到其他 Broker 上.
                                   
                    b. getOrMaybeCreateGroup 方法, 若组信息不存在, 就根据 createIfNotExist 参数值决定是否需要添加该消费者组.
                       该方法是在消费者组第一个成员加入组时被调用的, 用于把组创建出来.
                       
            (2) 移除消费者组元数据
                    当 Broker 卸任某些消费者组的 Coordinator 角色时, 需要将这些消费者组从 groupMetadataCache 中全部移除掉, 这就是
                    removeGroupsForPartition 方法要做的事情.
                    
                 removeGroupsForPartition 方法
                    异步任务从 ownedPartitions 中移除给定位移主题分区, 遍历消费者组元数据缓存中的所有消费者组对象, 如果消费者组正是
                    在给定位移主题分区下保存的, 则进行 removeGroupsAndOffsets 方法.
                    
                 removeGroupsAndOffsets 方法
                    第一步, 调用 onGroupUnloaded 方法执行组卸载逻辑, 上层组件 GroupCoordinator 传过来, 将消费者组状态变更到
                           Dead 状态, 封装异常表示 Coordinator 已发生变更, 然后调用回调函数返回
                    第二步, 把消费者组信息从 groupMetadataCache 中移除
                    第三步, 把消费者组从 producer 对应的组集合中移除, 给 Kafka 事务用
                    第四步, 增加已移除组计数器
                    第五步, 更新已移除位移值计数
                    
            (3) 添加消费者组元数据
                    给定组添加进 groupMetadataCache 中
                  
            (4) 加载消费者组元数据
                    第一步: 通过 initializeOffsets 方法, 将位移值添加到 offsets 字段标识的消费者组提交位移元数据中, 实现加载
                           消费者组订阅分区提交位移的目的.
                    第二步: 调用 addGroup 方法, 将该消费者组元数据对象添加进消费者组元数据缓存, 实现加载消费者组元数据的目的
                    
    4. 消费者组位移管理
            (1) 当消费者组程序在查询位移时, Kafka 总是从内存中的位移缓存数据查询, 而不会直接读取底层的位移主题数据
            (2) 保存消费者组位移
                    a. 保存消费者组位移的全部流程: 过滤出满足特定条件的待保存位移信息, 特定条件(validateOffsetMetadataLength 方法),
                       是指位移提交记录的自定义数据大小,要小于 Broker 端参数 offset.metadata.max.bytes 的值(默认值是 4KB)
                       如果没有一个分区满足条件, 就构造 OFFSET_METADATA_TOO_LARGE 异常, 并调用回调函数(执行发送位移提交 Response
                       的动作). 如果有分区满足, 判断当前 Broker 是不是该消费者组的 Coordinator, 如果不是就构造 NOT_COORDINATOR 
                       异常, 并提交给回调函数； 如果是就构造位移主题消息, 并将消息写入进位移主题下, 调用一个 putCacheCallback 
                       内置方法, 填充 groupMetadataCache 中各个消费者组元数据中的位移值, 最后调用回调函数返回.
                       
                    b. 定义
                           
                            def storeOffsets(
                                // group：消费者组元数据
                                group: GroupMetadata,
                                
                                // consumerId：消费者组成员ID
                                consumerId: String,
                                
                                 // offsetMetadata：待保存的位移值, 按照分区分组
                                offsetMetadata: immutable.Map[TopicPartition, OffsetAndMetadata],
                                
                                // responseCallback：处理完成后的回调函数
                                responseCallback: immutable.Map[TopicPartition, Errors] => Unit,
                                
                                 // producerId：事务型Producer ID, 与 Kafka 事务相关
                                producerId: Long = RecordBatch.NO_PRODUCER_ID,
                                
                                // producerEpoch：事务型Producer Epoch值, 与 Kafka 事务相关
                                producerEpoch: Short = RecordBatch.NO_PRODUCER_EPOCH): Unit = {
                            ......
                            }
                            
                    c. putCacheCallback 方法
                            将多个消费者组位移值填充到 GroupMetadata 的 offsets 元数据缓存中.
                            第一步: 确保位移消息写入到指定位移主题分区, 否则就抛出异常
                            第二步: 更新已提交位移数指标, 判断写入结果是否有错误, 如果没有错误, 只要组状态不是 Dead 状态, 
                                   就调用 GroupMetadata 的 onOffsetCommitAppend 方法填充元数据, onOffsetCommitAppend 是将
                                   消费者组订阅分区的位移值写入到 offsets 字段保存的集合中.
                           第三步: 如果写入结果有错误, 通过 failPendingOffsetWrite 方法取消未完成的位移消息写入, 进行异常操作.
                           
            (3) 查询消费者组位移
                    a. 读取 groupMetadataCache 中的组元数据(在缓存中), 如果不存在对应的记录, 则返回空数据集, 如果存在, 则判断组是否
                       处于 Dead 状态.  如果是 Dead 状态, 说明消费者组已经被销毁, 位移数据不可用, 返回空数据集；
                       若状态不是 Dead, 就提取出消费者组订阅的分区信息, 依次为它们获取对应的位移数据并返回.
                            
```

### 位移主题组成(GroupMetadataManager 中的部分)
```shell
    1. 概念
        (1) 位移主题目的是, 保存消费者组的注册消息和提交位移消息, 注册消息能够标识消费者组的身份信息, 提交位移消息保存消费者组消费的
            进度信息
            
        (2) 通过 kafka-console-consumer 命令行查看位移主题中的位移消息, 需要指定参数 
               --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter"
               
        (3) 对位移主题进行读写的前提就是要能找到正确的 Coordinator.
        (4) 消息类型
                a. 位移主题有两类消息, 消费者组注册消息(包含消费者组名称以及成员分区消费分配方案)和消费者组的已提交位移消息.
                b. GroupTopicPartition 类封装 < 消费者组名, 主题, 分区号 > 的三元组.
        (5) 消费者组必须要先向 Coordinator 组件注册, 才能提交位移.
        
    2. 注册消息
        (0) 概念
                a. 注册消息的 Key 就是(格式版本 + 消费者组名) 
                b. 注册消息的 value, 格式版本 + 协议类型 + Generation ID + 分区分配策略 +  Leader 成员 ID + 
                                    最近一次状态变更时间(可选) + 所有成员组信息
                                     
                                     所有成员组信息: 成员的 ID + Client ID + 主机名 + 会话超时时间信息 + Rebalance 超时时间(可选)
                                                    + Group Instance ID(可选) + 订阅信息 + 成员消费分配信息
        (1) 发送的时机(消费者组向位移主题写入注册类的消息)
                a. 所有成员都加入组后: Coordinator 向位移主题写入注册消息, 只是该消息不含分区消费分配方案；
                b. Leader 成员发送方案给 Coordinator 后: 当 Leader 成员将分区消费分配方案发给 Coordinator 后, Coordinator 写入
                                                      带分配方案的注册消息.
                                                      
        (2) GroupMetadataManager 中的 groupMetadataKey 方法
                负责将注册消息的 Key 转换成字节数组, 用于后面构造注册消息
                
        (3)  groupMetadataValue 方法
                将消费者组重要的元数据写入到字节数组.
                第一步: 根据传入的 apiVersion 字段, 确定要使用哪个格式版本, 并创建对应版本的结构体来保存这些元数据, apiVersion 的
                       取值是 Broker 端参数 inter.broker.protocol.version 的值
                       
                第二步: 依次向结构体写入消费者组的协议类型(Protocol Type), Generation ID, 分区分配策略(Protocol Name) 和 
                       Leader 成员 ID, 对于普通的消费者组而言, 协议类型就是"consumer"字符串, 分区分配策略可能是"range"
                       "round-robin"等. 格式版本 ≥2 的结构体, 还会写入消费者组状态最近一次变更的时间戳.
                       
                第三步: 遍历消费者组的所有成员, 为每个成员构建专属的结构体对象, 并依次向结构体写入成员的 ID, Client ID, 主机名以及
                       会话超时时间信息. 对于格式版本 ≥0 的结构体, 要写入成员配置的 Rebalance 超时时间, 对于格式版本 ≥3 的结构体,
                       还要写入用于静态消费者组管理的 Group Instance ID. groupMetadataValue 方法必须要确保消费者组选出分区
                       分配策略, 否则就抛出异常, 再依次写入成员消费订阅信息和成员消费分配信息.
                       
                第四步: 向 Buffer 依次写入版本信息和元数据信息, 并返回 Buffer 底层的字节数组
                
    3. 已提交位移信息
        (0) 概念
                a. 提交位移信息的 key 就是(格式版本 + 消费者组名 + 主题 + 分区号)
                b. 提交位移信息的 value 是(消息格式版本 + 位移值 + Leader Epoch 值 + 自定义元数据 + 时间戳)
                
        (1) offsetCommitKey 方法
                负责将建一个结构体对象, 依次写入消费者组名, 主题和分区号. 再构造 ByteBuffer, 写入格式版本和结构体, 最后返回
                底层的字节数组.
                
        (2) offsetCommitValue 方法
                确定消息格式版本以及创建对应的结构体对象, 最新版本 V3 , 结构体的元素包括位移值, Leader Epoch 值, 自定义元数据和时间戳.
                再构建 ByteBuffer, 写入消息格式版本和结构体, 返回 ByteBuffer 底层字节数组.
                
    4. Tombstone 消息
        (1)  Tombstone 消息 是 Value 为 null 的消息, 注册消息和提交位移消息都有对应的 Tombstone 消息, 作用是让 Kafka 识别
             哪些 Key 对应的消息是可以被删除的, 这样内部位移主题不会持续增加磁盘占用空间.
             
    5. Coordinator 组件
        (1) 概念
                a. Coordinator 组件是操作位移主题的唯一组件, 在内部对位移主题进行读写操作.
                
        (2) 每个 Broker 在启动时, 都会启动 Coordinator 组件, 但一个消费者组只能对于唯一一个 Coordinator 组件, 位移主题该消费者组
            对应分区 Leader 副本所在的 Broker 被选定为指定消费者组的 Coordinator. 确认消费者组对应于哪一个分区号, 则通过消费者组名
            哈希值与位移主题分区数求模的绝对值结果. 实例如下:
                位移主题默认是 50 个分区, 消费者组名是“testgroup”, Math.abs(“testgroup”.hashCode % 50) 的结果是 27, 那么目标
                分区号就是 27. 这个消费者组的注册消息和提交位移消息都会写入到位移主题的分区 27 中, 而分区 27 的 Leader 副本所在的
                 Broker, 就成为该消费者组的 Coordinator.
```

### 位移主题读写
```shell
    1. 概念
            a. GroupMetadataManager 类还是连接 GroupMetadata 和 Coordinator 的重要纽带, Coordinator 利用 
               GroupMetadataManager 类实现操作 GroupMetadata.
               
            b. GroupMetadata 是组元数据信息, 而 GroupMetadataManager 类是管理组元数据
    2. 向位移主题写入消息
             (1) storeGroup 方法
                    向 Coordinator 注册消费者组
                    
                    第一步: 调用 getMagic 方法, 判断当前 Broker 是否是该消费者组的 Coordinator 组件, 判断的依据是尝试去获取
                           位移主题目标分区的底层日志对象, 如果能够获取到, 就说明当前 Broker 是 Coordinator, 否则构造一个
                            NOT_COORDINATOR 异常返回
                       
                    第二步: 调用 groupMetadataKey 和 groupMetadataValue 方法, 去构造注册消息的 Key 和 Value 字段
                    
                    第三步: 使用 Key 和 Value 构建待写入消息集合(保存在内存中)
                    
                    第四步: 调用 partitionFor 方法, 计算要写入的位移主题目标分区
                    
                    第五步: 调用 appendForGroup 方法, 将待写入消息插入到位移主题的目标分区下
                    
    3. 读取位移主题
            (1) 对于 Coordinator 读取位移数据时, 会从 GroupMetadata 元数据缓存中查找对应的位移值, 而不会真正读取位移主题, 只有当
                Broker 当选 Coordinator 时, 即 Broker 成为位移主题某分区的 Leader 副本时, 才会读取位移主题填充自己的 
                GroupMetadata 元数据.
                
            (2) doLoadGroupsAndOffsets 方法主要就是加载消费者组, 加载消费者组的位移.
            (3) logEndOffset 方法: 获取位移主题指定分区的 LEO 值, 如果当前 Broker 不是该分区的 Leader 副本, 则返回 -1.  
                Kafka 依靠 logEndOffset 方法来判断分区的 Leader 副本是否发生变更, 如果发生变更, 那么在当前 Broker 执行
                logEndOffset 方法的返回值为 -1.
                
            (4) doLoadGroupsAndOffsets 方法会 (初始化 4 个列表 + 读取位移主题), (处理读到的数据, 并填充 4 个列表), 
                (分别处理这 4 个列表) 这 3 个部分.
                
                第一部分:
                        第一步: 创建 4 个列表, loadedOffsets(已完成位移值加载的分区列表), pendingOffsets(位移值加载中的分区列表),
                               loadedGroups(已完成组信息加载的消费者组列表), removedGroups(待移除的消费者组列表)
                               
                        第二步: 创建 ByteBuffer 缓冲区, 用于保存消息集合, 计算位移主题目标分区的日志起始位移值, 是要读取的起始位置,
                               定义一个布尔类型的变量, 该变量表示本次至少要读取一条消息
                               
                        第三步: 需要同时满足 读取位移值小于日志 LEO 值, 布尔变量值是 True, GroupMetadataManager 未关闭三个条件,才
                               进行 while 循环, 
                               
                               读取位移主题目标分区的日志对象, 从日志中取出真实的消息数据, 当读取到完整的日志之
                               后, doLoadGroupsAndOffsets 方法会查看返回的消息集合, 如果一条消息都没有返回,  则取消
                               把刚才的布尔变量值设置为 False.
                               
                               创建保存在内存中的消息集合对象(MemoryRecords),
                               
                第二部分:
                        依然是在 while 循环中.
                        第一步: 遍历该消息批次下的所有消息, 先记录消息批次中第一条消息的位移值.
                        第二步: 读取消息 Key, 并判断 Key 的类型. 
                        
                               如果是提交位移消息, 就判断消息有无 Value. 如果没有 value, 则将目标分区从已完成位移值加载的分区列表中
                               移除. 如果有 value, 则将目标分区加入到已完成位移值加载的分区列表中.
                                
                               如果是注册消息, 判断消息有无 Value. 如果存在 Value, 就把该消费者组从待移除消费者组列表中移除, 
                               并加入到已完成加载的消费组列表；如果不存在 Value, 就是一条 Tombstone 消息. 那么把该消费者组从
                               已完成加载的组列表中移除, 并加入到待移除消费组列表.
                               
                               如果是未知类型的 Key, 就直接抛出异常
                               
                        第三步: 更新读取位置(整个消息批次中最后一条消息的位移值 +1), 等待下次 while 循环
                        
                第三部分:
                        第一步: 对 loadedOffsets 进行分组, 将那些已经完成组加载的消费者组位移值分到一组, 保存在字段 groupOffsets 中
                               将有位移值但没有对应组信息的分成另外一组, 将数据保存在字段 emptyGroupOffsets 中.
                               
                        第二步: 为 loadedGroups 中的所有消费者组执行加载组操作, 以及加载之后的操作 onGroupLoaded. onGroupLoaded
                               方法是上层调用组件 Coordinator 传入的, 作用是处理消费者组下所有成员的心跳超时设置, 并指定下一次心跳
                               的超时时间
                               
                        第三步: 为 emptyGroupOffsets 的所有消费者组, 创建空的消费者组元数据, 然后执行和上一步相同的组加载逻辑以及
                               加载后的逻辑
                               
                        第四步: 检查 removedGroups 中的所有消费者组, 确保它们不能出现在消费者组元数据缓存中, 否则将抛出异常
                        
            (5) 通过 doLoadGroupsAndOffsets 方法,  Coordinator 成功地读取位移主题目标分区下的数据, 并把它们填充到消费者组元数
                据缓存中
                               
```

### Rebalance 的流程 - 加入组
```shell
    1. 概念
            a. 决定单次 Rebalance 所用最大时长的参数是 Consumer 端的 max.poll.interval.ms.
            b. Consumer 端参数 session.timeout.ms 判断向 Coordinator 心跳超时时间.
            c. 加入组(JoinGroup), 是指消费者组下的各个成员向 Coordinator 发送 JoinGroupRequest 请求加入进组的过程, 这个过程有
               一个超时时间, 如果有成员在超时时间之内, 无法完成加入组操作, 就不参加这轮 Rebalance
            d. handleJoinGroup 是执行加入组的顶层方法, 被 KafkaApis 类调用, 该方法根据给定消费者组成员是否设置成员 ID 来决定是
               调用 doUnknownJoinGroup 还是 doJoinGroup, 前者对应于未设定成员 ID 的情形, 后者对应于已设定成员 ID 的情形. 而
               这两个方法都会调用 addMemberAndRebalance, 执行真正的加入组逻辑.
               
    2. handleJoinGroup 方法(Coordinator 端处理成员加入组申请的方法)
            (1) 定义
                    def handleJoinGroup
                    (
                        groupId: String, // 消费者组名
                        memberId: String, // 消费者组成员ID, 如果成员是新加入的, 该字段是空字符串
                        
                        /* 组实例ID, 用于标识静态成员(2.4 版本引入), 可以避免因系统升级或程序更新而导致的 Rebalance 场景.
                        groupInstanceId: Option[String], 
                        
                        /* 是否要求成员必须设置 ID, 如果为 True, Kafka 要求消费者组成员必须设置 ID, 未设置 ID 的成员会被拒绝加入组,
                         * 直到它设置 ID 后才能重新加入组.
                        requireKnownMemberId: Boolean, 
                        clientId: String, // 消费者端参数 client.id 值, Coordinator 使用它来生成 memberId
                        clientHost: String, // 消费者程序主机名
                        rebalanceTimeoutMs: Int, // Rebalance 超时时间, 默认是 max.poll.interval.ms 值
                        
                        /* 会话超时时间, 如果消费者组成员无法在这段时间内向 Coordinator 上报心跳, 会超时进行新一轮 Rebalance
                        sessionTimeoutMs: Int, 
                        protocolType: String, // 协议类型
                        protocols: List[(String, Array[Byte])], // 按照分配策略分组的订阅分区
                        
                        /* 回调函数, 完成加入组之后的回调逻辑方法, 当消费者组成员成功加入组后, 需要执行该方法
                        responseCallback: JoinCallback 
                    ): Unit = { ...... }
                    
            (2) 流程
                    第一步: 调用 validateGroupStatus 方法验证消费者组状态的合法性, 即消费者组名 groupId 不能为空, 以及 
                           JoinGroupRequest 请求发送给正确的 Coordinator, 这两个必须同时满足. 否则会封装相应的错误, 并调用
                           回调函数返回
                           
                    第二步: 检验 sessionTimeoutMs 的值是否介于[group.min.session.timeout.ms, group.max.session.timeout.ms]
                           之间, 如果不是封装一个对应的异常调用回调函数返回, 这两个参数分别表示消费者组允许配置的最小和最大会话
                           超时时间.
                           
                    第三步: 获取当前成员的 ID 信息, 并看它是否为空. 通过 GroupMetadataManager 获取消费者组的元数据信息, 如果该组的 
                           元数据信息不存在, 看当前成员 ID 是否为空, 如果为空, 就创建一个空的元数据对象. 如果成员 ID 不为空, 
                           则会封装 “未知成员 ID”的异常, 调用回调函数返回.
                           
                    第四步: 检查当前消费者组是否已满员, 通过 acceptJoiningMember 方法, 该方法根据消费者组状态确定是否满员, 
                           消费者组状态有三种:
                                状态一: 如果是 Empty 或 Dead 状态, 不会是满员, 直接返回 True, 表示可以接纳申请入组的成员
                                状态二: 如果是 PreparingRebalance 状态, 成员入组的条件是必须满足两个条件之一, 第一个条件是
                                       该成员是之前已有的成员, 且当前正在等待加入组. 第二个条件是当前等待加入组的成员数小于
                                       Broker 端参数 group.max.size 值.
                                状态三: 如果是其他状态, 入组的条件是该成员是已有成员, 或者是当前组总成员数小于 Broker 端参数
                                       group.max.size 值. 这里比较的是组当前的总成员数, 而不是等待入组的成员数, 因为一旦 
                                       Rebalance 过渡到 CompletingRebalance 后, 没有完成加入组的成员, 就会被移除
                                       
                           如果成员入组失败, 则需要将该成员从元数据缓存中移除, 同时封装异常, 并调用回调函数返回；如果成员被入组成功,
                           则根据 Member ID 是否为空, 就执行 doUnknownJoinGroup 或 doJoinGroup 方法
                           
                    第五步: 尝试完成 JoinGroupRequest 请求的处理, 如果消费者组处于 PreparingRebalance 状态, 那么就将该请求放入
                           Purgatory, 尝试立即完成；如果是其它状态, 则无需将请求放入 Purgatory. 处理的是加入组的逻辑, 而此时消
                           费者组的状态应该要变更到 PreparingRebalance 后, Rebalance 才能完成加入组操作.
                           
    3. doUnknownJoinGroup 方法
            (1) 适用与全新的消费者组成员加入组, 因为 Member ID 尚未生成.
            (2) 流程
                    第一步: 检查消费者组的状态.
                    
                           如果是 Dead 状态, 则封装异常, 调用回调函数返回. 消费者组 Dead 状态情况是, 在成员加入组的同时, 可能存在
                           另一个线程, 已经把组的元数据信息从 Coordinator 中移除,例如, 组对应的 Coordinator 发生变更, 移动到
                           其他的 Broker 上, 这时会封装一个异常返回给消费者程序, 消费者程序会去寻找最新的 Coordinator, 重新发起加入
                           组操作
                           
                           如果状态不是 Dead, 就检查该成员的协议类型以及分区消费分配策略, 是否与消费者组当前支持的方案匹配, 
                           不匹配则封装异常, 然后调用回调函数返回. 匹配是指成员的协议类型与消费者组的是否一致, 以及成员设定的分区消费
                           分配策略是否被消费者组下的其它成员支持.
                           
                    第二步: 调用 generateMemberId 方法生成成员 ID, 生成规则是 clientId-UUID
                    第三步: 如果 requireKnownMemberId 取值为 true, 则将该成员加入到待决成员列表(Pending Member List)中, 然
                           后封装一个异常以及生成好的成员 ID, 将该成员的入组申请“打回去”，令其分配好成员 ID 之后再重新申请.
                           
                           如果为 False, 则无需这么苛刻, 直接调用 addMemberAndRebalance 方法将其加入到组中.
                           
    4. doJoinGroup 方法
            (1) 适用与 为那些设置成员 ID 的成员, 执行加入组
            (2) 流程:
                    第一部分:
                        与 doUnknownJoinGroup 方法一致, 判断是否处于 Dead 状态, 并且检查协议类型和分区消费分配策略是否与
                        消费者组的相匹配, 同时当前申请入组的成员是否属于待决成员, 如果是, 这成员已经分配好成员 ID,  直接调用 
                        addMemberAndRebalance 方法令其入组；
                        如果不是, 进入到第 2 部分, 即处理一个非待决成员的入组申请.
                        
                    第二部分:
                        第一步: 获取成员的元数据信息
                        第二步: 查询消费者组的当前状态
                                    如果是 PreparingRebalance 状态, 说明消费者组正要开启 Rebalance 流程, 调用 
                                    updateMemberAndRebalance 方法更新成员信息, 并开始准备 Rebalance.
                                    
                                    如果是 CompletingRebalance 状态, 判断该成员的分区消费分配策略与订阅分区列表, 是否和已保存
                                    记录中的一致, 如果相同说明该成员已经发起过加入组的操作, 并且 Coordinator 已经批准, 只是该成员
                                    没有收到, 这时构造一个 JoinGroupResult 对象, 直接返回当前的组信息给成员. 如果 protocols 
                                    不相同,  说明成员变更了订阅信息或分配策略, 就调用 updateMemberAndRebalance 方法, 
                                    更新成员信息, 并开始准备新一轮 Rebalance.
                                    
                                    如果是 Stable 状态,  判断该成员是否是 Leader 成员, 或者是它的订阅信息或分配策略发生变更, 就调用
                                    updateMemberAndRebalance 方法强迫一次新的 Rebalance. 否则,返回当前组信息给该成员即可, 通知它们
                                    可以发起 Rebalance 的下一步操作.
                                    
                                    如果这些状态都不是, 而是 Empty 或 Dead 状态,  就封装 UNKNOWN_MEMBER_ID 异常, 并调用
                                    回调函数返回
                        
            (3) updateMemberAndRebalance 方法
                    就是变更消费者组状态, 以及处理延时请求并放入 Purgatory.
                    
                    第一部分:
                            更新组成员信息: 调用 GroupMetadata 的 updateMember 方法来更新消费者组成员
                    
                    第二部分:
                            准备 Rebalance, 是将消费者组状态变更到 PreparingRebalance, 然后创建 DelayedJoin 对象, 并交由
                             Purgatory, 等待延时处理加入组操作
                             
    5. addMemberAndRebalance 方法
            (1) doUnknownJoinGroup 和 doJoinGroup 方法都会用到的 addMemberAndRebalance 方法, addMemberAndRebalance 方法作用
                是向消费者组添加成员, 准备 Rebalance.
            (2) 流程
                    第一步: 根据传入参数创建一个 MemberMetadata 对象实例,并设置 isNew 字段为 True, 标识是一个新成员. isNew 字段
                           与心跳设置相关联
                           
                    第二步: 判断消费者组是否是首次开启 Rebalance.如果是, 就把 newMemberAdded 字段设置为 True；这个字段的作用, 
                           是 Kafka 为消费者组 Rebalance 流程做的性能优化, 是在消费者组首次进行 Rebalance 时, 让 Coordinator 
                           多等待一段时间, 从而让更多的消费者组成员加入到组中, 以免后来者申请入组而反复进行 Rebalance. 这等待的时间
                           就是 Broker 端参数 group.initial.rebalance.delay.ms 的值, 这里 newMemberAdded 字段就是用于判断
                           是否需要多等待这段时间的一个变量.
                           
                    第三步: 调用 GroupMetadata 的 add 方法, 将新成员信息加入到消费者组元数据中, 同时设置该成员的下次心跳超期时间
                    
                    第四步: 将该成员从待决成员列表中移除, 它已经正式加入到组中, 就不需要在待决列表中了
                    
                    第五步: 调用 maybePrepareRebalance 方法, 准备开启 Rebalance
                        
```

### Rebalance 的流程 - 组同步
```shell
    1. 概念
            a. 组同步(SyncGroup), 是指当所有成员都成功加入组后, Coordinator 指定其中一个成员为 Leader, 将订阅分区信息发给
               Leader 成员.  所有成员(包括 Leader 成员)向 Coordinator 发送 SyncGroupRequest 请求.  只有 Leader 成员发送的请
               求中包含订阅分区消费分配方案, 在其他成员发送的请求中, 这部分的内容为空. 当 Coordinator 接收到分配方案后, 会通过向成员
               发送响应的方式, 通知各个成员要消费哪些分区.
               
            b. 组同步给成员下发分配方案外, 还需要在元数据缓存中注册组消息, 以及把组状态变更为 Stable. 一旦完成组同步操作, Rebalance 
               流程结束, 消费者组开始正常工作.
               
    2. handleSyncGroup 方法
            (1) 定义
                    def handleSyncGroup
                    (
                        groupId: String, // 消费者组名, 标识这个成员属于哪个消费者组, 
                        
                        /* 消费者组 Generation 号, 标识 Coordinator 负责为该消费者组处理的 Rebalance 次数, 
                         * 每当有新的 Rebalance 开启, Generation 加 1
                        generation: Int, 
                        
                        /* 消费者组成员 ID, 成员 ID 的值不是由客户端程序直接指定的, 但可以通过 client.id 参数间接影响该字段的取值
                        memberId: String, 
                        protocolType: Option[String], // 协议类型
                        protocolName: Option[String], // 分区消费分配策略名称
                        groupInstanceId: Option[String], // 静态成员Instance ID
                        
                        /* 按照成员分组的分配方案, 只有 Leader 成员发送的 SyncGroupRequest 请求, 才包含这个方案
                        groupAssignment: Map[String, Array[Byte]], 
                        responseCallback: SyncCallback // 回调函数
                    ): Unit = { ...... }
                    
            (2) 流程
                    第一步: 校验消费者组状态及合法性
                                第一: 消费者组名不能为空
                                第二: Coordinator 组件处于运行状态
                                第三: Coordinator 组件当前没有执行加载过程, 当 Coordinator 变更到其他 Broker 上时触发加载过程,即需要
                                     需要从内部位移主题中读取消息数据, 并填充到内存上的消费者组元数据缓存
                                第四: SyncGroupRequest 请求发送给了正确的 Coordinator 组件, 可能存在 Coordinator 变更
                                
                    第二步: 获取该消费者组的元数据信息, 如果找不到对应的元数据, 就封装 UNKNOWN_MEMBER_ID 异常, 调用回调函数返回；
                           如果找到元数据信息, 就调用 doSyncGroup 方法执行真正的组同步逻辑
                           
    3. doSyncGroup 方法
            (1) 流程
                    第一部分: 对消费者组做各种校验, 如果没有通过校验, 就封装对应的异常给回调函数；
                    
                        第一步: 判断消费者组的状态是否是 Dead, 如果是说明该组的元数据信息已经被其他线程从 Coordinator 中移除, 
                               很可能是因为 Coordinator 发生变更. 最佳的做法是拒绝该成员的组同步操作, 封装 
                               COORDINATOR_NOT_AVAILABLE 异常, 告知去寻找最新 Coordinator 所在的 Broker 节点, 然后再尝试重新
                               加入组
                        第二步: 判断 memberId 字段标识的成员是否属于这个消费者组, 如果不属于就封装 UNKNOWN_MEMBER_ID 异常, 并
                               调用回调函数返回
                        第三步: 判断成员的 Generation 是否和消费者组的相同. 如果不同则封装 ILLEGAL_GENERATION 异常给回调函数.
                        第四步: 判断成员和消费者组的协议类型是否一致. 如果不一致，则封装 INCONSISTENT_GROUP_PROTOCOL 异常给回调函数
                        第五步: 判断成员和消费者组的分区消费分配策略是否一致. 如果不一致, 封装 INCONSISTENT_GROUP_PROTOCOL 异常
                               给回调函数.
                               
                    第二部分: 根据不同的消费者组状态选择不同的执行逻辑
                        
                        第一: 如果消费者组的当前状态是 Empty 或 PreparingRebalance, 封装对应的异常给回调函数.
                        第二: 如果是 Stable 状态, 则说明消费者组已处于正常工作状态, 无需进行组同步的操作, 简单返回消费者组当前的
                             分配方案给回调函数, 供后面发送给消费者组成员
                        第三: 如果是 Dead 状态, 这是一个异常的情况, 理论上不应该为处于 Dead 状态的组执行组同步, 只能选择抛出 
                             IllegalStateException 异常
                        第四: 如果是 CompletingRebalance 状态(最有可能)
                                    第一步: 为该消费者组成员设置组同步回调函数, 通过 Response 的方式发送给消费者组成员.
                                    第二步: 判断当前成员是否是消费者组的 Leader 成员. 如果不是 Leader 成员,  直接结束. 只有 
                                           Leader 成员的 groupAssignment 字段才携带分配方案, 其他成员
                                           Leader 成员的 groupAssignment 字段才携带分配方案, 其他成员是没有分配方案的；
                                           如果是 Leader 成员. 则进入到下一步
                                    第三步: 为没有分配到任何分区的成员创建一个空的分配方案, 并赋值给这些成员, 目的是构造一个统一格式
                                           的分配方案字段 assignment
                                    第四步: 调用 storeGroup 方法, 保存消费者组信息到消费者组元数据, 同时写入到内部位移主题中
                                           如果 storeGroup 方法有错误, 则清空分配方案并发送给所有成员, 同时准备开启新一轮的 
                                           Rebalance；
                                           
                                           如果 storeGroup 方法没有错误, 则在消费者组元数据中保存分配方案, 发送给所有成员, 
                                           并将消费者组状态变更到 Stable
```