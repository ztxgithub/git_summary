# RabbitMQ

## 背景
```shell
    1. AMQP
        (1). AMQP 协议: Advanced Message Queuing Protocol , 提供统一消息服务的应用层标准高级消息队列协议，
        　　　　　　　　为面向消息的中间件设计
        (2). AMQ 基于模块化通过 Exchange 和 Message Queue 两个组件组合实现消息路由分发
        (3) AMQP 协议包含三层
                a. Module Layer: 协议的最高层,供客户端调用的命令, 实现客户端的业务逻辑,例如队列的声明 Basic.Consume
                b. Session Layer: 位于中间层，主要负责将客户端的命令发送给服务器，再将服务端的应
                                  答返回给客户端，为客户端与服务器之间的通信提供可靠性同步机制和错误处理。
                c. Transport Layer: 位于最底层，主要传输二进制数据流 ，提供帧的处理、信道复用、错
                                    误检测和数据表示等。
        
    2. RabbitMQ
        (1) Message Queue : 
                共享持久化消息队列：将发送的消息存储到磁盘，然后将消息转发给订阅该队列的所有消费者；
                私有临时消息队列： RabbitMQ 支持 rpc 调用，在调用过程中消费者都会临时生成一个消息队列，
                　　　　　　　　  只有当前消费者可见，且由服务端生成，调用完就会销毁队列。
        (2) RabbitMQ 采用 Erlang 语言实现 AMQP 的消息中间件.RabbitMQ 除了支持 AMQP 协议外,还支持 STOMP, MQTT 协议.
        (3) RabbitMQ 是一个生产者与消费者模型,主要负责消息的接收,存储,转发.在该模式下,消息一般包含 2 个部分: 消息体(payload) 
            和标签(Label), 标签用来描述这条消息的类型等, RabbitMQ 会根据标签将消息发送给指定的对象.
                
    3. 消息队列中间件(Message Queue Middleware)
            (1) 也称为消息队列或则消息中间件,其主要功能是进行消息的存储和转发,
                有 2 中消息模式, 第一种点对点模式,通过队列的形式.第二种订阅/发布模式(sub/publish).
                消息中间件避免的消息的耦合,使得消息的发送者和消息的接受者相互独立,同时消息中间件可以更好得进行分布式部署.
            (2) 作用
                    a. 解耦, 消息的发送者和消息的接受者可以异步的方式进行消息的通信
                    b. 冗余存储, 可以将消息持久化,确保因为消费者内部的原因(数据库有问题等), 导致当前消息无法正常处理,
                       此时中间件可以持久化消息,直到确保数据被安全的使用.
                    c. 扩展性, 对与中间件而言可以进行集群分布, 对与客户端而言,可以增加新的进程提高处理效率.
                    e. 削峰, 使用消息中间件能够使关键组件支撑突发访问压力(中间件有缓存的消息的特点)
                    f. 顺序保证, 大部分消息中间件支持一定程度的顺序性.
                    g. 异步通信.   
```

## Ubuntu 环境
```shell
    1. > sudo apt install rabbitmq-server
    2. 启动 RabbitMQ web 管理插件
       > sudo rabbitmq-plugins enable rabbitmq_management
    3. 重启服务器
       > sudo systemctl restart rabbitmq-server
    4. 打开浏览器输入 http://localhost:15672，默认用户名密码：guest/guest，就可以看到管理界面了 
```

## Linux troubleshoot
```shell
    1. error:
        Exception in thread "main" com . rabbitmq . c1ient.AuthenticationFai1ureException :
       ACCESS REFUSED - Login was refused using authentication mechanism PLAIN . For details
       see the broker 1ogfi1e.
       
       解决方案:
            重新新建一个账号,设置权限(set_permissions), 设置管理员权限(set_user)
    
```

## windows RabbitMQ 
```shell
    1. 安装 RabbitMQ 服务 windos 版(https://blog.csdn.net/qq_31634461/article/details/79377256)
    2. 在 cmd 中启动 RabbitMQ , cmd > net start RabbitMQ
       停止 RabbitMQ 服务, cmd> net stop RabbitMQ
    3. 测试地址 http://localhost:15672/ 
       默认的用户名：guest 
       默认的密码为：guest
    4. 
        (1) rabbitMQ 服务端添加用户
                在 http://localhost:15672/#/users 的 Admin 标签中添加用户名和密码.
        (2) 设置 Virtual Hosts 
                点击 Name 为 "/" 的一栏,设置 permission 加入到 root 用户中
```
## 概念
```shell
  1. Exchange：消息交换机，指定消息按什么规则，路由到哪个队列, 可以 exchange 路由到 queue,
               也可以是 exchange 路由到 另一个 exchange(这个属性 internal 为 true)
  2. Binding：绑定，把 exchange 和 queue 按照路由规则 binding 起来, 出于多租户和安全因素设计的，
              把 AMQP 的基本组件(exchange, queue)划分到一个虚拟的分组中，类似于网络中的 namespace 概念。
              当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，
              每个用户在自己的 vhost 创建 exchange/queue 等
  3. Queue：消息队列，每个消息都会被投入到一个或者多个队列里, 多个消费者可以订阅同一个队列,队列中的消息会以轮询(Round-Robin)
            方式发送到消费者中,不是每一个消费者都收到队列中的所有消息. 如果想要获得一个队列的所有消息,那么则新建一个队列
            将原来的 exchange 绑定到该队列中．
  4. Routing Key：路由关键字(发送的时候用到)，exchange 根据这个关键字进行消息投递，
  　　　　　　　　　这时应为在刚开始时进行 exchange 与 queue 绑定是通过 bindingKey 进行绑定，在生产者进行消息发送时，
  　　　　　　　　　将　Routing Key 与　bindingKey 进行匹配，发送对应的 queue.
  5. Vhost：虚拟主机，一个 broker 里可以开设多个 vhost，用作不同用户的权限分离
  6. Channel：消息通道，在客户端的每个连接里，可建立多个 channel，每个 channel 代表一个会话任务, 如果应用程序支持多线程，
              通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客户端和 message broker
              识别 channel，所以 channel 之间是完全隔离的。Channel 作为轻量级的 Connection 减少操作系统建立
              TCP connection 的开销。 如果每个 Channel 中流通的数据量很小,则可以复用一个 Connection, 如果 Channel 
              流通数据量很大,则需要独占一个 Connection
  7. 消息队列的使用过程大概如下：
          消息接收
            (1) 客户端连接到消息队列服务器，打开一个 channel。
            (2) 客户端声明一个 exchange，并设置相关属性。
            (3) 客户端声明一个 queue，并设置相关属性。
            (4) 客户端使用 routing key，在 exchange 和 queue 之间建立好绑定关系。
          消息发布
            (1) 客户端投递消息到 exchange。
            (2) exchange 接收到消息后，就根据消息的 key 和已经设置的 binding，进行消息路由，
                将消息投递到一个或多个队列里
  8. Exchange通常分为四种：
        (1) fanout：该类型路由规则非常简单，会把发送到该 Exchange 与它绑定的所有 Queue 中不管绑定的 Bindingkey　是什么，
                    相当于广播功能
        (2) direct：该类型路由规则会将消息路由到 binding key 与 routing key 完全匹配的 Queue 中, 同一个 exchange
                    与同一个 queue 会存在多个 BingdingKey
        (3) topic：与 direct 类型相似，只是规则没有那么严格，可以模糊匹配和多条件匹配, 其中　RoutingKey 为
        　　　　　　　一个个 "." 分隔开来的字符串(例如 com.rabbitmq.client). 同时 RoutingKey 和 BindingKey
                    存在特殊字符串，　通配符 "*" 代表一个单词， "#" 代表匹配到零个或多个单词,同一个 exchange
                    与同一个 queue 会存在多个 BingdingKey. 例如一个 exchange(type = "topic") 与 queue 的
                    BindingKey 有 "*.rabbitmq.*" 和 "com.#", 那么消息发送时 RoutingKey 为 "com.rabbitmq.client"
                    匹配，　"com.hidden.demo" 匹配．
        (4) headers：该类型不依赖于 routing key 与 binding key 的匹配规则来路由消息，而是根据发送的消息内容中的
                     headers 属性 与 BindingKey　进行匹配，不常用．
  9. 生产者只负责将消息送给交换机，而交换机确切地知道什么消息应该送到哪。
  10. RabbitMQ 消费模式
            (1) push(推模式), 采用 Basic.Consume 进行消费, 不同的订阅采用不同的消费者标签(consumerTag),
                最常用的做法是一个 Channel 对应一个消费者,这样为了防止同一个 Channel 中多个消费者调用相同的 callback
                 阻塞.对与 rabbitmq-c 而言, 先调用 amqp_basic_consume() 函数, 再调用 amqp_consume_message() ,
                 这时 rabbitmq 服务端进行消息的推送,推送的消息个数还是会受到 Basic.Qos 的限制.
            (2) pull(拉模式), 采用 Basic.Get 进行消费, 如果只想从 rabbitmq 中获取一条消息,则可以采用 Basic.Get 拉模式.
                如果需要持续的从 rabbitmq 接收消息, 则最好还是采用 push 模式
  11. 消费者的确认
        在 amqp_basic_consume() 时候设置 autoAck 值 , 当 autoAck 参数置为 false ，对于 RabbitMQ 服务端而言 ，
        队列中的消息分成了两个部分 : 一部分是等待投递给消费者的消息: 一部分是己经投递给消费者，但是还没有收到消费者确认
        信号的消息。 如果 RabbitMQ 一直没有收到消费者的确认信号，并且消费此消息的消费者己经断开连接，
        则 RabbitMQ 会安排该消息重新进入队列，等待投递给下一个消费者，当然也有可能还是原来的那个消费者。
        RabbitMQ 不会为未确认的消息设置过期时间，它判断此消息是否需要重新投递给消费者的条件是消费该消息的消费者连接是否己经断开，
        这么设计的原因是 RabbitMQ 允许消费者消费一条消息的时间可以很久.
        在 RabbtiMQ 的 Web 管理平台中, 当前队列的 "Ready" 状态是等待投递给消费者的消息数. "Unacknowledged" 是
        己经投递给消费者但是未收到确认信号的消息数.
  12. 消息的拒绝
        在收到消息后,如果对这条消息进行拒绝,可以用 basicReject(deliveryTag, requeue), 其中 deliveryTag 代表消息的编号,
        如果 requeue 置为 true ，则 RabbitMQ 会重新将这条消息存入队列，以便可以发送给下一个订阅的消费者;
        如果 requeue 置为 false，则 RabbitMQ 立即会把消息从队列中移除，而不会把它发送给新的消费者
        Basic.Reject 一次只能拒绝一条消息 ，如果想要批量拒绝消息 ，则可以使用 Basic.Nack 这个命令.
  13. 将 channel.basicReject 或者 channel.basicNack 中的 requeue 设直为 false ，可以启用"死信队列"的功能。
      死信队列可以通过检测被拒绝或者未送达的消息来追踪问题.
  14. 消息的未确认的重新发送
         channel.basicRecover(requeue)用来请求 RabbitMQ 重新发送还未被确认的消息 。 如果 requeue 参数设置为 true ，
         则未被确认的消息会被重新加入到队列中，这样对于同一条消息，可能会被分配给与之前不同的消费者。
         如果 requeue 参数设置为 false，那么同一条消息会被分配给与之前相同的消费者.
  15. 可靠性保证:
        设置 mandatory 参数或者备份交换器 (immediate 参数己被陶汰);
        设置 publisher confirm 机制或者事务机制;
        设置交换器、队列和消息都为持久化;
        设置消费端对应的 autoAck 参数为 false 并在消费完消息之后再进行消息确认
```

## 使用

```shell
  1. Fanout Exchange
        所有发送到 Fanout Exchange 的消息都会被转发到与该 Exchange 绑定(Binding)的所有 Queue 上。 Fanout Exchange
        不需要处理 RouteKey 。只需要简单的将队列绑定到 exchange 上, 这样发送到 exchange 的消息都会被转发到与该交换机绑定
        的所有队列上, Fanout Exchange 转发消息是最快的
  2. Direct Exchange
        所有发送到 Direct Exchange 的消息被转发到 RouteKey 中指定的 Queue。 Direct 模式，可以使用 RabbitMQ 
        自带的 Exchange：default Exchange 。所以不需要将 Exchange 进行任何绑定(binding)操作 。消息传递时，
        RouteKey 必须完全匹配，才会被队列接收，否则该消息会被抛弃
  3. Topic Exchange
        (1) 消息发布
                通配符 "*" 代表一个单词， "#" 代表匹配到零个或多个单词.
                
  4. 当消费者在收到消息后，给 rabbitMq 发送 ack 确认消息，则 rabbitMq 将对应的消息删除.
  5. 在客户端中在消费之前可以设置最大能保持的未确认的消息数.通过 AMQP 的 Basic.Qos
  6. 在真正消费之前消费者需要向 Broker 发送 Basic.Consume 命令将 Channel 置为接收模式,之后 Broker 回执
     Basic.Consume - Ok 以告诉消费者客户端准备好消费消息。紧接着 Broker 向消费者客户端推
     送 (Push) 消息，即 Basic.Deliver 命令，这个和 Basic.Publish 命令一样会携带 Content Header 和 Content Body.
     消费者收到消息后会向 Broker 发送 ack 确认消息.
  7. AMQP 命令
        (1) Queue.Purge : 清除队列中的内容
        (2) Queue.delete : 删除队列
  8. 在删除 exchnage 过程中, ifUnused 标识代表是否在 exchange 没有被使用的情况下删除. ifUnused 为 true 代表只有在 exchange
     没有被使用时才能删除. ifUnused 为 false 代表无论如何都可以被删除
  9. 删除队列 queue, ifUnused 标识代表是否在 queue 没有被使用的情况下删除. ifUnused 为 true 代表只有在 queue
     没有被使用时才能删除. ifUnused 为 false 代表无论如何都可以被删除, ifEmpty 设置为 true 表示在队列为空
     (队列里面没有任何消息堆积)的情况下才能够删除。
  10. exchangeBind 是将 exchange 与 exchange 进行绑定
```

### 消息的传递过程
```shell
    1. mandatory 和 immediate 是 channel.basicPublish 方法中的两个参数，它们都有
       当消息传递过程中不可达目的地时将消息返回给生产者的功能。 RabbitMQ 提供的备份交换器
       (Altemate Exchange) 可以将未能被交换器路由的消息(没有绑定队列或者没有匹配的绑定〉存
       储起来，而不用返回给客户端。
    2. mandatory 为 true 代表如果 exchange 无法根据路由键无法匹配到对应的队列,则 rabbitMQ 服务端调用 Basic.Return 命令将消息
       返回给生产者, 为 false 则匹配不到则直接丢弃.
    3. immediate 代表如果 exchange 根据路由键路路由到队列时,发现队列上并不存在任何消费者,则这条消息不会存入队列.
       当与路由键匹配的所有队列都没有消费者时, 该消息会通过 Basic.Return 返回至生产者, RabbitMQ 3.0 以后不再支持
       immediate, 可以采用 TTL 和 DLX 的方法替换.
       调用时客户端报错: 
        [WARN] - [An unexpected connection driver error occured (Exception message :
                 Connection reset)] - [com.rabbitmq.c1ient.imp1.ForgivingExceptionHand1er:120]
                 
       服务端报错:($RABBITMQ_HOME/var/log/rabbitmq/rabbit@$HOSTNAME.log)
                =ERROR REPORT==== 25-May-2017 : : 15 :1 0 : 25 ===
                Error on AMQP connection <0.25319.2> (192 . 168.0.2:55254->192.168.0.3 : 5672, vhost :
                ' / ', user: 'root ' , state: running) , channe1 1:{amqp_error, not_imp1emented , "
                immediate=true" , 'basic . pub1ish ' }
    4. 备份交换器(Alternate Exchange), 主要用于不满足匹配条件时,则消息通过备份交换器暂存在 RabbitMQ 中,实现的方式,
            (1). 
                声明交换器(调用 channel.exchangeDeclare 方法)的时候添加 alternate-exchange 参数来实现(
                 amqp_table_t arguments 加入对应的参数, args.put("a1ternate-exchange" , "myAe");)
               例如:
                   args.put("a1ternate-exchange" , "myAe");
                   // 将 myAe exchange 设置为 norma1Exchange 的备份交换器
                   channe1.exchangeDec1are( "norma1Exchange" , "direct" , true , fa1se , args);  
                   channe1.exchangeDec1are( "myAe " , "fanout" , true, fa1se , nu11) ;
                   channe1.queueDec1are( "norma1Queue " , true , fa1se , fa1se , nu11);
                   channe1.queueBind( " norma1Queue " ， " norma1Exchange" , " norma1Key");
                   channe1.queueDec1are( "unroutedQueue " , true , fa1se , fa1se , nu11);
                   channel.queueBind( "unroutedQueue ", "myAe ", "");
                   
                   声明了两个交换器 nonnallixchange 和 myAe，分别绑定了 nonnalQueue 和
                   umoutedQueue 这两个队列，同时将 myAe 设置为 nonnallixchange 的备份交换器, 同时 myAe 的类型为 fanout.
                   这样生产者如果发送消息时可以匹配到 norma1Key, 则消息到 norma1Queue 队列,否则到 unroutedQueue 队列.
                   
            (2) 采用 Policy 的方式来设置备份交换器
                    rabbitmqctl set_policy AE " ^normalExchange$" `{"alternate-exchange": "myAE"}'
            (3) 果备份交换器(exchange)和 mandatory 参数一起使用，那么 mandatory 参数无效.
                  
```
### TTL(过期时间)
```shell
    1. 设置消息的 ttl
            (0) 如果不设置 ttl, 则消息永远不会过期,如果设置为 0, 则表示除非此时可以直接将消息投递到消费者，否则该消息会被立即丢弃，
                这个特性可以部分替代 RabbitMQ 3.0 版本之前的 immediate 参数，之所以部分代替，是因为 immediate 参数在投递失败时
                会用 Basic.Return 将消息返回(这个功能可以用死信队列来实现)
            (1) 消息 ttl(方案一)
                通过申明队列时,设置队列的属性(增加参数选项 args), 这样队列中所有消息都有相同的过期时间. 
                在 channel.queueDeclare 方法中加入 x-message -ttl 参数实现的，单位是毫秒。
                方法一:
                    argss.put("x-message-ttl " , 6000);
                    channel . queueDeclare(queueName , durable , exclusive , autoDelete , argss) ;
                方法二:
                    rabbitmqctl set_policy TTL ".*" '{"message-ttl":60000}' --apply-to queues
                方法三: 调用 HTTP API 接口设置
                    $ curl -i -u root:root -H "content-type:application/json"-X PUT
                      -d'{"auto_delete":false, "durable":true, "arguments":{"x-message-ttl": 60000}}'
                      http://localhost:15672/api/queues/{vhost}/{queuename}
                      
                这种设置方式,其消息一旦过期,则在队列中就会消失.因为这是对队列中的所有消息进行相同的过期时间, 其队列
                中已过期的消息肯定在队列头部, RabbitMQ 只要定期从队头开始扫描是否有过期时间.
                      
            (2) 消息 ttl(方案二)
                 channel.basicPublish 方法中加入 expiration 的属性参数，单位为毫秒。
                 方法一: amqp_basic_publish() 中对 properties 进行操作
                 方法二: 通过 HTTPAPI 接口设置
                            $ curl -i - u root : root -H " content-type:application/json " -X POST -d
                            '{ " properties " :{ " expiration ":" 60000"} ， " routing_key": " routingkey" ,
                            "payload" :"my body", "payload_encoding " : " string" }' ,
                            http : //localhost : 15672/api/exchanges/{vhost}/{exchangename}/publish
                            
                这种设置方式,当消息过期时,并不会马上从队列中,因为每一个消息的过期时间都不同, Rabbit 不会每次都轮询
                队列看看消息是否过期,所以是该消息即将被消费时才进行判断.
                
    2. 设置队列的 ttl 
            channel.queueDeclare 方法中的 x-expires 参数(增加参数选项 args) , 其在 x-expires 毫秒的时间后,
            该队列没有被使用(队列上没有任何的消费者,队列也没有重新申明,也没有调用 Basic.Get 命令),则将该队列删除掉.
            如果 RabbitMQ 重启了,则其队列的过期时间会被重新计算.
            实例:
                args.put( "x-expires" , 1800000);
                channel.queueDeclare("myqueue " , false , false , false , args);
                             
```

### 队列类型
```shell
    1. 死信队列
            (1) 消息变成死信的情况,第一种:消息被拒绝(Basic.Reject / Basic.Nack), 并且设置 requeue 为 false
                                第二种: 消息过期   第三种: 队列达到最大长度
            (2) 当消息在队列中变为死信(dead message)后, 会被重新发送到死信交换器(DLX Dead-Letter-Exchange),
                在路由到死信队列.这个特性和将消息的 TTL 设置为 0 结合达到 immediate 参数的效果
                例如:
                channel.exchangeDeclare("dlx_exchange" , "direct "); //创建 DLX: dlx_exchange
                Map<String, Object> args = new HashMap<String, Object>();
                args.put("x-dead-letter-exchange" , "dlx exchange");
                //为队列 myqueue 添加 DLX
                channel.queueDeclare("myqueue" , false , false , false , args);
                // 为 DLX 指定路由键, 如果不指定,则使用原队列的路由键
                args.put("x-dead-letter-routing-key" , "dlx-routing-key");
                
                channel.exchangeDeclare("exchange.dlx" , "direct" , true);
                channel.exchangeDeclare("exchange.normal" , "fanout" , true);
                Map<String , Object> args = new HashMap<String, Object>( );
                args.put("x-message-ttl" , 10000); // 设置整个队列中消息的过期时间
                args.put("x-dead-letter-exchange" , "exchange.dlx");  // 设置死信交换器
                args.put( "x-dead-letter-routing-key", "routingkey"); // 为 DLX 指定路由键
                channe1.queueDec1are("queue.norma1" , true , fa1se , fa1se , args);
                channe1.queueBind("queue.normal" , "exchange.normal" , "");
                channe1.queueDec1are("queue.d1x" , true , false , false , null) ; // 声明死信队列
                channel.queueBind("queue.dlx" , "exchange.dlx " , Wroutingkey");
                channel.basicPublish( "exchange.normal" , " rk " ,
                MessageProperties.PERSISTENT_TEXT_PLAIN, "dlx " .getBytes()) ;
                
                生产者首先发送一条携带路由键为 " rk " 的消息，然后经过交换器 exchange.normal 顺利地存储到队列 queue.normal 中 。
                由于队列 queue.normal 设置了过期时间为 10s ， 在这 10s 内没有消费者消费这条消息，那么判定这条消息为过期。
                由于设置了 DLX ， 过期的时候， 消息被丢给交换器 exchange.dlx 中，这时找到与 exchange.dlx 匹配的队列 queue.dlx,
                最后消息被存储在 queue.dlx 这个死信队列中。
    2. 延迟队列
            1. 当消息发送后,其消息并不想立刻被消费者消费,而是等待特定的时间,消费者才能消费这个消息.使用的场景,
                (1). 用户下单后 30 分钟内要进行支付, 30 分钟后为未处理(过期时间),直接路由到死信队列(延迟队列)
                (2). 用户希望通过手机远程遥控家里的智能设备在指定的时间进行工作。这时可以将
                   用户指令发送到延迟队列，当指令设定的时间到了再将指令推送到智能设备
            2. 可以通过 DLX 和 TTL 来模拟延迟队列.根据应用需求的不同，生产者在发送消息的时候通过设置不同的路由键，
              以此将消息发送到与交换器绑定的不同的队列中。这里队列分别设置了过期时间为 5 秒、 10 秒、 30 秒、 1 分钟，同时也
              分别配置对应的 DLX 和相应的死信队列。当相应的消息过期时，就会转存到相应的死信队列.
              
    3. 优先级队列
            1. channel.queueDeclare 方法中的 x-max-priority 参数(增加参数选项 args)
               args.put("x-rnax-priority", 10) ;  // 设置队列的优先级为 10, 与发送消息带的优先级一一对应
               channel.queueDeclare("queue.priority" , true, fa1se, false, args);
               
               AMQP.BasicProperties.Bui1der builder = new AMQP.BasicProperties.Builder() ;
               builder.priority(5) ;
               AMQP.BasicProperties properties = builder.build() ;
               channel.basicPub1ish("exchange_priority" , "rk_priority" , properties , 
                                    ("messages" ).getBytes()) 
                                    
               消息发送的优先级最高为 10(队列设置的优先级), 消息设置的优先级越高(值越高),则消息优先被处理                 
```

### 持久化
```shell
    1. 交换器持久化, 在声明 exchange 时将 durable 置为 true, RabbitMQ 服务器重启后,不需要再重新声明 exchange.
    2. 队列持久化, 在声明队列时将 durable 置为 true, 如果队列没有持久化,则 RabbitMQ 重启后,队列和消息都不存在
    3. 消息的持久化,在 publish 时  amqp_basic_properties_t.delivery_mode 置为 2 设置为消息的持久化.
       要进行消息的持久化必须保证队列的持久化.
    4. 消息持久化可能出现的问题
            (1) 如果在订阅消费队列时将 autoAck 参数设置为 true，当消费者接收到相关消息之后，
                还没来得及处理就宕机，这样也算数据丢失。这种情况,解决方案: 将 autoAck 参数设置为 false ,
                并且消费端处理完后进行手动确认
            (2) 在持久化的消息正确存入 RabbitMQ 之后,不可能每一条都实时同步到磁盘中,那么在这个过程中宕机,保存在
                系统缓冲区内的数据将会丢失,采用的解决方案是: RabbitMQ 的镜像队列机制，相当
                于配置了副本，如果主节点(master)在此特殊时间内挂掉，可以自动切换到从节点(slave),
                保证高可用性，除非整个集群都挂掉。这样要比没有配置镜像队列的可靠性要高，在实际生产环境中的
                关键业务队列一般都会设置镜像队列。
            (3) 发送端引入事务机制或者发送方确认机制来保证消息己经正确地发送并存储至 RabbitMQ
```
### 生产者确认
```shell
    1. 问题:在 RabbitMQ 持久化消息之前,怎么确保生产者成功发送消息到 RabbitMQ 服务端.
    2. 方案一: 事务机制(AMQP 层协议)
            (1) channel.txSelect 用于将当前的信道设置为事务模式
                channel.txCommit 用于提交事务
                channel.txRollback 用于事务回滚
                在发布消息之前,先通过 channel.txSelect 设置为事务模式, 在进行消息的发布,
                在 channel.txCommit 提交事务之前,如果 RabbitMQ 异常崩溃或则异常,则可以调用 channel.txRollback 进行事务的回滚, 
                当 channel.txCommit 成功执行后,则消息一定到达 RabbitMQ.
                
                实例:
                    channel.txSelect();
                    channel.basicPublish(EXCHANGE NAME , ROUTING KEY ,
                    MessageProperties . PERSISTENT TEXT_PLAIN,
                    "transaction messages".getBytes());
                    channel.txCommit();
                    
                发送多条消息
                    channel.txSelect();
                    for (int i=O ; i < LOOP TIMES; i++) {
                    try {
                        channel.basicPublish ("exchange" , "routingKey", null,
                                              ("messages" + i).getBytes()) ;
                        channel.txCommit();
                    } catch (IOException e) {
                        e.printStackTrace() ;
                        channel.txCommit();
                   }
            (2) 事务机制通过事务的开始,再发布数据,在事务的提交确保消息到达 RabbitMQ, 这种频繁的操作会造成 RabbitMQ 的性能的
                降低
    3. 方案二: 发送方确认机制(publish confirm)
            (1) 生产者通过调用 channel.confirmSelect 方法(即 Confirm.Select 命令)将信道设置为 confirm 模式(针对发送者而言)，
                之后 RabbitMQ 会返回 Confirm.Select-Ok 命令表示同意生产者将当前信道设置为 confirm 模式。
                所有被发送的后续消息都被 ack 或者 nack 一次，不会出现一条消息既被 ack 又被 nack 的情况 
            (2) publish confirm 的优势并不在于同步确认,而是批量 confirm 方法或则异步 confirm 方法.
                在批量 confirm 方法中，客户端程序需要定期或者定量(达到多少条)，亦或者两者结合起
                来调用 channel.waitForConfirms 来等待 RabbitMQ 的确认返回。相比于普通单条 confirm 方法，
                批量极大地提升了 confmn 的效率，但是问题在于出现返回 Basic.Nack 或者超时情况时，客户端需要将这一批次的消息
                全部重发，这会带来明显的重复消息数量，并且当消息经常丢失时，批量 confirm 的性能应该是不升反降的。
            (3) 推荐采用异步 confirm 方法     
```

### 消费者要点
```shell
    1. 消息分发
            (1) RabbitMQ 分发一般采用轮询(round-robin) 的分发方式给消费者, 每条消息只会发送给订阅列表里的一个消费者, 
                这种方式适合扩展. 
            (2) 存在问题: 如果有 n 个消费者，那么 RabbitMQ 会将第 m 条消息分发给第 m%n (取余的方式)个消费者，
                RabbitMQ 不管消费者是否消费并己经确认 (Basic.Ack) 消息。如果某些消费者任务繁重，来不及消费那么多的消
                息，而某些其他消费者由于某些原因(比如业务逻辑简单、机器性能卓越等)很快地处理完了所分配到的消息，进而进程空闲，
                这样就会造成整体应用吞吐量的下降。
            (3) 解决方案:
                    采用 channel.basicQos 方法允许限制信道上的消费者所能保持的最大未确认消息的数量, 这样 RabbitMQ 检测
                    到该消费者未确认数量达到设置的上限时,就不会消息发送给该消费者.
            (4) Basic.Qos 的使用对于拉模式的消费方式无效. prefetchCount 设置为 0 则表示没有上限.
                prefetchSize 这个参数表示消费者所能接收未确认消息的总体大小的上限， 单位为 Byte ，设置为 0 则表示没有上限。
            (5) 实例:
                    Channel channel =
                    Consumer consumer1 = . ..;
                    Consumer consumer2 = ...;
                    channel . basicQos(10) ; // Per consumer limit
                    channel.basicConsume( "my-queue1 " , false , consumer1);
                    channel.basicConsume( "my-queue2 " , false , consumer2);
                    各自能接收到未确认消息的上限为 10 
                    
    2. 消息的顺序性
            (1) 消息的顺序性是指消费者消费到的消息和发送者发布的消息的顺序是一致的, RabbitMQ 存在多种消息顺序的乱位
                例如: 第一种情况: 事务的回滚. 
                     第二种情况: 启用 publisher confirrn 时，在发生超时、中断， 又或者是收到 RabbitMQ 的 Basic.Nack 命令时，
                               那么需要补偿发送，结果与事务机制一样会错序.
                     第三种情况: 如果生产者发送的消息设置了不同的超时时间，井且也设置了死信队列， 整体上来说相当于一个延迟队列，
                                那么消费者在消费这个延迟队列的时候，消息的顺序必然不会和生产者发送消息的顺序一致。
                     第四种情况: 队列的优先级
            (2) 保证消息的顺序性, 需要业务方使用 RabbitMQ 之后做进一步的处理，比如在消息体内添加全局有序标识(类似 Sequence ID) 
                来实现 。
                     
```

### 消息传输保障
```shell
    1. 分为 3 个级别,
            (1) at most once: 最多一次。消息可能会丢失 ，但绝不会重复传输 。
            (2) at least once: 最少一次。消息绝不会丢失，但可能会重复传输。
            (3) exactly once: 恰好一次。每条消息肯定会被传输一次且仅传输一次。
    2. RabbitMQ 支持 "最多一次" 和 "最少一次", 很难保证"恰好一次", 只能是客户端进行 GUID (Globally Unique Identifier) 去重.
       
```

## 接口使用
```shell
  1.  amqp_connection_state_t amqp_new_connection(void);
      描述: 分配和初始化一个新的 amqp_connection_state_t 对象
      注意: 不使用时应该调用 amqp_destroy_connection()
  2. amqp_socket_t *amqp_tcp_socket_new(amqp_connection_state_t state)
      描述: 创建一个 socket
      注意: 不使用时需要调用 amqp_connection_close() 进行释放
  3. int amqp_socket_open(amqp_socket_t *self, const char *host, int port)
      描述: 打开 socket 连接
      参数:
          self : 指向 amqp_socket_t 的指针
          host : RabbitMQ Server 的主机(可以是域名，ip)
          port: RabbitMQ Server 端口号
      返回值:
          AMQP_STATUS_OK : 成功 （enum amqp_status_enum_）
      注意: 
          可以再调 amqp_socket_open() 函数之前进行 socket option 设置 amqp_set_socket()
          
  3.1 int amqp_socket_open_noblock(amqp_socket_t *self, const char *host,
                                             int port, struct timeval *timeout);
     描述:
        非阻塞的打开 scoket
          
   4. amqp_rpc_reply_t amqp_login(amqp_connection_state_t state, char const *vhost,
                                  int channel_max,int frame_max,int heartbeat,
                                  amqp_sasl_method_enum sasl_method, ...);
      描述: 用于登录 RabbitMQ server，主要目的为了进行权限管理
      参数说明:
            state : amqp connection(amqp_connection_state_t)
            vhost : rabbit-mq 的虚机主机，是 rabbit-mq 进行权限管理的最小单位, 默认是 "/"
            channel_max:  最大链接数， 0 代表无限制。 AMQP_DEFAULT_MAX_CHANNELS：默认值
                          最大链接数为 65535 
            frame_max:  和客户端通信时所允许的最大的 frame size. 默认值为 131072(128KB)，
                        默认值是 AMQP_DEFAULT_FRAME_SIZE
                        增大这个值有助于提高吞吐，降低这个值有利于降低时延
            heartbeat:  代表心跳包发送间隔. 0: 代表屏蔽心跳包, 这个功能现阶段有限制，只能在
                        amqp_basic_publish() and amqp_simple_wait_frame()/amqp_simple_wait_frame_noblock()
                        函数使用
            sasl_method:  用于SSL鉴权,
                          AMQP_SASL_METHOD_PLAIN 后跟 username, password
                          AMQP_SASL_METHOD_EXTERNAL 后跟 identity
      返回值:
          (1) amqp_rpc_reply_t.reply_type == AMQP_RESPONSE_NORMAL 代表登录成功
          (2) amqp_rpc_reply_t.reply_type == AMQP_RESPONSE_LIBRARY_EXCEPTION ，出现这种情况一般是因为
              amqp_rpc_reply_t.library_error 被置为 AMQP_STATUS_CONNECTION_CLOSED, 而被置为该状态的原因
              是无效的 vhost, 或则认证失败(authentication failure)
          (3) amqp_rpc_reply_t.reply_type == AMQP_RESPONSE_SERVER_EXCEPTION, 分为 2 中情况
              第一种： amqp_rpc_reply_t.reply.id == AMQP_CHANNEL_CLOSE_METHOD , 其 channel 出现异常 
                      r.reply.decoded to amqp_channel_close_t* to see details of the exception。
                      amqp_channel_close_t *m = (amqp_channel_close_t *)x.reply.decoded;
                      fprintf(stderr, "%s: server channel error %uh, message: %.*s\n",
                                context, m->reply_code, (int)m->reply_text.len,
                                (char *)m->reply_text.bytes);
                                
                      需要做的操作 amqp_send_method() a amqp_channel_close_ok_t
                      如果想要重新使用则需要跟 channel 相关的资源重新打开.任何资源与 channel 相关
                      (auto-delete exchanges, auto-delete queues, consumers) 都无效,要想使用
                      得重新 create
              第二种: 
                       amqp_rpc_reply_t.reply.id == AMQP_CONNECTION_CLOSE_METHOD， 
                       查看错误信息
                       amqp_connection_close_t *m =
                                     (amqp_connection_close_t *)x.reply.decoded;
                       fprintf(stderr, "%s: server connection error %uh, message: %.*s\n",
                             context, m->reply_code, (int)m->reply_text.len,
                             (char *)m->reply_text.bytes);
                       需要调用 amqp_send_method() 发送一个 amqp_connection_close_ok_t and 
                       disconnect from the broker
    5. amqp_channel_open_ok_t * amqp_channel_open(amqp_connection_state_t state, amqp_channel_t channel)
        描述: 打开一个 channel 
        参数:
            amqp_connection_state_t : 连接
            amqp_channel_t ： 一般从 1 开始
        实例:
            amqp_channel_open(conn, 1);
            
    6. amqp_rpc_reply_t amqp_get_rpc_reply(amqp_connection_state_t state)
       描述: 获取上一步对 amqp_connection_state_t 操作的状态, 这个方法 amqp_get_rpc_reply() 根据
             大部分的同步的 AMQP 方法的指针返回值来解析出调用的结果. 调用此方法 amqp_get_rpc_reply()
             主要是用于哪些 method 返回值不是 amqp_rpc_reply_t 类型, 例如 
              amqp_channel_open_ok_t *amqp_channel_open(...) 函数
            
       返回值:
              amqp_rpc_reply_t

    7. amqp_exchange_declare_ok_t* amqp_exchange_declare( amqp_connection_state_t state, 
                                                          amqp_channel_t channel,
                                                          amqp_bytes_t exchange, 
                                                          amqp_bytes_t type, 
                                                          amqp_boolean_t passive, 
                                                          amqp_boolean_t durable, 
                                                          amqp_boolean_t auto_delete, 
                                                          amqp_boolean_t internal, 
                                                          amqp_table_t arguments)
        描述: 声明 exchange 
        参数:
              amqp_connection_state_t : 打开的连接
              channel : 打开的 channel
              exchange : exchange 名字
              type ： exchange 类型:  "fanout"  "direct" "topic" "headers"
              passive : 被动:0 (默认写 0), 如果 exchange 不存在则创建 exchange，调用成功返回。
                                          如果 exchange 已经存在，并且匹配当前 exchange 的属性,如果一致则成功返回，
                                          如果不一致则 exchange 声明失败。属性有 exchange 的 type, durable, auto_delete
                                          internal 等
                        主动 1: 若队列存在则命令成功返回（申明该 exchange 时其他参数有变化没有用，不会影响原来的 exchange 属性）
                       　　　　　若不存在不会创建 exchange，返回错误。这个可以判断某个 exchange　是否存在 
              durable ： 1：持久化, 将 exchange 信息持久化, rabbitMQ 服务器重启时加载对应的 exchange 
                         0: 暂时
              auto_delete ：  1: 自动删除
                              自动删除的前提是至少有一个队列(或则 exchange) 与这个 exchange 进行绑定.
                              自动删除就是这个 exchange 与所有的与之绑定队列(或则 exchange)解绑.
              internal : 0 (默认写 0)
                         1: clients cannot publish to this exchange directly. It can only be used with 
                            exchange to exchange bindings
              arguments: amqp_empty_table
        注意:
            申明前,该 exchange 必须已经存在
              
      8. amqp_queue_declare_ok_t *AMQP_CALL amqp_queue_declare(amqp_connection_state_t state, 
                                                               amqp_channel_t channel, 
                                                               amqp_bytes_t queue,
                                                               amqp_boolean_t passive, 
                                                               amqp_boolean_t durable,
                                                               amqp_boolean_t exclusive,
                                                               amqp_boolean_t auto_delete, 
                                                               amqp_table_t arguments)
         描述: 声明 queue
         参数:
              amqp_connection_state_t : 打开的连接
              channel : 打开的 channel
              queue: 队列的名字
              passive 被动:0 (默认写 0), 如果 exchange 不存在则创建 exchange，调用成功返回。
                                        如果 exchange 已经存在，并且匹配当前 exchange 的属性,如果一致则成功返回，
                                        如果不一致则 exchange 声明失败。属性有 exchange 的 type, durable, auto_delete
                                        internal 等
                      主动 1: 若队列存在则命令成功返回（申明该 exchange 时其他参数有变化没有用，不会影响原来的 exchange 属性）
                     　　　　　若不存在不会创建 exchange，返回错误。这个可以判断某个 exchange　是否存在 
              durable  队列是否持久化
              exclusive  默认为0, 设置是否排他，如果一个队列被申明为排他队列，该队列仅对首次声明它的连接(connection) 可见
              　　　　　　并在连接断开时自动删除. 排他性是针对连接(Connection)而言的, 该连接下所有的 Channel 可以访问这个
                         排他队列，"首次"是指如果一个连接(Connection)己经声明了一个排他队列，其他连接(Connection)是不允许
                         建立同名的排他队列的，这个与普通队列不同:即使该队列是持久化的，一旦连接关闭或者客户端退出，
                         该排他队列都会被自动删除，这种队列适用于一个客户端同时发送和读取消息的应用场景。
              aoto_delete :队列是否自动删除, 其前提条件是至少有一个消费者连接到这个队列，同时所有与这个队列连接的消费者
              　　　　　　　　都断开才会被自动删除.如果生产者客户端创建这个队列，或则没有消费者连接到这个队列都是不会自动删除这个
                            队列．
              arguments 用于拓展参数，amqp_empty_table
         注意:
            申明前,该 queue 必须已经存在.
              
       9. amqp_queue_bind_ok_t * amqp_queue_bind(amqp_connection_state_t state, amqp_channel_t channel, 
                                              amqp_bytes_t queue, amqp_bytes_t exchange, 
                                              amqp_bytes_t routing_key, amqp_table_t arguments)
       描述: 进行 bind, 在打开的 channel 的情况下，将已经声明的 queue 和 已经声明的 exchange 通过 routing_key 绑定
             在一起.
       实例；
            amqp_queue_bind(conn, 1, amqp_cstring_bytes(queue),
                  amqp_cstring_bytes(exchange), amqp_cstring_bytes(routing_key),
                  amqp_empty_table);
            die_on_amqp_error(amqp_get_rpc_reply(conn), "binding");
            
       10. int amqp_basic_publish( amqp_connection_state_t state, amqp_channel_t channel,
                                   amqp_bytes_t exchange, amqp_bytes_t routing_key, 
                                   amqp_boolean_t mandatory, amqp_boolean_t immediate, 
                                   struct amqp_basic_properties_t_ const *properties,
                                   amqp_bytes_t body)
            描述: 向 RabbitMQ 发送消息, 向 exchange 发送消息附带 routing key
            
            参数:
                amqp_connection_state_t
                amqp_channel_t
                exchange
                routing_key
                mandatory : 强制性的, 当置为 1 时, 消息必须发到路由到 queue 上，否则返回错误. 一般写 0
                            如果 exchange 无法根据路由键无法匹配到对应的队列,则 rabbitMQ 服务端调用 Basic.Return 命令将消息
                            返回给生产者,
                immediate: 即时性，置为 1 时，消息必须分发到 consumer 中，否则返回错误 ,一般写 0
                           如果 exchange 根据路由键路路由到队列时,发现队列上并不存在任何消费者,则这条消息不会存入队列.
                           当与路由键匹配的所有队列都没有消费者时, 该消息会通过 Basic.Return 返回至生产者
                properties: 跟消息相关的属性
                            amqp_basic_properties_t._flags : AMQP_BASIC_CONTENT_TYPE_FLAG  
                                                             AMQP_BASIC_DELIVERY_MODE_FLAG
                            amqp_basic_properties_t.content_type : amqp_cstring_bytes("text/plain");
                            amqp_basic_properties_t.delivery_mode : 2: 可持久化，消息会持久化在服务器中.
                            实例:
                            props._flags = AMQP_BASIC_CONTENT_TYPE_FLAG | AMQP_BASIC_DELIVERY_MODE_FLAG;
                            props.content_type = amqp_cstring_bytes("text/plain");
                            props.delivery_mode = 2; /* persistent delivery mode */
                            
                            其他属性：
                                1. 设置过期时间(expiration)
                body: 消息体
            返回值：
                  AMQP_STATUS_OK ： 成功
                  AMQP_STATUS_TIMER_FAILURE:系统时间返回错误
                  AMQP_STATUS_HEARTBEAT_TIMEOUT : 心跳超时，导致消息未发送
                  AMQP_STATUS_NO_MEMORY : 内存不足，导致消息未发送
                  AMQP_STATUS_TABLE_TOO_BIG： a table in the properties was too large to fit in a single frame
                  AMQP_STATUS_CONNECTION_CLOSED： 连接关闭
                  
                  
            注意: amqp_basic_publish 函数是一个异步的方式，如果发送不存在的 exchange ，则不会体现在返回值上。这个返回值
                  只是体现了消息成功被传输到 broker 
                  
       11. amqp_basic_consume_ok_t *AMQP_CALL amqp_basic_consume( amqp_connection_state_t state, 
                                                                    amqp_channel_t channel,
                                                                    amqp_bytes_t queue,
                                                                    amqp_bytes_t consumer_tag,
                                                                    amqp_boolean_t no_local, 
                                                                    amqp_boolean_t no_ack,
                                                                    amqp_boolean_t exclusive,
                                                                    amqp_table_t arguments)
              描述: 告知 rabbitmq 服务器开始一个消费者，在从 queue 中接受消息之前先调用一次 amqp_basic_consume 接口,
                    将消费的模式置为推模式
              参数:
                  state
                  channel
                  queue : 队列名
                  consumer_tag: 消费者标签, 用来区分多个消费者
                  no_local: 1: 表示不能将同一个 Connection 中生产者发送的消息给这个 Connection 中的消费者
                  no_ack : 是否需要确认消息后再从队列中删除消息, 
                            1 : 收到消息后代表自动回复 ack
                            0 : 需要调用 amqp_basic_ack() 函数显式进行 ack 回复, 这样 rabbitmq
                           才会将对应的消息从服务器中删除.建议使用这个模式,不用担心处理消息过程中消费者进程挂掉后
                           消息丢失的问题
                  exclusive
                  arguments
              
              实例:
              amqp_basic_consume(conn, 1, queuename, amqp_empty_bytes, 0, 1, 0,
                     amqp_empty_table);
                     
                     
         
       12. amqp_rpc_reply_t amqp_consume_message(amqp_connection_state_t state,
                                                    amqp_envelope_t *envelope,
                                                    struct timeval *timeout,
                                                    int flags)
              描述: 等待并且消费消息内容
              参数：
                  amqp_connection_state_t
                  envelope: 调用者调用完后并用完 envelope 后，需要调用 amqp_destroy_envelope(),需要自己管理 
                            envelope
                  timeout: 非阻塞等待消息的， 值为 NULL 为阻塞等待
                  flags: 默认值为 0
              返回值:
                  ret.reply_type == AMQP_RESPONSE_NORMAL : 成功
                  If ret.reply_type == AMQP_RESPONSE_LIBRARY_EXCEPTION, and 
                  ret.library_error == AMQP_STATUS_UNEXPECTED_STATE, a frame 不是
                  AMQP_BASIC_DELIVER_METHOD 这个类型收到, the caller should call amqp_simple_wait_frame()
                  to read this frame and take appropriate action.
              注意:
                  监听所有通道 channel 的消息，并返回.如果有其他方法收到在 basic.deliver 之前, 则会返回
                  错误, ret.reply_type == AMQP_RESPONSE_LIBRARY_EXCEPTION && 
                  ret.library_error == AMQP_STATUS_UNEXPECTED_STATE, The caller should then
                 call amqp_simple_wait_frame() to read this frame and take appropriate action.
                     
          13. void amqp_maybe_release_buffers(amqp_connection_state_t state)
            描述: 释放 conn 连接自身的内存，包括关联的所有 channel 的资源, 在调用　amqp_consume_message() 函数之前
                 先调用　amqp_maybe_release_buffers() 函数
            
          14. void amqp_destroy_envelope(amqp_envelope_t *envelope)
            描述: 释放 amqp_envelope_t 结构体中的内存数据, 在调用完　amqp_consume_message(), 并使用完 amqp_envelope_t
                  记得 amqp_destroy_envelope(), 防止内存泄露
                  
          
       rpc call           
       13. int amqp_simple_wait_frame_noblock(amqp_connection_state_t state,
                                              amqp_frame_t *decoded_frame,
                                              struct timeval *timeout)
           描述:
                非阻塞等待从 broker 读取下一个 amqp_frame_t frame, 当接收到消息时,调用 amqp_simple_wait_frame_noblock() 直接
                返回 amqp_frame_t 数据, 不用再进行阻塞读( blocking read()), 
                
           参数:
                amqp_connection_state_t state :
                amqp_frame_t *decoded_frame
                struct timeval *timeout : 
                                            (1) 结构体里面的值都为 0, 则立刻检测,立刻返回
                                            (2) timeout 值为 NULL, 则一直阻塞等待,直到接到消息
                 
           返回值:
                AMQP_STATUS_OK : 成功
                失败情况:
                AMQP_STATUS_TIMEOUT : 消息超时
                AMQP_STATUS_INVALID_PARAMETER : struct timeval *timeout 传入参数的值有问题
                AMQP_STATUS_NO_MEMORY: 内存不足
                AMQP_STATUS_BAD_AMQP_DATA: AMQP data 数据异常,需要将该连接(connection) 断开
                AMQP_STATUS_UNKNOWN_METHOD: 错误的 rpc 方法, 可能是协议有问题,需要将该连接(connection) 断开
                AMQP_STATUS_HEARTBEAT_TIMEOUT: 在等待的过程中,心跳超时
                AMQP_STATUS_SOCKET_ERROR : 连接被关闭
                
       13.1 int  amqp_simple_wait_frame(amqp_connection_state_t state,
                                                 amqp_frame_t *decoded_frame)
            描述:
                阻塞等待
                
       14. amqp_boolean_t  amqp_frames_enqueued(amqp_connection_state_t state)
            描述:
                查看 amqp_connection_state_t 是否有进来的 amqp_frame_t 消息,如果有消息,
                则 amqp_simple_wait_frame() 函数或则 amqp_simple_wait_frame_noblock() 函数
                则会有消息返回(而不是用系统的 block read())
                
            返回值:
                ture : 有消息
                
       15. amqp_boolean_t amqp_data_in_buffer(amqp_connection_state_t state)
            描述:
                检查在 receive buffer 中是否有数据, 
            
            
          
                  
```
## 实例
```shell
    1. 
         amqp_exchange_declare(m_connState, 1, amqp_cstring_bytes(strExchange.c_str()), 
                               amqp_cstring_bytes("fanout"), 0, 1, 0, 0, amqp_empty_table);

         amqp_queue_declare(m_connState, 1, amqp_cstring_bytes(strQueue.c_str()), 0, 0, 0, 1,
                            amqp_empty_table);
                            
         amqp_queue_bind(m_connState, 1, amqp_cstring_bytes(strQueue.c_str()),              
                         amqp_cstring_bytes(strExchange.c_str()), amqp_empty_bytes, 
                         amqp_empty_table);
```