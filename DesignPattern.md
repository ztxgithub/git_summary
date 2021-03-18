# 设计模式

# 创建型

## 单例模式
```shell
    1.概念
            (1) 
                为什么要使用单例？
                单例存在哪些问题？
                单例与静态类的区别？
                有何替代的解决方案？
                
            (2) 
                一个类只允许创建一个对象(或者实例), 那这个类就是一个单例类, 这种设计模式就叫作单例设计模式, 简称单例模式
                
            (3) 
                如何理解单例模式中的唯一性？
                如何实现线程唯一的单例？
                如何实现集群环境下的单例？
                如何实现一个多例模式？
            
    2. 单例模式目的
            (1) 处理资源访问冲突
                    如果多个线程要对同一个对象进行操作(向同一个日志文件打印), 则需要加锁, 对固定的同一个对象可以用单例模式. 例如
                 将 Logger 设计成一个单例类, 那么只允许创建一个 Logger 对象, 所有的线程共享使用的这一个 Logger 对象, 共享一个 
                 FileWriter 对象, 而 FileWriter 本身是对象级别线程安全的, 避免多线程情况下写日志会互相覆盖的问题.
                 
            (2) 表示全局唯一类
                    例如配置信息类, 按道理程序中只会有一份这样的信息, 这种情况下可以用单例模式.
                    
    3. 设计单例模式
            (1) 考虑要求
                    a. 构造函数需要是 private 访问权限, 避免外部通过 new 创建实例
                    b. 考虑对象创建时的线程安全问题
                    c. 考虑是否支持延迟加载
                    d. 考虑 getInstance() 性能是否高(是否加锁)
                    
            (2) 饿汉式设计
                    在类加载时, instance 静态实例就已经创建初始化好, 其 instance 实例的创建过程是线程安全的, 这样的实现方式不支持
                    延迟加载(在真正用到 IdGenerator 时, 再创建实例)
                    
                    例如:
                    
                        public class IdGenerator {
                         private AtomicLong id = new AtomicLong(0);
                         private static final IdGenerator instance = new IdGenerator();  // 类加载时单例实例就被初始化
                         
                         private IdGenerator() {}
                         
                         public static IdGenerator getInstance() {
                         return instance;
                         }
                         
                         public long getId() {
                         return id.incrementAndGet();
                         }
                        }
                        
                    优点: 一开始就加载完资源, 一般情况下单例实例在服务过程中都有被用到, 可以考虑在程序刚开始时就进行资源加载, 这样等
                    　　　真正用到的时候省去加载时间.
                    
            (3) 懒汉式设计
                    支持延迟加载.
                    
                    例如:
                    
                        public class IdGenerator {
                         private AtomicLong id = new AtomicLong(0);
                         private static IdGenerator instance;
                         
                         private IdGenerator() {}
                         
                         // 当调用 getInstance() 才触发
                         public static synchronized IdGenerator getInstance() {
                         if (instance == null) {
                         instance = new IdGenerator();
                         }
                         return instance;
                         }
                         
                         public long getId() {
                         return id.incrementAndGet();
                         }
                        }
                        
                    缺点: getInstance() 里面还有创建实例逻辑, 为了防止重复创建实例, 需要加锁, 这样导致这个函数的并发度很低. 
                         如果频繁得调用这个函数, 频繁加锁, 释放锁及并发度低等问题. 会导致性能瓶颈, 这种实现方式就不可取
                         
            (4) 双重检测设计
                    既支持延迟加载, 又支持高并发的单例实现方式, 只要 instance 被创建后, 即便再调用 getInstance() 函数也不会再
                    进入到加锁逻辑中
                    
                    例如:
                        
                        public class IdGenerator {
                         private AtomicLong id = new AtomicLong(0);
                         private static IdGenerator instance;  // 可以加上 volatile, 防止指令重排序
                         
                         private IdGenerator() {}
                         
                         public static IdGenerator getInstance() {
                         if (instance == null) {
                         synchronized(IdGenerator.class) { // 此处为类级别的锁
                         if (instance == null) {
                         instance = new IdGenerator();
                         }
                         }
                         }
                         return instance;
                         }
                         
                         public long getId() {
                         return id.incrementAndGet();
                         }
                        }
                        
                    对于"指令重排序" 问题, 即 存在　IdGenerator 对象被 new 出来, 并且赋值给 instance, 但 IdGenerator 还没来得及
                    初始化(执行构造函数中的代码逻辑), 就被另一个线程使用. 针对这个问题,  在 instance 成员变量加上 volatile 关键字
                    
            (5) 静态内部类设计
                    有点类似饿汉式, 但又能做到延迟加载.
                    例如:
                    
                        public class IdGenerator {
                         private AtomicLong id = new AtomicLong(0);
                         private IdGenerator() {}
                         
                         private static class SingletonHolder{
                         private static final IdGenerator instance = new IdGenerator();
                         }
                         
                         public static IdGenerator getInstance() {
                         return SingletonHolder.instance;
                         }
                         
                         public long getId() {
                         return id.incrementAndGet();
                         }
                        }
                        
                    这种实现方法既保证线程安全, 又能做到延迟加载
                    
            (6) 枚举
                    通过枚举类型本身的特性, 保证实例创建的线程安全性和实例的唯一性.
                    
                    例如:
                    
                        public enum IdGenerator {
                         INSTANCE;
                         private AtomicLong id = new AtomicLong(0);
                         
                         public long getId() {
                         return id.incrementAndGet();
                         }
                        }
                        
    5. 单例模式中的唯一性
            单例模式创建的对象是进程唯一的, 当一个进程 fork() 出子进程时, 单例类在老进程中存在且只能存在一个对象, 在新进程中也会存在且
            只能存在一个对象, 这两个对象并不是同一个对象, 即单例类中对象的唯一性的作用范围是进程内, 在进程间是不唯一的.
            
    6. 实现线程唯一的单例
            进程唯一: 指进程内唯一, 进程间不唯一, 同时在线程内和线程间也是唯一的.
            线程唯一: 指线程内唯一, 线程间可以不唯一.
            
            线程唯一单例的代码实现是通过一个 HashMap 来存储对象, 其中 key 是线程 ID, value 是对象, 不同的线程对应不同的对象, 同一个
            线程只能对应一个对象
            
            例如:
            
                public class IdGenerator {
                 private AtomicLong id = new AtomicLong(0);
                 private static final ConcurrentHashMap<Long, IdGenerator> instances
                 = new ConcurrentHashMap<>();
                 
                 private IdGenerator() {}
                 
                 public static IdGenerator getInstance() {
                 Long currentThreadId = Thread.currentThread().getId();
                 instances.putIfAbsent(currentThreadId, new IdGenerator());
                 return instances.get(currentThreadId);
                 }
                 
                 public long getId() {
                 return id.incrementAndGet();
                 }
                }
                
    7. 集群唯一的单例
            集群相当于多个进程构成的一个集合, "集群唯一" 就相当于是进程内唯一, 进程间也唯一, 即不同的进程间共享同一个对象, 不能创建
        同一个类的多个对象.
        
            具体的实现方式: 
                把这个单例对象序列化并存储到外部共享存储区(比如文件), 进程在使用这个单例对象时, 需要先从外部共享存储区中将它读取到内存,
            并反序列化成对象, 然后再使用, 使用完成之后还需要再存储回外部共享存储区. 为了保证任何时刻, 在进程间都只有一份对象存在, 
            一个进程在获取到对象之后, 需要对对象加锁, 避免其他进程再将其获取. 在进程使用完这个对象后, 还需要显式地将对象从内存中删除, 
            并且释放对对象的加锁
                例如:
                    public class IdGenerator {
                     private AtomicLong id = new AtomicLong(0);
                     private static IdGenerator instance;
                     private static SharedObjectStorage storage = FileSharedObjectStorage(....);
                     private static DistributedLock lock = new DistributedLock();
                     
                     private IdGenerator() {}
                     
                     public synchronized static IdGenerator getInstance() {
                     if (instance == null) {
                     lock.lock();
                     instance = storage.load(IdGenerator.class);
                     }
                     return instance;
                     }
                     
                     public synchroinzed void freeInstance() {
                     storage.save(this, IdGeneator.class);
                     instance = null; //释放对象
                     lock.unlock();
                     }
                     
                     public long getId() {
                     return id.incrementAndGet();
                     }
                    }
                    
                    // IdGenerator使用举例
                    IdGenerator idGeneator = IdGenerator.getInstance();
                    long id = idGenerator.getId();
                    IdGenerator.freeInstance();
                    
    8. 多例模式实现
            "多例" 指的是一个类可以创建多个对象, 但是个数是有限制的.
            
            第一种多例模式: 一个类创建固定个对象
            
                public class BackendServer {
                 private long serverNo;
                 private String serverAddress;
                 
                 private static final int SERVER_COUNT = 3;
                 private static final Map<Long, BackendServer> serverInstances = new HashMaP();
                 
                 static {
                 serverInstances.put(1L, new BackendServer(1L, "192.134.22.138:8080"));
                 serverInstances.put(2L, new BackendServer(2L, "192.134.22.139:8080"));
                 serverInstances.put(3L, new BackendServer(3L, "192.134.22.140:8080"));
                 }
                 
                 private BackendServer(long serverNo, String serverAddress) {
                 this.serverNo = serverNo;
                 this.serverAddress = serverAddress;
                 }
                 
                 public BackendServer getInstance(long serverNo) {
                 return serverInstances.get(serverNo);
                 }
                 
                 public BackendServer getRandomInstance() {
                 Random r = new Random();
                 int no = r.nextInt(SERVER_COUNT)+1;
                 return serverInstances.get(no);
                 }
                }
                
            第二种多例模式: 虽然是同一类, 但是如果属性不同, 则可以实例化出不同的实例.
            
                public class Logger {
                 private static final ConcurrentHashMap<String, Logger> instances
                 = new ConcurrentHashMap<>();
                 
                 private Logger() {}
                 
                 public static Logger getInstance(String loggerName) {
                 instances.putIfAbsent(loggerName, new Logger());
                 return instances.get(loggerName);
                 }
                 
                 public void log() {
                 //...
                 }
                }
                
                //l1==l2, l1!=l3
                Logger l1 = Logger.getInstance("User.class");
                Logger l2 = Logger.getInstance("User.class");
                Logger l3 = Logger.getInstance("Order.class");
                        
```

### 单例模式问题
```shell
    1. 概念
            (1) 
                单例这种设计模式存在哪些问题？
                为什么会被称为反模式？
                如果不用单例, 该如何表示全局唯一类？有何替代的解决方案？
                
    2. 问题一: 单例对 OOP 特性的支持不友好
            单例这种设计模式对于其中的抽象, 继承, 多态都支持得不好, 违背了基于接口而非实现的设计原则.
            例如:
            
                如果 getId 的实现方法发生了改变
                public class Order {
                 public void create(...) {
                 //...
                 long id = IdGenerator.getInstance().getId();
                 // 需要将上面一行代码，替换为下面一行代码
                 long id = OrderIdGenerator.getIntance().getId();
                 //...
                 }
                }
                
                public class User {
                 public void create(...) {
                 // ...
                 long id = IdGenerator.getInstance().getId();
                 // 需要将上面一行代码，替换为下面一行代码
                 long id = UserIdGenerator.getIntance().getId();
                 }
                }
                
    3. 问题二: 单例会隐藏类之间的依赖关系
            通过构造函数, 参数传递等方式声明类之间的依赖关系, 通过查看函数的定义, 就能识别出来, 单例类不需要显示创建, 不需要依赖参数传递, 
            在函数中直接调用单例方法, 如果代码比较复杂, 这种调用关系就会非常隐蔽, 在阅读代码的时候就需要仔细查看每个函数的代码实现, 
            才能知道这个类到底依赖了哪些单例类
            
    4. 问题三: 单例对代码的扩展性不友好
            单例类只能有一个对象实例, 如果以后业务需求, 想拓展为多个实例就很麻烦, 例如数据库连接池, 系统设计初期只设计一个数据库连接池,
       方便控制对数据库连接资源的消耗, 但随着业务不断扩展, 有些 SQL 语句运行得非常慢, 长时间占用数据库连接资源导致其他 SQL 请求无法响应,
       这时就需要用两个数据库连接池, 将快慢的 SQL 分开来.
            数据库连接池、线程池这类的资源池最好不要用单例
            
    5. 问题四: 单例对代码的可测试性不友好
            例如, 单例类依赖的外部资源, 比如 DB, 在写单元测试时, 希望通过 mock 的方式将它替换掉, 而单例类这种硬编码的使用方式, 导致
            无法实现 mock 替换.
            
    6. 问题五: 单例不支持有参数的构造函数
            单例不支持有参数的构造函数, 比如创建一个连接池的单例对象, 没法通过参数来指定连接池的大小.
            
            解决方案一:
                创建完实例后, 再调用 init() 函数传递参数, 在使用这个单例类时, 要先调用 init() 方法, 然后才能调用 getInstance() 
            方法, 否则会代码异常.
                例如:
                
                    public class Singleton {
                     private static Singleton instance = null;
                     private final int paramA;
                     private final int paramB;
                     
                     private Singleton(int paramA, int paramB) {
                     this.paramA = paramA;
                     this.paramB = paramB;
                     }
                     
                     public static Singleton getInstance() {
                     if (instance == null) {
                     throw new RuntimeException("Run init() first.");
                     }
                     return instance;
                     }
                     
                     public synchronized static Singleton init(int paramA, int paramB) {
                     if (instance != null){
                     throw new RuntimeException("Singleton has been created!");
                     }
                     instance = new Singleton(paramA, paramB);
                     return instance;
                     }
                    }
                    
                    Singleton.init(10, 50); // 先init, 再使用
                    Singleton singleton = Singleton.getInstance();
                    
            解决方案二:
                将参数放到 getIntance() 方法中
                例如:
                
                    public class Singleton {
                     private static Singleton instance = null;
                     private final int paramA;
                     private final int paramB;
                     
                     private Singleton(int paramA, int paramB) {
                     this.paramA = paramA;
                     this.paramB = paramB;
                     }
                     
                     public synchronized static Singleton getInstance(int paramA, int paramB) {
                     if (instance == null) {
                     instance = new Singleton(paramA, paramB);
                     }
                     return instance;
                     }
                    }
                    
                    Singleton singleton = Singleton.getInstance(10, 50);
                    
                问题是第一次调用 Singleton.getInstance(10, 50);有效果, 但紧接着第二次调用 Singleton.getInstance(20, 30), 则
                参数没有意义, 不生效.
                
            解决方案三:(推荐使用)
                将参数放到另外一个全局变量中, 参数加载可以是静态定义, 也可以是从配置文件中加载
                
                public class Config {
                    public static final int PARAM_A = 123;
                    public static fianl int PARAM_B = 245;
                }
                
                public class Singleton {
                 private static Singleton instance = null;
                 private final int paramA;
                 private final int paramB;
                 
                 private Singleton() {
                 this.paramA = Config.PARAM_A;
                 this.paramB = Config.PARAM_B;
                 }
                 
                 public synchronized static Singleton getInstance() {
                 if (instance == null) {
                 instance = new Singleton();
                 }
                 return instance;
                 }
                }
                
    7. 单例的替代方案
            如果要实现全局唯一类, 替代方案可以是 工厂模式. 
            
    8. 总结
            (1) 如果单例类并没有后续扩展的需求, 并且不依赖外部系统, 那设计成单例类就没有太大问题
```
