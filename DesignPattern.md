# 设计模式

```shell
    1.  很多设计模式都是试图将庞大的类拆分成更细小的类, 然后再通过某种更合理的结构组装在一起
    2. 一切设计模式重要依据是在确保可读性的情况下, 保证需求频繁变更(新加需求), 有可扩展性, 如果需求很固定, 那么不需要过度设计.
    3. 设计模式要干的事情就是解耦, 创建型模式是将创建和使用代码解耦, 结构型模式是将不同功能代码解耦, 行为型模式是将不同的行为代码解耦.
       借助设计模式, 利用更好的代码结构, 将一大坨代码拆分成职责更单一的小类, 让其满足开闭原则, 高内聚松耦合等特性, 以此来控制和应对
       代码的复杂性, 提高代码的可扩展性, 设计模式就是为了解决复杂代码问题而产生的.
       
    4. 每个设计模式都由两部分组成：
            第一部分是应用场景, 即这个模式可以解决哪类问题；
            第二部分是解决方案, 即这个模式的设计思路和具体的代码实现(代码实现不是必须的)
            
       单纯地只关注解决方案(甚至只关注代码实现), 就会觉得大部分设计模式都很相似(在代码层面)
       
    5. 设计模式之间的主要区别还是在于设计意图(应用场景), 单纯地看设计思路或者代码实现, 有些模式确实很相似, 比如策略模式和工厂模式,
       策略模式包含策略的定义, 创建和使用三部分, 从代码结构上非常像工厂模式, 它们的区别在, 策略模式侧重“策略”或“算法”这个特定的应用场景,
       用来解决根据运行时状态从一组策略中选择不同策略的问题, 而工厂模式侧重封装对象的创建过程, 这里的对象没有任何业务场景的限定, 
       可以是策略, 但也可以是其他东西, 从设计意图上来, 这两个模式完全是两回事儿
       
    6. 应用设计模式只是方法, 最终的目的还是提高代码质量(提高代码的可读性, 可扩展性, 可维护性)
    7. 设计的过程是先有问题后有方案, 先要分析代码存在问题(可读性不好, 可扩展性不好等等), 再针对性利用设计模式去改造.
    8. 需要具备分析问题, 解决问题的能力, 看到某段代码后, 说出好的地方, 不好的地方, 为什么好, 为什么不好, 不好的如何改善, 
       可以应用哪种设计模式, 应用了之后有哪些副作用要控制等等.
    9. 在真正有痛点的时候, 再去考虑用设计模式来解决, 而不是一开始就为不一定实现的未来需求而应用设计模式.
    10. 要学会刻意练习, 针对某个知识点进行实践.
    11. 要有代码质量意识, 设计意识, 在写代码之前, 要多想想未来会有哪些扩展的需求, 哪部分是会变的, 哪部分是不变的, 这样写会不会导致
        之后添加新的功能比较困难, 代码的可读性好不好等代码质量问题
    
```

# 创建型(就是创建对象的模式)
```shell
    1. 创建型模式主要解决对象的创建问题, 封装复杂的创建过程, 解耦对象的创建代码和使用代码
       
        (1) 单例模式用来创建全局唯一的对象.
        (2) 工厂模式用来创建不同但是相关类型的对象(继承同一父类或者接口的一组子类), 由给定的参数来决定创建哪种类型的对象.
        (3) 建造者模式是用来创建复杂对象, 可以通过设置不同的可选参数, “定制化”地创建不同的对象.
        (4) 原型模式针对创建成本比较大的对象, 利用对已有对象进行复制的方式进行创建, 以达到节省创建时间的目的
```

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
                
            (4) 如果单例类并没有后续扩展的需求, 并且不依赖外部系统, 那设计成单例类就没有问题
            
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

## 工厂模式(Factory Design Pattern)
```shell
    1. 概念
            (1) 工厂模式分为简单工厂, 工厂方法和抽象工厂. 简单工厂, 工厂方法在项目中用的比较多.
            (2) 什么时候该用工厂模式？
                相比于直接 new 来创建对象, 用工厂模式来创建有什么好处呢？
                
            (3) 之所以将某个代码块剥离出来, 独立为函数或者类, 原因是这个代码块的逻辑过于复杂, 剥离之后能让代码更加清晰, 更加可读, 
                可维护. 如果代码块本身并不复杂, 就几行代码而已, 就没必要将它拆分成单独的函数或者类
                
            (4) 在简单工厂和工厂方法中, 类只有一种分类方式. 如在规则配置解析的例子中, 解析器类只会根据配置文件格式(Json, Xml, Yaml……)
                来分类
                
    2. 简单工厂(Simple Factory)
            (1) 简单工厂模式的代码实现中, 有多处 if 分支判断逻辑, 虽然违背开闭原则, 但权衡扩展性和可读性, 这样的代码实现在大多数情况下
                (比如, 不需要频繁地添加 parser, 也没有太多的 parser) 是没有问题的
                
            (2) 大部分工厂类都是以 “Factory” 单词结尾的, 工厂类中创建对象的方法一般都是 create 开头, 比如代码中的 createParser(), 
                但有的也命名为 getInstance(), createInstance(), newInstance(), 有的甚至命名为 valueOf()(比如 Java String
               类的 valueOf() 函数）等等.
                
            (3) 简单工厂模式的演变过程
                    原始代码:
                    
                        public class RuleConfigSource {
                         public RuleConfig load(String ruleConfigFilePath) {
                         String ruleConfigFileExtension = getFileExtension(ruleConfigFilePath);
                         
                         // 根据不同文件的后缀名创建不同类对象, part1 
                         IRuleConfigParser parser = null;
                         if ("json".equalsIgnoreCase(ruleConfigFileExtension)) {
                         parser = new JsonRuleConfigParser();
                         } else if ("xml".equalsIgnoreCase(ruleConfigFileExtension)) {
                         parser = new XmlRuleConfigParser();
                         } else if ("yaml".equalsIgnoreCase(ruleConfigFileExtension)) {
                         parser = new YamlRuleConfigParser();
                         } else if ("properties".equalsIgnoreCase(ruleConfigFileExtension)) {
                         parser = new PropertiesRuleConfigParser();
                         } else {
                         throw new InvalidRuleConfigException(
                         "Rule config file format is not supported: " + ruleConfigFilePa
                         }
                         String configText = "";
                         //从ruleConfigFilePath文件中读取配置文本到configText中
                         RuleConfig ruleConfig = parser.parse(configText);
                         return ruleConfig;
                         }
                         
                         
                         private String getFileExtension(String filePath) {
                         //...解析文件名获取扩展名，比如rule.json，返回json
                         return "json";
                         }
                        }
                        
                    
                    重构演变 level1:
                        可以将 part1 封装为一个 createParser() 函数
                        
                    重构演变 level2:
                        让类的职责更加单一, 代码更加清晰, 进一步将 createParser() 函数剥离到一个独立的类, 让这个类只负责对象的创建, 
                        这就是简单工厂模式类
                        例如:
                                
                                public class RuleConfigParserFactory {
                                 private static final Map<String, RuleConfigParser> cachedParsers = new HashMap;
                                 
                                 static {
                                 cachedParsers.put("json", new JsonRuleConfigParser());
                                 cachedParsers.put("xml", new XmlRuleConfigParser());
                                 cachedParsers.put("yaml", new YamlRuleConfigParser());
                                 cachedParsers.put("properties", new PropertiesRuleConfigParser());
                                 }
                                 
                                 public static IRuleConfigParser createParser(String configFormat) {
                                 if (configFormat == null || configFormat.isEmpty()) {
                                 return null;
                                 }
                                 
                                 IRuleConfigParser parser = cachedParsers.get(configFormat.toLowerCase())
                                 return parser;
                                 }
                                }
                                
    3. 工厂方法(Factory Method)
            (1) 就是当一级工厂类里面需要很复杂的 if-else 创建逻辑, 需要简化一级工厂类里面的逻辑代码, 就采用工厂方法, 将创建业务逻辑
            　　下沉到二级工厂类.
            (2) 当创建逻辑比较复杂, 考虑使用工厂模式, 封装对象的创建过程, 将对象的创建和使用相分离, 有以下 2 种复杂情况
                    a. 第一种情况, 类似规则配置解析的例子, 代码中存在 if-else 分支判断, 动态地根据不同的类型创建不同的对象, 
                       针对这种情况, 就考虑使用工厂模式, 将这一大坨 if-else 创建对象的代码抽离出来, 放到工厂类中
                       
                    b. 第二种情况, 尽管不需要根据不同的类型创建不同的对象, 单个对象本身的创建过程比较复杂, 比如前面提到的要组合其他类
                       对象, 做各种初始化操作, 在这种情况下, 也可以考虑使用工厂模式, 将对象的创建过程封装到工厂类中
            
    4. 抽象工厂(Abstract Factory): 不常用
            (1) 对一个产品由多个维度进行划分, 共同组成的.
            (2) 
                public interface IConfigParserFactory {
                 IRuleConfigParser createRuleParser();
                 ISystemConfigParser createSystemParser();
                 //此处可以扩展新的parser类型，比如IBizConfigParser
                }
                
                public class JsonConfigParserFactory implements IConfigParserFactory {
                 @Override
                 public IRuleConfigParser createRuleParser() {
                 return new JsonRuleConfigParser();
                 }
                 
                 @Override
                 public ISystemConfigParser createSystemParser() {
                 return new JsonSystemConfigParser();
                 } 
                }
                
                public class XmlConfigParserFactory implements IConfigParserFactory {
                 @Override
                 public IRuleConfigParser createRuleParser() {
                 return new XmlRuleConfigParser();
                 }
                 
                 @Override
                 public ISystemConfigParser createSystemParser() {
                 return new XmlSystemConfigParser();
                 }
                }
                
                // 省略YamlConfigParserFactory和PropertiesConfigParserFactory代码
                
    5. 使用工厂模式的判断依据
            封装变化：创建逻辑有可能变化, 封装成工厂类之后, 创建逻辑的变更对调用者透明
            代码复用：创建代码抽离到独立的工厂类之后可以复用
            隔离复杂性：封装复杂的创建逻辑, 调用者无需了解如何创建对象
            控制复杂度：将创建代码抽离出来, 让原本的函数或类职责更单一, 代码更简洁
```

## DI 容器
```shell
    1. 概念
            (1) 依赖注入框架, 又叫依赖注入容器(Dependency Injection Container)
            (2) 
                DI 容器跟工厂模式有何区别和联系？
                DI 容器的核心功能有哪些, 以及如何实现一个简单的 DI 容器？
                
            (3) DI 容器其实就是基于工厂模式的设计思路, DI 容器相当于一个大的工厂类, 负责在程序启动时, 根据配置(要创建哪些类对象,
               每个类对象的创建需要依赖哪些其他类对象)事先创建好对象, 当应用程序需要使用某个类对象时, 直接从容器中获取. 正因为持有一
               堆对象, 所以这个框架才被称为"容器".
               
            (4) DI 容器的核心功能有配置解析, 对象创建和对象生命周期管理.
                
    2. DI 容器跟工厂模式区别
            (1) 一个工厂类只负责某个类对象或者某一组相关类对象(继承自同一抽象类或者接口的子类)的创建, 而 DI 容器负责的是整个应用中
                所有类对象的创建.
                
            (2) DI 容器负责的事情要比单纯的工厂模式要多, 如它还包括配置的解析, 对象创建和对象生命周期的管理.
            
    3. 配置的解析
            (1) 工厂类要创建哪个类对象是事先确定好的, 并且是写死在工厂类代码中, 而 DI 容器则是读取配置文件, 根据配置文件提供的信息来
                创建对象, 例如 Spring 容器读取配置文件, 解析出要创建的两个对象：rateLimiter 和 redisCounter, 并且得到两者的依赖关系
                rateLimiter 依赖 redisCounter
                
                例如:
                    public class RateLimiter {
                        private RedisCounter redisCounter;
                        
                        public RateLimiter(RedisCounter redisCounter) {
                            this.redisCounter = redisCounter;
                        }
                        
                        public void test() {
                            System.out.println("Hello World!");
                        }
                        //...
                    }
                    
                    public class RedisCounter {
                        private String ipAddress;
                        private int port;
                        
                        public RedisCounter(String ipAddress, int port) {
                            this.ipAddress = ipAddress;
                            this.port = port;
                        }
                        //...
                    }
                    
                    配置文件beans.xml：
                    <beans>
                        <bean id="rateLimiter" class="com.xzg.RateLimiter">
                            <constructor-arg ref="redisCounter"/>
                        </bean>
                        
                        <bean id="redisCounter" class="com.xzg.redisCounter">
                            <constructor-arg type="String" value="127.0.0.1">
                            <constructor-arg type="int" value=1234>
                        </bean>
                    </beans>
                    
    4. 对象创建
            在 DI 容器中, 所有对象的创建, 都放到一个工厂类中完成就可以了, 比如 BeansFactory, 它能在程序运行的过程中, 动态地加载类, 
            创建对象, 不需要事先在代码中写死要创建哪些对象, 不管是创建一个对象还是十个对象, BeansFactory 工厂类代码都是一样的.
            
    5. 对象的生命周期管理
            通过一些配置参数动态加载
            (1) 通过配置 scope 属性, 来区分这两种不同类型的对象, scope=prototype 表示返回新创建的对象, scope=singleton 表示
                返回单例对象
                
            (2) 配置对象是否支持懒加载, 如果 lazy-init=true, 对象在真正被使用到时(比如：BeansFactory.getBean(“userService”))
                才被被创建；如果 lazyinit=false, 对象在应用启动的时候就事先创建好.
                
            (3) 配置对象的 init-method 和 destroy-method 方法, 比如 init-method=loadProperties(), 
                destroy-method=updateConfigFile(). DI 容器在创建好对象之后, 会主动调用 init-method 属性指定的方法来初始化对象.
                在对象被最终销毁之前, DI 容器会主动调用 destroy-method 属性指定的方法来做一些清理工作, 比如释放数据库连接池、关闭文件
    
```

## 建造者模式
```shell
    1. 概念
            (1) 建造者模式又被称为 Builder 模式, 构建者模式, 生成器模式
            (2) 
                直接使用构造函数或者配合 set 方法就能创建对象, 为什么还需要建造者模式来创建呢？
                建造者模式和工厂模式都可以创建对象, 它们两个的区别在哪里？
                
    2. 建造者模式使用场景
            (1) 在普通的创建对象时, 对于对象内属性是必填的, 则可以通过构造函数传入, 如果是选填的属性, 则可以通过 set() 函数传入, 此时
            　　没有必要用建造者模式
            (2) 需要使用建造者模式场景
                    a. 如果必填的配置项有很多, 把这些必填配置项都放到构造函数中设置, 那构造函数就又会出现参数列表很长的问题, 如果把
                       必填项也通过 set() 方法设置, 那校验这些必填项是否已经填写的逻辑就有问题
                       
                    b. 假设配置项之间有一定的依赖关系, 如, 如果用户设置 maxTotal, maxIdle, minIdle 其中一个, 就必须显式地设置
                       另外两个；或者配置项之间有一定的约束条件, 如 maxIdle 和 minIdle 要小于等于 maxTotal. 如果继续使用构造函数和
                       set() 函数设计思路, 那这些配置项之间的依赖关系或者约束条件的校验逻辑就不好处理
                       
                    c. 希望对象在创建好之后, 就不能再修改内部的属性值, 要实现这个功能, 就不能在 ResourcePoolConfig 类中暴露 
                       set() 方法.
                       
            (3) 具体方法
                    把校验逻辑放置到 Builder 类中, 先创建建造者, 并且通过 set() 方法设置建造者的变量值, 在使用 build() 方法真正
                    创建对象之前, 做集中的校验, 校验通过之后才会创建对象, 把 ResourcePoolConfig 的构造函数改为 private 私有权限,
                    就只能通过建造者来创建 ResourcePoolConfig 类对象, 并且 ResourcePoolConfig 没有提供任何 set() 方法, 这样
                    创建出来的对象就是不可变对象
                    
                    例如:
                        public class ResourcePoolConfig {
                         private String name;
                         private int maxTotal;
                         private int maxIdle;
                         private int minIdle;
                         
                         private ResourcePoolConfig(Builder builder) {
                         this.name = builder.name;
                         this.maxTotal = builder.maxTotal;
                         this.maxIdle = builder.maxIdle;
                         this.minIdle = builder.minIdle;
                         }
                         //...省略getter方法...
                         
                         /***********************************************************/
                         //将 Builder 类设计成 ResourcePoolConfig 的内部类。
                         //也可以将 Builder 类设计成独立的非内部类 ResourcePoolConfigBuilder。
                         public static class Builder {
                         private static final int DEFAULT_MAX_TOTAL = 8;
                         private static final int DEFAULT_MAX_IDLE = 8;
                         private static final int DEFAULT_MIN_IDLE = 0;
                         
                         private String name;
                         private int maxTotal = DEFAULT_MAX_TOTAL;
                         private int maxIdle = DEFAULT_MAX_IDLE;
                         private int minIdle = DEFAULT_MIN_IDLE;
                         
                         public ResourcePoolConfig build() {
                         // 校验逻辑放到这里来做，包括必填项校验、依赖关系校验、约束条件校验等
                         if (StringUtils.isBlank(name)) {
                         throw new IllegalArgumentException("...");
                         }
                         if (maxIdle > maxTotal) {
                         throw new IllegalArgumentException("...");
                         }
                         if (minIdle > maxTotal || minIdle > maxIdle) {
                         throw new IllegalArgumentException("...");
                         }
                         
                         return new ResourcePoolConfig(this);
                         }
                         
                         public Builder setName(String name) {
                         if (StringUtils.isBlank(name)) {
                         throw new IllegalArgumentException("...");
                         }
                         this.name = name;
                         return this;
                         }
                         
                         public Builder setMaxTotal(int maxTotal) {
                         if (maxTotal <= 0) {
                         throw new IllegalArgumentException("...");
                         }
                         this.maxTotal = maxTotal;
                         return this;
                         }
                         
                         public Builder setMaxIdle(int maxIdle) {
                         if (maxIdle < 0) {
                         throw new IllegalArgumentException("...");
                         }
                         this.maxIdle = maxIdle;
                         return this;
                         }
                         
                         public Builder setMinIdle(int minIdle) {
                         if (minIdle < 0) {
                         throw new IllegalArgumentException("...");
                         }
                         this.minIdle = minIdle;
                         return this;
                         }
                         }
                         /********************************************************/
                         
                        }
                        
                        // 这段代码会抛出IllegalArgumentException，因为minIdle>maxIdle
                        ResourcePoolConfig config = new ResourcePoolConfig.Builder()
                         .setName("dbconnectionpool")
                         .setMaxTotal(16)
                         .setMaxIdle(10)
                         .setMinIdle(12)
                         .build();
                         
    3. 与工厂模式区别
            工厂模式是用来创建不同但是相关类型的对象(继承同一父类或者接口的一组子类), 由给定的参数来决定创建哪种类型的对象.
            建造者模式是用来创建 一种类型 的复杂对象, 通过设置不同的可选参数, "定制化"地创建不同的对象.
            
            通俗区别是 顾客走进一家餐馆点餐, 利用工厂模式, 根据用户不同的选择, 来制作不同的食物, 比如披萨、汉堡、沙拉. 对于披萨来说,
                     用户又有各种配料可以定制, 比如奶酪、西红柿、起司, 通过建造者模式根据用户选择的不同配料来制作披萨.
```

## 原型模式(Prototype Design Pattern)
```shell
    1. 概念
            (1) 原型模式是利用对已有对象(原型)进行复制(或者叫拷贝)的方式来创建新对象, 以达到节省创建对象的成本(针对某一种场景, 
                对象的创建成本大, 而同一个类的不同对象之间差别不大(大部分字段都相同))
                
    2. 创建对象成本大
            (1) 创建对象时数据来源需要经过复杂的计算才能得到(比如排序, 计算哈希值), 或者需要从 RPC, 网络, 数据库, 文件系统等非常慢速的
                IO 中读取, 这时就可以利用原型模式, 从其他已有对象中直接拷贝得到.
                
            (2) 例如原来从数据库中加载所有的数据(磁盘 IO 操作耗时), 再通过哈希散列算法构造, 这中间会消耗很长时间, 所以如果要重新更新
                数据库中的值, 又全量加载不合适, 可以先通过内存数据 A 拷贝一份到数据 B, 再从数据库中有更新的差集进行更新.
                
    3. 深拷贝,浅拷贝
            (1) 深拷贝方法
                    第一种: 递归拷贝对象, 对象的引用对象以及引用对象的引用对象……直到要拷贝的对象只包含基本数据类型数据, 
                           没有引用对象为止
                           
                    第二种: 先将对象序列化, 然后再反序列化成新的对象
```

# 结构型
```shell
    1. 结构型模式主要总结一些类或对象组合在一起的经典结构, 这些经典的结构可以解决特定应用场景的问题.
```

## 代理模式
```shell
    1. 概念
            (1) 代理模式(Proxy Design Pattern), 在不改变原始类(被代理类)代码的情况下, 通过引入代理类来给原始类附加功能
            (2) 代理模式中, 代理类附加的是跟原始类无关的功能, 而在装饰器模式中, 装饰器类附加的是跟原始类相关的增强功能
            
    2. 实现
            (1) 为了将框架代码和业务代码解耦, 通过代理模式, 例如, 代理类 UserControllerProxy 和原始类 UserController 两个
                实现相同的接口 IUserController. UserController 类只负责业务功能, 代理类 UserControllerProxy 负责在业务代码
                执行前后附加其他逻辑代码, 并通过委托的方式调用原始类来执行业务代码.
                
                改动前：
                        
                        // 业务和非业务功能耦合在一起, 一旦非业务功能需要改动, 则都要改.
                        public class UserController {
                          //...省略其他属性和方法...
                          private MetricsCollector metricsCollector; // 依赖注入
                        
                          public UserVo login(String telephone, String password) {
                            long startTimestamp = System.currentTimeMillis();
                        
                            // ... 省略login逻辑...
                        
                            long endTimeStamp = System.currentTimeMillis();
                            long responseTime = endTimeStamp - startTimestamp;
                            RequestInfo requestInfo = new RequestInfo("login", responseTime, startTimestamp);
                            metricsCollector.recordRequest(requestInfo);
                        
                            //...返回UserVo数据...
                          }
                        
                          public UserVo register(String telephone, String password) {
                            long startTimestamp = System.currentTimeMillis();
                        
                            // ... 省略register逻辑...
                        
                            long endTimeStamp = System.currentTimeMillis();
                            long responseTime = endTimeStamp - startTimestamp;
                            RequestInfo requestInfo = new RequestInfo("register", responseTime, startTimestamp);
                            metricsCollector.recordRequest(requestInfo);
                        
                            //...返回UserVo数据...
                          }
                        }
                        
                     
                改动后(代理增强)
                
                    // 定义抽象接口
                    public interface IUserController {
                      UserVo login(String telephone, String password);
                      UserVo register(String telephone, String password);
                    }
                    
                    // UserController 类纯粹业务
                    public class UserController implements IUserController {
                      //...省略其他属性和方法...
                    
                      @Override
                      public UserVo login(String telephone, String password) {
                        //...省略login逻辑...
                        //...返回UserVo数据...
                      }
                    
                      @Override
                      public UserVo register(String telephone, String password) {
                        //...省略register逻辑...
                        //...返回UserVo数据...
                      }
                    }
                    
                    // UserControllerProxy 包含了 UserController 类, 通过委托形式将业务传给 UserController 类
                    public class UserControllerProxy implements IUserController {
                    
                      private MetricsCollector metricsCollector;
                      private UserController userController;
                    
                      public UserControllerProxy(UserController userController) {
                        this.userController = userController;
                        this.metricsCollector = new MetricsCollector();
                      }
                    
                      @Override
                      public UserVo login(String telephone, String password) {
                        long startTimestamp = System.currentTimeMillis();
                    
                        // 委托
                        UserVo userVo = userController.login(telephone, password);
                    
                        long endTimeStamp = System.currentTimeMillis();
                        long responseTime = endTimeStamp - startTimestamp;
                        RequestInfo requestInfo = new RequestInfo("login", responseTime, startTimestamp);
                        metricsCollector.recordRequest(requestInfo);
                    
                        return userVo;
                      }
                    
                      @Override
                      public UserVo register(String telephone, String password) {
                        long startTimestamp = System.currentTimeMillis();
                    
                        UserVo userVo = userController.register(telephone, password);
                    
                        long endTimeStamp = System.currentTimeMillis();
                        long responseTime = endTimeStamp - startTimestamp;
                        RequestInfo requestInfo = new RequestInfo("register", responseTime, startTimestamp);
                        metricsCollector.recordRequest(requestInfo);
                    
                        return userVo;
                      }
                    }
                    
                    /* UserControllerProxy 使用举例
                     * 因为原始类和代理类实现相同的接口，是基于接口而非实现编程, 将 UserController 类对象替换为 
                     * UserControllerProxy 类对象,不需要改动太多代码
                    IUserController userController = new UserControllerProxy(new UserController());
                    
                    
            (2) 第二中代理类实现, 如果实现类并不是自己开发维护的(来自一个第三方的类库), 则采用继承的方式, 让代理类继承原始类,
                然后扩展附加功能
                继承类:
                        public class UserControllerProxy extends UserController {
                        
                          private MetricsCollector metricsCollector;
                        
                          public UserControllerProxy() {
                            this.metricsCollector = new MetricsCollector();
                          }
                        
                          public UserVo login(String telephone, String password) {
                            long startTimestamp = System.currentTimeMillis();
                        
                            UserVo userVo = super.login(telephone, password);
                        
                            long endTimeStamp = System.currentTimeMillis();
                            long responseTime = endTimeStamp - startTimestamp;
                            RequestInfo requestInfo = new RequestInfo("login", responseTime, startTimestamp);
                            metricsCollector.recordRequest(requestInfo);
                        
                            return userVo;
                          }
                        
                          public UserVo register(String telephone, String password) {
                            long startTimestamp = System.currentTimeMillis();
                        
                            UserVo userVo = super.register(telephone, password);
                        
                            long endTimeStamp = System.currentTimeMillis();
                            long responseTime = endTimeStamp - startTimestamp;
                            RequestInfo requestInfo = new RequestInfo("register", responseTime, startTimestamp);
                            metricsCollector.recordRequest(requestInfo);
                        
                            return userVo;
                          }
                        }
                        
                        //UserControllerProxy使用举例
                        UserController userController = new UserControllerProxy();
                        
                        
    3. 动态代理
            (1) 像静态代理方式, 一个实现类就得需要对应一个代理类, 同时需要重写实现类所有方法, 则对应的重复代码量多, 开发成本高.
                解决方案就是动态代理, 不事先为每个原始类编写代理类, 而是在运行时, 动态地创建原始类对应的代理类, 然后在系统中用代理类
                替换掉原始类.
                
            (2) 用 Java 的动态代理来实现, Spring AOP 底层的实现原理就是基于动态代理, 用户配置好需要给哪些类创建代理, 并定义好在
                执行原始类的业务代码前后执行哪些附加功能, Spring 为这些类创建动态代理对象, 并在 JVM 中替代原始类对象, 原本在代码中
                执行的原始类的方法, 被换作执行代理类的方法, 也就实现了给原始类添加附加功能的目的
                
    4. 应用场景
            (1) 业务系统的非功能性需求开发
                    监控, 统计, 鉴权, 限流, 事务, 幂等, 日志, 将这些附加功能与业务功能解耦, 放到代理类中统一处理, 只需要关注业务方面
                的开发.  搜集接口请求信息的例子, 就是这个应用场景的一个典型例子.
                
            (2)  代理模式在 RPC, 缓存中的应用
                    a. RPC
                        RPC 框架也可以看作一种代理模式, 类似与远程代理, 通过远程代理, 将网络通信, 数据编解码等细节隐藏起来, 客户端在
                       使用 RPC 服务的时, 就像使用本地函数一样, 无需了解跟服务器交互的细节,  RPC 服务的开发者也只需要开发业务逻辑, 就像
                       开发本地使用的函数一样, 不需要关注跟客户端的交互细节
                 
                    b.代理模式在缓存中的应用
                        接口请求的缓存功能, 对于某些接口请求, 如果入参相同, 在设定的过期时间内, 直接返回缓存结果, 
                     而不用重新进行逻辑处理, 这时需要动态代理, 基于 Spring 框架来开发, 就可以在 AOP 切面中完成接口缓存的功能, 在应用
                     启动的时, 从配置文件中加载需要支持缓存的接口, 以及相应的缓存策略(比如过期时间)等. 当请求到来的时候, 在 AOP 切面
                     中拦截请求, 如果请求中带有支持缓存的字段 (比如 http://…?..&cached=true), 便从缓存(内存缓存或者 Redis 缓存等)
                     中获取数据直接返回
                                
```

## 桥接模式
```shell
    1. 概念
            (1) 桥接模式(Bridge Design Pattern), 是将抽象和实现解耦, 让它们可以独立变化, 定义中的“抽象”, 并非"抽象类"或"接口". 
                而是被抽象出来的一套 "类库", 只包含骨架代码, 真正的业务逻辑需要委派给定义中的“实现”来完成. 而定义中的“实现”, 
                也并非“接口的实现类”, 而是的一套独立的“类库”. “抽象”和“实现”独立开发, 通过对象之间的组合关系, 组装在一起
            (2)
    2. 桥接模式的抽象和实现
            (1) 例如 JDBC 驱动
            
                JDBC 本身就相当于“抽象”, 这里所说的“抽象”, 指的并非“抽象类”或“接口”, 而是跟具体的数据库无关的, 
                被抽象出来的一套“类库”.
                 
                具体的 Driver(如 com.mysql.jdbc.Driver)就相当于"实现", 这里所说的“实现”, 也并非指“接口的实现类”, 而是跟具体数据库
                相关的一套“类库”. JDBC 和 Driver 独立开发, 通过对象之间的组合关系, 组装在一起. JDBC 的所有逻辑操作, 最终都
                委托给 Driver 来执行
                
            (2) 重构前 Notification 类中 if-else 很多, 并且里面业务很复杂
            
                    public class Notification {
                        .....
                    
                     public void notify(NotificationEmergencyLevel level, String message) {
                     if (level.equals(NotificationEmergencyLevel.SEVERE)) {
                     //...自动语音电话
                     } else if (level.equals(NotificationEmergencyLevel.URGENCY)) {
                     //...发微信
                     } else if (level.equals(NotificationEmergencyLevel.NORMAL)) {
                     //...发邮件
                     } else if (level.equals(NotificationEmergencyLevel.TRIVIAL)) {
                     //...发邮件
                     }
                     }
                     
                    }
                    
                 重构后, 将不同渠道的发送逻辑剥离出来, 形成独立的消息发送类 (MsgSender 相关类), Notification 类相当于抽象,
                        MsgSender 类相当于实现, 两者可以独立开发, 通过组合关系(也就是桥梁)任意组合在一起, 不同紧急程度的消息和
                        发送渠道之间的对应关系, 不是在代码中固定写死的, 可以动态地去指定(如通过读取配置来获取对应关系)
                        
                        
                    public interface MsgSender {
                     void send(String message);
                    }
                    
                    public class TelephoneMsgSender implements MsgSender {
                     private List<String> telephones;
                     
                     public TelephoneMsgSender(List<String> telephones) {
                     this.telephones = telephones;
                     }
                     
                     @Override
                     public void send(String message) {
                     //...
                     }
                    }
                    
                    public class EmailMsgSender implements MsgSender {
                     // 与TelephoneMsgSender代码结构类似，所以省略...
                    }
                    
                    public class WechatMsgSender implements MsgSender {
                     // 与TelephoneMsgSender代码结构类似，所以省略...
                    }
                    
                    
                    public abstract class Notification {
                     protected MsgSender msgSender;
                     
                     public Notification(MsgSender msgSender) {
                     this.msgSender = msgSender;
                     }
                     
                     public abstract void notify(String message);
                    }
                    
                    
                    public class SevereNotification extends Notification {
                     public SevereNotification(MsgSender msgSender) {
                     super(msgSender);
                     }
                     
                     @Override
                     public void notify(String message) {
                     msgSender.send(message);
                     }
                    }
                    public class UrgencyNotification extends Notification {
                     // 与SevereNotification代码结构类似，所以省略...
                    }
                    
                    public class NormalNotification extends Notification {
                     // 与SevereNotification代码结构类似，所以省略...
                    }
```

## 装饰器模式(Decorator Pattern)
```shell
    1. 概念
    2. 装饰器模和"用组合代替继承"区别
            (1) 装饰器类和原始类继承同样的父类, 可以对原始类"嵌套"多个装饰器类, 对 FileInputStream 嵌套了两个装饰器类：
                BufferedInputStream 和 DataInputStream, 让它既支持缓存读取, 又支持按照基本数据类型来读取数据.
                例如:
                        // FileInputStream 为原始类
                        InputStream in = new FileInputStream("/user/wangzheng/test.txt");
                        InputStream bin = new BufferedInputStream(in);
                        DataInputStream din = new DataInputStream(bin);
                        int data = din.readInt();
                        
            (2) 装饰器类是对功能的增强, 是装饰器模式应用场景的一个重要特点. 代理模式和装饰器模式结构差不多, 代理模式中, 
                代理类附加的是跟原始类无关的功能, 而在装饰器模式中, 装饰器类附加的是跟原始类相关的增强功能
                
                    // 代理模式的代码结构(下面的接口也可以替换成抽象类)
                    public interface IA {
                        void f();
                    }
                    public class A impelements IA {
                        public void f() { }
                    }
                    
                    public class AProxy impements IA {
                        private IA a;
                        public AProxy(IA a) {
                            this.a = a;
                        }
                        
                        public void f() {
                            // 新添加的代理逻辑, 与原始类功能毫无关系
                            a.f();
                            // 新添加的代理逻辑
                        }
                    }
                    
                    // 装饰器模式的代码结构(下面的接口也可以替换成抽象类)
                    public interface IA {
                        void f();
                    }
                    public class A impelements IA {
                        public void f() { }
                    }
                    
                    public class ADecorator impements IA {
                        private IA a;
                        public ADecorator(IA a) {
                            this.a = a;
                        }
                        
                        public void f() {
                            // 功能增强代码
                            a.f();
                            // 功能增强代码
                        }
                    }
                    
    3. 装饰器模式主要解决继承关系过于复杂的问题, 通过组合来替代继承, 作用是给原始类添加增强功能, 这是判断是否该用装饰器模式的一个重要的
       依据, 装饰器模式还有一个特点, 就是可以对原始类嵌套使用多个装饰器, 为了满足这个应用场景, 在设计时, 装饰器类需要跟原始类继承相同的
       抽象类或者接口
```

## 适配器模式
```shell
    1. 概念
            (1) 适配器模式(Adapter Design Pattern), 主要是用来做适配的, 将不兼容的接口转换为可兼容的接口, 让原本由于接口不兼容而
                不能一起工作的类可以一起工作.
                
            (2) 适配器模式有两种实现方式: 类适配器和对象适配器, 类适配器使用继承关系来实现, 对象适配器使用组合关系来实现.
    2. 适配器模式实现
            (1) ITarget 表示要转化成的接口定义, Adaptee 是一组不兼容 ITarget 接口定义的接口, Adaptor 将 Adaptee 转化成一组符合
                ITarget 接口定义的接口.
                
                a. 基于继承
                    // 类适配器: 基于继承
                    public interface ITarget {
                     void f1();
                     void f2();
                     void fc();
                    }
                    
                    public class Adaptee {
                     public void fa() { ... }
                     public void fb() { ... }
                     public void fc() { ... }
                    }
                    
                    public class Adaptor extends Adaptee implements ITarget {
                     public void f1() {
                     super.fa();
                     }
                     
                     public void f2() {
                     //...重新实现f2()...
                     }
                     
                     // 这里fc()不需要实现，直接继承自Adaptee，这是跟对象适配器最大的不同点
                    }
                    
                    
                b. 基于组合   
                    // 对象适配器：基于组合
                    public interface ITarget {
                     void f1();
                     void f2();
                     void fc();
                    }
                    
                    public class Adaptee {
                     public void fa() { ... }
                     public void fb() { ... }
                     public void fc() { ... }
                    }
                    
                    public class Adaptor implements ITarget {
                     private Adaptee adaptee;
                     
                     public Adaptor(Adaptee adaptee) {
                     this.adaptee = adaptee;
                     }
                     
                     public void f1() {
                     adaptee.fa(); //委托给 Adaptee
                     }
                     
                     public void f2() {
                     //...重新实现f2()...
                     }
                     
                     public void fc() {
                     adaptee.fc();
                     }
                    }
                    
            (2) 两种适配器实现方式的选择
            
                    如果 Adaptee 接口并不多, 那两种实现方式都可以.
                    
                    如果 Adaptee 接口很多, 而且 Adaptee 和 ITarget 接口定义大部分都相同, 推荐使用类适配器, 因为 Adaptor 复用
                    父类 Adaptee 的接口, 比起对象适配器的实现方式, Adaptor 的代码量要少一些
                    
                    如果 Adaptee 接口很多, 而且 Adaptee 和 ITarget 接口定义大部分都不相同, 那推荐使用对象适配器, 因为组合结构相对
                    于继承更加灵活
                    
    3. 适配器模式应用场景
            (1) 适配器模式可以看作一种"补偿模式", 用来补救设计上的缺陷
            (2) 封装有缺陷的接口设计
                    依赖的外部系统在接口设计方面有缺陷(如包含大量静态方法), 引入之后会影响到自身代码的可测试性, 为了隔离设计上的缺陷,
                希望对外部系统提供的接口进行二次封装, 抽象出更好的接口设计, 这个时候就可以使用适配器模式.
                    例如:
                        //这个类来自外部 sdk，我们无权修改它的代码
                        public class CD { 
                         //...
                         public static void staticFunction1() { ... }
                         public void uglyNamingFunction2() { ... }
                         public void tooManyParamsFunction3(int paramA, int paramB, ...) { ... }
                         public void lowPerformanceFunction4() { ... }
                        }
                        
                        // 使用适配器模式进行重构
                        public class ITarget {
                         void function1();
                         void function2();
                         void fucntion3(ParamsWrapperDefinition paramsWrapper);
                         void function4();
                         //...
                        }
                        
                        // 注意：适配器类的命名不一定非得末尾带Adaptor
                        public class CDAdaptor extends CD implements ITarget {
                         //...
                         public void function1() {
                         super.staticFunction1();
                         }
                         
                         public void function2() {
                         super.uglyNamingFucntion2();
                         }
                         
                         public void function3(ParamsWrapperDefinition paramsWrapper) {
                         super.tooManyParamsFunction3(paramsWrapper.getParamA(), ...);
                         }
                         
                         public void function4() {
                         //...reimplement it...
                         }
                        }
                        
            (3)  统一多个类的接口设计
                    某个功能的实现依赖多个外部系统(或者类),每个外部系统的接口(包括接口名, 传入参数等)都是不一样的,  通过适配器模式, 
                    将它们的接口适配为统一的接口定义, 然后就可以使用多态的特性来复用代码逻辑.
                    
                    例如: 对用户输入的文本内容做敏感词过滤, 为了提高过滤率, 引入多款第三方敏感词过滤系统, 依次对用户输入的内容进行过滤,
                         过滤掉尽可能多的敏感词, 但是每个系统提供的过滤接口都是不同的, 意味着没法复用一套逻辑来调用各个系统. 解决方法是
                         使用适配器模式, 将所有系统的接口适配为统一的接口定义, 这样可以复用调用敏感词过滤的代码.
                         
                         // A敏感词过滤系统提供的接口
                         public class ASensitiveWordsFilter { 
                          //text 是原始文本，函数输出用***替换敏感词之后的文本
                          public String filterSexyWords(String text) {
                          // ...
                          }
                          
                          public String filterPoliticalWords(String text) {
                          // ...
                          }
                         }
                         
                         // B敏感词过滤系统提供的接口
                         public class BSensitiveWordsFilter { 
                          public String filter(String text) {
                          //...
                          }
                         }
                         
                         // C敏感词过滤系统提供的接口
                         public class CSensitiveWordsFilter { 
                          public String filter(String text, String mask) {
                          //...
                          }
                         }
                         
                         重构前：
                         // 未使用适配器模式之前的代码：代码的可测试性、扩展性不好
                         public class RiskManagement {
                          private ASensitiveWordsFilter aFilter = new ASensitiveWordsFilter();
                          private BSensitiveWordsFilter bFilter = new BSensitiveWordsFilter();
                          private CSensitiveWordsFilter cFilter = new CSensitiveWordsFilter();
                         
                          public String filterSensitiveWords(String text) {
                          String maskedText = aFilter.filterSexyWords(text);
                          maskedText = aFilter.filterPoliticalWords(maskedText);
                          maskedText = bFilter.filter(maskedText);
                          maskedText = cFilter.filter(maskedText, "***");
                          return maskedText;
                          }
                         }
                         
                         
                         重构后：
                         // 使用适配器模式进行改造
                         public interface ISensitiveWordsFilter { // 统一接口定义
                          String filter(String text);
                         }
                         
                         public class ASensitiveWordsFilterAdaptor implements ISensitiveWordsFilter {
                          private ASensitiveWordsFilter aFilter;
                          public String filter(String text) {
                          String maskedText = aFilter.filterSexyWords(text);
                          maskedText = aFilter.filterPoliticalWords(maskedText);
                          return maskedText;
                          }
                         }
                         //...省略BSensitiveWordsFilterAdaptor、CSensitiveWordsFilterAdaptor...
                         
                         // 扩展性更好，更加符合开闭原则，如果添加一个新的敏感词过滤系统，
                         // 这个类完全不需要改动；而且基于接口而非实现编程，代码的可测试性更好。
                         public class RiskManagement {
                          private List<ISensitiveWordsFilter> filters = new ArrayList<>();
                          
                          public void addSensitiveWordsFilter(ISensitiveWordsFilter filter) {
                          filters.add(filter);
                          }
                          
                          public String filterSensitiveWords(String text) {
                          String maskedText = text;
                          for (ISensitiveWordsFilter filter : filters) {
                          maskedText = filter.filter(maskedText);
                          }
                          return maskedText;
                          }
                         }
                         
            (4) 替换依赖的外部系统
                    把项目中依赖的一个外部系统替换为另一个外部系统时, 利用适配器模式, 可以减少对代码的改动.
                    
                        // 外部系统A
                        public interface IA {
                         //...
                         void fa();
                        }
                        
                        public class A implements IA {
                         //...
                         public void fa() { ... }
                        }
                        
                        // 在项目中, 外部系统 A 的使用示例
                        public class Demo {
                         private IA a;
                         public Demo(IA a) {
                         this.a = a;
                         }
                         //...
                        }
                        
                        Demo d = new Demo(new A());
                        
                        // 将外部系统 A 替换成外部系统 B
                        public class BAdaptor implemnts IA {
                         private B b;
                         public BAdaptor(B b) {
                         this.b= b;
                         }
                         public void fa() {
                         //...
                         b.fb();
                         }
                        }
                        // 借助 BAdaptor, Demo 的代码中, 调用 IA 接口的地方都无需改动, 
                        // 只需要将 BAdaptor 如下注入到 Demo 即可
                        Demo d = new Demo(new BAdaptor(new B()));
                        
            (5) 兼容老版本接口
                    在做版本升级时, 对于一些要废弃的接口, 不直接将其删除, 而是暂时保留, 并且标注为 deprecated, 将内部实现逻辑委托为
                    新的接口实现, 让使用它的项目有个过渡期, 而不是强制进行代码修改
                    
            (6) 适配不同格式的数据
                    可以用在不同格式的数据之间的适配, 如把从不同征信系统拉取的不同格式的征信数据, 统一为相同的格式, 以方便存储和使用.
                    Java 中的 Arrays.asList() 也可以看作一种数据适配器, 将数组类型的数据转化为集合容器类型
                    
                    List<String> stooges = Arrays.asList("Larry", "Moe", "Curly");
                    
    4. 适配器开源项目应用
            Slf4j 日志框架提供了一套打印日志的统一接口规范, 提供了统一的接口定义, 还提供了针对不同日志框架的适配器. 对不同日志框
       架的接口进行二次封装, 适配成统一的 Slf4j 接口定义
            例如:
                    // slf4j统一的接口定义
                    package org.slf4j;
                    public interface Logger {
                     public boolean isTraceEnabled();
                     public void trace(String msg);
                     public void trace(String format, Object arg);
                     public void trace(String format, Object arg1, Object arg2);
                     public void trace(String format, Object[] argArray);
                     public void trace(String msg, Throwable t);
                     
                     public boolean isDebugEnabled();
                     public void debug(String msg);
                     public void debug(String format, Object arg);
                     public void debug(String format, Object arg1, Object arg2)
                     public void debug(String format, Object[] argArray)
                     public void debug(String msg, Throwable t);
                     //...省略info、warn、error等一堆接口
                    }
                    
                    // log4j 日志框架的适配器
                    // Log4jLoggerAdapter实现了LocationAwareLogger接口，
                    // 其中LocationAwareLogger继承自Logger接口，
                    // 也就相当于 Log4jLoggerAdapter 实现了Logger接口。
                    package org.slf4j.impl;
                    public final class Log4jLoggerAdapter extends MarkerIgnoringBase
                     implements LocationAwareLogger, Serializable {
                     final transient org.apache.log4j.Logger logger; // log4j
                     
                     public boolean isDebugEnabled() {
                     return logger.isDebugEnabled();
                     }
                     
                     public void debug(String msg) {
                     logger.log(FQCN, Level.DEBUG, msg, null);
                     }
                     
                     public void debug(String format, Object arg) {
                     if (logger.isDebugEnabled()) {
                     FormattingTuple ft = MessageFormatter.format(format, arg);
                     logger.log(FQCN, Level.DEBUG, ft.getMessage(), ft.getThrowable());
                     }
                     }
                     
                     public void debug(String format, Object arg1, Object arg2) {
                     if (logger.isDebugEnabled()) {
                     FormattingTuple ft = MessageFormatter.format(format, arg1, arg2);
                     logger.log(FQCN, Level.DEBUG, ft.getMessage(), ft.getThrowable());
                     }
                     }
                     
                     public void debug(String format, Object[] argArray) {
                     if (logger.isDebugEnabled()) {
                     FormattingTuple ft = MessageFormatter.arrayFormat(format, argArray);
                     logger.log(FQCN, Level.DEBUG, ft.getMessage(), ft.getThrowable());
                     }
                     }
                     
                     public void debug(String msg, Throwable t) {
                     logger.log(FQCN, Level.DEBUG, msg, t);
                     }
                     //...省略一堆接口的实现...
                    }
                    
    5. 代理, 桥接, 装饰器, 适配器设计模式区别
            都称为 Wrapper 模式, 即通过 Wrapper 类二次封装原始类, 但其要解决的问题, 应用场景不同.
            
            代理模式：代理模式在不改变原始类接口的条件下, 为原始类定义一个代理类, 目的是控制访问(非功能性的需求, 统计, 幂等),
                    而非加强功能, 这是它跟装饰器模式最大的不同
                    
            桥接模式：桥接模式目的是将接口部分和实现部分分离, 从而让它们较为容易, 也相对独立地加以改变.
            
            装饰器模式: 装饰者模式在不改变原始类接口的情况下, 对原始类功能进行增强, 并且支持多个装饰器的嵌套使用
                 
            适配器模式：适配器模式是一种事后的补救策略, 适配器提供跟原始类不同的接口, 而代理模式, 装饰器模式提供的都是跟原始类相同的接口.
```

## 门面模式
```shell
    1. 概念
            (1) 门面模式也叫外观模式(Facade Design Pattern), 门面模式为子系统提供一组统一的接口, 定义一组高层接口让子系统更易用,
                例如, 假设有一个系统 A, 提供了 a, b, c, d 四个接口, 系统 B 完成某个业务功能, 需要调用 A 系统的 a, b, d 接口.
                利用门面模式, 提供一个包含 a, b, d 接口调用的门面接口 x, 给系统 B 直接使用, 由 3 个接口统一成 1 个接口, 如果这个
                接口是移动网络传输的, 可以减少延迟.
            (2) 主要在接口设计方面使用, 解决可复用性（通用性）和易用性之间的矛盾, 可复用性是期待函数颗粒度小, 单一功能好调用, 但随着
            　　 而来就是调用者要实现一个大功能又得调 n 个函数, 不易用.
            (3) 
                如果门面接口不多, 完全可以将它跟非门面接口放到一块, 也不需要特殊标记, 当作普通接口来用.
                如果门面接口很多, 可以在已有的接口之上, 再重新抽象出一层, 专门放置门面接口, 从类, 包的命名上跟原来的接口层做区分.
                如果门面接口特别多, 并且很多都是跨多个子系统的, 可以将门面接口放到一个新的子系统中.
            
    2. 应用场景
            (1) 解决易用性问题
                    门面模式可以用来封装系统的底层实现, 隐藏系统的复杂性, 提供一组更加简单易用, 更高层的接口. Linux 系统调用函数就是
                门面模式设计, 封装底层更基础的 Linux 内核调用.
                
            (2) 解决性能问题
                    通过一个接口替换掉多个接口, 减少网络通信成本, 提高 App 客户端的响应速度.
                    
            (3) 解决分布式事务问题
                    金融系统, 用户和钱包的业务, 想要实现在用户注册时, 不仅要创建用户(在数据库 User 表中), 还要给用户创建一个钱包
                (在数据库的 Wallet 表中), 可以利用数据库事务或者 Spring 框架提供的事务, 在一个事务中, 执行创建用户和创建钱包这两个
                 SQL 操作, 这要求两个 SQL 操作要在一个接口中完成, 可以借鉴门面模式的思想, 再设计一个包裹这两个操作的新接口, 让新接口
                 在一个事务中执行两个 SQL 操作. 
```

## 组合模式
```shell
    1. 概念
            (1) 组合模式(Composite Design Pattern), 组合模式跟之前讲的面向对象设计中的"组合关系(通过组合来组装两个类), 完全
                是两码事, 这里的“组合模式”, 主要是用来处理树形结构数据, “数据” 可以简单理解为一组对象集合.
                
                定义: 将一组对象组织(Compose)成树形结构, 以表示一种 "部分 - 整体"的层次结构, 组合让客户端(代码的使用者) 可以统一单个
                     对象和组合对象的处理逻辑.
    2. 应用场景
            (1) 文件系统
                    将一组对象(文件和目录)组织成树形结构, 以表示一种‘部分 - 整体’的层次结构(目录与子目录的嵌套结构), 组合模式让客户端
                    可以统一单个对象(文件)和组合对象(目录)的处理逻辑(递归遍历)
                    
            (2) OA 系统
                需求: 在内存中构建整个公司的人员架构图(部门, 子部门, 员工的隶属关系), 并且提供接口计算出部门的薪资成本(隶属于这个部门的
                     所有员工的薪资和)
                     
                原理: 部门包含子部门和员工, 这是一种嵌套结构, 可以表示成树这种数据结构, 计算每个部门的薪资开支这样一个需求, 也可以通过
                     在树上的遍历算法来实现, 这种场景可以通过组合的形式.
```

## 享元模式
```shell
    1. 概念
            (1) 享元模式(Flyweight Design Pattern), 被共享的单元, 享元模式目的是复用对象, 节省内存(前提是享元对象是不可变对象).
                即当一个系统中存在大量重复对象时, 如果这些重复的对象是不可变对象, 就可以利用享元模式将对象设计成享元, 在内存中只保留
                一份实例, 供多处代码引用, 可以减少内存对象的数量, 起到节省内存的目的. 不仅仅相同对象可以设计成享元, 对于相似对象, 
                也可以将这些对象中相同的部分(字段)提取出来, 设计成享元, 让这些大量相似对象引用这些享元.
    2. 享元模式应用场景
            (1) 棋牌游戏
                需求:
                        一个棋牌游戏(比如象棋), 一个游戏厅中有成千上万个“房间”, 每个房间对应一个棋局, 棋局要保存每个棋子的数据, 比如
                   棋子类型(将, 相, 士, 炮等), 棋子颜色(红方, 黑方), 棋子在棋局中的位置. 利用这些数据, 就能显示一个完整的棋盘给玩家.
                   ChessPiece 类表示棋子, ChessBoard 类表示一个棋局, 里面保存象棋中 30 个棋子的信息.
                   
                问题: 如果给每个房间都创建一个 ChessBoard 棋局对象, 那么同一时间有成百上千的棋局对象, 这会消耗很大一部分内存.
                
                解决方案: 可以将棋子的 id, text, color 属性拆分出来, 设计成独立的类, 并且作为享元供多个棋盘复用. 棋盘只需要记录
                         每个棋子的位置信息就可以了.
                         
                         利用工厂类来缓存 ChessPieceUnit 信息(id, text, color), 通过工厂类获取到的 ChessPieceUnit 就是享元, 
                         所有的 ChessBoard 对象共享这 30 个 ChessPieceUnit 对象(因为象棋中只有 30 个棋子), 在使用享元模式前, 记
                         录 1 万个棋局, 则需要创建 30 万(30*1 万)个棋子的 ChessPieceUnit 对象, 利用享元模式, 只需要创建 30 个
                         享元对象供所有棋局共享使用, 大大节省了内存
                         
                         public class ChessPieceUnit {
                             private int id;
                             private String text;
                             private Color color;
                         
                             public ChessPieceUnit(int id, String text, Color color) {
                                 this.id = id;
                                 this.text = text;
                                 this.color = color;
                             }
                             public static enum Color{
                                 RED, BLACK
                             }
                             //...省略其他属性和getter/setter方法...
                         }
                         
                         public class ChessPieceUnitFactory {
                             private static final Map<Integer,ChessPieceUnit> pieces = new HashMap<>();
                             static {
                                 pieces.put(1,new ChessPieceUnit(1,"车", ChessPieceUnit.Color.BLACK));
                                 pieces.put(2,new ChessPieceUnit(2,"马", ChessPieceUnit.Color.BLACK));
                                 //...省略摆放其他棋子的代码...
                             }
                             public static ChessPieceUnit getChessPiece(int chessPieceId){
                                 return pieces.get(chessPieceId);
                             }
                         }
                         
                         public class ChessPiece {
                             private ChessPieceUnit chessPieceUnit;
                             private int positionX;
                             private int positionY;
                         
                             public ChessPiece(ChessPieceUnit chessPieceUnit, int positionX, int positionY) {
                                 this.chessPieceUnit = chessPieceUnit;
                                 this.positionX = positionX;
                                 this.positionY = positionY;
                             }
                             //省略getter、setter方法
                         }
                         
                         public class ChessBoard {
                             private Map<Integer,ChessPiece> chessPieces = new HashMap<>();
                             public ChessBoard(){
                                 init();
                             }
                             private void init(){
                                 chessPieces.put(1,new ChessPiece(ChessPieceUnitFactory.getChessPiece(1),0,0));
                                 chessPieces.put(1,new ChessPiece(ChessPieceUnitFactory.getChessPiece(2),1,0));
                                 //...省略摆放其他棋子的代码...
                             }
                             public void move(int chessPieceId,int toPositionX,int toPositionY){
                                 //...省略...
                             }
                         }
                         
                实现方式: 主要是通过工厂模式, 在工厂类中, 通过一个 Map 来缓存已经创建过的享元对象, 来达到复用的目的
                
                
            (2) 文本编辑器
                    需求:
                        文本编辑器在内存中需要记录文字和格式两部分信息, 其中格式包括文字的字体, 大小, 颜色等信息.
                        
                    问题:
                        每敲一个文字, 就会创建一个新的 Character 对象(这里面包含字体, 大小, 颜色).如果一个文本文件, 有上万, 十
                        几万, 几十万的文字, 那就要在内存中存储这么多 Character 对象.
                        
                    解决方案:
                        对于字体格式, 可以将它设计成享元, 让不同的文字共享使用(已指针的方式共享).
                        
                        public class CharacterStyle {
                            private Font font;
                            private int size;
                            private int colorRGB;
                        
                            public CharacterStyle(Font font, int size, int colorRGB) {
                                this.font = font;
                                this.size = size;
                                this.colorRGB = colorRGB;
                            }
                        
                            @Override
                            public boolean equals(Object obj) {
                                CharacterStyle otherStyle = (CharacterStyle) obj;
                                return font.equals(otherStyle.font)
                                        && size==otherStyle.size
                                        && colorRGB==otherStyle.colorRGB;
                            }
                        }
                        
                        public class CharacterStyleFactory {
                            private static final List<CharacterStyle> styles = new ArrayList<>();
                            public static CharacterStyle getStyle(Font font,int size,int colorRGB){
                                CharacterStyle newStyle = new CharacterStyle(font,size,colorRGB);
                                for (CharacterStyle style:styles){
                                    if (style.equals(newStyle)){
                                        return style;
                                    }
                                }
                                styles.add(newStyle);
                                return newStyle;
                            }
                        }
                        
                        public class Character {
                            private char c;
                            private CharacterStyle style;
                        
                            public Character(char c, CharacterStyle style) {
                                this.c = c;
                                this.style = style;
                            }
                        }
                        
                        public class Editor {
                            private List<Character> chars = new ArrayList<>();
                            public void appendCharacter(char c, Font font, int size, int colorRGB){
                                Character character = new Character(c,CharacterStyleFactory.getStyle(font, size, colorRGB));
                                chars.add(character);
                            }
                        }
                        
    3. 享元模式 vs 单例、缓存、对象池
            (1) 享元模式跟单例的区别
                    单例模式中, 一个类只能创建一个对象, 而在享元模式中, 一个类可以创建多个对象, 每个对象被多处代码引用共享. 从解决场景
                 来看, 应用享元模式是为了对象复用, 节省内存, 而应用多例模式是为了限制对象的个数.
                 
            (2) 享元模式跟对象池
                    池化技术中的"复用"理解为"重复使用", 主要目的是节省时间(如从数据库池中取一个连接, 不需要重新创建). 在任意时刻, 
                    每一个对象, 连接, 线程, 并不会被多处使用, 而是被一个使用者独占, 当使用完成后, 才会放回到池中, 再由其他使用者重复
                    利用. 享元模式中的"复用"理解为“共享使用”, 在整个生命周期中, 都是被所有使用者共享, 主要目的是节省空间
                    
    4. 享元模式在 java 中应用
            (1) 
                Integer i1 = 56;
                Integer i2 = 56;
                Integer i3 = 129;
                Integer i4 = 129;
                System.out.println(i1 == i2);
                System.out.println(i3 == i4);
                
                首先 Integer i1 = 56; 这中间有进行隐式转换, 将 int 型通过 Integer.valueof() 转化为 Integer, 如果创建的 Integer
                对象的值在 -128 到 127 之间, 会从 IntegerCache 类中直接返回, 否则才调用 new 方法创建, 则刚开始进行初始化时, 
                i1 和 i2 指向的是同一个地址, 之后值有发生变化才不相等.
                
            (2) 
                    
```

# 行为型
```shell
    1. 行为型设计模式解决的是"类或对象之间的交互"问题
```

## 观察者模式
```shell
    1. 概念
            (1) 实现方式, 有同步阻塞的实现方式, 也有异步非阻塞的实现方式；
                         有进程内的实现方式, 也有跨进程的实现方式
                         
            (2) 观察者模式(Observer Design Pattern)也被称为发布订阅模式(Publish-Subscribe Design Pattern), 在对象之间定义一个
                一对多的依赖, 当一个对象状态改变时, 所有依赖的对象都会自动收到通知.
                
            (3) 被依赖的对象叫作被观察者(Observable), 依赖的对象叫作观察者(Observer), 如 Subject-Observer, 
                Publisher-Subscriber, Producer-Consumer,  EventEmitter-EventListener, Dispatcher-Listener
                
            (4) 同步阻塞是最经典的实现方式, 主要是为了代码解耦；异步非阻塞除了能实现代码解耦之外, 还能提高代码的执行效率；
                进程间的观察者模式解耦更加彻底, 一般是基于消息队列来实现, 用来实现不同进程间的被观察者和观察者之间的交互
    2. 经典的观察者模式(有点类似于函数回调)
        
                public interface Subject {
                    void registerObserver(Observer observer);
                    void removeObserver(Observer observer);
                    void notifyObservers(Message message);
                }
                
                public interface Observer {
                    void update(Message message);
                }
                
                public class ConcreteSubject implements Subject {
                    private List<Observer> observers = new ArrayList<>();
                    @Override
                    public void registerObserver(Observer observer) {
                        observers.add(observer);
                    }
                
                    @Override
                    public void removeObserver(Observer observer) {
                        observers.remove(observer);
                    }
                
                    @Override
                    public void notifyObservers(Message message) {
                        for (Observer observer:observers){
                            observer.update(message);
                        }
                    }
                }
                
                public class ConcreteObserverOne implements Observer {
                    @Override
                    public void update(Message message) {
                        //todo 获取消息通知，执行自己的逻辑
                        System.out.println("ConcreteObserverOne is notified");
                    }
                }
                
                public class ConcreteObserverTwo implements Observer {
                    @Override
                    public void update(Message message) {
                        //todo 获取消息通知，执行自己的逻辑
                        System.out.println("ConcreteObserverTwo is notified");
                    }
                }
                
                public class Demo {
                    public static void main(String[] args) {
                        ConcreteSubject subject = new ConcreteSubject();
                        subject.registerObserver(new ConcreteObserverOne());
                        subject.registerObserver(new ConcreteObserverTwo());
                        subject.notifyObservers(new Message());
                    }
                }
                
    3. 业务场景
            (1) P2P 投资理财系统中,  register() 函数如果频繁修改需求, 会使得逻辑变得越来越复杂, 会影响到代码的可读性和可维护性, 这个
            　　时候观察者就比较合适.
            
                    public interface RegObserver {
                        void handleRegSuccess(long userId);
                    }
                    
                    public class RegPromotionObserver implements RegObserver {
                        private PromotionService promotionService;
                        @Override
                        public void handleRegSuccess(long userId) {
                            promotionService.issueNewUserExperienceCash(userId);
                        }
                    }
                    public class RegNotificationObserver implements RegObserver {
                        private NotificationService notificationService;
                        @Override
                        public void handleRegSuccess(long userId) {
                            notificationService.sendInboxMessage(userId,"Welcome...");
                        }
                    }
                    public class UserController {
                        private UserService userService;
                        private List<RegObserver> regObservers = new ArrayList<>();
                        //一次性设置好，之后也不可能动态的修改
                        public void setRegObservers(List<RegObserver> observers){
                            regObservers.addAll(observers);
                        }
                        
                        public Long register(String telephone,String password){
                            //省略输入参数的校验代码
                            //省略userService.register()异常的try-catch代码
                            long userId = userService.register(telephone,password);
                            
                            // 用接口同一代码
                            for (RegObserver observer:regObservers){
                                observer.handleRegSuccess(userId);
                            }
                            
                            return userId;
                        }
                    }
                    
                这是一种同步阻塞的实现方式, 观察者和被观察者代码在同一个线程内执行, 被观察者一直阻塞, 直到所有的观察者代码都执行完成
                后, 才执行后续的代码, 如果性能有需求, 可以通过创建新的线程来进行回调函数处理.
                
    4. 异步非阻塞观察者模式
            (1) 第一种方式, 可以通过创建新的线程来进行回调函数处理, 这样主线程就不会一致阻塞.
            (2) 第二种方式, 基于 EventBus 来实现
            (3) 第三种方式, 消息队列.
            
    5. EventBus 框架功能
              
```

## 模板模式
```shell
    1. 概念
            (1) 模板模式主要是用来解决复用和扩展两个问题
            (2) 模板模式(模板方法设计模式), Template Method Design Pattern, 定义为模板方法模式在一个方法中定义一个算法(业务逻辑)骨架, 
                并将某些步骤推迟到子类中实现, 模板方法模式可以让子类在不改变算法(业务逻辑)整体结构的情况下, 重新定义算法中的某些步骤. 
    2. 实例代码
            public abstract class AbstractClass {
                public final void templateMethod(){
                    //...
                    method1();
                    //...
                    method2();
                    //...
                }
                protected abstract void method1();
                protected abstract void method2();
            }
            
            public class ConcreteClass1 extends AbstractClass {
                @Override
                protected void method1() {
                    //...
                }
            
                @Override
                protected void method2() {
                    //...
                }
            }
            
            public class ConcreteClass2 extends AbstractClass {
                @Override
                protected void method1() {
                    //...
                }
            
                @Override
                protected void method2() {
                    //...
                }
            }
            
            AbstractClass demo = new ConcreteClass1();
            demo.templateMethod();
            
    3. 模板模式作用(复用)
            (1) 模板模式把一个业务逻辑中不变的流程抽象到父类的模板方法 templateMethod() 中, 将可变的部分 method1(), method2() 
                留给子类 ContreteClass1 和 ContreteClass2 来实现, 所有的子类都可以复用父类中模板方法定义的流程代码.
                
            (2) 应用
                    a. InputStream
                        read(....) 函数是一个模板方法, 定义了读取数据的整个流程, 并暴露一个可以由子类来定制的抽象方法, 
                        不过这个方法也被命名 read(), 只是参数跟模板方法不同
                        
                    b. AbstractList
                    
                            public abstract class AbstractList<E> {
                            
                                public boolean addAll(int index, Collection<? extends E> c){
                                    rangeCheckForAdd(index);
                                    boolean modified = false;
                                    for (E e:c){
                                        add(index++,e);
                                        modified = true;
                                    }
                                    return modified;
                                }
                                
                                // 这个就相当于 abstract, 继承子类如果不重写, 则基类的 add() 会抛异常.
                                public void add(int index, E element){
                                    throw new UnsupportedOperationException();
                                }
                              //...
                            }
                            
    4. 模板模式作用(扩展)
            (1) 扩展是指框架的扩展性, 模板模式常用在框架的开发中, 让框架用户可以在不修改框架源码的情况下, 定制化框架的功能.
            (2) 应用
                    a Java Servlet
                    
                        public class HttpServlet {
                            public void service(ServletRequest req, ServerHttpResponse res) throws Exception{
                                HttpServletRequest request;
                                HttpServletResponse response;
                                if (!(req instanceof HttpServletRequest && res instanceof HttpServletResponse)){
                                    throw new ServletException("non-HTTP request or response");
                                }
                                request = (HttpServletRequest)req;
                                response = (HttpServletResponse)res;
                                service(request,response);
                            }
                            protected void service(HttpServletRequest req,HttpServletResponse resp)throws Exception{
                                if (method.equals(METHOD_HEAD)){
                                    long lastModified = getLastModified(req);
                                    maybeSetLastModified(resp,lastModified);
                                    doHead(req,resp);
                                }else if (method.equals(METHOD_POST)){
                                    doPost(req,resp);
                                }else if (method.equals(METHOD_PUT)){
                                    doPut(req,resp);
                                }else if (method.equals(METHOD_DELETE)){
                                    doDelete(req,resp);
                                }else if (method.equals(METHOD_OPTIONS)){
                                    doOptions(req,resp);
                                }else if (method.equals(METHOD_TRACE)){
                                    doTrace(req,resp);
                                }else {
                                    String errMsg = lString.getString("http.method_not_implemented");
                                    Object[] errArgs = new Object[1];
                                    errArgs[0] = method;
                                    errMsg = MessageFormat.format(errMsg,errArgs);
                                    resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED,errMsg;
                                }
                            }
                          //...
                          } 
                          
                        HttpServlet 的 service() 方法就是一个模板方法, 实现整个 HTTP 请求的执行流程, doHead(), doPost() 是模板中
                        可以由子类来定制的部分, 让框架用户在不用修改 Servlet 框架源码的情况下, 将业务代码通过扩展点镶嵌到框架中执行.
                        
                    b. JUnit TestCase
                          在使用 JUnit 测试框架来编写单元测试时, 编写的测试类都要继承框架提供的 TestCase 类, 在 TestCase 类中, 
                          runBare() 函数是模板方法, 定义了执行测试用例的整体流程：先执行 setUp() 做些准备工作, 然后执行 runTest()
                         运行真正的测试代码, 最后执行 tearDown() 做扫尾工作
                         
    5. 回调
            (1) 概念
                    a.  A 类事先注册某个函数 F 到 B类, A 类在调用 B 类的 P 函数的时, B 类反过来调用 A 类注册给它的 F 函数. F 函数就是
                        “回调函数”. A 调用 B, B 反过来又调用 A, 这种调用机制就叫作"回调"
                        
                    b. 回调可以分为同步回调和异步回调(延迟回调), 同步回调指在函数返回之前执行回调函数；异步回调指的是在函数返回之后
                       执行回调函数.从应用场景上来看, 同步回调看起来更像模板模式, 异步回调看起来更像观察者模式
                       
            (2) JdbcTemplate 应用
                    a. JdbcTemplate, RedisTemplate, RestTemplate 这些都是基于同步回调
                    
            (3) setClickListener() 应用
                    给控件注册事件监听器,
                    
                        Button button = (Button)findViewById(R.id.button);
                        button.setOnclickListener(new OnclickListener(){
                         @Override
                         public void onclick(View v){
                         System.out.println("I am clicked.");
                         }
                        });
                        
                    这个是异步回调函数, setOnClickListener() 函数中注册好回调函数后, 并不需要等待回调函数执行, 
                    异步回调更像观察者模式
                    
            (4) addShutdownHook() 应用
                    
    6. 模板模式 VS 回调
            (1) 从应用场景上看, 同步回调跟模板模式几乎一致, 异步回调跟观察者模式差不多.
            (2) 从代码实现上来看, 回调和模板模式完全不同. 回调基于组合关系来实现, 把一个对象传递给另一个对象, 是一种对象之间的关系；
                模板模式基于继承关系来实现, 子类重写父类的抽象方法, 是一种类之间的关系, 回调相对于模板模式会更加灵活
                        
```

## 策略模式
```shell
    1. 概念
            (0) 策略模式的原理和实现, 以及如何用它来避免分支判断逻辑, 策略模式的应用场景以及真正的设计意图.
            (1) 可以使用策略模式避免冗长的 if-else 或 switch 分支判断
            (2) 策略模式(Strategy Design Pattern), 定义一族算法类, 将每个算法分别封装起来, 让它们可以互相替换. 策略
                模式可以使算法的变化独立于使用它们的使用侧, 策略模式解耦的是策略的定义, 创建, 使用这三部分
            (3) 策略模式主要的作用还是解耦策略的定义, 创建和使用, 控制代码的复杂度, 让每个部分都不至于过于复杂, 代码量过多.
                策略模式还能让其满足开闭原则, 添加新策略时, 最小化、集中化代码改动, 减少引入 bug 的风险.额外规避 if-else
    2. 策略的定义
            (1) 包含一个策略接口和一组实现这个接口的策略类(每个策略类实现业务不同), 因为所有的策略类都实现相同的接口, 客户端代码基于
                接口而非实现编程, 可以灵活地替换不同的策略
                
                    public interface Strategy {
                        void algorithmInterface();
                    }
                    
                    public class ConcreteStrategyA implements Strategy {
                        @Override
                        public void algorithmInterface() {
                            //...具体的算法...
                        }
                    }
                    
                    public class ConcreteStrategyB implements Strategy {
                        @Override
                        public void algorithmInterface() {
                            //...具体的算法...
                        }
                    }
                    
    3. 策略的创建
            (1) 策略模式包含一组策略, 在使用它们时, 会通过类型(type)来判断创建哪个策略来使用, 为了封装创建逻辑, 需要对客户端代码
                屏蔽创建细节, 可以把根据 type 创建策略的逻辑抽离出来, 放到工厂类
                
                a. 
                    这是策略类是无状态的, 不包含成员变量, 可以进行共享, 不需要每次都创建新的策略对象, 可以事先创建好每个策略对象, 缓存到
                    工厂类中, 用的时候直接返回
                
                    public class StrategyFactory {
                        private static final Map<String, Strategy> strategies = new HashMap<>();
                        static {
                            strategies.put("A",new ConcreteStrategyA());
                            strategies.put("B",new ConcreteStrategyB());
                        }
                        
                        public static Strategy getStrategy(String type){
                            if (type == null || type.isEmpty()){
                                throw new IllegalArgumentException("type should not be empty");
                            }
                            return strategies.get(type);
                        }
                    }
                    
                b.
                    如果策略类是有状态的, 根据业务场景的需要, 每次从工厂方法中, 获得的都是新创建的策略对象, 而不是缓存好可共享的
                    策略对象
                    
                    public class StrategyFactory {
                    
                        public static Strategy getStrategy(String type){
                            if (type == null || type.isEmpty()){
                                throw new IllegalArgumentException("type should not be empty.");
                            }
                            
                            if (type.equals("A")){
                                return new ConcreteStrategyA();
                            }else if (type.equals("B")){
                                return new ConcreteStrategyB();
                            }
                            
                            return null;
                        }
                    }
              
    4. 策略的使用
            (1) 策略模式最典型的使用应用场景是, 运行时动态确定使用哪种策略.
            
                    public class UserCache {
                        private Map<String, User> cacheData = new HashMap<>();
                        private EvictionStrategy eviction;
                        public UserCache(EvictionStrategy eviction){
                            this.eviction = eviction;
                        }
                        //...
                    }
                    
                    public class Application {
                        //运行时动态确定, 根据配置文件的配置决定使用哪种策略
                        public static void main(String[] args) throws Exception {
                            EvictionStrategy evictionStrategy = null;
                            Properties properties = new Properties();
                            // 加载配置文件
                            properties.load(new FileInputStream("./config.properties"));
                            String type = properties.getProperty("eviction_type");
                            
                            evictionStrategy = EvictionStrategyFactory.getEvictionStrategy(type);
                            UserCache userCache = new UserCache(evictionStrategy);
                            //...
                        }
                    }
                    
            (2) 利用策略模式避免分支判断
                     策略模式适用于根据不同类型动态, 决定使用哪种策略这样一种应用场景.　这得益于策略工厂类, 更本质上是借助“查表法”,
                     根据 type 查表来替代根据 type 分支判断.
                     
                     重构前:
                            public class OrderService {
                                public double discount(Order order){
                                    double discount = 0.0;
                                    OrderType type = order.getType();
                                    if (type.equals(OrderType.NORMAL)){
                                        //普通订单
                                        //...省略折扣计算算法代码
                                    }else if (type.equals(OrderType.GROUPON)){
                                    
                                        //团购订单  ...省略折扣计算算法代码
                                        
                                    }else if (type.equals(OrderType.PROMOTION)){
                                        //促销订单 ...省略折扣计算算法代码
                                    }
                                    return discount;
                                }
                            }
                            
                     重构后(将不同类型订单的打折策略设计成策略类, 并由工厂类来负责创建策略对象)
                     
                            public interface DiscountStrategy {
                                double calDiscount(Order order);
                            }
                            
                            // 策略定义
                            public class NormalDiscountStrategy implements DiscountStrategy {
                            
                             double calDiscount(Order order)
                             {
                             ......
                             }
                            }
                            //省略 GrouponDiscountStrategy PromotionDiscountStrategy的策略定义
                            
                            //策略的创建, 工厂模式
                            public class DiscountStrategyFactory {
                                private static final Map<OrderType,DiscountStrategy> strategies = new HashMap<>();
                                static {
                                    strategies.put(OrderType.NORMAL,new NormalDiscountStrategy());
                                    strategies.put(OrderType.GROUPON,new GrouponDiscountStrategy());
                                    strategies.put(OrderType.PROMOTION,new PromotionDiscountStrategy());
                                }
                                public static DiscountStrategy getDiscountStrategy(OrderType type){
                                    return strategies.get(type);
                                }
                            }
                            
                            public class OrderService {
                                public double discount(Order order){
                                    OrderType type = order.getType();
                                    DiscountStrategy discountStrategy = DiscountStrategyFactory.getDiscountStrategy(type);
                                    return discountStrategy.calDiscount(order);
                                }
                            }
                    
            
            (3)　消除 if-else 业务, 通过构建表
                    重构前:
                            public class Sorter {
                                private static final long GB = 1000*1000*1000;
                                public void sortFile(String filePath){
                                    //省略校验逻辑
                                    File file = new File(filePath);
                                    long fileSize = file.length();
                                    ISortAlg sortAlg;
                                    if (fileSize < 6*GB){
                                        sortAlg = SortAlgFactory.getSortAlg("QuickSort");
                                    }else if (fileSize < 10*GB){
                                        sortAlg = SortAlgFactory.getSortAlg("ExternalSort");
                                    }else if (fileSize < 100*GB){
                                        sortAlg = SortAlgFactory.getSortAlg("ConcurrentExternalSort");
                                    }else {
                                        sortAlg = SortAlgFactory.getSortAlg("MapReduceSort");
                                    }
                                    sortAlg.sort(filePath);
                                }
                            }
                            
                    重构后:
                    
                        把可变的部分隔离到策略工厂类和 Sorter 类中的静态代码段中, 当要添加一个新的排序算法时, 只需要修改策略工厂类和 
                        Sort 类中的静态代码段, 其他代码都不需要修改, 这样就将代码改动最小化、集中化
                        
                        public class Sorter {
                            private static final long GB = 1000*1000*1000;
                            private static final List<AlgRange> algs = new ArrayList<>();
                            static {
                                algs.add(new AlgRange(0,6*GB, SortAlgFactory.getSortAlg("QuickSort")));
                                algs.add(new AlgRange(6*GB,10*GB, SortAlgFactory.getSortAlg("ExternalSort")));
                                algs.add(new AlgRange(10*GB,100*GB, SortAlgFactory.getSortAlg("ConcurrentExternalSort")));
                                algs.add(new AlgRange(100*GB,Long.MAX_VALUE, SortAlgFactory.getSortAlg("MapReduceSort")));
                            }
                        
                            public void sortFile(String filePath) {
                                //省略校验逻辑
                                File file = new File(filePath);
                                long fileSize = file.length();
                                ISortAlg sortAlg = null;
                                for (AlgRange algRange:algs){
                                    if (algRange.inRange(fileSize)){
                                        sortAlg = algRange.getAlg();
                                        break;
                                    }
                                }
                                sortAlg.sort(filePath);
                            }
                            
                            private static class AlgRange{
                                private long start;
                                private long end;
                                private ISortAlg alg;
                        
                                public AlgRange(long start, long end, ISortAlg alg) {
                                    this.start = start;
                                    this.end = end;
                                    this.alg = alg;
                                }
                        
                                public ISortAlg getAlg() {
                                    return alg;
                                }
                                
                                public boolean inRange(long size){
                                    return size >= start && size < end;
                                }
                            }
                        }
                    
```

## 职责链模式
```shell
    1. 概念
            (1) 职责链模式(Chain Of Responsibility Design Pattern), 多个处理器(接收对象)依次处理同一个请求, 一个请求先经过 A 
                处理器处理, 然后再把请求传递给 B 处理器, B 处理器处理完后再传递给 C 处理器, 以此类推, 形成一个链条, 链条上的每个处理器
                各自承担各自的处理职责.
                
    2. 职责链实现
            包含处理器接口(IHandler)或抽象类(Handler), 以及处理器链(HandlerChain)
            
            public interface IHandler {
                boolean handle();
            }
            
            public class HandlerA implements IHandler {
                @Override
                public boolean handle() {
                    boolean handled = false;
                    //...
                    return handled;  // 是否执行成功
                }
            }
            
            public class HandlerB implements IHandler {
                @Override
                public boolean handle() {
                    boolean handled = false;
                    //...
                    return handled;  // 是否执行成功
                }
            }
            
            public class HandlerChain {
                private List<IHandler> handlers = new ArrayList<>();
                public void addHandler(IHandler handler){
                    this.handlers.add(handler);
                }
                public void handle(){
                    for (IHandler handler:handlers){
                        boolean handled = handler.handle();
                        
                        // 如果执行成功就退出
                        if (handled){
                            break;
                        }
                    }
                }
            }
            
            public class Application {
                public static void main(String[] args) {
                    HandlerChain chain = new HandlerChain();
                    chain.addHandler(new HandlerA());
                    chain.addHandler(new HandlerB());
                    chain.handle();
                }
            }
            
3. 职责链的应用场景   
            (1) 论坛发布的应用场景
            (2) 职责链模式最常用是用来开发框架的过滤器和拦截器.
```

## 状态模式
```shell
    1. 概念
            (1) 状态模式一般用来实现状态机, 状态机在游戏, 工作流引擎中比较常见. 
            (2) 有限状态机(FSM, Finite State Machine), 状态机有 3 个组成部分, 状态(State), 事件(Event), 动作(Action). 事件也称
                为转移条件(Transition Condition), 事件触发状态(State)的转移和动作的执行, 但是动作不是必须的, 也可能只转移状态,
                不执行任何动作
                
    2.  原始
            public enum State {
                SMALL(0),
                SUPER(1),
                FIRE(2),
                CAPE(3);
                private int value;
                private State(int value){
                    this.value = value;
                }
            
                public int getValue() {
                    return this.value;
                }
            }
            
            public class MarioStateMachine {
                private int score;
                private State currentState;
                public  MarioStateMachine(){
                    this.score = 0;
                    this.currentState = State.SMALL;
                }
                public void obtainMushRoom(){
                    //todo
                }
                public void obatainCape(){
                    //todo
                }
                public void obtainFireFlower(){
                    //todo
                }
                public void meetMonster(){
                    //todo
                }
                public int getScore(){
                    return this.score;
                }
                public State getCurrentState(){
                    return this.currentState;
                }
            }
            
            public class ApplicationDemo {
                public static void main(String[] args) {
                    MarioStateMachine mario = new MarioStateMachine();
                    mario.obtainMushRoom();
                    int score = mario.getScore();
                    State state = mario.getCurrentState();
                    System.out.println("mario score: "+score+"; state: "+state);
                }
            }
            
    3.  第一种方法: 分支逻辑法
            参照状态转移图, 将每一个状态转移, 原模原样地直译成代码, 这会导致存在大量的 if-else 逻辑.
            对于简单的状态机, 分支逻辑法可以接受, 但是对于复杂的状态机, 则可读性很差, 后续也不易修改代码, 维护困难.
            
                public class MarioStateMachine {
                    private int score;
                    private State currentState;
                    public  MarioStateMachine(){
                        this.score = 0;
                        this.currentState = State.SMALL;
                    }
                    public void obtainMushRoom(){
                       if (currentState.equals(State.SMALL)){
                           this.currentState = State.SUPER;
                           this.score += 100;
                       }
                    }
                    public void obatainCape(){
                        if (currentState.equals(State.SMALL) || currentState.equals(State.SUPER)){
                            this.currentState = State.CAPE;
                            this.score += 200;
                        }
                    }
                    public void obtainFireFlower(){
                        if (currentState.equals(State.SMALL) || currentState.equals(State.SUPER)){
                            this.currentState = State.FIRE;
                            this.score += 300;
                        }
                    }
                    public void meetMonster(){
                        if (currentState.equals(State.SUPER)){
                            this.currentState = State.SMALL;
                            this.score -= 100;
                            return;
                        }
                        if (currentState.equals(State.CAPE)){
                            this.currentState = State.SMALL;
                            this.score -= 200;
                            return;
                        }
                        if (currentState.equals(State.FIRE)){
                            this.currentState = State.SMALL;
                            this.score -= 300;
                            return;
                        }
                    }
                    public int getScore(){
                        return this.score;
                    }
                    public State getCurrentState(){
                        return this.currentState;
                    }
                }
                
    4. 第二种方法: 查表法
            在两张二维表中, 第一维表示当前状态, 第二维表示事件, 值表示当前状态经过事件之后, 转移到的新状态及其执行的动作, 同时可以把两个
            二维数组存储在配置文件中, 当需要修改状态机时, 只需要修改对应的配置文件就行.
            
            查表法的适用范围是简单得操作(加减乘除等), 不太适用满足条件后的一系列复杂的逻辑操作(如加减积分,再写数据库, 还有可能发送消息
            通知等等)
            
                public enum State {
                    SMALL(0),
                    SUPER(1),
                    FIRE(2),
                    CAPE(3);
                    private int value;
                    private State(int value){
                        this.value = value;
                    }
                
                    public int getValue() {
                        return this.value;
                    }
                }
                            
                public enum Event {
                    GOT_MUSHROOM(0),
                    GOT_CAPE(1),
                    GOT_FIRE(2),
                    MET_MONSTER(3);
                    private int value;
                    private Event(int value){
                        this.value = value;
                    }
                    public int getValue() {
                        return this.value;
                    }
                }
                
                public class MarioStateMachine {
                    private int score;
                    private State currentState;
                    private static final State[][] transitionTable = {
                            {SUPER,CAPE,FIRE,SMALL},
                            {SUPER,CAPE,FIRE,SMALL},
                            {CAPE,CAPE,CAPE,SMALL},
                            {FIRE,FIRE,FIRE,SMALL},
                    };
                    private static final int[][] actionTable = {
                            {+100,+200,+300,+0},
                            {+0,+200,+300,-100},
                            {+0,+0,+0,-200},
                            {+0,+0,+0,-300},
                    };
                    public MarioStateMachine(){
                        this.score = 0;
                        this.currentState = SMALL;
                    }
                    public void obtainMushRoom(){
                        executeEvent(Event.GOT_MUSHROOM);
                    }
                    public void obatainCape(){
                        executeEvent(Event.GOT_CAPE);
                    }
                    public void obtainFireFlower(){
                        executeEvent(Event.GOT_FIRE);
                    }
                    public void meetMonster(){
                        executeEvent(Event.MET_MONSTER);
                    }
                    
                    private void executeEvent(Event event){
                        int stateValue = currentState.getValue();
                        int eventValue = event.getValue();
                        this.currentState = transitionTable[stateValue][eventValue];
                        this.score = actionTable[stateValue][eventValue];
                    }
                    public int getScore(){
                        return this.score;
                    }
                    public State getCurrentState(){
                        return this.currentState;
                    }
                }
                
    5. 第三种方法: 状态模式
            通过将事件触发的状态转移和动作执行, 拆分到不同的状态类中, 来避免分支判断逻辑. 如, IMario 是状态的接口, 定义了所有的事件.
            SmallMario, SuperMario, CapeMario, FireMario 是 IMario 接口的实现类, 分别对应状态机中的 4 个状态. 原来所有的状态转移
            和动作执行的代码逻辑都集中在 MarioStateMachine 类中, 现在这些代码逻辑被分散到这 4 个状态类中.
            
                // 接口类
                public interface IMario {
                    State getName();
                    //以下是定义的事件
                    void obtainMushRoom();
                    void obtainCape();
                    void obtainFireFlower();
                    void meetMonster();
                }
                
                // SmallMario 子类
                public class SmallMario implements IMario {
                    private MarioStateMachine stateMachine;
                    public SmallMario(MarioStateMachine stateMachine){
                        this.stateMachine = stateMachine;
                    }
                    @Override
                    public State getName() {
                        return State.SMALL;
                    }
                
                    @Override
                    public void obtainMushRoom() {
                        stateMachine.setCurrentState(new SuperMario(stateMachine));
                        stateMachine.setScore(stateMachine.getScore()+100);
                    }
                
                    @Override
                    public void obtainCape() {
                        stateMachine.setCurrentState(new CapeMario(stateMachine));
                        stateMachine.setScore(stateMachine.getScore()+200);
                    }
                
                    @Override
                    public void obtainFireFlower() {
                        stateMachine.setCurrentState(new FireMario(stateMachine));
                        stateMachine.setScore(stateMachine.getScore()+300);
                    }
                
                    @Override
                    public void meetMonster() {
                        //...do nothing...
                    }
                }
                
                // 子类
                public class SuperMario implements IMario {
                    private MarioStateMachine stateMachine;
                    public SuperMario(MarioStateMachine stateMachine){
                        this.stateMachine = stateMachine;
                    }
                    @Override
                    public State getName() {
                        return State.SUPER;
                    }
                
                    @Override
                    public void obtainMushRoom() {
                        //...do nothing...
                    }
                
                    @Override
                    public void obtainCape() {
                        stateMachine.setCurrentState(new CapeMario(stateMachine));
                        stateMachine.setScore(stateMachine.getScore()+200);
                    }
                
                    @Override
                    public void obtainFireFlower() {
                        stateMachine.setCurrentState(new FireMario(stateMachine));
                        stateMachine.setScore(stateMachine.getScore()+300);
                    }
                
                    @Override
                    public void meetMonster() {
                        stateMachine.setCurrentState(new SmallMario(stateMachine));
                        stateMachine.setScore(stateMachine.getScore()-100);
                    }
                }
                
                //...省略CapeMario FireMario类...
                
                public class MarioStateMachine {
                    private int score;
                    private IMario currentState;//不再使用枚举类表示状态
                    public MarioStateMachine(){
                        this.score = 0;
                        this.currentState = new SmallMario(this);
                    }
                    public void obtainMushRoom(){
                        this.currentState.obtainMushRoom();
                    }
                    public void obtainCape(){
                        this.currentState.obtainCape();
                    }
                    public void obtainFireFlower(){
                        this.currentState.obtainFireFlower();
                    }
                    public void meetMonster(){
                        this.currentState.meetMonster();
                    }
                    public int getScore(){
                        return this.score;
                    }
                    public State getCurrentState(){
                        return this.currentState.getName();
                    }
                    public void setScore(int score){
                        this.score = score;
                    }
                    public void setCurrentState(IMario currentState){
                        this.currentState = currentState;
                    }
                }
                
    6. 方法的选用
            (1) 像游戏这种比较复杂的状态机, 即包含的状态比较多，　优先推荐使用查表法, 因为状态模式会引入非常多的状态类, 会导致代码比较
                难维护
                
            (2) 像电商下单, 外卖下单这种类型的状态机, 它们的状态并不多, 状态转移也比较简单, 但事件触发执行的动作包含的业务逻辑可能会
                比较复杂, 推荐使用状态模式来实现.
            
```

## 迭代器模式
```shell
    1. 概念
            (1) 迭代器模式(Iterator Design Pattern), 也叫作游标模式(Cursor Design Pattern).
            (2) 迭代器的设计思路, 需要定义 hasNext(), currentItem(), next() 三个最基本的方法, 待遍历的容器对象通过依赖注
                入传递到迭代器类中, 容器通过 iterator() 方法来创建迭代器.
            (3) 迭代器模式主要作用是解耦容器代码和遍历代码
                
    2. 迭代器优点
            (1) 对于一些简单的数据结构(数组, 链表), 元素的遍历用 for 较为简单, 但是对于复杂的数据结构(比如树, 图), 它们的遍历算法
            　　比较复杂, 树有前中后序, 按层遍历, 图有深度优先, 广度优先遍历. 这些算法如果由客户端实现, 则开发难度大, 代码复杂, 容易
            　　出错, 如果将这部分遍历的逻辑写到容器类中, 也会导致容器类代码的复杂性, 所以可以将这些逻辑从类中拆分出来, 形成迭代类, 
               如针对图的遍历, 就可以定义 DFSIterator, BFSIterator 两个迭代器类, 让它们分别来实现深度优先遍历和广度优先遍历.
               
            (2) 每个迭代器独享各自的游标信息, 可以创建不同迭代器, 对同一个容器进行遍历而互不影响
            (3) 容器和迭代器都提供了抽象的接口, 是基于抽象而非实现的编程, 这样后续扩展方便, 当需要切换新的遍历算法时, 只需要将迭代器类
               从 LinkedIterator 切换为 ReversedLinkedIterator 即可, 其他代码都不需要修改.
               
    3. 迭代器适用的问题
            (1) 如果在使用迭代器遍历集合的同时增加, 删除集合中的元素, 会发生什么情况？应该如何应对？如何在遍历的同时安全地删除集合元素？
            (2) 在通过迭代器来遍历集合元素的同时, 增加或者删除集合中的元素, 会导致不可预期的行为, 要根据删除或则新增元素是在游标的前面
            　　位置还是后面位置.
            
    4. 遍历时改变集合导致的未决行为的解决方案
            增删元素之后让遍历报错, 即在 ArrayList 中定义一个成员变量　modCount(记录集合被修改的次数), 每次增加或删除元素就会给
        modCount 加 1, 当通过调用集合上的 iterator() 函数来创建迭代器时, 把 当前的 modCount 值传递给迭代器的 expectedModCount 
        成员变量, 之后每次调用迭代器上的 hasNext()、next()、currentItem() 函数都会检查集合上的 modCount 是否等于 
        expectedModCount，也就是看，在创建完迭代器之后，modCount 是否改变过, 不相等则说明元素个数发生改变, 选择 fail-fast 解决方式,
        抛出运行时异常, 结束掉程序
      
    5. 涉及具备快照功能的迭代器
            (1) 首先创建迭代器 1 时, 快照这个时刻的数据, 之后不管该集合新增, 删除元素, 该迭代器 1 都不会发生变化. 能想到的最简单的办法是
            　　 在迭代器中保存集合变量, 创建迭代器时进行集合拷贝, 这样会导致多个迭代器都保存冗余的集合元素.
            (2) 优化的方案是在容器中为每个元素新增两个时间戳(创建时间戳和删除时间戳),实现方式是创建两个数组, 迭代器本身也保存创建时间戳,
                当要进行迭代集合时, 通过迭代器的创建时间戳, 和每个元素的创建时间戳和删除时间戳比较, 如果满足条件, 则说明创建迭代器那个
                时刻该元素还存在, 可以进行访问.
```

## 访问者模式
```shell
    1. 概念
            (1) 访问者模式即通过多态的模式减少代码开发, 复用代码, 但同时对同一个类型的资源做不同的处理, 需要解耦出不同的方法, 怎么
            　　考虑不同类型资源用同一套代码, 同时又得在运行时进行不同处理
            
                     public abstract class ResourceFile {
                         protected String filePath;
                         public ResourceFile(String filePath){
                             this.filePath = filePath;
                         }
                         abstract public void accept(Extractor extractor);
                     }
                     
                     public class PdfFile extends ResourceFile {
                         public PdfFile(String filePath) {
                             super(filePath);
                         }
                     
                         @Override
                         public void accept(Extractor extractor) {
                             extractor.extract2txt(this);
                         }
                     }
                     public class PPTFile extends ResourceFile {
                         public PPTFile(String filePath) {
                             super(filePath);
                         }
                     
                         @Override
                         public void accept(Extractor extractor) {
                             extractor.extract2txt(this);
                         }
                     }
                     public class WordFile extends ResourceFile {
                         public WordFile(String filePath) {
                             super(filePath);
                         }
                     
                         @Override
                         public void accept(Extractor extractor) {
                             extractor.extract2txt(this);
                         }
                     }
                     
                     public class Extractor {
                         public void extract2txt(PdfFile pdfFile) {
                             //...
                             System.out.println("Extract PDF.");
                         }
                         public void extract2txt(PPTFile pdfFile) {
                             //...
                             System.out.println("Extract PPT.");
                         }
                         public void extract2txt(WordFile pdfFile) {
                             //...
                             System.out.println("Extract word.");
                         }
                     }
          
                     public class ToolApplication {
                         public static void main(String[] args) {
                             Extractor extractor = new Extractor();
                             List<ResourceFile> resourceFiles = listAllResourceFiles(args[0]);
                             for (ResourceFile resourceFile:resourceFiles){ 
                                 resourceFile.accept(extractor);
                             }
                         }
                         private static List<ResourceFile> listAllResourceFiles(String arg) {
                             List<ResourceFile> resourceFiles = new ArrayList<>();
                             //...根据后缀（pdf/ppt/word）由工厂方法创建不同的类对象
                             resourceFiles.add(new PdfFile("a.pdf"));
                             resourceFiles.add(new WordFile("b.word"));
                             resourceFiles.add(new PPTFile("c.ppt"));
                             return resourceFiles;
                         }
                     }
                     
            (2) 访问者模式: 允许一个或者多个操作应用到一组对象上, 解耦操作和对象本身
             
          
    2. 访问者代码
            public abstract class ResourceFile {
                protected String filePath;
                public ResourceFile(String filePath){
                    this.filePath = filePath;
                }
                abstract public void accept(Visitor visitor);
            }
            
            public class PdfFile extends ResourceFile {
                public PdfFile(String filePath) {
                    super(filePath);
                }
            
                @Override
                public void accept(Visitor visitor) {
                    visitor.visit(this);
                }
            }
            public class PPTFile extends ResourceFile {
                public PPTFile(String filePath) {
                    super(filePath);
                }
            
                @Override
                public void accept(Visitor visitor) {
                    visitor.visit(this);
                }
            }
            public class WordFile extends ResourceFile {
                public WordFile(String filePath) {
                    super(filePath);
                }
            
                @Override
                public void accept(Visitor visitor) {
                    visitor.visit(this);
                }
            }
            
            public interface Visitor {
                void visit(PdfFile pdfFile);
                void visit(PPTFile pptFile);
                void visit(WordFile wordFile);
            }
            
            public class Extractor implements Visitor {
                @Override
                public void visit(PdfFile pdfFile) {
                    //...
                    System.out.println("Extract pdf.");
                }
            
                @Override
                public void visit(PPTFile pptFile) {
                    //...
                    System.out.println("Extract ppt.");
                }
            
                @Override
                public void visit(WordFile wordFile) {
                    //...
                    System.out.println("Extract word.");
                }
            }
            
            public class Compressor implements Visitor {
                @Override
                public void visit(PdfFile pdfFile) {
                    //...
                    System.out.println("Compress pdf.");
                }
            
                @Override
                public void visit(PPTFile pptFile) {
                    //...
                    System.out.println("Compress ppt.");
                }
            
                @Override
                public void visit(WordFile wordFile) {
                    //...
                    System.out.println("Compress word.");
                }
            }
            
            public class ToolApplication {
                public static void main(String[] args) {
                    Extractor extractor = new Extractor();
                    List<ResourceFile> resourceFiles = listAllResourceFiles(args[0]);
                    for (ResourceFile resourceFile:resourceFiles){
                        resourceFile.accept(extractor);
                    }
                    
                    Compressor compressor = new Compressor();
                    for (ResourceFile resourceFile:resourceFiles){
                        resourceFile.accept(compressor);
                    }
                }
                private static List<ResourceFile> listAllResourceFiles(String arg) {
                    List<ResourceFile> resourceFiles = new ArrayList<>();
                    //...根据后缀（pdf/ppt/word）由工厂方法创建不同的类对象
                    resourceFiles.add(new PdfFile("a.pdf"));
                    resourceFiles.add(new WordFile("b.word"));
                    resourceFiles.add(new PPTFile("c.ppt"));
                    return resourceFiles;
                }
            }
            
            
            
    3. 访问者模式的应用场景
            访问者模式针对的是一组类型不同的对象(PdfFile, PPTFile, WordFile) 但它继承相同的父类(ResourceFile)或者实现相同的接口,
       在不同的应用场景下, 需要对这组对象进行一系列不相关的业务操作(抽取文本, 压缩等), 但为了避免不断添加功能导致类(PdfFile, PPTFile, 
       WordFile) 不断膨胀, 职责越来越不单一, 以及避免频繁地添加功能导致的频繁代码修改, 使用访问者模式, 将对象与操作解耦, 将这些业务操作
       抽离出来, 定义在独立细分的访问者类(Extractor, Compressor)中
       
    4. 双分派(Double Dispatch)
            (1) Single Dispatch 是执行哪个对象的方法, 根据对象的运行时类型来决定；执行对象的哪个方法, 根据方法参数的编译时类型来决定.
                Double Dispatch 是执行哪个对象的方法, 根据对象的运行时类型来决定；执行对象的哪个方法, 根据方法参数的运行时类型来决定
                
                Dispatch 就是指方法调用, 也可以理解为消息传递, 一个对象调用另一个对象的方法, 就相当于给它发送一条消息, 这条消息起码要
                包含对象名, 方法名, 方法参数
                
                Single 是因为执行哪个对象的哪个方法, 只跟“对象”的运行时类型有关,  Double 是因为执行哪个对象的哪个方法, 跟“对象”和
                “方法参数”两者的运行时类型有关.
                
                面向对象编程语言(Java, C++, C#) 都只支持 Single Dispatch, 不支持 Double Dispatch
                
    5. 代替访问者模式的解决方案
            可以用工厂模块 + 多态
            
                public abstract class ResourceFile {
                    protected String filePath;
                    public ResourceFile(String filePath){
                        this.filePath = filePath;
                    }
                    public abstract ResourceFileType getType();
                }
                
                public class PdfFile extends ResourceFile {
                    public PdfFile(String filePath) {
                        super(filePath);
                    }
                
                    @Override
                    public ResourceFileType getType() {
                        return ResourceFileType.PDF;
                    }
                }
                
                public class PPTFile extends ResourceFile {
                    public PPTFile(String filePath) {
                        super(filePath);
                    }
                
                    @Override
                    public ResourceFileType getType() {
                        return ResourceFileType.PPT;
                    }
                }
                
                public class WordFile extends ResourceFile {
                    public WordFile(String filePath) {
                        super(filePath);
                    }
                
                    @Override
                    public ResourceFileType getType() {
                        return ResourceFileType.WORD;
                    }
                }
                
                public interface Extractor {
                    void extract2txt(ResourceFile resourceFile);
                }
                
                public class PdfExtractor implements Extractor {
                    @Override
                    public void extract2txt(ResourceFile resourceFile) {
                        //...
                    }
                }
                
                public class PPTExtractor implements Extractor {
                    @Override
                    public void extract2txt(ResourceFile resourceFile) {
                        //...
                    }
                }
                
                public class WordExtractor implements Extractor {
                    @Override
                    public void extract2txt(ResourceFile resourceFile) {
                        //...
                    }
                }
                
                public class ExtractorFactory {
                    private static final Map<ResourceFileType,Extractor> extractors = new HashMap<>();
                    static {
                        extractors.put(ResourceFileType.PDF,new PdfExtractor());
                        extractors.put(ResourceFileType.PPT,new PPTExtractor());
                        extractors.put(ResourceFileType.WORD,new WordExtractor());
                    }
                    public static Extractor getExtractor(ResourceFileType type){
                        return extractors.get(type);
                    }
                }
                
                public class ToolApplication {
                    public static void main(String[] args) {
                        List<ResourceFile> resourceFiles = listAllResourceFiles(args[0]);
                        for (ResourceFile resourceFile:resourceFiles){
                            Extractor extractor = ExtractorFactory.getExtractor(resourceFile.getType());
                            extractor.extract2txt(resourceFile);
                        }
                    }
                    private static List<ResourceFile> listAllResourceFiles(String arg) {
                        List<ResourceFile> resourceFiles = new ArrayList<>();
                        //...根据后缀（pdf/ppt/word）由工厂方法创建不同的类对象
                        resourceFiles.add(new PdfFile("a.pdf"));
                        resourceFiles.add(new WordFile("b.word"));
                        resourceFiles.add(new PPTFile("c.ppt"));
                        return resourceFiles;
                    }
                }
                
    6. 取舍
            对于资源文件处理, 如果工具提供的功能并不是非常多, 只有几个而已, 更推荐使用工厂模式的实现方式, 毕竟代码更加清晰、易懂. 
            如果工具提供非常多的功能, 比如有十几个, 更推荐使用访问者模式, 访问者模式需要定义的类要比工厂模式的实现方式少很多, 类太多也会
            影响到代码的可维护性
                
```

## 备忘录模式
```shell
    1. 概念
            (1) 备忘录模式主要是用来防丢失, 撤销, 恢复.
            (2) 备忘录模式也叫快照(Snapshot)模式, Memento Design Pattern, 在不违背封装原则的前提下, 捕获一个对象的内部状态, 并在
                该对象之外保存这个状态, 以便之后恢复对象为先前的状态.
                
            (3) 
                为什么存储和恢复副本会违背封装原则?
                备忘录模式是如何做到不违背封装原则的?
    2. 
        存储和恢复副本相当于快照, 
            问题一: 为了能用快照恢复 InputText 对象, 在 InputText 类中定义了 setText() 函数, 但这个函数有可能会被其他业务使用, 
                   暴露不应该暴露的函数违背了封装原则
                   
            问题二: 快照本身是不可变的, 不应该包含任何 set() 等修改内部状态的函数, 在上面的代码中, “快照“这个业务模型复用了 
                   InputText 类的定义, 而 InputText 类本身有一系列修改内部状态的函数, 用 InputText 类来表示快照违背封装原则.
                   
        解决方案: 
                 第一, 定义一个独立的类(Snapshot 类)来表示快照, 而不是复用 InputText 类, Snapshot 类只暴露 get() 方法, 没有
                      set() 等任何修改内部状态的方法. 
                 第二, 在 InputText 类中, 把 setText() 方法重命名为 restoreSnapshot() 方法, 用意更加明确, 只用来恢复对象
        重构后:
                public class InputText {
                    private StringBuilder text = new StringBuilder();
                 
                    public String getText(){
                        return text.toString();
                    }
                    public void append(String input){
                        text.append(input);
                    }
                    public Snapshot createSnapshot(){
                        return new Snapshot(text.toString());
                    }
                    public void restoreSnapshot(Snapshot snapshot){
                        this.text.replace(0,this.text.length(),snapshot.getText());
                    }
                }
                
                public class Snapshot {
                    private String text;
                    public Snapshot(String text){
                        this.text = text;
                    }
                    public String getText(){
                        return this.text;
                    }
                }
                
                public class SnapshotHolder {
                    private Stack<Snapshot> snapshots = new Stack<>();
                    public Snapshot popSnapshot(){
                        return snapshots.pop();
                    }
                    public void pushSnapshot(Snapshot snapshot){
                        snapshots.push(snapshot);
                    }
                }
                
                public class ApplicationMain {
                    public static void main(String[] args) {
                        InputText inputText = new InputText();
                        SnapshotHolder snapshotHolder = new SnapshotHolder();
                        Scanner scanner = new Scanner(System.in);
                        while (scanner.hasNext()){
                            String input = scanner.next();
                            if (input.equals(":list")){
                                System.out.println(inputText.toString());
                            }else if (input.equals(":undo")){
                                Snapshot snapshot = snapshotHolder.popSnapshot();
                                inputText.restoreSnapshot(snapshot);
                            }else{
                                snapshotHolder.pushSnapshot(inputText.createSnapshot());
                                inputText.append(input);
                            }
                        }
                    }
                }
                
    3. 备份时机
            采用 “低频率全量备份” 和 “高频率增量备份”相结合的方法
```

## 命令模式
```shell
    1. 概念
            (1) 命令模式(Command Design Pattern), 命令模式将请求(命令)封装为一个对象, 可以使用不同的请求参数化其他对象(将不
                同请求依赖注入到其他对象), 并且能够支持请求(命令)的排队执行, 记录日志, 撤销等(附加控制)功能.
                
            (2) 命令模式的核心是将函数封装成对象, 具体做法是设计一个包含这个函数的类, 实例化一个对象, 就可以实现把函数当对象使用, 
            　　同时就可以存储下来
            (3) 命令模式的主要作用和应用场景, 是用来控制命令的执行, 如, 异步, 延迟, 排队执行命令, 撤销重做命令, 存储命令, 给命令记录
                日志等等
                
    2. 命令模式应用
            (1) 服务器轮询获取客户端发来的请求, 获取到请求后, 借助命令模式, 把请求包含的数据和处理逻辑封装为命令对象, 存储在内存队列中,
                再从队列中取出一定数量的命令来执行, 执行完成后, 再重新开始新的一轮轮询
                
            (2) 示例
                    public interface Command {
                        void execute();
                    }
                    
                    public class GotDiamondCommand implements Command {
                        //省略成员变量
                        public GotDiamondCommand(){
                            //...
                        }
                        @Override
                        public void execute() {
                            //执行相应的逻辑
                        }
                    }
                    
                    public class GotStartCommand implements Command {
                        //省略成员变量
                        public GotStartCommand(){
                            //...
                        }
                        @Override
                        public void execute() {
                            //执行相应的逻辑
                        }
                    }
                    
                    
                    public class GameApplication {
                        private static final int MAX_HANDLED_REQ_COUNT_PER_LOOP=100;
                        private Queue<Command> queue = new LinkedList<>();
                        public void mainloop(){
                            while (true){
                                List<Request> requests = new ArrayList<>();
                                //省略从epoll或select获取数据，并封装为Request的逻辑
                                //注意设置超时时间，如果很长时间没有接收到请求，就继续下面的逻辑处理
                                for (Request request:requests){
                                    Event event = request.getEvent();
                                    Command command = null;
                                    if (event.equals(Event.GOT_DIAMOND)){
                                        command = new GotDiamondCommand();
                                    }else if (event.equals(Event.GOT_STAR)){
                                        command = new GotStartCommand();
                                    }
                                    queue.add(command);
                                }
                                int handledCount = 0;
                                while (handledCount < MAX_HANDLED_REQ_COUNT_PER_LOOP){
                                    if (queue.isEmpty()){
                                        break;
                                    }
                                    Command command = queue.poll();
                                    command.execute();
                                }
                            }
                        }
                    }
                    
    3. 命令模式 VS 策略模式
            业务常见不一样, 在策略模式中, 不同的策略具有相同的目的(业务上), 不同的实现, 互相之间可以替换. 如 BubbleSort,
       SelectionSort 都是为了实现排序的, 只不过一个是用冒泡排序算法来实现, 另一个是用选择排序算法来实现. 命令模式, 是不同的命令具有
       不同的目的, 对应不同的处理逻辑, 并且互相之间不可替换
```

## 解释器模式
```shell
    1. 概念
            (1) 解释器模式描述如何构建一个简单的“语言”解释器, 比如编译器, 规则引擎, 正则表达式.
            (2) 解释器模式(Interpreter Design Pattern), 为某个语言定义它的语法表示, 并定义一个解释器用来处理这个语法, 用来实现
                根据语法规则解读“句子”的解释器
                
            (3) 解释器模式的代码实现比较灵活, 没有固定的模板. 应用设计模式主要是应对代码的复杂性, 解释器模式也不例外, 核心思想
                就是将语法解析的工作拆分到各个小类中, 以此来避免大而全的解析类. 一般的做法是将语法规则拆分一些小的独立的单元, 然后对
                每个单元进行解析, 最终合并为对整个语法规则的解析.
                
    2. 设计告警规则(key1 > 100 && key2 < 1000 || key3 == 200)
            public interface Expression {
                boolean interpret(Map<String,Long> stats);
            }
            
            public class GreaterExpression implements Expression {
                private String key;
                private long value;
                public GreaterExpression(String strExpression){
                    String[] elements = strExpression.trim().split("\\s+");
                    if (elements.length != 3 || !elements[1].trim().equals(">")){
                        throw new RuntimeException("Expression is invalid: "+strExpression);
                    }
                    this.key = elements[0].trim();
                    this.value = Long.parseLong(elements[2].trim());
                }
                
                public GreaterExpression(String key,long value){
                    this.key = key;
                    this.value = value;
                }
                @Override
                public boolean interpret(Map<String, Long> stats) {
                    if (!stats.containsKey(key)){
                        return false;
                    }
                    long statValue = stats.get(key);
                    return statValue > value;
                }
            }
            
            public class LessExpression implements Expression {
                private String key;
                private long value;
                public LessExpression(String strExpression){
                    String[] elements = strExpression.trim().split("\\s+");
                    if (elements.length != 3 || !elements[1].trim().equals("<")){
                        throw new RuntimeException("Expression is invalid: "+strExpression);
                    }
                    this.key = elements[0].trim();
                    this.value = Long.parseLong(elements[2].trim());
                }
            
                public LessExpression(String key,long value){
                    this.key = key;
                    this.value = value;
                }
                @Override
                public boolean interpret(Map<String, Long> stats) {
                    if (!stats.containsKey(key)){
                        return false;
                    }
                    long statValue = stats.get(key);
                    return statValue < value;
                }
            }
            
            public class EqualExpression implements Expression {
                private String key;
                private long value;
                public EqualExpression(String strExpression){
                    String[] elements = strExpression.trim().split("\\s+");
                    if (elements.length != 3 || !elements[1].trim().equals("==")){
                        throw new RuntimeException("Expression is invalid: "+strExpression);
                    }
                    this.key = elements[0].trim();
                    this.value = Long.parseLong(elements[2].trim());
                }
                public EqualExpression(String key,long value){
                    this.key = key;
                    this.value = value;
                }
                @Override
                public boolean interpret(Map<String, Long> stats) {
                    if (!stats.containsKey(key)){
                        return false;
                    }
                    long statValue = stats.get(key);
                    return statValue == value;
                }
            }
            
            public class AndExpression implements Expression {
                private List<Expression> expressions = new ArrayList<>();
                public AndExpression(String strAndExpression){
                    String[] strExpressions = strAndExpression.split("&&");
                    for (String strExpr:strExpressions){
                        if (strExpr.contains(">")){
                            expressions.add(new GreaterExpression(strExpr));
                        }else if (strExpr.contains("<")){
                            expressions.add(new LessExpression(strExpr));
                        }else if (strExpr.contains("==")){
                            expressions.add(new EqualExpression(strExpr));
                        }else {
                            throw new RuntimeException("Expression is invalid: "+strAndExpression);
                        }
                    }
                }
                
                public AndExpression(List<Expression> expressions){
                    this.expressions.addAll(expressions);
                }
                @Override
                public boolean interpret(Map<String, Long> stats) {
                    for (Expression expr:expressions){
                        if (!expr.interpret(stats)){
                            return false;
                        }
                    }
                    return true;
                }
            }
            
            public class OrExpression implements Expression {
                private List<Expression> expressions = new ArrayList<>();
                public OrExpression(String strOrExpression){
                    String[] orExpressions = strOrExpression.split("\\|\\|");
                    for (String orExpr : orExpressions){
                        expressions.add(new OrExpression(orExpr));
                    }
                }
                
                public OrExpression(List<Expression> expressions){
                    this.expressions.addAll(expressions);
                }
                @Override
                public boolean interpret(Map<String, Long> stats) {
                    for (Expression expr:expressions){
                        if (expr.interpret(stats)){
                            return true;
                        }
                    }
                    return false;
                }
            }
            
            public class AlertRuleInterpreter {
                private Expression expression;
                public AlertRuleInterpreter(String ruleExpression){
                    this.expression = new OrExpression(ruleExpression);
                }
                public boolean interpret(Map<String,Long> stats){
                    return expression.interpret(stats);
                }
            }
            
            public class DemoTest {
                public static void main(String[] args) {
                    String rule = "key1 > 100 && key2 < 30 || key3 < 100 || key4 == 88";
                    AlertRuleInterpreter interpreter = new AlertRuleInterpreter(rule);
                    Map<String,Long> stats = new HashMap<>();
                    stats.put("key1",101L);
                    stats.put("key3",121L);
                    stats.put("key4",88L);
                    boolean alert = interpreter.interpret(stats);
                    System.out.println(alert);
                }
            } 
```

## 中介模式
```shell
    1. 概念
            (1) 中介模式(Mediator Design Pattern), 中介模式定义一个单独的(中介)对象, 来封装一组对象之间的交互, 将这组对象之间的
                交互委派给与中介对象交互, 来避免对象之间的直接交互. 通过引入中介这个中间层, 将一组对象之间的交互关系(或者说依赖关系)
                从多对多(网状关系)转换为一对多(星状关系), 即原来一个对象要跟 n 个对象交互, 现在只需要跟一个中介对象交互, 从而最小化
                对象之间的交互关系, 降低代码的复杂度, 提高代码的可读性和可维护性.
                
    2. 实例
            (1) 航空管制
                    每架飞机都需要知道其他飞机每时每刻的位置, 这就需要时刻跟其他飞机通信, 飞机通信形成的通信网络就会很复杂,  通过引
                入“塔台”这样一个中介, 让每架飞机只跟塔台来通信, 发送自己的位置给塔台, 由塔台来负责每架飞机的航线调度, 大大简化通信网络
               
            (2) 各个控件只跟中介对象交互, 中介对象负责所有业务逻辑的处理
                    
    3. 观察者模式 VS　中介模式
            在观察者模式中, 交互关系往往都是单向的, 一个参与者要么是观察者, 要么是被观察者, 不会兼具两种身份, 即在观察者模式的应用场景中,
        参与者之间的交互关系比较有条理.
        
            中介模式正好相反, 只有当参与者之间的交互关系错综复杂, 维护成本很高时, 才考虑使用中介模式, 中介模式的应用会带来一定的副作用,
        有可能会产生大而复杂的上帝类, 如果一个参与者状态的改变, 其他参与者执行的操作有一定先后顺序的要求, 中介模式就可以利用中介类, 
        通过先后调用不同参与者的方法来实现顺序的控制, 而观察者模式是无法实现这样的顺序要求的
```

