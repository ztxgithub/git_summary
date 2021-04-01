# 开源实战
```shell
    1. 
```

## JavaJDK　源码
```shell
    1. 工厂模式在 Calendar 类中的应用, 根据参数类型, 创建出相关类型(继承同一父类)的子类
    2. 建造者模式在 Calendar 类中的应用, 根据参数定制化相同的类型的对象.
    3. 装饰器模式在 Collections 类中的应用
```

## Unix 源码
```shell
    1. 应对复杂软件开发的方法论
       从设计原则和思想的角度来看, 如何应对庞大而复杂的项目开发?
       从研发管理和开发技巧的角度来看, 如何应对庞大而复杂的项目开发?
       聚焦在 Code Review 上来看, 如何通过 Code Reviwe 保持项目的代码质量?
       
    2. 从设计原则和思想的角度
        (1) 封装与抽象
                linux 中"一切皆文件”就体现了封装和抽象的设计思想, cpu, 内存使用率, 都可以通过 read 某个文件. 抽象和封装控制代码的复杂程度,
           提供简单, 统一的访问接口, 基于抽象而非实现的编程.
           
        (2) 分层与模块化
                系统划分成各个独立的模块, 比如进程调度, 进程通信, 内存管理, 虚拟文件系统, 网络接口等模块, 不同的模块之间通过接口来进行通信,
           模块之间耦合小
           
                Unix 系统也是基于分层开发的, 分为三层, 分别是 内核, 系统调用, 应用层. 每一层都对上层封装实现细节, 暴露抽象的接口来调用.
           任意一层都可以被重新实现, 不会影响到其他层的代码.
           
                善于应用分层技术, 把容易复用, 跟具体业务关系不大的代码, 尽量下沉到下层, 把容易变动, 跟具体业务强相关的代码, 尽量上移到上层
                
        (3) 基于接口通信
                接口从命名到定义都要抽象一些, 尽量少涉及具体的实现细节, 例如 open() 文件操作函数, 底层实现非常复杂, 涉及权限控制, 并发控制,
           物理存储.
           
        (4) 为扩展而设计
                提前思考项目中未来可能会有哪些功能需要扩展, 提前预留好扩展点, 以便在未来需求变更时, 在不改动代码整体结构的情况下, 轻松地添加
           新功能, 同时满足开闭原则, 识别出代码可变部分和不可变部分, 将可变部分封装起来, 隔离变化, 提供抽象化的不可变接口, 供上层系统使用.
           当具体的实现发生变化时, 只需要基于相同的抽象接口, 扩展一个新的实现, 替换掉老的实现即可, 上游系统的代码几乎不需要修改.
           
        (5) 最小惊奇原则
                就是遵守开发规范, 代码规范, 确保良好可读性.
                
    3. 从研发管理和开发技巧
        (1) 严格执行代码规范
        (2) 编写高质量的单元测试
        (3) Code Review
        (4) 先详细写好概要设计
        (5) 持续重构
                持续的小迭代
```

## Google Guava
```shell
    1. 概念
            (1) Google Guava 是 Google 开源的 Java 开发库
            (2) 如何在业务开发中, 发现通用的功能模块, 以及如何将它开发成类库、框架或者功能组件.
            (3) 跟业务无关的通用功能模块, 有类库(library), 框架(framework), 功能组件(component), Google Guava 属于类库,
                提供一组 API 接口. EventBus, DI 容器属于框架, 提供骨架代码, 能让业务开发人员聚焦在业务开发部分, 在预留的扩展点里填充
                业务代码. ID 生成器, 性能计数器属于功能组件, 提供一组具有某一特殊功能的 API 接口, 相比于类库, 但更加聚焦和重量级, 
                如 ID 生成器有可能会依赖 Redis 等外部系统, 不像类库那么简单
            
    2. 如何发现通用的功能模块
            (1) 通用功能模块的标准就是复用和与业务无关.
            
    3. 如何开发通用的功能模块
            (1) 对于一个技术产品, 需要达到 bug 少, 性能好, 是否易用, 易集成, 易插拔, 文档是否全面, 是否容易上手.
            (2) 产品意识, 服务意识, 代码质量意识, 不要重复造轮子等
            
```
### Google Guava 设计模式
```shell
    1. 概念
    2. Builder 模式在 Guava 中的应用
            Guava 缓存的实现运用了 build 模式
            
                public class CacheDemo {
                 public static void main(String[] args) {
                 Cache<String, String> cache = CacheBuilder.newBuilder()
                 .initialCapacity(100)
                 .maximumSize(1000)
                 .expireAfterWrite(10, TimeUnit.MINUTES)
                 .build();
                 
                 cache.put("key1", "value1");
                 String value = cache.getIfPresent("key1");
                 System.out.println(value);
                 }
                }
                
            真正构造 Cache 对象时, 必须做一些必要的参数校验, 如果采用无参默认构造函数 加 setXXX() 方法的方案, 这校验代码就无处安放, 
            而不经过校验, 创建的 Cache 对象有可能是不合法、不可用的
            
    3. Wrapper 模式在 Guava 中的应用
            (1) 代理模式, 装饰器, 适配器模式可以统称为 Wrapper 模式, 都可以通过组合的方式(原始类依赖注入 wrapper 类), 
                将 Wrapper 类的函数委托给原始类的函数来实现.
                
            (2) 简化 Wrapper 模式的代码实现, Guava 提供一系列缺省的 Forwarding 类, 用户在实现自己的 Wrapper 类时, 基于缺省的
               Forwarding 类来扩展, 就可以只实现自己关心的方法, 其他不关心的方法使用缺省 Forwarding 类的实现, 而不是基于 Collection
               最基础的抽象类开发(需要对每一个抽象类中抽象方法进行实现)
               
    4. Immutable 模式在 Guava 中的应用
            (1) Immutable 模式是不变模式, 涉及的类就是不变类(Immutable Class), 对象就是不变对象(Immutable Object)
            (2) 不变模式可以分为两类
                    第一类是普通不变模式(通常意义的不变模式), 对象中包含的引用对象是可以改变的
                    第二类是深度不变模式(Deeply Immutable Pattern), 对象包含的引用对象也不可变
                    
            (3) 不变集合, Google Guava 针对集合类(Collection, List, Set, Map…) 提供对应的不变集合类(ImmutableCollection, 
                ImmutableList, ImmutableSet, ImmutableMap…), 这个不变集合属于普通不变模式, 集合中的对象不会增删, 但是对象的成员
                变量（或叫属性值）是可以改变的.
                
                        public class ImmutableDemo {
                          public static void main(String[] args) {
                          
                            List<String> originalList = new ArrayList<>();
                            originalList.add("a");
                            originalList.add("b");
                            originalList.add("c");
                        
                            List<String> jdkUnmodifiableList = Collections.unmodifiableList(originalList);
                            List<String> guavaImmutableList = ImmutableList.copyOf(originalList);
                        
                            jdkUnmodifiableList.add("d"); // 抛出 UnsupportedOperationException
                            guavaImmutableList.add("d"); // 抛出 UnsupportedOperationException
                            
                            // 修改源数据本身
                            originalList.add("d");
                        
                            print(originalList); // a b c d
                            print(jdkUnmodifiableList); // a b c d
                            print(guavaImmutableList); // a b c
                          }
                        
                        
                        }
```

### Google Guava 函数式编程
```shell
    1. 概念
            (1) 函数式编程的特殊性, 适合用在科学计算, 数据处理, 统计分析等领域
            (2) 函数式编程中的“函数", 不是编程语言中的函数, 而是数学“函数”或者“表达式”(y=f(x)), 但在编程实现时, 对于数学“函数”或
                “表达式”, 一般习惯性地将它们设计成函数. 笼统来讲, 函数式编程中的“函数”也可以理解为编程语言中的“函数”
            (3) 每个编程范式都有自己特点
                    面向过程编程最大的特点是：以函数作为组织代码的单元, 数据与方法相分离
                    面向对象编程最大的特点是：以类、对象作为组织代码的单元以及它的四大特性
                     
                    面向函数编程最大特点是: 编程思想, 程序可以用一系列数学函数或表达式的组合来表示, 函数式编程是程序面向数学的更底层
                                         的抽象, 适用于一些科学计算领域, 在实现上它的函数是无状态的, 函数不会共享类成员变量, 
                                         函数的执行结果只与入参有关, 跟其他任何外部变量无关
                                         
    2. Java 对函数式编程的支持
            (1) 
                    Optional<Integer> result = Stream.of("f", "ba", "hello")
                                               .map(s -> s.length())
                                               .filter(l -> l <= 3)
                                               .max((o1, o2) -> o1-o2);
                                               
               Java 为函数式编程引入了三个新的语法概念：Stream 类, Lambda 表达式和函数接口(Functional Inteface). Stream 类用来
               支持通过“.”级联多个函数操作的代码编写方式；引入 Lambda 表达式的作用是简化代码编写；函数接口的作用是可以把函数包裹成
               函数接口, 来实现把函数当做参数一样来使用.
                                               
            (2) Stream 类
                    对于表达式 (3 - 1) * 2 + 5, 
                        普通函数调用 add(multiply(subtract(3,1),2),5) 
                        函数式调用 subtract(3,1).multiply(2).add(5); 
                        
                        java 规定每个函数都返回一个通用的类型(Stream 类对象), 在 Stream 类上的操作有两种中间操作和终止操作,
                        中间操作返回的仍然是 Stream 类对象, 而终止操作返回的是确定的值结果
                        例如:
                            Optional<Integer> result = Stream.of("f", "ba", "hello") // of返回Stream<String>对象
                                                             .map(s -> s.length()) // map返回Stream<Integer>对象
                                                             .filter(l -> l <= 3) // filter返回Stream<Integer>对象
                                                             .max((o1, o2) -> o1-o2); // max终止操作：返回Optional<Integer>
                                                             
            (3) Lambda 表达式
            
                    // Stream 中 map函数的定义(map 的传入参数是函数对象)
                    public interface Stream<T> extends BaseStream<T, Stream<T>> {
                      <R> Stream<R> map(Function<? super T, ? extends R> mapper);
                      //...省略其他函数...
                    }
                    
                    // Stream 中 map 的常规使用方法：
                    Stream.of("fo", "bar", "hello").map(new Function<String, Integer>() {
                      @Override
                      public Integer apply(String s) {
                        return s.length();
                      }
                    });
                    
                    // 用 Lambda 表达式简化后的写法：
                    Stream.of("fo", "bar", "hello").map(s -> s.length());
                    
                    Lambda 表达式包括三部分：输入, 函数体, 输出
                        标准写法如下:
                            (a, b) -> { 语句1； 语句2；...; return 输出; } //a,b是输入参数
                            
                        简化写法:
                            如果输入参数只有一个, 可以省略 (), 直接写成 a->{…}
                            如果没有入参, 可以直接将输入和箭头都省略掉，只保留函数体；
                            如果函数体只有一个语句, 那可以将{}省略掉；
                            如果函数没有返回值, return 语句就可以不用写
                            
            (4) 函数接口
                    java 中的函数接口要求只包含一个未实现的方法, 只有这样 Lambda 表达式才能明确知道匹配的是哪个接口, 如果有两个未
                实现的方法, 并且接口入参, 返回值都一样, Java 在翻译 Lambda 表达式时, 就不知道表达式对应哪个方法
                
            
                
    3. Guava 对函数式编程的增强
            (1) 如果增加, 则增加 Stream 类上的操作(类似 map, filter, max 这样的终止操作和中间操作), 
                         增加更多的函数接口(类似 Function, Predicate 这样的函数接口)
                
            (2) Guava 并没有提供太多函数式编程的支持, 仅仅封装几个遍历集合操作的接口, 过度地使用函数式编程, 会导致代码可读性变差.
                对遍历集合操作做优化的原因是不使用函数式编程, 只能 for 循环, 一个一个的处理集合中的数据, 使用函数式编程, 可以大大简化
                遍历集合操作的代码编写, 一行代码就能搞定, 而且在可读性方面也没有太大损失
```

## Spring
```shell
    1. 概念
```
### Sping 设计思想
```shell
    1. 概念
            (1)  Sping 设计思想包括约定大于配置, 低侵入松耦合, 模块化轻量级等
            (2) Sping 框架是 Spring Framework 基础框架, Spring Framework 是整个 Spring 生态的基石, Spring MVC 是支持 Web 开发
                的 MVC 框架, 提供 URL 路由, Session 管理, 模板引擎等跟 Web 开发相关的一系列功能.
                
                Spring Boot 是基于 Spring Framework 开发的, 专注于微服务开发, 快速地实现一个项目的开发, 部署和运行, 包括集成很多
                第三方开发包、简化配置(规约优于配置), 集成内嵌 Web 容器(Tomcat, Jetty)等, 适合单机微服务开发.
                
                Spring Cloud 适用于构建整个微服务集群, 负责微服务集群的服务治理工作, 包含很多独立的功能组件, 如 Spring Cloud 
                Sleuth 调用链追踪, Spring Cloud Config 配置中心等
                
    2. 约定优于配置
            (1) 在用 Spring MVC 来开发 Web 应用时, 尽量简化配置, 有两种方法, 第一种是基于注解, 第二种是基于约定.
            (2) 基于注解的配置方式, 在指定类上使用指定的注解, 来替代集中的 XML 配置. 如使用 @RequestMapping 注解, 在 Controller 类
                或者接口上, 标注对应的 URL. 使用 @Transaction 注解表明支持事务等.
            (3) 约定的配置方式(约定优于配置), 优先使用默认值, 也可以单独设置, 如在 Spring JPA(基于 ORM 框架, JPA 规范的基础上,封装的
                一套 JPA 应用框架), 约定类名默认跟表名相同, 属性名默认跟表字段名相同, String 类型对应数据库的 varchar 类型, 
                long 类型对应数据库中的 bigint 类型. 代码中定义的 Order 类就对应数据库中的“order”表, 只有在偏离这一约定时, 如
                需要数据库中表命名为“order_info”而非“order”, 需要显示地去配置类与表的映射关系(Order 类 ->order_info 表),
                很好地体现了“二八法则", 只有 20 % 需要定制配置.
                
    3. 低侵入, 松耦合
            (1) 低侵入是框架代码很少耦合在业务代码中, 即要替换一个框架时, 对原有的业务代码改动会很少.
            (2) Spring 提供的 IOC 容器,  Bean 不需要继承任何父类或者实现任何接口, 仅仅通过配置, 就能纳入进 Spring 的管理中. 如果
                换一个 IOC 容器, 也只是重新配置一下就可以, 原有的 Bean 都不需要任何修改
                
    4. 模块化, 轻量级
            
    5. 再封装, 再抽象
            (1) Spring Cache, 实际上也是一种再封装, 再抽象. 定义统一, 抽象的 Cache 访问接口, 这些接口不依赖具体的 Cache 实现(
                Redis, Guava Cache, Caffeine 等)
```