# ActiveMQ 使用

## 概述

```
	1. 默认用户名和密码是 admin
        
	3. ActiveMQ 消息管理后台系统: http://localhost:8161/admin
	4. JMS 规范
      (1) JMS 是 Java 消息服务（Java Message Service）是一个 Java 平台中关于面向消息中间件（MOM）的 API，
          用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。
          JMS 提供与厂商无关的访问方法，以访问消息收发服务，许多厂商都支持 JMS。
          消息是 JMS 中的一种类型对象，由两部分组成：报头和消息主体。
                  1.报头由路由信息以及有关该消息的元数据组成。
                  2.消息主体则携带着应用程序的数据或有效负载。根据有效负载的类型来划分，可以将消息分为几种类型，简单文本(TextMessage)、
                    可序列化的对象 (ObjectMessage)、属性集合 (MapMessage)、字节流 (BytesMessage)、原始值流 (StreamMessage)，还有无有效负载的消息 (Message)
      (2) 
          a.连接工厂。连接工厂（ConnectionFactory）是由管理员创建，并绑定到 JNDI 树中。客户端使用 JNDI 查找连接工厂，然后利用连接工厂创建一个 JMS 连接。
                     先根据 mqServerIp , mqServerPort, mqServerUser, mqServerPwd 创建连接使用的工厂类 JMS ConnectionFactory
          b.JMS 连接。JMS 连接（Connection）表示 JMS 客户端和服务器端之间的一个活动的连接，是由客户端通过调用连接工厂的方法建立的。
                      使用管理对象JMS ConnectionFactory, 建立 cms::Connection(连接Connection), 并启动 start
          c.JMS 会话。JMS会话（Session）表示 JMS 客户与 JMS 服务器之间的会话状态。JMS 会话建立在 JMS 连接上，表示客户与服务器之间的一个会话线程。
                      由于会话是单线程的，所以消息是连续的，消息是按照发送的顺序一个一个接收的。会话的好处是它支持事务。如果用户选择了事务支持，
                      会话上下文将保存一组消息，直到事务被提交才发送这些消息。在提交事务之前，用户可以使用回滚操作取消这些消息。
                      一个会话允许用户创建消息生产者来发送消息，创建消息消费者来接收消息.
                      使用 cms::Connection(连接Connection),建立会话 Session(cms::Session)
          d.JMS 目的。JMS目的（Destination），又称为消息队列，是实际的消息源。
                      通过会话和队列名创建指定的队列(cms::Destination)
          e.JMS 生产者和消费者。生产者（Message Producer）和消费者（Message Consumer）对象由Session对象创建，用于发送和接收消息。
                               通过 cms::Destination 创建 cms::MessageProducer 
          f.JMS消息通常有两种类型：
              A. PTP 模式，点对点（Point-to-Point）。在点对点的消息系统中，消息分发给一个单独的使用者。点对点消息往往与队列（javax.jms.Queue）相关联。
              B. 发布/订阅（Publish/Subscribe）。发布/订阅消息系统支持一个事件驱动模型，消息生产者和消费者都参与消息的传递。生产者发布事件，
                 而使用者订阅感兴趣的事件，并使用事件。该类型消息一般与特定的主题（javax.jms.Topic）关联
                 
  5. ActiveMQ 的优势
      (1) 与 rpc(remote procedural call) 远程过程调用不同， ActiveMQ 是松耦合的方式, 调用者无需等待被调用者，就能异步
          的执行.
      (2) ActiveMQ 采用异步通信的方式.
```

## ActiveMQ-cpp 简介
```shell
    1. CMS (C++ Messaging Service), CMS 是 C++ 程序与消息中间件进行通信的一种标准接口，可以通过 CMS 接口与类似 ActiveMQ 
       消息中间件进行通信
    2. ActiceMQ-CPP 是 CMS 的一种实现，是一个能够与 ActiveMQ 服务端进行通信的 C++ 客户端库，ActiveMQ-CPP 的架构设计能够支持
       可插拨的传输协议和消息封装格式
```

## CMS 应用程序接口
```shell
    1. ConnectionFactory
            namespace cms {
            
                class Connection;
                class ExceptionListener;
                class MessageTransformer;
            
                class CMS_API ConnectionFactory {
                public:
                    virtual ~ConnectionFactory();
                    /* 采用默认的用户身份创建 conn, 并且默认 message stop mode, 需要显式调用 start method,
                       才能收到 message, 返回值是 Connection 的指针，调用方不用时需要关闭连接和 delete.
                        出现错误时会抛异常
                    */ 
                    virtual cms::Connection* createConnection() = 0;
                    /* 采用默认的用户身份创建 conn, 并且默认 message stop mode, 需要显式调用 start method,
                       才能收到 message, 返回值是 Connection 的指针，调用方不用时需要关闭连接和 delete.
                       出现错误时会抛异常
                       传入的用户名和密码不会对内部默认用户名和密码进行修改，如果下一次使用无参数的 createConnection(),
                       还是会使用内部的用户名和密码
                    */ 
                    virtual cms::Connection* createConnection(const std::string& username, const std::string& password) = 0;
                    /*
                        共性的信息同上，
                        主要是显式传入 connct 的　clientId, 如果 clientId 值为 "", 则内部采用随机数产生
                     */
                    virtual cms::Connection* createConnection(const std::string& username, const std::string& password, const std::string& clientId) = 0;
                    /*
                        设置一个 ExceptionListener 实例，这个实例对该 ConnectionFactory 创建所有 conn 对象都有效   
                    */
                    virtual void setExceptionListener(cms::ExceptionListener* listener) = 0;
                    /*
                      CMS 代码不会拥有 ExceptionListener 指针的所有权，这意味着客户端代码必须确保该对象在已分配 
                       ExceptionListener 的 CMS对 象的生存期内保持有效
                    */
                    virtual cms::ExceptionListener* getExceptionListener() const = 0;
                    /*
                        设置一个 MessageTransformer 实例，这个实例对该 ConnectionFactory 创建所有 conn 对象都有效 
                        CMS 代码不会拥有 MessageTransformer 指针的所有权，这意味着客户端代码必须确保该对象在已分配 
                        MessageTransformer 的 CMS对 象的生存期内保持有效
                     */
                    virtual void setMessageTransformer(cms::MessageTransformer* transformer) = 0;
                    virtual cms::MessageTransformer* getMessageTransformer() const = 0;
            
                public:
                    /*
                        返回的是派生类　ConnectionFactory，　调用者负责去 delete 这个派生类的指针
                    */
                    static cms::ConnectionFactory* createCMSConnectionFactory(const std::string& brokerURI);
            
                };
            
            }
        创建连接工厂, 在通过连接工厂得到 cms::Connection, 这个好处可以隐藏 cms::Connection 的实现方式．
        
    2. Connection
            Connection 代表应用程序和消息服务器之间的通信链路。在获得 ConnectionFactory 后，
            就可以创建一个与 JMS 提供者(例如 activeMq)的连接。
            根据不同的连接类型，连接允许用户创建会话，以发送和接收队列和主题到目标.
            
            namespace cms {
                class ExceptionListener;
                class MessageTransformer;
            
                class CMS_API Connection : public Startable, public Stoppable, public Closeable {
                public:
            
                    virtual ~Connection();
                    /*
                        关闭此连接以及有该连接创建任何 session(包括该 session 产生的生产者及消费者)
                    */
                    virtual void close() = 0;
                    /*
                        获取 ConnectionMetaData*, 调用者对它的生命周期不可控, 用之前需要判断一下是否 NULL
                    */
                    virtual const ConnectionMetaData* getMetaData() const = 0;
                    /*
                        Creates an AUTO_ACKNOWLEDGE Session.
                    */
                    virtual Session* createSession() = 0;
                    /*
                        AcknowledgeMode 种类有:
                            SESSION_TRANSACTED (支持事务，提供回滚等机制)
                    */
                    virtual Session* createSession(Session::AcknowledgeMode ackMode) = 0;
                    virtual std::string getClientID() const = 0;
                    /*
                        分配 conn clientID 最好的方法是通过　ConnectionFactory　创建 connection 时传入 clientID.
                        一般不太建议显式调用 setClientID(), 调用这个 setClientID() 条件很苛刻, 需要在创建 connection
                        后并且对该连接没有任何动作时立即调用 setClientID(), 否则出现编程错误.
                        
                    */
                    virtual void setClientID(const std::string& clientID) = 0;
                    virtual ExceptionListener* getExceptionListener() const = 0;
                    virtual void setExceptionListener(ExceptionListener* listener) = 0;
                    /*
                        对该 connect 下所有 session 都有效，调用者需要对 MessageTransformer 生命周期负责
                    */
                    virtual void setMessageTransformer(cms::MessageTransformer* transformer) = 0;
                    virtual cms::MessageTransformer* getMessageTransformer() const = 0;
                };
            }
            连接支持并发使用，connection 由以下几个作用:
                (1) 提供了与 JMS provider 的之间的连接
                (2) 这个对象提供了客户端身份验证的功能
                (3) 提供 ConnectionMetaData 对象
            通常的做法是创建一个 connection, 在这个 connection 上创建一个或则多个 session, 并且在 session 基础上创建消息发布者，
            或则订阅者.在创建 connection 后初始为 stop mode,不会接受到任何消息．通常流程是在这个 connection 上都创建好 message
            consumers 后在进行 start method, 进行消息接收,这是为了减少 connection 在进行设置时异步收到发生消息冲突．
            A message producer can send messages while a connection is stopped.
            
    3. Session 
         会话是单线程的(支持消息的发生和接收)，所以消息是连续的，消息是按照发送的顺序一个一个接收的。会话支持事务。如果事务支持，
         会话上下文将保存一组消息，直到事务被提交才发送这些消息。在提交事务之前，用户可以使用回滚操作取消这些消息。
         一个会话允许用户创建消息生产者来发送消息，创建消息消费者来接收消息。
         
         namespace cms {
         
             class MessageTransformer;
             class CMS_API Session : public Closeable, public Startable, public Stoppable {
             public:
                 enum AcknowledgeMode {
                     /*
                        当 session call rev message 成功返回, 或则 message listen 成功调用处理函数, 则自动确认.
                      */
                     AUTO_ACKNOWLEDGE,// 使用此确认模式，会话自动进行, 
                     /*
                        当 session call rev message 成功返回, 或则 message listen 成功调用处理函数, 则自动确认.
                        报文回复可能会延迟以提供性能,需要重新发送客户端失败的消息.
                     */
                     DUPS_OK_ACKNOWLEDGE,
                     /*
                        客户端自行调用消息回复方法
                     */
                     CLIENT_ACKNOWLEDGE,
                     /*
                        当事务提交时,消息会被消费
                     */
                     SESSION_TRANSACTED,
                     /*
                        单独确认每一条消息.
                        与之相对的是发送一条 ack 确认,处理对这条进行确认,还会对此之前消息都进行确认.
                     */
                     INDIVIDUAL_ACKNOWLEDGE
                 };
         
             public:
                 virtual ~Session();
                 /*
                    关闭 session, 以及对应下面的消费者和生产者
                 */
                 virtual void close() = 0;
                 /*
                    通过 session 进行消息事务的提交,并且释放掉持有的锁.
                    注意： 该 session 的 mode 应该为 SESSION_TRANSACTED
                 */
                 virtual void commit() = 0;
                 /*
                    session 的回滚.
                 */
                 virtual void rollback() = 0;
                 /*
                    进行操作:
                        (1) 停止消息的传输
                        (2) 将所有传输过的但为进行 ack 恢复的消息标记为 redelivered
                        (3) 重新按顺序传递消息，包括未 ack 的消息
                        
                    注意: 该　session 的 mode 不能为 SESSION_TRANSACTED
                 */
                 virtual void recover() = 0;
                 /*
                    destination 为接收消息的来源，
                    调用者需要自行管理 MessageConsumer* 的生命周期, 例如自己去 delete
                 */
                 virtual MessageConsumer* createConsumer(const Destination* destination) = 0;
                 /*
                    selector 为消息的选择器
                 */
                 virtual MessageConsumer* createConsumer(const Destination* destination, const std::string& selector) = 0;
                 /*
                    noLocal : 条件是 destination 一定为 topic 类型， 如果是 queue 有问题. 
                              值为 true, 代表不会收到由自己 connection 发布的消息.
                 */
                 virtual MessageConsumer* createConsumer(const Destination* destination, const std::string& selector, bool noLocal) = 0;
                 /*
                    通过 selector , 创建一个可持久化的 topic 的订阅者, 当程序重新建立 connection(clientId), 要使用之前相同的
                    clientId, 这样就能收到断开连接后的消息.
                 */
                 virtual MessageConsumer* createDurableConsumer(const Topic* destination, const std::string& name, const std::string& selector, bool noLocal = false) = 0;
                 virtual MessageProducer* createProducer(const Destination* destination = NULL) = 0;
                 /*
                    创建一个 QueueBrower 来查看 message 
                 */
                 virtual QueueBrowser* createBrowser(const cms::Queue* queue) = 0;
                 /*
                    selector: 筛选某些消息
                 */
                 virtual QueueBrowser* createBrowser(const cms::Queue* queue, const std::string& selector) = 0;
                 /*
                    创建一个 queue 类型, Queue* 需要调用者自行管理其生命周期
                 */
                 virtual Queue* createQueue(const std::string& queueName) = 0;
                 virtual Topic* createTopic(const std::string& topicName) = 0;
                 /*
                     创建一个 TemporaryQueue* 需要调用者自行管理其生命周期
                 */
                 virtual TemporaryQueue* createTemporaryQueue() = 0;
                 virtual TemporaryTopic* createTemporaryTopic() = 0;
                 virtual Message* createMessage() = 0;
                 virtual BytesMessage* createBytesMessage() = 0;
                 virtual BytesMessage* createBytesMessage(const unsigned char* bytes, int bytesSize) = 0;
                 virtual StreamMessage* createStreamMessage() = 0;
                 virtual TextMessage* createTextMessage() = 0;
                 virtual TextMessage* createTextMessage(const std::string& text) = 0;
                 virtual MapMessage* createMapMessage() = 0;
                 /*
                    获取 session 的 AcknowledgeMode
                 */
                 virtual AcknowledgeMode getAcknowledgeMode() const = 0;
                 /*
                    判断这个 session 是不是事务型(SESSION_TRANSACTED)
                 */
                 virtual bool isTransacted() const = 0;
                 /*
                    取消持久化订阅端,与 createDurableConsumer() 相对应.这个方法是将 Provider 端状态进行删除.
                    当在对应的订阅 topic 有活跃的 MessageConsumer 或则 Subscriber ,或则在事务的过程中,或则消费的
                    消息还没被 ack , 则调用 unsubscribe() 会报错.
                 */
                 virtual void unsubscribe(const std::string& name) = 0;
                  /*
                    对该 session 下所有 MessageConsumer, MessageProducer 都有效，
                    调用者需要对 MessageTransformer 生命周期负责
                   */
                 virtual void setMessageTransformer(cms::MessageTransformer* transformer) = 0;
                 virtual cms::MessageTransformer* getMessageTransformer() const = 0;
             };
         }
         #endif /*_CMS_SESSION_H_*/
        session 提供以下作用:
            (1) 通过它来提供消息生产者和消费者
            (2) 提供 TemporaryTopics and TemporaryQueues.
            (3) 通过传入 destination_name 创建 Queue 或则 topic
            (4) 支持事务操作，保证消息的有序性
            (5) 可以保存消费后的消息，直到该消息被确认．
            
        (1) 一个常用的使用方法是，有一个 thread block 进行一个　MessageConsumer 类的消息同步处理，同时可以进行多个 MessageProducers
            的发送.
        (2) 如果客户端想要通过一个 product message 供其他多个 consumer 进行消费，则建议单独创建一个 session 进行发送消息．
            对于已经关闭的 session 无需再关闭 producers 和 consumers. close 方法会一直阻塞直到接收消息或则 message listen 
            调用完成. 当 session 已经关闭时, blocked message consumer 会收到 NULL.要关闭一个事务 session ,必须要先回滚事务.
            因为 session 是 单线程的,所以 close 方法是唯一一个在多个线程并发操作的方法,其他的 session 的方法调用已经
            关闭的 session 抛 IllegalStateException 异常. 关闭已关闭的会话不得引发任何异常.
        (3) 对于一个 MessageProducer , 通过 producer 发送的消息不会发送到 commit 的 Provider unit 中, 
            当事务回滚时 produce 产生的 message 都会被丢弃.
        (4) 对于一个 MessageConsumer 而言, 一个接收事务没有 commit 提交其接收的消息是不会应答,所以当事务回滚时客户端会
            重新收到相同的消息 the Provider 可以规定重传会应答消息的最大个数.
        (5) 虽然 Session 类提供了 Startable and Stoppable 的接口,但这些接口不需要实现, 如果对应的 CMS provider 实现不可用,
            则调用 Startable and Stoppable 接口会抛 UnsupportedOperation 异常.
            
    4. Destination
            消息目标(message Destination)是指消息发布和接收的地点(是队列，或者是主题)
                class CMS_API Destination {
                public:
                    enum DestinationType {
                        TOPIC,
                        QUEUE,
                        TEMPORARY_TOPIC,
                        TEMPORARY_QUEUE
                    };
            
                public:
                    virtual ~Destination();
                    /*
                        获取 DestinationType 类型
                    */
                    virtual DestinationType getDestinationType() const = 0;
                    /*
                        连同 DestinationType 和 content 一起拷贝
                    */
                    virtual cms::Destination* clone() const = 0;
                    /*
                        只拷贝 content 
                    */
                    virtual void copy(const cms::Destination& source) = 0;
                    virtual bool equals(const cms::Destination& other) const = 0;
                    virtual const CMSProperties& getCMSProperties() const = 0;
                };
                
            (1) 所有 CMS DestinationType 支持并发访问
            (2) 继承关系:
                    class CMS_API Queue : public Destination
                    class CMS_API Topic : public Destination
    5. MessageConsumer
            由会话(session)创建的对象，用于接收 发送到目标(Destination ) 的消息。消费者可以同步地（阻塞模式），
            或异步（非阻塞）接收队列和主题类型的消息
            
            namespace cms {
            
                class MessageTransformer;
                class CMS_API MessageConsumer : public Closeable, public Startable, public Stoppable {
                public:
                    virtual ~MessageConsumer();
                    /*
                        同步接收消息, Message* 需要自行管理生命周期
                    */
                    virtual Message* receive() = 0;
                    /*
                        阻塞等待 millisecs 时间, timeout 过后还没有收到消息,则返回 NULL 
                    */
                    virtual Message* receive(int millisecs) = 0;
                    /*
                        非阻塞等待,在调用的哪个时刻看是否消息,没有则返回 NULL
                    */
                    virtual Message* receiveNoWait() = 0;
                    virtual void setMessageListener(MessageListener* listener) = 0;
                    virtual MessageListener* getMessageListener() const = 0;
                    virtual std::string getMessageSelector() const = 0;
                    /*
                        设置 MessageTransformer 实例, 该实例应用于所有的 cms::Message 对象
                    */
                    virtual void setMessageTransformer(cms::MessageTransformer* transformer) = 0;
                    virtual cms::MessageTransformer* getMessageTransformer() const = 0;
                    /*
                        设置 MessageAvailableListener 实例, 其 consumer 处于 synchronous 消费模式,这样
                        当新的 message 到达时, 会将 events 传给 MessageAvailableListener 这个对象.
                    */
                    virtual void setMessageAvailableListener(cms::MessageAvailableListener* listener) = 0;
                    virtual cms::MessageAvailableListener* getMessageAvailableListener() const = 0;
                };
            }
            (1) MessageConsumer 可以同步阻塞接收消息, 可以通过调用 receive()/receive(int millisecs)/receiveNoWait()
                函数来请求下一个消息.
            (2) MessageConsumer 异步接收消息, 通过 setMessageListener() 函数进行将 MessageListener* listener 注册,
                这样当消息到达时,会触发 MessageListener.onMessage 的回调函数.
            (3) 如果 MessageConsumer 的异步消息正在进行或则同步 receive() 函数正在运行, MessageConsumer.close 会被阻塞,
                直到它们完成. 如果 MessageConsumer.close 已经被调用,则再调用 receive() 会返回 NULL.
            (4) 虽然 MessageConsumer 类提供了 Startable and Stoppable 的接口,但这些接口不需要实现, 
                如果对应的 CMS provider 实现不可用, 则调用 Startable and Stoppable 接口会抛 UnsupportedOperation 异常.
               
    6. MessageProducer
            由会话(session)创建的对象，用于发送消息(message)到目标(Destination)。
            用户可以创建某个目标(Destination)的发送者(MessageProducer)，
            也可以创建一个通用的发送者，在发送消息(message)时指定目标(Destination)
            
            namespace cms {
                class MessageTransformer;
                class CMS_API MessageProducer : public Closeable {
                public:
                    virtual ~MessageProducer();
                    /*
                        向 MessageProducer 初始化时 Destination 发送消息, 调用者需要自行管理 Message* 的生命周期
                        (要调用者删除).使用默认的发送方式(delivery mode), 优先级(priority), 消息发送的存活时间(ms)
                        在发送失败时会抛异常(CMSException, MessageFormatException, InvalidDestinationException, 
                        UnsupportedOperationException)
                    */
                    virtual void send(Message* message) = 0;
                    /*
                        向 MessageProducer 初始化时 Destination 发送消息, 调用者需要自行管理 Message* 的生命周期
                        (要调用者删除).使用默认的发送方式(delivery mode), 优先级(priority), 消息发送的存活时间(ms)
                        在发送失败时会抛异常(CMSException, MessageFormatException, InvalidDestinationException, 
                        UnsupportedOperationException)
                        参数 AsyncCallback* 的设置, 这个 send 方法会立马返回,当 provider 有消息 ack 或则错误时, 
                        会通知到 AsyncCallback 对象.
                        注意: 这个 onComplete 在发送完成后一定要考虑 delete, 或则 connection 断开后也要释放掉
                    */
                    virtual void send(Message* message, AsyncCallback* onComplete) = 0;
                    /*
                        timeToLive : 单位是 ms 
                     */
                    virtual void send(Message* message, int deliveryMode, int priority, long long timeToLive) = 0;
                    virtual void send(Message* message, int deliveryMode, int priority,
                                      long long timeToLive, AsyncCallback* onComplete) = 0;
                    /*
                        发送消息到指定的目的地, message 生命周期需要自行管理(自主删除)
                    */
                    virtual void send(const Destination* destination, Message* message) = 0;
                    virtual void send(const Destination* destination, Message* message, AsyncCallback* onComplete) = 0;
                    virtual void send(const Destination* destination, Message* message,
                                      int deliveryMode, int priority, long long timeToLive) = 0;
                    virtual void send(const Destination* destination, Message* message, int deliveryMode,
                                      int priority, long long timeToLive, AsyncCallback* onComplete) = 0;
                                      
                    /*
                        cms::DeliveryMode::PERSISTENT 可持久化(在消息的传输中不能被丢失)
                        cms::DeliveryMode::NON_PERSISTENT 不可持久化
                        这个 DeliveryMode 只适用于发送端,不适用于接收端,不管发送端设置为 cms::DeliveryMode::PERSISTENT
                        接收端可以因为内存的限制,消息过滤而将消息丢弃.
                        
                    */
                    virtual void setDeliveryMode(int mode) = 0;
                    virtual int getDeliveryMode() const = 0;
                    /*
                        设置是否使能 MessageID for product
                        value:
                                true: 使能
                                false: 禁止
                    */
                    virtual void setDisableMessageID(bool value) = 0;
                    virtual bool getDisableMessageID() const = 0;
                    /*
                        设置是否使能 MessageTimeStamp for product
                        value:
                                true: 使能
                                false: 禁止
                    */
                    virtual void setDisableMessageTimeStamp(bool value) = 0;
                    virtual bool getDisableMessageTimeStamp() const = 0;
                    virtual void setPriority(int priority) = 0;
                    virtual int getPriority() const = 0;
                    /*
                        设置默认消息发送的存活时间,如果 send 函数没有显式带有存活时间,
                        则用这个默认存活时间.
                    */
                    virtual void setTimeToLive(long long time) = 0;
                    virtual long long getTimeToLive() const = 0;
                    /*
                        设置 MessageTransformer 实例, 该实例应用于所有的 cms::Message 对象
                    */
                    virtual void setMessageTransformer(cms::MessageTransformer* transformer) = 0;
                    virtual cms::MessageTransformer* getMessageTransformer() const = 0;
                };
            }
            (1) 通过 MessageProducer 对象将 message 发送到 Destination. MessageProducer 对象通过 session 的
            　　 createProducer(const Destination* destination = NULL)  进行创建.
            (2) MessageProducer 创建时可以不带 Destination, 但要求每次发送都必须带 Destination, 这种发送方式常用于
            　　 在收到消息进行应答时需要指定的 Destination
            (3) MessageProducer 可以设置消息的发送方式(delivery mode), 优先级(priority), 消息发送的存活时间(ms)
                
            
```

## c++ ActiveMQ 客户端使用

```shell
    1. 
       m_mqUrl = "tcp://ip:port";
       // 创建连接工厂
       ActiveMQConnectionFactory* pConnectionFactory = new (std::nothrow) ActiveMQConnectionFactory(m_mqUrl, strMqUserName, strMqPwd);
       // 在连接工厂上创建连接
       cms::Connection *m_pconnection = pConnectionFactory->createConnection();
       // 开始连接服务
       m_pconnection->start();
       
       // 创建生产者
       // 创建一个 seesion
       cms::Session* producer_notifySession = m_pconnection->createSession(cms::Session::AUTO_ACKNOWLEDGE);
       // 创建一个队列名
       cms::Destination* producer_notifyDestination = producer_notifySession->createTopic("producer_queue_name");
       
       // 在 session 中根据队列,创建生产者
       cms::MessageProducer* producer_notifyProducer = producer_notifySession->createProducer(producer_notifyDestination);
       // 设置生产者的配置信息
       producer_notifyProducer->setDeliveryMode((int)cms::DeliveryMode::NON_PERSISTENT);
       producer_notifyProducer->setTimeToLive(1000 * 60 * 10);
       // 发送消息
       TextMessage* textMsg= producer_notifySession->createTextMessage(message);
       producer_notifyProducer->send(textMsg);
       
       // 创建消费者
       // 创建一个 seesion
       cms::Session* consumer_pConsumerSession = m_pconnection->createSession(cms::Session::AUTO_ACKNOWLEDGE);
       // 创建一个队列名
       cms::Destination* consumer_pConsumerDestination = consumer_pConsumerSession->createTopic("consumer_queue_name");
        // 在 session 中根据队列,创建消费者
       cms::MessageConsumer* consumer_pConsumerConsumer = consumer_pConsumerSession->createConsumer(consumer_pConsumerDestination);
       // 设置消息接受的回调函数
       consumer_pConsumerConsumer->setMessageListener(this);
       // this 是 cms::MessageListener 对象,需要覆盖写 cms::MessageListener::onMessage
       
```

## ActiveMQ 特性
```shell
  1. JMX 管理（java Management Extensions，即 java 管理扩展）
  2. 主从管理（master/salve，这是集群模式的一种，主要体现在可靠性方面，当主中介（代理）出现故障，
     那么从代理会替代主代理的位置，不至于使消息系统瘫痪）
  3. 消息组通信（同一组的消息，仅会提交给一个客户进行处理）
  4. 有序消息管理（确保消息能够按照发送的次序被接受者接收）。
  5. 消息优先级（优先级高的消息先被投递和处理）
  6. 订阅消息的延迟接收（订阅消息在发布时，如果订阅者没有开启连接，那么当订阅者开启连接时，
     消息中介将会向其提交之前的，其未处理的消息）
  7. 接收者处理过慢（可以使用动态负载平衡，将多数消息提交到处理快的接收者，这主要是对PTP消息所说）
  8. 虚拟接收者（降低与中介的连接数目）
  9. 成熟的消息持久化技术（部分消息需要持久化到数据库或文件系统中，当中介崩溃时，信息不会丢失）
  10. 支持游标操作（可以处理大消息）
  11. 支持消息的转换
  12. 通过使用 Apache 的 Camel 可以支持 EIP
  13. 使用镜像队列的形式轻松的对消息队列进行监控等
```

## ActiveMQ 存储
```shell
  1. AMQ 消息存储(默认消息存储)
        基于文件存储的消息数据库并且不依赖第三方数据库。配置如下
        <amqPersistenceAdapter directory="${activemq.base}/data" maxFileLength="32mb"/>
  2. KahaDB 消息存储
        提供容量的提升和恢复能力, 配置如下
        <kahaDB directory="${activemq.data}/kahadb" />
  3. JDBC 消息存储
        基于JDBC存储
        <persistenceAdapter>
        <jdbcPersistenceAdapter  dataSource="#mysql-ds"/></persistenceAdapter>
        <bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource"

                            destroy-method="close"> <property name="driverClassName" value="com.mysql.jdbc.Driver"/>

                            <property name="url" value="jdbc:mysql://localhost/activemq?relaxAutoCommit=true"/>

                            <property name="username" value="activemq"/> <property name="password" value="activemq"/>

                            <property name="maxActive" value="200"/> <property name="poolPreparedStatements"

                            value="true"/>

        </bean>
  4. Memory 消息存储
        基于内容的消息存储,将消息保存在内存中， 只要将 Broker 的 “prsistent” 属性设置为 “false” 即可。
```

## ActiveMQ 负载均衡
```shell
  1. ActiveMQ 可以实现多个 mq 之间进行路由，假设有两个 mq，分别为 brokerA 和 brokerB，当有一条消息发送到 brokerA 
     的队列 test 中，有一个客户端连接到 brokerB 上，并且要求获取 test 队列的消息时，brokerA 中队列 test 的消息
     就会路由到 brokerB 上
```

## ActiveMQ 主从配置
```shell
  1. Master-Slave 模式分为三类：Pure Master Slave、Shared File System Master Slave 和 JDBC Master Slave。
     以上三种方式的集群都不支持负载均衡，但可以解决单点故障的问题，以保证消息服务的可靠性。
  2. Pure Master Slave 模式
        需要两个 Broker，一个作为 Master，另一个作为 Slave，运行时，Slave 通过网络实时从 Master 处复制数据
        如果 Slave 和 Master 失去连接，Slave 就会自动升级为 Master，继续为客户端提供消息服务，
        这种方式的 Slave 只能有一个。这种方式的主备不需要对 Master Broker 做特殊的配置，只要在 Slave Broker 
        中指定他的 Master 就可以了，指定 Master 有两种方式，
        第一种最简单的配置就是在 broker 节点中添加 masterConnectorURI =”tcp://localhost:61616″
        有一种方式就是添加一个 services 节点，可以指定连接的用户名和密码，配置如下：
          <services>
              <masterConnector remoteURI= "tcp://localhost:61616" userName="system" password="manager"/>
          </services>
   3. SharedFile System Master Slave
          SharedFile System Master Slave 就是利用共享文件系统做 ActiveMQ 集群，基于 ActiveMQ 的默认数据库 kahaDB 
          完成的，kahaDB 的底层是文件系统。这种方式的集群，Slave 的个数没有限制，哪个 ActiveMQ 实例先获取
          共享文件的锁，那个实例就是 Master，其它的 ActiveMQ 实例就是 Slave，如果当前的 Master 失效，
          其它的 Slave 就会去竞争共享文件锁，谁竞争到谁就是 Master。这种模式的好处就是当 Master 失效时不用手动去配置，
          只要有足够多的 Slave。如果各个 ActiveMQ 实例需要运行在不同的机器，就需要用到分布式文件系统
   4. JDBCMaster Slave
          JDBCMaster Slave 模式和 Shared File Sysytem Master Slave 模式的原理是一样的，只是把共享文件系统
          换成共享数据库。在所有的 ActiveMQ 的主配置文件中添加数据源，所有的数据源都指向同一个数据库，
          然后修改持久化适配器。这种方式的集群相对 Shared File System Master Slave 更加简单，更加容易地进行分布式部署，
          但是如果数据库失效，那么所有的 ActiveMQ 实例都将失效。
```
## ActiveMQ 拦截器
```shell
  1. ActiveMQ 中使用拦截器和过滤器的使用多采用插件的形式实现
  2. 日志拦截器
        日志拦截器是 Broker 的一个拦截器，默认的日志级别为 INFO。这个日志拦截器支持 Commons-log 和 Log4j 两种日志
        <plugins>
            <loggingBrokerPlugin logAll="true" logConnectionEvents="false"/>
        </plugins>
        
        logAll: 记录所有事件的日志
  3. 统计拦截器
```

## ActiveMQ 安全配置
```shell
  1. ActiveMQ 可以对各个主题和队列设置用户名和密码
  2.<plugins>

  <!-- Configure authentication; Username, passwords and groups -->

  <simpleAuthenticationPlugin>

      <users>

          <authenticationUser username="testUser" password="123456" groups="testGroup"/>

      </users>

  </simpleAuthenticationPlugin>

  <!--  Lets configure a destination based authorization mechanism -->

  <authorizationPlugin>

    <map>

      <authorizationMap>

        <authorizationEntries>

          <authorizationEntry queue="queue.group.uum" read="users" write="users" admin="users" />

        </authorizationEntries>

      </authorizationMap>

    </map>

  </authorizationPlugin>

</plugins>
```
## ActiveMQ 消息游标(Pending Message Cursors)
```shell
  1. 当 producer 发送的持久化消息到达 broker 后，broker 会先把它保存在持久存储中。如果发现当前有活跃的 consumer，
     而且这个 consumer 消费消息的速度能跟上 producer 生产消息的速度，那么 ActiveMQ 会直接把消息传递给 broker 内部
     跟这个 consumer 关联的 dispatch（派遣、调度） queue；如果当前没有活跃的 consumer 或者 consumer 消费消息的速度
     跟不上 producer 生产消息的速度，那么 ActiveMQ 会使用 Pending Message Cursors 保存对消息的引用。在需要的时候，
     Pending Message Cursors 把消息引用传递给 broker 内部跟这个 consumer 关联的 dispatch queue。
     以下是两种Pending Message Cursors：
        (1) VM Cursor。在内存中保存消息的引用。
        (2) File Cursor。首先在内存中保存消息的引用，如果内存使用量达到上限， 那么会把消息引用保存到临时文件中。

```

## ActiveMQ 严格调度策略（Strict order dispatch policy）
```shell
  1. 由于网络原因，不同的消费者(consumer) 可能受到消息的顺序不同, P 发送了 P1、P2 和 P3 三个消息；
     Q 发送 Q1 和Q2 两个消息。两个不同的 consumer 可能会以以下顺序接收到消息：
     consumer1: P1 P2 Q1 P3 Q2
     consumer2: P1 Q1 Q2 P2 P3
     如果使用了 Strict order dispatch policy , 则两个消费者都会收到相同的顺序，例如 P1 P2 Q1 P3 Q2
  2. 配置文件
         <destinationPolicy>

                 <policyMap>

                     <policyEntries>

                        <policyEntry   topic="FOO.>">

                             <dispatchPolicy>

                                <strictOrderDispatchPolicy  />

                              </dispatchPolicy>

                          </policyEntry>

                     </policyEntries>

                </policyMap>

           </destinationPolicy>

```

## ActiveMQ 轮转调度策略(roundrobin)
```shell
  1. ActiveMQ 的缺省参数是针对处理大量消息时的高性能和高吞吐量而设置的。所以缺省的 prefetch 参数比较大，
     而且缺省的 dispatch policies 会尝试尽可能快的填满 prefetch 缓冲.
  2. 一个业务场景, 少量的消息而且单个消息的处理时间比较长，那么在缺省的 prefetch 和 dispatch policies 下，
     这些少量的消息总是倾向于被分发到个别的 consumer 上。这样就会因为负载的不均衡分配而导致处理时间的增加.
     这种情况下采用轮转的方式比较符合这个业务
  3. 
    <destinationPolicy>     

          <policyMap>     

             <policyEntries>     

                 <policyEntry  topic="FOO.>">     

                      <dispatchPolicy>     

                         <roundRobinDispatchPolicy  />     

                    </dispatchPolicy>     

                 </policyEntry>     

              </policyEntries>     

       </policyMap>     

    </destinationPolicy>

```

## ActiveMQ 消费者设置异步/同步消息
```shell
  1. 根据不同的业务场景对 consumer(消费者) 设置不同的发送机制, 对于 slow Consumer , 则使用异步的消息传递.
  2. 用 Connection URI 来配置 Async 如下：
        ActiveMQConnectionFactory("tcp://locahost:61616?jms.useAsyncSend=true");
     用 ConnectionFactory 配置 Async 如下：
        ((ActiveMQConnectionFactory)connectionFactory).setUseAsyncSend(true);
     用 Connection 配置 Async 如下：
        ((ActiveMQConnection)connection).setUseAsyncSend(true);

```
## ActiveMQ 支持批量确认消息(默认)
```shell
  1. 批量确认消息更有利于性能优化,禁用的操作
     cf = new ActiveMQConnectionFactory ("tcp://locahost:61616?jms.optimizeAcknowledge=false"); 
    ((ActiveMQConnectionFactory)connectionFactory).setOptimizeAcknowledge(false); 
    ((ActiveMQConnection)connection).setOptimizeAcknowledge(false); 

```

## ActiveMQ destination（队列）
```shell
  1. ActiveMQ 的混合发送模式
     a. 允许一个虚拟的 destination 代表多个 destinations，多个 destination之间用“，”分割
        Queue queue = new ActiveMQQueue("USERS.First,USERS.Sconder");
     b. 如果需要不同类型的destination，需要加上前缀queue:// 或topic://
        Queue queue = new ActiveMQQueue("USERS.First,USERS.Sconder,topic://USERS.topic1");
  2. ActiveMQ 的接收 Mirrored 模式(影子模式)
        为了监视生产者和消费者之间的消息流, ActiveMQ 支持 Mirrored Queues。Broker 会把发送到某个 queue 的所有消
        息转发到一个名称类似的 topic，因此监控程序可以订阅这个 mirrored queue topic。
        操作流程:
            先要将 BrokerService 的 useMirroredQueues 属性设置成 true，然后可以通过 destinationInterceptors 
            设置其它属性
            <broker xmlns="http://activemq.apache.org/schema/core"
            brokerName="localhost" dataDirectory="${activemq.data}"
            useMirroredQueues="true">
            <amq:destinationInterceptors>
                <amq:mirroredQueue copyMessage="true" postfix="Mirror.Topic">
                </amq:mirroredQueue>
            </amq:destinationInterceptors>
   3. ActiveMQ 的接收 Wildcards 模式
          Wildcards 通配符: "." 用于作为路径上名字间的分隔符, "*" 用于匹配路径上的任何名字。
                            ">" 用于递归地匹配任何以这个名字开始的 destination。

```

## ActiveMQ 消费者特性
```shell
  1. 独有消费者或者独有队列(Exclusive consumer)
        考虑到一个一个业务场景, 多个 consumer (消费者，不同的线程)从相同的 queue 提前消息，但每个队列里面每个消息是
        有先后顺序的, 所以不能让各个 consumer 取到消息的一部分, 可以设置独有消费者, Broker 会从多个 consumers 中挑选一个
        consumer 来处理 queue 中所有的消息，从而保证了消息的有序处理。如果这个 consumer 失效，那么 broker 会自动切换到其它的 consumer. 
        queue  =  new  ActiveMQQueue("TEST.QUEUE?consumer.exclusive=true");     
        consumer  =  session.createConsumer(queue);
  2. Message Group
        Message Groups 可以看成是一种并发的 Exclusive Consumer。JMS 消息属性JMSXGroupID 被用来区分 message group。
        Message Groups 特性保证所有具有相同 JMSXGroupID 的消息会被分发到相同的 consumer（只要这个consumer 保持active）。
        在一个消息被分发到 consumer 之前，broker 首先检查消息 JMSXGroupID 属性。如果存在，broker 会检查是否有某个 
        consumer 拥有这个 message group。如果没有, broker 会选择一个consumer，并将它关联到这个 message group。此后，
        这个 consumer 会接收这个 message group 的所有消息，直到：Consumer 被关闭
        开启 Message Group：
            TextMessage message = session.createTextMessage("ActiveMq 发送的消息");
            message.setStringProperty("JMSXFroupID", "TEST_GROUP_A");
        关闭 Message Group：
            TextMessage message = session.createTextMessage("ActiveMq 发送的消息");
            message.setStringProperty("JMSXFroupID", "TEST_GROUP_A");
            message.setIntProperty("JMSXGroupSeq", -1);
  3. Message Slelectors
        在订阅中，基于消息属性和 Xpath 语法对进行消息的过滤。
        consumer = session.createConsumer(destination,  "JMSType  =  'car'  AND  weight  >  2500");

```

## ActiveMQ 消息的优先级
```shell
  1. 定义了十个消息优先级值，0 是最低的优先级，9 是最高的优先级。客户端应当将0‐4  看作普通优先级，5‐9  看作加急优先级.
    queue = new ActiveMQQueue("TEST.QUEUE?consumer.priority=10");
    consumer = session.createConsumer(queue);
```

## 总结
```shell
  1. 在配置好 xml 的情况下，activemq 对消息传输上是没有问题，发送的消息都可以全部接收，发送多少条就接收多少条，
     持久模式支持断电续传功能。虽然功能上没有什么问题但对 cpu 的占用率就比较大了，发送或接受消息的时候都达到了100%，
     内存到不会很大
```