# 设计原则与思想

## 概要
```shell
    1. 参考资料: https://blog.csdn.net/aha_jasper/category_9790051_3.html

```
## java 语义
```shell
    1. Java 语言使用 extends 关键字来实现继承, 子类继承父类的所有非 private 属性和方法.
            public class Animal { // 父类
                protected int age;
                protected int weight;
                
                public Animal(int age, int weight) {
                    this.age = age;
                    this.weight = weight;
                }
            }
            
            public class Dog extends Animal { // 子类
                public Dog(int age, int weight) { // 构造函数
                    super(age, weight); // 调用父类的构造函数
                }
            }
            
    2. 接口
            Java 语言通过 interface 关键字来定义接口, 接口中只能声明方法, 不能包含实现, 也不能定义属性, 类通过 implements 关键字
            来实现接口中定义的方法
            
            public interface Runnable {
                void run();
            }
            
            public class Dog implements Runnable {
                private int age; // 属性或者成员变量
                private int weight;
                
                @Override
                public void run() { // 实现接口中定义的 run() 方法
                // ...
                }
            }
            
    3. 异常
            关键字 throw 来抛出一个异常, 通过 try-catch-finally 语句来捕获异常
            
            // 自定义一个异常
            public class UserNotFoundException extends Exception { 
                public UserNotFoundException() {
                    super();
               } 
               
                public UserNotFoundException(String message, Throwable e) {
                    super(message, e);
                }
            }
            
            
            public class UserService {
                private UserRepository userRepo;
                public UserService(UseRepository userRepo) {
                    this.userRepo = userRepo;
                } 
                
                public User getUserById(long userId) throws UserNotFoundException {
                    User user = userRepo.findUserById(userId);
                    if (user == null) { // throw 用来抛出异常
                        throw new UserNotFoundException();// 代码从此处返回
                    }
                    
                    return user;
                }
            }
            
            public class UserController {
                private UserService userService;
                public UserController(UserService userService) {
                    this.userService = userService;
                } 
                
                public User getUserById(long userId) {
                    User user = null;
                    
                    try { // 捕获异常
                            user = userService.getUserById(userId);
                        } 
                    catch (UserNotFoundException e) 
                    {
                        System.out.println("User not found: " + userId);
                    } 
                    finally 
                    { 
                        // 不管异常会不会发生，finally 包裹的语句块总会被执行
                        System.out.println("I am always printed.");
                    }
                    
                    return user;
                }
            }
            
    4. 
        通过 import 关键字来引入类或者 package
```
## 概要
```shell
    1. 通过设计模式, 设计原则来解决
        如何分层, 分模块, 怎么划分类, 每个类应该具有哪些属性, 方法. 怎么设计类之间的交互. 该用继承还是组合, 
        该使用接口还是抽象类, 怎样做到解耦, 高内聚低耦合. 该用单例模式还是静态方法, 用工厂模式创建对象还是直接 new 出来. 
        如何避免引入设计模式提高扩展性的同时带来的降低可读性问题.
        
    2. 好代码需要可读性强, 易扩展, 易维护, 需要从多维度去审视, 从扩展性, 可读性, 可维护性等等.
       评价标准
            (1) 可维护性(maintainability)
                    维护无外乎就是修改 bug, 修改老的代码, 添加新的代码之类的工作. 
                    代码易维护指在不破坏原有代码设计, 不引入新的 bug 的情况下, 能够快速地修改或者添加代码. 代码的可读性好, 简洁, 
                可扩展性好, 就会使得代码易维护, 更细节的话就是, 代码分层清晰, 模块化好, 高内聚低耦合, 遵从基于接口而非实现编
                程的设计原则等等, 就意味着代码易维护.
                
            (2) 可读性(readability)
                    衡量可读性的标准是代码是否符合编码规范, 命名是否达意, 注释是否详尽, 函数是否长短合适, 模块划分是否清晰, 
                 是否符合高内聚低耦合.
                 
            (3) 可扩展性(extensibility)
                    在不修改或少量修改原有代码的情况下, 通过扩展的方式添加新的功能代码.
                    
            (4) 灵活性(flexibility)
                    当添加一个新的功能代码时, 原有的代码已经预留好扩展点, 不需要修改原有的代码, 只要在扩展点上添加新的代码即可, 
                    除了可以说代码易扩展, 也可以说代码写得好灵活．
                    
                    当要实现一个功能时, 发现原有代码中, 已经抽象出很多底层可以复用的模块, 类等代码, 可以拿来直接使用. 
                    可以说代码易复用之外,  还可以说代码写得好灵活.
                    
                    当使用某组接口时, 如果这组接口可以应对各种使用场景, 满足各种不同的需求, 除了可以说接口易用之外, 还可以说这个接口
                    设计得好灵活或者代码写得好灵活
                    
            (5) 简洁性(simplicity)
                    KISS 原则, Keep It Simple, Stupid. 代码简单, 逻辑清晰, 比较难
                    
            (6) 可复用性(reusability)
                    尽量减少重复代码的编写, 复用已有的代码, 代码可复用性跟 DRY(Don’t Repeat Yourself) 设计原则有关联.
                    
            (7) 可测试性(testability)
                    
    3. 要写出高质量的代码, 需要相关的编程方法论, 包括面向对象设计思想, 设计原则, 设计模式, 编码规范, 重构技巧等.
       例如, 面向对象中的继承、多态能写出可复用的代码；编码规范能写出可读性好的代码；
       设计原则中的单一职责, DRY, 基于接口而非实现, 里式替换原则等, 可以写出可复用, 灵活, 可读性好, 易扩展, 易维护的代码；
       设计模式可以写出易扩展的代码. 持续重构可以时刻保持代码的可维护性.
       
    4. 面向对象
        编程范式有面向过程, 面向对象和函数式编程, 其中面向对象最主流, 具有丰富的特性(封装, 抽象, 继承, 多态), 可以实现很多复杂的
        设计思路, 是很多设计原则, 设计模式编码实现的基础
        以下需要掌握知识点:
            (1) 面向对象的四大特性：封装, 抽象, 继承, 多态
            (2) 面向对象编程与面向过程编程的区别和联系
            (3) 面向对象分析, 面向对象设计, 面向对象编程
            (4) 接口和抽象类的区别以及各自的应用场景
            (5) 基于接口而非实现编程的设计思想
            (6) 多用组合少用继承的设计思想
            (7) 面向过程的贫血模型和面向对象的充血模型
    
    5. 设计原则
            设计原则是指导代码设计的一些经验总结, 对于每一种设计原则, 需要掌握设计初衷, 能解决哪些编程问题, 有哪些应用场景.
            掌握以下设计原则
                SOLID 原则 - SRP 单一职责原则
                SOLID 原则 - OCP 开闭原则
                SOLID 原则 - LSP 里式替换原则
                SOLID 原则 - ISP 接口隔离原则
                SOLID 原则 - DIP 依赖倒置原则
                DRY 原则, KISS 原则, YAGNI 原则, LOD 法则
                
    6. 设计模式
            设计模式是针对软件开发中经常遇到的一些设计问题, 总结出来的一套解决方案或者设计思路, 大部分设计模式要解决的都是代码的
       可扩展性问题. 难点是了解它们都能解决哪些问题, 掌握典型的应用场景, 并且懂得不过度应用.
           使用设计模式可以提高代码的可扩展性, 但过度不恰当地使用, 也会增加代码的复杂度, 影响代码的可读性. 在开发初期, 除非特别必须, 
       一定不要过度设计应用复杂的设计模式, 而是当代码出现问题的时候, 再针对问题, 应用原则和模式进行重构, 这样就能有效避免前期的过度设计
       
            设计模式分为三大类：创建型, 结构型, 行为型
            (1) 创建型
                    常用的有：单例模式, 工厂模式(工厂方法和抽象工厂), 建造者模式
                    不常用的有：原型模式
                    
            (2) 结构型
                    常用的有：代理模式, 桥接模式, 装饰者模式, 适配器模式
                    不常用的有：门面模式, 组合模式, 享元模式
                    
            (3) 行为型
                    常用的有：观察者模式, 模板模式, 策略模式, 职责链模式, 迭代器模式, 状态模式
                    不常用的有：访问者模式, 备忘录模式, 命令模式, 解释器模式, 中介模式
                    
    7. 编程规范
            主要解决的是代码的可读性问题, 比如, 如何给变量、类、函数命名, 如何写代码注释, 函数不宜过长, 参数不能过多等等
            
    8. 代码重构
            重构的工具就是那些面向对象设计思想, 设计原则, 设计模式, 编码规范.保证重构不出错的技术手段：单元测试和代码的可测试性
            
    9.
        面向对象、设计原则、设计模式、编程规范、代码重构本质就是为编写高质量代码服务的, 在某个场景该不该用这个设计模式, 就看能不能提高
        代码的可扩展性；要不要重构, 就看代码是否存在可读, 可维护问题等
    
```

## 面向对象
```shell
    1. 概念
            (1) 面向对象编程(OOP,  Object Oriented Programming)
                面向对象编程语言(OOPL, Object Oriented Programming Language)
                面向对象分析(OOA, Object Oriented Analysis)
                面向对象设计(OOD, Object Oriented Design)
                
            (2) 面向对象编程是一种编程范式或编程风格, 它以类或对象作为组织代码的基本单元, 并将封装、抽象、继承、多态四个特性, 
                作为代码设计和实现的基石. 其实面向对象就是将对象或类作为代码组织的基本单元, 来进行编程的一种编程范式或者编程风格, 
                并不一定需要封装, 抽象, 继承, 多态这四大特性的支持, 但是在进行面向对象编程的过程中, 人们不停地总结发现, 有了这四大特性,
                就能更容易地实现各种面向对象的代码设计思路,  例如 is-a 这种类关系(比如狗是一种动物), 通过继承这个特性就能很好地支持
                这种 is-a 的代码设计思路, 并且解决代码复用的问题, 所以继承就成了面向对象编程的四大特性之一
                
                面向对象编程语言是支持类或对象的语法机制, 并有现成的语法机制. 能方便地实现面向对象编程四大特性)封装、抽象、继承、多态)
                的编程语言
                
            (3) 继承这种特性容易造成层次不清, 代码混乱
            (4) 面向对象分析, 面向对象设计是围绕着对象或类来做需求分析和设计的.分析和设计两个阶段最终的产出是类的设计, 包括程序被拆解
                为哪些类, 每个类有哪些属性方法, 类与类之间如何交互等等.面向对象分析就是要搞清楚做什么, 面向对象设计就是要搞清楚怎么做,
                面向对象编程就是将分析和设计的的结果翻译成代码的过程.
```

### 面向对象-　四大特性
```shell
    1. 封装(Encapsulation)
            (1) 封装也叫作信息隐藏或者数据访问保护,类通过暴露有限的访问接口, 外部仅能通过类提供的方法(或者叫函数)来访问内部信息或者数据.
            (2) 封装需要编程语言本身提供一定的语法机制来支持, 即访问权限控制(public, private)
            (3) 封装的意义: 
                一方面是保护数据不被随意修改, 提高代码的可维护性；另一方面是仅暴露有限的必要接口, 提高类的易用性
                
                不做封装, 意味着任何代码可以修改类内的任何数据, 过度灵活导致不可控, 所有修改逻辑都在各个地方, 影响可读性和
            　　可维护性, 同时有些类内部变量是有关系的, 可能会导致数据间不统一. 同时封装提高类的易用性, 类内业务如果很复杂, 只需对外提供
            　　必要接口, 无需关注内部成员变量实现及关系.
            
    2. 抽象(Abstraction)
            (1) 抽象讲的是如何隐藏方法的具体实现, 让调用者只需要关心方法提供了哪些功能, 并不需要知道这些功能是如何实现的.
            (2) 该特性有时候又不是面向对象唯一区别的特性, 并不需要编程语言提供特殊的语法机制来支持, 只需要提供“函数”这一非常基础的
                语法机制, 就可以实现抽象特性.
            (3) 很多设计原则都体现抽象这种设计思想, 比如基于接口而非实现编程, 开闭原则(对扩展开放, 对修改关闭), 
                代码解耦(降低代码的耦合性)等
            (4) 意义: 一方面是提高代码的可扩展性, 维护性, 修改实现不需要改变定义, 减少代码的改动范围；
                   另一方面是处理复杂系统的有效手段, 能有效地过滤掉不必要关注的信息。
                
    3. 继承(Inheritance)
            (1) 继承是用来表示类之间的 is-a 关系, 比如猫是一种哺乳动物.
            (2) 为了实现继承这个特性, 编程语言需要提供特殊的语法机制来支持, 比如 Java 使用 extends 关键字来实现继承, 
                C++ 使用冒号（class B : public A）
            (3) 意义: 从具体实现上是为了代码复用, 从逻辑认知上是 is - a 关系.
            (4) 缺点: 如果继承层太深, 不单一, 代码可读性, 可维护性变差, 为了看懂代码得一层层跟踪, 修改父类也会影响到所有继承的子类. 
            
    4. 多态(Polymorphism)
            (1) 多态这种特性也需要编程语言提供特殊的语法机制来实现, 多态特性的实现方式是利用 继承 + 方法重写(java , c++)
                也可以有其他的继承实现方式, 
            (2) 利用接口类来实现多态特性(java)
            
                    // 接口类 interface
                    public interface Iterator {
                        String hasNext();
                        String next();
                    }
                    
                    // 实现类 implements
                    public class Array implements Iterator {
                        private String[] data;
                        public String hasNext() { ... }
                        public String next() { ... }
                        //... 省略其他方法...
                    } 
                    
                    // 实现类 implements
                    public class LinkedList implements Iterator {
                        private LinkedListNode head;
                        public String hasNext() { ... }
                        public String next() { ... }
                        //... 省略其他方法...
                    }
                    
                    public class Demo {
                    
                        // 多态
                        private static void print(Iterator iterator) {
                            while (iterator.hasNext()) {
                            System.out.println(iterator.next());
                            }
                        } 
                    
                    
                        public static void main(String[] args) {
                        
                            Iterator arrayIterator = new Array();
                            // 打印 Array 实现的方法
                            print(arrayIterator);
                            
                            // 打印 LinkedList 实现的方法
                            Iterator linkedListIterator = new LinkedList();
                            print(linkedListIterator);
                        }
                    }
                    
            (3) 利用 duck-typing 语法(python)
                    class Logger:
                        def record(self):
                            print(“I write a log into file.”)
                            
                    class DB:
                        def record(self):
                            print(“I insert data into db. ”)
                            
                    def test(recorder):
                        recorder.record()
                        
                        
                    def demo():
                        logger = Logger()
                        db = DB()
                        
                        test(logger)
                        test(db)
            
                    Logger 和 DB 两个类没有任何关系, 既不是继承关系, 也不是接口和实现的关系, 但是只要它们都有定义了
                    record() 方法, 就可以被传递到 test() 方法中, 在实际运行的时候, 执行对应的 record() 方法
                    
            (4) 意义: 提高代码的可扩展性和复用性, 可扩展是在现有的基础上, 只需要新增实现方法, 其余不变. 复用性是对于 print(xxx) 函数
            　　　　　　而言不需要重复定义多个传入参数不同的的 print(xxx) 函数.
            
            (5) 多态也是很多设计模式, 设计原则, 编程技巧的代码实现基础, 比如策略模式, 基于接口而非实现编程, 依赖倒置原则, 
                里式替换原则, 利用多态去掉冗长的 if-else 语句等等
                       
```

### 面向对象 VS 面向过程
```shell
    1. 问题
            (1) 什么是面向过程编程与面向过程编程语言？
            (2) 面向对象编程相比面向过程编程有哪些优势？
            (3) 为什么说面向对象编程语言比面向过程编程语言更高级？
            (4) 有哪些看似是面向对象实际是面向过程风格的代码？
            (5) 在面向对象编程中, 为什么容易写出面向过程风格的代码？
            (6) 面向过程编程和面向过程编程语言就真的无用武之地了吗？
            
    2. 
        面向过程编程是一种编程范式或编程风格, 它以过程(可以为理解函数)作为组织代码的基本单元, 以数据(可以理解为变量)与函数相分离为
        最主要的特点. 面向过程风格是一种流程化的编程风格, 通过拼接一组顺序执行的方法来操作数据完成一项功能.
        
        面向过程编程语言首先是一种编程语言, 它最大的特点是不支持类和对象两个语法概念, 不支持丰富的面向对象编程特性(比如继承, 多态, 封装),
        仅支持面向过程编程
        
    3. 面向对象和面向过程最大区别是代码的组织方式, 面向过程风格的代码被组织成一组方法集合及其数据结构(struct User), 方法和数据结构的
       定义是分开的. 面向对象风格的代码被组织成一组类, 方法和数据结构被绑定一起, 定义在类中.
       
    4. 面向对象的优势
            (1) OOP(面向对象编程)更适合大规模复杂程序的开发
                    对于简单程序的开发, 不管是用面向过程编程, 还是面向对象编程, 差别不会很大, 甚至有的时候, 面向过程的编程风格反倒
                更有优势, 因为需求足够简单, 整个程序的处理流程只有一条主线, 很容易被划分成顺序执行的几个步骤, 然后逐句翻译成代码, 
                这就非常适合采用面向过程这种面条式的编程风格来实现.
                   对于大规模复杂程序, 整个程序的处理流程错综复杂, 并非只有一条主线. 如果把整个程序的处理流程画出来会是一个网状结构.
                如果再用面向过程编程这种流程化、线性的思维方式, 去翻译这个网状结构, 思考如何把程序拆解为一组顺序执行的方法, 就会比较吃力.
                面向对象的编程风格比较有优势.
                    面向对象编程是以类为单位, 在面向对象编程时, 不是一上来就去思考, 如何将复杂的流程拆解为一个一个方法, 而是先去思考
                如何给业务建模, 如何将需求翻译为类, 如何给类之间建立交互关系, 而完成这些工作完全不需要考虑错综复杂的处理流程. 当有了类
                的设计后, 然后再像搭积木一样, 按照处理流程, 将类组装起来形成整个程序, 这种开发模式、思考问题的方式, 能让我们在应对复杂
                程序开发时, 思路更清晰
                    面向对象编程提供一种更加清晰的, 更加模块化的代码组织方式, 对于有数百个函数, 数百个数据结构, 通过类的形式, 将函数
                和数据结构组织起来
                
            (2) OOP(面向对象编程)风格的代码更易复用, 易扩展, 易维护
                    由于面向对象的四大特性, 封装, 抽象, 继承, 多态.
                    
                    封装特性 
                        是面向对象编程相比于面向过程编程的一个最基本的区别, 因为它基于类的概念, 只有面向对象编程, 由于提供了有限
                    访问方法, 更容易维护.
                     
                     抽象特性
                        函数本身就是一种抽象, 隐藏了具体的实现, 面向对象编程和面向过程编程都有. 但面向对象编程还提供其他抽象特性的
                     实现方式.这些实现方式是面向过程编程所不具备的, 比如基于接口实现的抽象, 基于接口的抽象, 可以不改变原有实现的情况下,
                     轻松替换新的实现逻辑, 提高代码的可扩展性
                     
                     继承特性
                        两个类有相同的方法和属性, 可以通过继承父类的形式, 提高代码的复用性
                        
                     多态特性
                         在需要修改一个功能实现时, 可以通过实现一个新的子类的方式, 在子类中重写原来的功能逻辑, 用子类替换父类, 
                     在实际的代码运行过程中, 调用子类新的功能逻辑, 而不是在原有代码上做修改.  遵从了“对修改关闭, 对扩展开放”的设计原则,
                     提高代码的扩展性. 利用多态特性, 不同的类对象可以传递给相同的方法, 执行不同的代码逻辑, 提高了代码的复用性
                     
            (3) OOP(面向对象编程)更偏于人类思维
                    从二进制, 汇编, 面向过程编程这些倾向于计算机思维方式, 而面向对象编程倾向于人的思维模式. 面向过程编程在思考如何
                设计一组指令, 告诉机器去执行这组指令, 操作某些数据,进行流式处理. 而面向对象编程在思考如何给业务建模, 如何将真实的世界
                映射为类或者对象, 以业务抽象出类.
                
    5. 面向对象使用误区
            (1) 滥用 getter、setter 方法
                    在设计实现类时, 除非真的需要, 否则尽量不要给属性定义 setter 方法,  尽管 getter 方法相对 setter 方法要安全些, 
                但是如果返回的是集合容器, 也要防范集合内部数据被修改的危险
                
            (2) 滥用全局变量和全局方法
                    大而全的 Constants 类缺点
                        第一, 影响代码的可维护性. 第二, 增加代码的编译时间, 每新增一个常量, 
                    引用这个 Constants 类都要重新编译. 第三, 会影响代码的复用性, 其他项目代码要用某个类, 而这个类有引用了 
                    Constants 类, Constants 类很多常量在这个项目中又没有用.
                    
                    尽量避免大而全的 Constants 类(常量类), 可以细分为各个方面常量类, 比如跟 MySQL 配置相关的常量, 放到 
                MysqlConstants 类中；跟 Redis 配置相关的常量, 放到 RedisConstants 类中
```

### 接口和抽象类
```shell
    1. 接口和抽象类的区别是什么？
       什么时候用接口？
       什么时候用抽象类？
       抽象类和接口存在的意义是什么, 能解决哪些编程问题？
       
    2. 抽象类
        特点(c++, java)
            (1) 抽象类不允许被实例化, 只能被继承
            (2) 抽象类可以包含属性和方法. 方法既可以进行代码实现, 也可以不进行代码实现(抽象方法)
            (3) 子类继承抽象类, 必须实现抽象类中的所有抽象方法
            
        意义
            (1) 通过子类继承抽象类, 为了代码复用.
            (2) 抽象类的抽象性(定义虚函数)可以更优雅的实现多态特性.
            
    3. 接口
        特点 (c++ 没有, java 有)
            (1) 接口不能包含属性(也就是成员变量)
            (2) 接口只能声明方法, 方法不能包含代码实现
            (3) 类实现接口的时候, 必须实现接口中声明的所有方法
            
            满足以上的条件就可以定义为接口.
            // c++ 没有接口语法特性, 可以模拟
            class Strategy { // 用抽象类模拟接口
                public:
                    ~Strategy();
                    virtual void algorithm()=0;
                protected:
                    Strategy();
            };
        意义
            (1) 接口就更侧重于解耦, 接口是对行为的一种抽象, 相当于一组协议, 调用者只需要关注抽象的接口, 不需要了解具体的实现,
                具体的实现代码对调用者透明. 接口可以降低代码间的耦合性, 提高代码的可扩展性.
                
        a. 接口表示具有某种行为特性
            
    4. 抽象类和接口区别
            (1) 抽象类是 is-a 关系, 接口表示一种 has-a 关系, 表示具有某些功能
            (2) 如果要表示一种 is-a 的关系, 并且是为了解决代码复用的问题, 用抽象类；
                如果要表示一种 has-a 关系, 并且是为了解决抽象(解耦)而非代码复用的问题, 就可以使用接口
                
    5. 基于接口而非实现编程
            (1) "基于接口而非实现编程" 这条设计原则可以将接口和实现相分离, 封装不稳定的实现, 暴露稳定的接口. 上游系统面向接口而
                非实现编程, 不依赖不稳定的实现细节, 这样当实现发生变化时, 上游系统的代码基本上不需要做改动, 以此来降低耦合性,
                提高扩展性. 
            (2) 原则
                    a. 函数的命名不能暴露任何实现细节. 比如 uploadToAliyun() 就不符合要求, 应该改为去掉 aliyun 改为更加抽象的
                       命名方式, 比如：upload()
                    b. 封装具体的实现细节. 比如跟阿里云相关的特殊上传（或下载）流程(获取 token 子类)不应该暴露给调用者.
                       对上传(或下载)流程进行封装, 对外提供一个包含所有上传(或下载)细节的方法, 给调用者使用
                    c. 为"实现类"定义抽象的接口. 具体的实现类都依赖统一的接口定义, 遵从一致的上传功能协议.使用者依赖接口, 而不是
                       具体的实现类来编程
                       
                    例如:
                            public interface ImageStore {
                                String upload(Image image, String bucketName);
                                Image download(String url);
                            }
                            
                            / 上传下载流程改变：私有云不需要支持 access token
                            public class PrivateImageStore implements ImageStore {
                            
                                public String upload(Image image, String bucketName) {
                                    createBucketIfNotExisting(bucketName);
                                    //... 上传图片到私有云...
                                    //... 返回图片的 url...
                                }
                                
                                public Image download(String url) {
                                    //... 从私有云下载图片...
                                }
                                
                                private void createBucketIfNotExisting(String bucketName) {
                                    // ... 创建 bucket...
                                    // ... 失败会抛出异常..
                                }
                            }
                            
                            ......// 以后有其他模式的云图片上传, 下载
                            
                            public void process() {
                                Image image = ...;// 处理图片，并封装为 Image 对象
                                ImageStore imageStore = new PrivateImageStore(...);
                                imagestore.upload(image, BUCKET_NAME);
                            }
                            
            (3) 在做软件开发时, 要有抽象意识, 封装意识, 接口意识. 在定义接口时, 不要暴露任何实现细节. 接口的定义只表明做什么, 而不是
                怎么做. 在设计接口时, 要多思考一下这样的接口设计是否足够通用, 是否能够做到在替换具体的接口实现的时候, 不需要任何
                接口定义的改动.
                
            (4) 根据"基于接口而非实现编程" 的初衷, 如果在业务场景中, 某个功能只有一种实现方式, 未来也不可能被其他实现方式替换, 那就
                没有必要为其设计接口, 也没有必要基于接口编程, 直接使用实现类就可以
```

### 组合优于继承
```shell
    1. 为什么不推荐使用继承？
       组合相比继承有哪些优势？
       如何判断该用组合还是继承？
      
    2. 继承是表示 is-a 关系, 主要是减少代码重复开发, 但继承层次过深, 过复杂, 会影响到代码的可维护性, 所有继承与父类的子类, 都要具备
       父类的接口, 这导致存在各种中间子类(比如鸟是最原始父类, 鸟要分会不会飞, 会不会叫, .....), 最终子类要继承这种中间类.
       第一: 导致代码的可读性变差, 因为要搞清楚某个类具有哪些方法, 属性, 必须阅读父类的代码, 父类的父类的代码……一直追溯到最顶层父类的代码.
       第二: 破坏类的封装特性, 将父类的实现细节暴露给子类, 子类的实现依赖父类的实现, 两者高度耦合, 一旦父类代码修改, 就会影响所有
             子类的逻辑
             
    3. 组合
            (1) 可以利用组合(composition), 接口, 委托(delegation) 来解决继承的问题. 
                    通过接口可以实现每个不同的行为, 但接口本身不能进行实现方法, 可以防止每一个类都要实现一遍相同的代码, 可以用组合.
                        // 会飞接口
                        public interface Flyable {
                            void fly();
                        }
                        
                        public class FlyAbility implements Flyable {
                            @Override
                            public void fly() { //... }
                        }
                        
                        public class Ostrich {// 鸵鸟
                        private FlyAbility flyAbility = new FlyAbility(); // 组合
                            //... 省略其他属性和方法...
                            
                            @Override
                            public void fly() {
                            flyAbility.fly(); // 委托
                            }
                        }
            (2)  is-a 关系可以通过组合和接口的 has-a 关系来替代；多态特性可以利用接口来实现；
                 代码复用可以通过组合和委托来实现
                 
    4. 组合和继承的选择
            (1) 如果类之间的继承结构稳定(不会轻易改变), 继承层次比较浅(比如, 最多有两层继承关系), 继承关系不复杂,  使用继承.反之
                系统越不稳定, 继承层次很深, 继承关系复杂, 尽量使用组合来替代继承.
            (2) 一些设计模式会固定使用继承或者组合. 比如 装饰者模式(decorator pattern), 策略模式(strategy pattern), 
                组合模式(composite pattern)等都使用组合关系, 而模板模式(template pattern)使用继承关系
```

### 面向对象实战
```shell
    0. 概念
            (1) 领域驱动设计(Domain Driven Design, 简称 DDD), 用来指导如何解耦业务系统, 划分业务模块, 定义业务领域模型及其交互
                做好领域驱动设计的关键是对所做业务的熟悉程度.
                
    1. 基于贫血模型的 MVC 三层架构开发模式
            (1) 
                什么是贫血模型？什么是充血模型？
                为什么说基于贫血模型的传统开发模式违反 OOP?
                基于贫血模型的传统开发模式既然违反 OOP, 那又为什么如此流行？
                什么情况下应该考虑使用基于充血模型的 DDD 开发模式？
            (2) MVC 三层架构中的 M 表示 Model, V 表示 View, C 表示 Controller. 
                后端项目分为 Repository 层(负责数据访问), Service 层(负责业务逻辑), Controller 层(负责暴露接口)
                
                大多数业务开发:
                    
                    ///////// Service+BO(Business Object) //////////
                    public class UserService {
                        private UserRepository userRepository; // 通过构造函数或者 IOC 框架注入
                        
                        public UserBo getUserById(Long userId) {
                            UserEntity userEntity = userRepository.getUserById(userId);
                            UserBo userBo = [...convert userEntity to userBo...];
                            return userBo;
                        }
                    } 
                    
                    public class UserBo {// 省略其他属性、get/set/construct 方法
                        private Long id;
                        private String name;
                        private String cellphone;
                    }
                    
                    UserBo 和 UserService 成了业务逻辑层, UserBo 只包含数据, 不包含任何业务逻辑. 业务逻辑集中在 UserService 中, 
                    通过 UserService 来操作 UserBo.
                    
                    像 UserBo 只包含数据, 不包含业务逻辑的类, 就叫作贫血模型(Anemic Domain Model), 贫血模型将数据与操作分离, 破
                    坏面向对象的封装特性, 是一种典型的面向过程的编程风格
                    
            (3) 贫血模型的 MVC 三层架构受欢迎的原因
                    a. 系统业务简单, 贫血模型足以应付这种简单业务的开发, 这种情况下精心设计的充血模型没多大用处.
                    b. 充血模型的设计难度要比贫血模型大, 事先就要设计好针对数据要暴露哪些操作, 定义哪些业务逻辑, 不像贫血模型一开始
                       就可以开发, 定义好数据结构, 需要什么操作定义好就可以了.
                    c. 思维已固化, 转型有成本
                    
            (4) 局限
                    不适用于复杂系统的开发, 往往这类 web 开发是, service 代码动不了, 不断新增 SQL 语句, 业务逻辑包裹在一个大的 
                    SQL 语句中, 而 Service 层可以做的事情很少, SQL 都是针对特定的业务功能编写的, 复用性差, 会存在大量的差不多, 
                    区别很小的 SQL 语句
                    
    2. 基于充血模型的 DDD 开发
            (1) 充血模型(Rich Domain Model): 数据和对应的业务逻辑被封装到同一个类, 体现封装性, 是 OOP 风格.
                基于充血模型的 DDD 开发模式, 更适合业务复杂的系统开发, 比如包含各种利息计算模型, 还款模型等复杂业务的金融系统
                
            (2) 基于充血模型的 DDD 开发跟基于贫血模型的传统开发模式的区别主要在 Service 层. 基于贫血模型的传统的开发模式, 
                重 Service 轻 BO；基于充血模型的 DDD 开发模式, 轻 Service 重 Domain(既有数据, 又有逻辑业务)
                
            (3) 基于充血模型的 DDD 的开发模式, 需要前期做大量的业务调研, 领域模型设计
            
            (4) Domain 有数据和逻辑, 而 Service 也有数据和逻辑, Service 作用是
                    a. Service 类负责与 Repository 交流, 具体代码是 VirtualWalletService 类负责与 Repository 层交互, 
                       调用 Respository 类的方法, 获取数据库中的数据, 转化成领域模型 VirtualWallet, 然后由领域模型 VirtualWallet
                       来完成业务逻辑, 最后调用 Repository 类的方法, 将数据存回数据库. 之所以让 VirtualWalletService 类与
                       Repository 打交道, 而不是让领域模型 VirtualWallet 与 Repository 打交道, 那是因为想保持领域模型的独立性, 
                       不与任何其他层的代码(Repository 层的代码)或开发框架(比如 Spring,  MyBatis)耦合在一起, 将流程性的
                       代码逻辑(比如从 DB 中取数据, 映射数据)与领域模型的业务逻辑解耦, 让领域模型更加可复用.
                       
                    b. Service 类负责跨领域模型的业务聚合功能, 有些业务需要跨其他层业务的, 不是领域模型专属的, 则无法将这部分业务
                       放在领域模型中, 那么暂时放在 Service 层中.
                       
                    c. Service 类负责一些非功能性及与三方系统交互的工作. 比如幂等、事务、发邮件、发消息、记录日志、调用其他系统的
                       RPC 接口等, 都可以放到 Service 类中
                       
            (4) Controller 层主要负责接口的暴露, Repository 层主要负责与数据库打交道, 这两层包含的业务逻辑并不多, 可以使用贫血模型.
                
    3. 基于充血模型的 DDD 开发模式跟基于贫血模型的传统开发模式相比, 主要区别在 Service 层. 在基于充血模型的开发模式下, 将部分原来在
       Service 类中的业务逻辑移动到了一个充血的 Domain 领域模型中, 让 Service 类的实现依赖这个 Domain 类
                             
                
```

### 面向对象项目流程(接口调用鉴权功能流程)
```shell
    1. 概念
        (1) 根据项目一般经历从基础的需求分析, 职责划分, 类的定义, 交互, 组装运行.
    2. 面向对象分析
            (1) 面向对象分析就是需求分析, 将笼统的需求细化到足够清晰, 可执行, 产出是详细的需求描述
            (2) 先从最简单的方案想起, 然后再优化, 以下进行 4 轮
                    第一轮:
                            通过用户名加密码来做认证, 给每个允许访问服务的调用方,分配用户名和密码, 调用方每次进行接口请求时, 都携带
                            自己的用户名和密码. 微服务在接收到接口调用请求, 进行校验.
                            
                    第二轮:
                            用户名和密码是明文的, 有风险, 需要加密, 通过 OAuth 的验证思路, 调用方将请求接口的 URL 跟 AppID, 密码
                            拼接在一起(http://www.xxx.com?id=123&appid=345&pwd=123) 然后进行加密, 生成一个 token. 
                            调用方在进行接口请求的时, 将这个 token 及 AppID, 随 URL 一块传递给微服务端
                            (http://www.xxx.com?id=123&appid=345&token=456). 微服务端接收到这些数据后, 根据 AppID 从数据库
                            中取出对应的密码, 并通过同样的 token 生成算法, 生成另外一个 token. 用这个新生成的 token 跟调用方传递
                            过来的 token 对比, 如果一致, 则允许接口调用请求；否则就拒绝接口调用请求. 
                            
                    第三轮: 
                           对于固定的 url, AppID, token 还是会存在重放攻击, 解决方案引入一个随机变量(时间戳), 让每次接口请求生成的
                           token 都不一样, 将 URL, AppID, 密码, 时间戳四者
                           (http://www.xxx.com?id=123&appid=345&pwd=123&ts=xxxxxxx) 进行加密来生成 token, 
                           微服务端在收到这些数据后(http://www.xxx.com?id=123&appid=345&token=xxxxxx&ts=xxxxxxx) , 
                           会验证当前时间戳跟传递过来的时间戳, 是否在一定的时间窗口内(比如一分钟). 如果超过一分钟, 则判定 token 过期, 
                           拒绝接口请求. 如果没有超过一分钟, 则说明 token 没有过期, 就再通过同样的 token 生成算法进行 token 校验.
                           
                    第四轮:
                            针对 AppID 和密码的存储, 能灵活地支持各种不同的存储方式, 比如 ZooKeeper, 本地配置文件, 自研配置中心, 
                            MySQL, Redis 等, 不一定针对每种存储方式都去做代码实现, 但起码要留有扩展点, 保证系统有足够的灵活性和
                            扩展性, 能够在切换存储方式时, 尽可能地减少代码的改动
                            
            (3) 需求分析的过程实际上是一个不断迭代优化的过程, 不要试图一下就能给出一个完美的解决方案, 而是先给出一个粗糙的, 基础的方案,
                有一个迭代的基础, 然后再慢慢优化
                
    3. 面向对象设计
            (1) 面向对象设计的产出就是类, 细化为以下部分
                    a. 划分职责进而识别出有哪些类；
                    b. 定义类及其属性和方法；
                    c. 定义类与类之间的交互关系；
                    d. 将类组装起来并提供执行入口
                    
            (2) 构造类, 根据需求描述, 把其中涉及的功能点, 一个一个罗列出来, 然后再去看哪些功能点职责相近, 操作同样的属性, 可否
                归为同一个类, 例如:
                    1. 把 URL, AppID, 密码, 时间戳拼接为一个字符串；
                    2. 对字符串通过加密算法加密生成 token；
                    3. 将 token, AppID, 时间戳拼接到 URL 中, 形成新的 URL；
                    4. 解析 URL, 得到 token, AppID, 时间戳等信息；
                    5. 从存储中取出 AppID 和对应的密码；
                    6. 根据时间戳判断 token 是否过期失效；
                    7. 验证两个 token 是否匹配
                    
                    1, 2, 6, 7 都是跟 token 有关, AuthToken 类负责 token 的生成, 验证；
                    3, 4 都是在处理 URL, Url 类负责 URL 的拼接, 解析
                    5 CredentialStorage 类是操作 AppID 和密码
                    
                针对复杂的需求开发, 首先要进行模块划分, 将需求先简单划分成几个小的, 独立的功能模块, 然后在模块内部, 应用功能点汇合方法,
                进行面向对象设计, 而模块的划分和识别, 跟类的划分和识别类似
                
            (3) 定义类及其属性和方法
                    定义原则有
                        第一, 从业务模型上来说, 不属于这个类的属性和方法, 就不应该被放到这个类里,  比如 URL, AppID 这些信息, 
                             从业务模型上不应该属于 AuthToken, 则可以通过参数方式传入.
                             
                        第二, 在设计类具有哪些属性和方法时, 不能单纯地依赖当下的需求, 还要分析这个类从业务模型上来讲, 理应具有哪些属性
                              和方法, 一方面保证类定义的完整性, 另一方面不仅为当下的需求还为未来的需求做些准备.
                              
            (4) 定义类与类之间的交互关系
                    类与类之间的交互关系有 泛化, 实现, 关联, 聚合, 组合, 依赖
                    
                    泛化(Generalization): 继承关系
                        public class A { ... }
                        public class B extends A { ... }
                        
                    实现(Realization): 接口和实现类之间的关系
                        public interface A {...}
                        public class B implements A { ... }
                        
                    聚合(Aggregation): 是一种包含关系, A 类对象包含 B 类对象, B 类对象的生命周期可以不依赖 A 类对象的生命周期, 即
                                       可以单独销毁 A 类对象而不影响 B 对象, 比如课程与学生之间的关系, 
                                       在 A 对象中传入参(已存在 B 对象)
                       public class A {
                           private B b;
                           
                           public A(B b) {
                            this.b = b;
                           }
                       }
                       
                    组合(Composition): 也是一种包含关系, A 类对象包含 B 类对象, B 类对象的生命周期依赖于 A 类对象的生命周期, 
                                       B 类对象不可单独存在, 比如鸟与翅膀之间的关系, 在 A 对象中 new B 对象
                                       
                       public class A {
                           private B b;
                           
                           public A() {
                            this.b = new B();
                           }
                       }
                       
                    关联(Association) : 是一种非常弱的关系, 包含聚合, 组合两种关系, 如果 B 类对象是 A 类的成员变量, 
                                       那 B 类和 A 类就是关联关系
                                       
                        public class A {
                            private B b;
                            
                            public A(B b) {
                                this.b = b;
                            }
                        }
                        
                        或者
                        
                        public class A {
                            private B b;
                            
                            public A() {
                                this.b = new B();
                            }
                        }
                        
                    依赖(Dependency): 是一种比关联关系更加弱的关系, 包含关联关系. 不管是 B 类对象是 A 类对象的成员变量, 还是 
                                     A 类的方法使用 B 类对象作为参数或者返回值, 局部变量, 只要 B 类对象和 A 类对象有任何使用关系, 
                                     都称它们有依赖关系
                                     
                    注意:
                            多用组合少用继承, 这里原则上的组合就是 UML 中的关联(只要是成员变量就行, 不管是什么形式产生的, 传参也好
                                                                          自己类内 new 也好)
                                                                          
            
            (5) 将类组装起来并提供执行入口
                     // 上层接口
                    public interface ApiAuthencator {
                     void auth(String url);
                     void auth(ApiRequest apiRequest);
                    } 
                    
                    // 实现接口类
                    public class DefaultApiAuthencatorImpl implements ApiAuthencator {
                     
                     // 组合
                     private CredentialStorage credentialStorage;
                     
                     public ApiAuthencator() {
                     this.credentialStorage = new MysqlCredentialStorage();
                     } 
                     
                     public ApiAuthencator(CredentialStorage credentialStorage) {
                     this.credentialStorage = credentialStorage;
                     } 
                     
                     @Override
                     public void auth(String url) {
                     ApiRequest apiRequest = ApiRequest.buildFromUrl(url);
                     auth(apiRequest);
                     } 
                     
                     @Override
                     public void auth(ApiRequest apiRequest) {
                     String appId = apiRequest.getAppId();
                     String token = apiRequest.getToken();
                     long timestamp = apiRequest.getTimestamp();
                     String originalUrl = apiRequest.getOriginalUrl();
                     AuthToken clientAuthToken = new AuthToken(token, timestamp);
                     
                     if (clientAuthToken.isExpired()) {
                     throw new RuntimeException("Token is expired.");
                     } 
                     
                     String password = credentialStorage.getPasswordByAppId(appId);
                     
                     AuthToken serverAuthToken = AuthToken.generate(originalUrl, appId, password);
                     if (!serverAuthToken.match(clientAuthToken)) {
                     throw new RuntimeException("Token verfication failed.");
                     }
                     }
                    }
            
```

## 设计原则

### 概念
```shell
    1. 设计原则需要了解原则的定义, 这些原则设计的初衷, 能解决哪些问题, 有哪些应用场景等
    2.  SOLID 原则分别由 5 个设计原则组成, 单一职责原则, 开闭原则, 里式替换原则, 接口隔离原则和依赖反转原则, 
        依次对应 SOLID 中的 S, O, L, I, D 这 5 个英文字母
```

### SRP 单一职责原则
```shell
    1. 单一职责原则(Single Responsibility Principle, SRP), 一个类或者模块只负责完成一个职责(或者功能), 即 不要设计大而全的类, 
       要设计粒度小, 功能单一的类, 例如, 一个类包含了两个或者两个以上业务不相干的功能, 则职责不够单一, 应该将它拆分成多个功能更加单一,
       粒度更细的类
       
       目的: 单一职责原则是为了实现代码高内聚、低耦合, 提高代码的复用性、可读性、可维护性
       
    2. 单一职责原则也要考虑到实际业务场景, 例如
            public class UserInfo {
                private long userId;
                private String username;
                private String email;
                private String telephone;
                private long createTime;
                private long lastLoginTime;
                private String avatarUrl;
                
                private String provinceOfAddress; // 省
                private String cityOfAddress; // 市
                private String regionOfAddress; // 区
                private String detailedAddress; // 详细地址
                
                // ... 省略其他属性和方法...
            }
            
            如果用户的地址信息跟其他信息一样, 只是单纯地用来展示,. 那 UserInfo 现在的设计是合理的. 如果该产品业务添加了电商的模
            块, 用户的地址信息还会用在电商物流中, 那最好将地址信息从 UserInfo 中拆分出来, 独立成用户物流信息(或者叫地址信息, 收货信息等)
            
    3. 不同的应用场景, 不同阶段的需求, 对同一个类的职责是否单一的判定, 可能都是不一样. 在某种应用场景或者当下的需求背景下, 一个
       类的设计可能已经满足单一职责原则, 但如果换个应用场景或着在未来的某个需求背景下, 可能就不满足了, 需要继续拆分成粒度更细的类
       
    4. 对于单一职责原则, 可以先写一个粗粒度的类, 满足业务需求. 随着业务的发展, 如果粗粒度的类越来越庞大, 代码越来越多, 这个时候就可以
       将这个粗粒度的类, 拆分成几个更细粒度的类, 这就是持续重构
       
    5. 单一职责指导方法
            (1) 类中的代码行数, 函数或属性过多, 会影响代码的可读性和可维护性, 就需要考虑对类进行拆分；
            (2) 类依赖的其他类过多, 或者依赖类的其他类过多, 不符合高内聚, 低耦合的设计思想,  就需要考虑对类进行拆分；
            (3) 私有方法过多, 需要考虑能否将私有方法独立到新的类中, 设置为 public 方法, 供更多的类使用, 从而提高代码的复用性；
            (4) 比较难给类起一个合适名字, 很难用一个业务名词概括, 或者只能用一些笼统的 Manager、Context 之类的词语来命名, 这就说明
                类的职责定义得可能不够清晰；
            (5) 类中大量的方法都是集中操作类中的某几个属性, 比如, 在 UserInfo 例子中, 如果一半的方法都是在操作 address 信息, 那就
                可以考虑将这几个属性和对应的方法拆分出来
                
    6. 不管是应用设计原则还是设计模式, 最终的目的还是提高代码的可读性, 可扩展性、复用性、可维护性等, 例如序列化和反序列化为了提供可维护性
       可以在同一个类内.
```

### OCP 开闭原则
```shell
    1. 概念
        (1) 怎样的代码改动才被定义为‘扩展’？
            怎样的代码改动才被定义为‘修改’？
            怎么才算满足或违反‘开闭原则’？
            修改代码就一定意味着违反‘开闭原则’吗？
            
            定义, 原因, 做法, 意义
            
        (2) 这条原则最有用, 扩展性是代码质量最重要的衡量标准之一, 在 23 种经典设计模式中, 大部分设计模式都是为了解决代码的扩展性问题
            而存在的, 主要遵从的设计原则就是开闭原则
            
        (3) 开闭原则(Open Closed Principle, OCP), 对扩展开放, 对修改关闭. 添加一个新的功能应该在已有代码基础上扩展代码(新增模块, 类,
            方法等), 而非修改已有代码(修改模块, 类, 方法等), 即只要它没有破坏原有的代码的正常运行, 没有破坏原有的单元测试.
            
        (4) 最常用来提高代码扩展性的方法有：多态, 依赖注入, 基于接口而非实现编程, 以及大部分的设计模式(装饰, 策略, 模板, 职责链, 状态等)
            
    2. 在函数内需要新增一个指标判断, 通过修改函数入参的形式会出现很多问题, 第一,调用方需要修改调用这个函数的参数. 第二, 单元测试也需要更改.
       那么重构可以根据开闭原则, 不进行修改, 新增的形式. 大致原理是, 每一个指标对应的判断是不一样的, 可以通过定义一个抽象类, 里面是
       判断指标的虚函数, 同时虚函数传参不是某个特定的指标变量, 而是所有指标汇总的一个结构体 strct info, 那么就可以定义不同的指标子类,
       继承抽象类, 再重写虚函数方法.
       
    3. 为了尽量写出扩展性好的代码, 要时刻具备扩展意识, 抽象意识, 封装意识, 即识别出代码可变部分和不可变部分之后, 将可变部分封装起来, 
       隔离变化, 提供抽象化的不可变接口, 给上层系统使用, 当具体的实现发生变化时, 只需要基于相同的抽象接口, 扩展一个新的实现, 替换掉老的
       实现即可, 上游系统的代码几乎不需要修改
            实例:
                // 这一部分体现了抽象意识
                 public interface MessageQueue { ... }
                 public class KafkaMessageQueue implements MessageQueue { ... }
                 public class RocketMQMessageQueue implements MessageQueue {...}
                 
                 public interface MessageFromatter { ... }
                 public class JsonMessageFromatter implements MessageFromatter {...}
                 public class ProtoBufMessageFromatter implements MessageFromatter {...}
                
                 public class Demo {
                 private MessageQueue msgQueue; // 基于接口而非实现编程
                 public Demo(MessageQueue msgQueue) 
                 {  // 依赖注入
                 this.msgQueue = msgQueue;
                 } 
                 
                 // msgFormatter：多态、依赖注入
                 public void sendNotification(Notification notification, MessageFormatter msg)
                 //...
                 }
                }
                
    4. 如果是一个业务导向的系统, 比如金融系统, 电商系统, 物流系统等, 要识别出尽可能多的扩展点, 就要对业务有足够的了解, 能够知道当下以及
       未来可能要支持的业务需求, 但是没有必要做过度的设计.
       
       如果开发的是跟业务无关的, 通用的, 偏底层的系统, 如框架, 组件, 类库, 需要了解“它们会被如何使用？今后打算添加哪些功能？
       使用者未来会有哪些更多的功能需求？”等问题
       
       
       最合理的做法是, 对于一些比较确定的, 短期内可能就会扩展, 或者需求改动对代码结构影响比较大的情况, 或者实现成本不高的扩展点, 
                      就可以事先做些扩展性设计. 但对于一些不确定未来是否要支持的需求, 或者实现起来比较复杂的扩展点, 可以等到有需求驱动时
                      再通过重构代码的方式来支持扩展的需求
                      
       需要在扩展性和可读性之间做权衡
      
```

### LSP 里式替换原则
```shell
    1. 概念
            (1) 里式替换原则(Liskov Substitution Principle, LSP), 子类对象(object of subtype/derived class) 能够替换程序
                (program) 中父类对象(object of base/parent class)出现的任何地方, 并且保证原来程序的逻辑行为(behavior)不变及
                正确性不被破坏.
                
    2. 和多态的区别
            多态是面向对象编程的一大特性, 也是面向对象编程语言的一种语法. 它是一种代码实现的思路, 而里式替换是一种设计原则, 
            是用来指导继承关系中子类该如何设计的, 子类的设计要保证在替换父类的时候, 不改变原有程序的逻辑以及不破坏原有程序的正确性.
            
    3.里式替换原则实际上是设计子类时要遵守父类的行为约定(或者叫协议), 父类定义函数的行为约定, 那子类可以改变函数的 内部实现逻辑, 
      但不能改变函数原有的行为约定. 这里的行为约定包括：函数声明要实现的功能；对输入、输出、异常的约定；甚至包括注释中所罗列的任何特殊说明
      违反里式替换原则实例如下:
            (1)  子类违背父类声明要实现的功能
                        父类定义的 sortOrdersByAmount() 订单排序函数, 是按金额从小到大来给订单排序. 而子类重写这个
                 sortOrdersByAmount() 订单排序函数是按照创建日期来给订单排序的.
                 
            (2) 子类违背父类对输入, 输出, 异常的约定
                    父类约定某个函数, 运行出错的时候返回 null, 获取数据为空的时候返回空集合(empty collection), 而子类重载函数后, 
                    实现变了, 运行出错返回异常(exception), 获取不到数据返回 null
                    
                    在父类某个函数约定, 输入数据可以是任意整数, 但子类实现时, 只允许输入数据是正整数, 负数就抛出, 子类对输入的数据的
                    校验比父类更加严格
                    
                    在父类中某个函数约定, 只会抛出 ArgumentNullException 异常, 那子类的设计实现中只允许抛出 
                    ArgumentNullException 异常, 任何其他异常的抛出，都会导致子类违背里式替换原则
                    
            (3) 子类违背父类注释中所罗列的任何特殊说明
                    父类中定义的 withdraw() 提现函数的注释是这么写的：“用户的提现金额不得超过账户余额……”, 而子类重写 withdraw()
                     函数之后, 针对 VIP 账号实现了透支提现的功能, 也就是提现金额可以大于账户余额, 那这个子类的设计也是不符合
                     里式替换原则的
```

### ISP 接口隔离原则
```shell
    1. 概念
            (1) 接口隔离原则(Interface Segregation Principle, ISP), 
            (2) 接口可以理解为三种情况,  第一, 一组 API 接口集合. 第二, 单个 API 接口或函数. 第三, OOP 中的接口概念
            
    2. 一组 API 接口集合
            (1) 接口原则: 调用者不应该强迫依赖它不需要的接口, 调用者不会使用的接口不应该继承,依赖.
    
            public interface UserService {
             boolean register(String cellphone, String password);
             boolean login(String cellphone, String password);
             UserInfo getUserInfoById(long id);
             UserInfo getUserInfoByCellphone(String cellphone);
            }
            
            public interface RestrictedUserService {
             boolean deleteUserByCellphone(String cellphone);
             boolean deleteUserById(long id);
            }
            
            public class UserServiceImpl implements UserService, RestrictedUserService {
            // ... 省略实现代码...
            }
            
            (2) 对于删除用户功能的受限接口, 只提供给后天管理系统, 不能随意公开, 所以最好不要在 UserService 中提供对外接口, 可以将删除接口
                单独放到另外一个接口 RestrictedUserService 中, 然后将 RestrictedUserService 只打包(implements)提供给后台管理系统
                来使用.
            
            (3) 如果部分接口只被部分调用者使用, 那就需要将这部分接口隔离出来, 单独给对应的调用者使用, 而不是强迫其他调用者也依赖这部分
                不会被用到的接口.
            
    3. 单个 API 接口或函数
            (1) 接口隔离原则: 函数的设计要功能单一, 不要将多个不同的功能逻辑在一个函数中实现.
            (2) 接口隔离原则与单一职责原则的区别
                    单一职责原则针对的是模块, 类, 接口的设计, 而接口隔离原则相对于单一职责原则, 第一它更侧重于接口的设计. 第二思考的
                角度不同, 它提供接口是否职责单一的标准：通过调用者如何使用接口来间接地判定, 如果调用者只使用部分接口或接口的部分功能, 
                那接口的设计就不够职责单一.
                
    4. 把“接口”理解为 OOP 中的接口概念
            (1) 理解为 OOP 中的接口, 比如 Java 中的 interface, 通过类似抽象接口的概念, 使用接口隔离原则, 每一个接口都实现单一职责, 
                再新增一个功能, 也只需新增一个实现接口, 设计思路更加灵活、易扩展、易复用.
                
                        public class ApiMetrics implements Viewer {...}
                        public class DbMetrics implements Viewer {...}
                        
                        public class Application {
                         ConfigSource configSource = new ZookeeperConfigSource();
                         
                         public static final RedisConfig redisConfig = new RedisConfig(configSource)
                         public static final KafkaConfig kafkaConfig = new KakfaConfig(configSource)
                         public static final MySqlConfig mySqlConfig = new MySqlConfig(configSource)
                         public static final ApiMetrics apiMetrics = new ApiMetrics();
                         public static final DbMetrics dbMetrics = new DbMetrics();
                         
                         public static void main(String[] args) {
                         SimpleHttpServer simpleHttpServer = new SimpleHttpServer(“127.0.0.1”, 2);
                         simpleHttpServer.addViewer("/config", redisConfig);
                         simpleHttpServer.addViewer("/config", mySqlConfig);
                         simpleHttpServer.addViewer("/metrics", apiMetrics);
                         simpleHttpServer.addViewer("/metrics", dbMetrics);
                         simpleHttpServer.run();
                         }
                        }
                      
```
### DIP 依赖反转原则
```shell
    1. 概念
            (1) "依赖反转"这个概念指的是“谁跟谁” 的 “什么依赖”被反转了？
                "反转”两个字该如何理解？
                "控制反转"和"依赖注入" 跟“依赖反转”有什么区别和联系呢？它们说的是同一个事情吗？
                
    2. 控制反转(IOC)
            (1) 控制反转(Inversion Of Control, IOC),  “控制” 是对程序执行流程的控制, "反转"是在没有使用框架之前, 程序员自己控制
                整个程序的执行, 在使用框架之后, 整个程序的执行流程可以通过框架来控制, 流程的控制权从程序员“反转”到了框架.
                
                控制反转并不是一种具体的实现技巧, 而是一个比较笼统的设计思想, 用来指导框架层面的设计.
            (2) 通过框架来实现 "控制反转", 框架提供了一个可扩展的代码骨架, 用来组装对象, 管理整个执行流程. 利用框架进行开发时, 只需
                要往预留的扩展点上, 添加跟自己业务相关的代码, 就可以利用框架来驱动整个程序流程的执行.
                
                例如:
                        public abstract class TestCase {
                         public void run()
                         {
                         if (doTest()) 
                         {
                         System.out.println("Test succeed.");
                         } 
                         else
                         {
                         System.out.println("Test failed.");
                         }
                         } 
                        
                         public abstract boolean doTest();
                        }
                        
                        public class JunitApplication {
                         private static final List<TestCase> testCases = new ArrayList<>();
                         
                         public static void register(TestCase testCase) {
                         testCases.add(testCase);
                         } 
                         
                         public static final void main(String[] args) {
                         for (TestCase case: testCases) {
                         case.run();
                         }
                         }
                        }
                        
                        public class UserServiceTest extends TestCase {
                         @Override
                         public boolean doTest() {
                         // ...
                         }
                        } 
                        
                        // 注册操作还可以通过配置的方式来实现, 不需要程序员显示调用 register()
                        JunitApplication.register(new UserServiceTest();
                        
    3. 依赖注入(DI)
            (1) 依赖注入(Dependency Injection, DI), 是一种具体的编码技巧, 是不通过 new() 的方式在类内部创建依赖类对象, 而是将依赖
                的类对象在外部创建好之后, 通过构造函数、函数参数等方式传递(或注入)给类使用.
                
                    // 为了更好的扩展性(例如不同的发送方式), MessageSender 可以定义为接口
                    public class MessageSender {
                     public void send(String cellphone, String message) {
                     //....
                     }
                    }
                    
                    // 依赖注入的实现方式
                    public class Notification {
                     private MessageSender messageSender;
                     
                     // 通过构造函数将 messageSender 传递进来
                     public Notification(MessageSender messageSender) {
                     this.messageSender = messageSender;
                     } 
                     
                     public void sendMessage(String cellphone, String message) {
                     //... 省略校验逻辑等...
                     this.messageSender.send(cellphone, message);
                     }
                    }
                    
                    // 使用 Notification
                    MessageSender messageSender = new MessageSender();
                    Notification notification = new Notification(messageSender);
                
            (2) 通过依赖注入的方式来将依赖的类对象传递进来, 提高代码的扩展性, 可以灵活地替换依赖的类.
            
    4. 依赖注入框架(DI Framework)
            (1) 背景
                    在采用依赖注入实现的 Notification 类中, 虽然不需要用类似 hard code 的方式(在类内部通过 new 来创建 
                MessageSender 对象), 但是这个创建对象, 组装(或注入)对象的工作仅仅是被移动到了更上层代码, 还是需要程序员自己来编写.
                
                    public class Demo {
                     public static final void main(String args[]) {
                     MessageSender sender = new SmsSender(); // 创建对象
                     Notification notification = new Notification(sender);// 依赖注入
                     notification.sendMessage("13918942177", " 短信验证码：2346");
                     }
                    }
                
            (2) 对象创建和依赖注入的工作, 本身跟具体的业务无关, 可以抽象成框架来自动完成, 这个框架就是“依赖注入框架”, 只需要通过依赖
                注入框架提供的扩展点, 简单配置一下所有需要创建的类对象, 类与类之间的依赖关系, 就可以实现由框架来自动创建对象, 
                管理对象的生命周期, 依赖注入等原本需要程序员来做的事情, 该框架有 Google Guice, Butterfly Container 等.
                
    5. 依赖反转原则(DIP)
            (1) 依赖反转原则(Dependency Inversion Principle, DIP), 高层模块(high-level modules) 不要依赖低层模块(low-level),
                高层模块和低层模块应该通过抽象(abstractions)来互相依赖.  抽象(abstractions)不要依赖具体实现细节(details), 
                具体实现细节(details)依赖抽象(abstractions), 这条原则主要还是用来指导框架层面的设计
```

### KISS 原则和 YAGNI 原则
```shell
    1. 概念
        (1) 怎么理解 KISS 原则中“简单”两个字？
            什么样的代码才算“简单”？
            怎样的代码才算“复杂”？
            如何才能写出“简单”的代码？
            YAGNI 原则跟 KISS 原则说的是一回事吗？
            
        (2) KISS 原则(Keep It Simple and Stupid), 尽量保持简单, KISS 原则就是保持代码可读和可维护的重要手段, 代码足够简单, 意味着
            容易读懂, 出现 bug 修复起来也比较简单.
            
    2. KISS 指导原则
            (1) 不要重复造轮子, 要善于使用已经有的工具类库. 自己去实现这些类, 出 bug 的概率会更高, 维护的成本也比较高
            (2) 不要过度优化. 不要过度使用一些奇技淫巧(如, 位运算代替算术运算, 复杂的条件语句代替 if-else, 使用一些过于底层的函数等)
                来优化代码, 牺牲代码的可读性
                
    3. YAGNI 原则
            (1) YAGNI 原则(You Ain’t Gonna Need It, YAGNI), 你不会需要它, 不要去设计当前用不到的功能；不要去编写当前用不到的代码,
                核心思想就是不要做过度设计
                
            (2) 如, 系统暂时只用 Redis 存储配置信息, 以后可能会用到 ZooKeeper. 根据 YAGNI 原则, 在未用到 ZooKeeper 之前, 
                没必要提前编写这部分代码,  这并不是说不需要考虑代码的扩展性, 还是要预留好扩展点, 等到需要的时候, 再去实现 ZooKeeper
                存储配置信息这部分代码.
```

### DRY 原则（Don’t Repeat Yourself）
```shell
    1. 概念
            (1) DRY 原则就是不要写重复的代码.
            (2) DRY 原则与代码的复用性也有一些联系
            
    2. 代码重复的情况
            (1) 实现逻辑重复
                    例如,  isValidUserName() 和 isValidPassword() 两个函数在现在的代码是一样的, 即代码的实现逻辑相同, 但是
                语义(功能)不同, 可能后续校验密码方式会调整, 这符合 DRY 原则, 对于包含重复代码的问题, 可以通过抽象成更细粒度函数的方式
                来解决. 比如将校验只包含 a~z、0~9、dot 的逻辑封装成 boolean onlyContains(String str, String charlist) 函数.
                
            (2) 功能语义重复
                    例如, isValidIp() 和 checkIfIpValid(), 虽然它们的代码逻辑不一样,实现的方式不一样(一个是用正则表达式, 一个是用
                 库函数), 但功能语义重复, 则不符合 DRY 原则, 这样后续的维护也不方便, 如果以后 255.255.255.255 也是非法的, 
                 得两处都要改.
                 
            (3) 代码执行重复
                    在一个函数内, 相同的校验函数被重复执行.
                    
    3. 代码复用性
            (1) 代码码复用: 表示一种行为,即在开发新功能时, 尽量复用已经存在的代码
                代码的可复用性: 表示一段代码可被复用的特性或能力, 在编写代码时, 让代码尽量可复用
                DRY 原则是一条原则, 不要写重复的代码
                
    4. 提高代码复用性
            (1) 减少代码耦合
                    高度耦合的代码会影响到代码的复用性, 因为高度耦合的代码会涉及到很多功能, 无法单独抽离出一个模块, 尽量减少代码耦合
            (2) 满足单一职责原则
                    功能单一, 越细粒度的代码, 代码的通用性会越好, 越容易被复用
            (3) 模块化
                    善于将功能独立的代码, 封装成模块.
            (4) 业务与非业务逻辑分离
                    越是跟业务无关的代码越是容易复用, 可以考虑抽取成一些通用的框架, 类库, 组件等
            (5) 通用代码下沉
                    越底层的代码越通用, 会被越多的模块调用, 越应该设计得足够可复用. 只允许上层代码调用下层代码及同层代码之间的调用, 
                杜绝下层代码调用上层代码.
            (6) 继承, 多态, 抽象, 封装
            (7) 应用模板等设计模式
                    模板模式利用多态来实现, 可以灵活地替换其中的部分代码, 整个流程模板代码可复用
                    
    5. 第一次编写代时, 不考虑复用性；第二次遇到复用场景时, 再进行重构使其复用
```

###  LOD 迪米特法则
```shell
    1. 概念
            (1) 迪米特法则是用于高内聚, 松耦合
            (2) 什么是“高内聚, 松耦合”？
                如何利用迪米特法则来实现“高内聚、松耦合”？
                有哪些代码设计是明显违背迪米特法则的？对此又该如何重构？
            (3) 迪米特法则(Law of Demeter, LOD), 也叫最小知识原则, 不该有直接依赖关系的类之间, 不要有依赖；有依赖关系的类之间, 
                尽量只依赖必要的接口.
                
    2. 高内聚,松耦合
            (1) 很多设计原则都以实现代码的“高内聚、松耦合”为目的, 比如单一职责原则, 基于接口而非实现编程等
            (2) “高内聚”用来指导类本身的设计, “松耦合”用来指导类与类之间依赖关系的设计
            (3) 高内聚, 就是指相近的功能应该放到同一个类中, 不相近的功能不要放到同一个类中, 相近的功能往往会被同时修改, 放到同一个类中,
                修改会比较集中, 代码容易维护. 单一职责原则是实现代码高内聚非常有效的设计原则.
            (4) 松耦合, 是指类与类之间的依赖关系简单清晰, 即使两个类有依赖关系, 一个类的代码改动不会或者很少导致依赖类的代码改动.
                依赖注入, 接口隔离, 基于接口而非实现编程, 以及迪米特法则都是为了实现代码的松耦合.
                
    3. 不该有直接依赖关系的类之间, 不要有依赖
            (1)　最底层 NetworkTransporter 通信库类不应该直接依赖太具体的发送对象(HtmlRequest), 可以修改为更通用的做法.
            
                    public class NetworkTransporter {
                        // 省略属性和其他方法...
                        public Byte[] send(HtmlRequest htmlRequest) 
                        {
                            //...
                        }
                    }
                    
                    修改为
                    
                    public class NetworkTransporter {
                        // 省略属性和其他方法...
                        public Byte[] send(String address, Byte[] data) {
                            //...
                        }
                    }
                    
            (2) 最顶层的类设计
            
                    // 有问题的代码
                    public class Document {
                     private Html html;
                     private String url;
                     
                     public Document(String url) {
                     this.url = url;
                     HtmlDownloader downloader = new HtmlDownloader();
                     this.html = downloader.downloadHtml(url);
                     } 
                    }
                    
                    第一: 构造函数内有 downloader.downloadHtml() 业务逻辑很重的代码, 耗时长, 会影响代码的可测试性.
                    第二: HtmlDownloader 对象是在 Document 类的构造函数中通过 new 来创建, 违反基于接口而非实现编程的设计思想
                    　　　(后续有其他的不是 html 类, 改动范围大), 也会影响到代码的可测试性
                    第三: 从业务上来讲, Document 网页文档没必要依赖 HtmlDownloader 类(直接传入 html 参数就行), 违背了迪米特法则
                    
                    修改为
                    
                        public class Document {
                         private Html html;
                         private String url;
                         public Document(String url, Html html) {
                         this.html = html;
                         this.url = url;
                         } 
                        } 
                        
                        // 通过一个工厂方法来创建 Document
                        public class DocumentFactory {
                         private HtmlDownloader downloader;
                         
                         public DocumentFactory(HtmlDownloader downloader) {
                         this.downloader = downloader;
                         } 
                         
                         public Document createDocument(String url) {
                         Html html = downloader.downloadHtml(url);
                         return new Document(url, html);
                         }
                        }
                        
    4. 有依赖关系的类之间, 尽量只依赖必要的接口
                           
```

### 业务系统设计(根据设计原则开发积分兑换系统)
```shell
    1. 概念
            (1) 需要独立负责一个系统的能力, 负责一个系统的工作, 包括前期的需求沟通分析, 中期的代码设计实现, 后期的系统上线维护等
    2. 需求分析
            (1) 积分系统两大功能点: 一个是赚取积分, 另一个是消费积分.
                a. 赚取积分
                        赚取积分功能包括积分赚取渠道, 比如下订单, 每日签到, 评论等；还包括积分兑换规则, 比如订单金额与积分的兑换比例,
                   每日签到赠送多少积分等.
                   
                        对于积分的有效期, 可以根据不同渠道, 设置不同的有效期, 积分到期之后会作废；在消费积分的时候, 优先使用快到期
                   的积分.
                   
                b. 消费积分
                        消费积分功能包括积分消费渠道, 比如抵扣订单金额, 兑换优惠券, 积分换购, 参与活动扣积分等；还包括积分兑换规
                        则, 比如多少积分可以换算成抵扣订单的多少金额, 一张优惠券需要多少积分来兑换等等
                        
                c. 积分及其明细查询
                        查询用户的总积分, 以及赚取积分和消费积分的历史记录
                        
    3. 系统设计
            (1)  合理地将功能划分到不同模块
                    合理地划分模块可以做到模块层面的高内聚, 低耦合, 架构整洁清晰.
                    
                    为了避免业务知识的耦合, 让下层系统更加通用, 不希望下层系统(被调用的系统)包含太多上层系统(调用系统)的业务信息,如
                    积分系统中最好不要包含太多跟订单, 优惠券, 换购等相关的信息. 上层系统可以包含下层系统的业务信息, 如订单系统, 
                    优惠券系统, 换购商城等作为调用"积分系统"的上层系统, 可以包含一些积分相关的业务信息, 积分系统所负责的工作只包含
                    积分的增、减、查询, 以及积分明细的记录和查询.
                    
            (2) 设计模块与模块之间的交互关系
                    确定有哪些系统跟积分系统之间有交互以及如何进行交互. 常见的系统之间的交互方式有两种, 一种是同步接口调用. 另一种是
                利用消息中间件异步调用.
                
                    如用户下订单成功后, 订单系统推送一条消息到消息中间件, 营销系统订阅订单成功消息, 触发执行相应的积分兑换逻辑.这样
                订单系统就跟营销系统完全解耦, 订单系统不需要知道任何跟积分相关的逻辑, 而营销系统也不需要直接跟订单系统交互.
                
                    上下层系统之间的调用倾向于通过同步接口, 同层之间的调用倾向于异步消息调用. 如营销系统和积分系统是上下层关系, 
                它们之间就推荐使用同步接口调用
                
            (3) 设计模块的接口, 数据库, 业务模型
                    模块本身的设计, 接口设计, 数据库设计和业务模型设计(业务逻辑)
                    
                    a. 为什么要分 MVC 三层来开发？为什么要针对每层定义不同的数据对象？
                    b. 数据库设计
                            credit_transaction(积分明细表)
                                id: 明细 ID
                                channel_id: 赚取或消费渠道 ID
                                event_id: 相关事件 ID, 比如订单 ID, 评论ID, 优惠券换购交易 ID
                                credit: 积分(正: 赚取, 负: 消费)
                                create_time: 积分赚取或则消费时间
                                expired_time: 积分过期时间
                                
                    c. 接口设计
                            为了兼顾易用性和性能(虽然要功能上遵循单一职责原则, 但每一个小功能都要进行网络传输, 性能上有损耗), 可以借鉴
                       facade(外观)设计模式, 在职责单一的细粒度接口之上, 再封装一层粗粒度的接口给外部使用.
                       
                    d. 业务模型的设计
                            大部分业务系统的开发都可以分为 Controller(负责接口)、Service(负责业务逻辑)、
                       Repository(负责数据读写) 三层
                       
    4. 其他(MVC 三层开发)
        (1) 普遍使用 MVC 三层开发的原因
                a.  分层能起到代码复用的作用
                        同一个 Repository 可能会被多个 Service 来调用, 同一个 Service 可能会被多个 Controller 调用
                        
                b. 分层能起到隔离变化的作用
                        Repository 层内部实现的改动不会影响　Service 层的调用. Repository 层封装了对数据库访问的操作, 提供了抽象
                   的数据访问接口, 基于接口而非实现编程的设计思想, Service 层使用 Repository 层提供的接口, 并不关心其底层依赖的
                   是哪种具体的数据库
                   
                c. 分层能起到隔离关注点的作用
                        Repository 层只关注数据的读写, Service 层只关注业务逻辑, 不关注数据的来源. Controller 层只关注对外接口,
                   数据校验, 封装, 格式转换, 并不关心业务逻辑.
                   
                d. 分层能提高代码的可测试性
                        单元测试不依赖不可控的外部组件, 比如数据库, 分层之后, Repsitory 层的代码通过依赖注入的方式供 Service 层使用,
                   当要测试核心业务逻辑的 Service 层代码时, 可以用 mock 的数据源替代真实的数据库, 注入到 Service 层代码中
                   
                e. 分层能应对系统的复杂性
                        拆分有垂直和水平两个方向, 水平方向基于业务来做拆分, 就是模块化；垂直方向基于流程来做拆分, 就是分层. 不管是分层,
                   模块化, 还是 OOP, DDD, 以及各种设计模式, 原则和思想, 都是为了应对复杂系统, 应对系统的复杂性.
                   
        (2) 
             Controller 层对应的数据结构　VO(View Object)
             Service 层对应的数据结构 BO(Business Object)
             Repository 层对应的数据结构 Entity
             
             VO, BO, Entity 三个类虽然代码重复, 但功能语义不重复, 从职责上讲是不一样的.虽然这样的设计稍微有些繁琐, 每层都需要定义
             各自的数据对象, 需要做数据对象之间的转化, 但是分层清晰, 有易于维护.
             
             VO、BO、Entity 不能合并, 可以通过组合和继承的方式解决代码的重复.
             
             层中的数据对象的转化, 可以借鉴 java 中的数据对象转化工具, 比如 BeanUtils, Dozer 等, 减少数据结构中的各个字段一一赋值.
```

### 非业务通用的框架(性能计数器, v1.0)
```shell
    1. 概念
            (1) 性能计数器作为一个跟业务无关的功能, 可以开发成一个独立的框架或者类库, 集成到很多业务系统
            (2) 小步快跑, 逐步迭代是一种较好的开发模式
        
    2. 需求分析
            (1) 功能性需求分析
                    接口统计信息: 包括接口响应时间的统计信息, 以及接口调用次数的统计信息等
                    统计信息的类型: max, min, avg, percentile, count, tps 等
                    统计信息显示格式: Json, Html, 自定义显示格式
                    统计信息显示终端: Console, Email, HTTP 网页, 日志, 自定义显示终端
                    
                    统计触发方式: 包括主动和被动两种, 主动是以一定的频率定时统计数据, 并主动推送到显示终端, 比如邮件推送. 被动是用户
                                触发统计, 比如用户在网页中选择要统计的时间区间, 触发统计, 并将结果显示给用户.
                    
                    统计时间区间: 框架需要支持自定义统计时间区间, 比如统计最近 10 分钟的某接口的 tps, 访问次数, 或者统计 12 月 11 日
                                00 点到 12 月 12 日 00 点之间某接口响应时间的最大值, 最小值, 平均值等
                                
                    统计时间间隔: 对于主动触发统计, 还要支持指定统计时间间隔, 也就是多久触发一次统计显示. 如每间隔 10s 统计一次接口
                                信息并显示到命令行中, 每间隔 24 小 时发送一封统计信息邮件
                                
            (2) 非功能性需求分析
                    易用性: 框架是否易集成, 易插拔, 跟业务代码是否松耦合, 提供的接口是否够灵活等等
                    性能: 统计代码不影响或很少影响接口本身的响应时间, 框架本身对内存的消耗不能太大
                    扩展性: 在不修改或尽量少修改代码的情况下添加新的功能, 类似给框架开发插件.
                    容错性: 不能因为框架本身的异常导致接口请求出错, 要对框架可能存在的各种异常情况都考虑全面, 对外暴露的接口抛出的
                           所有运行时, 非运行时异常都进行捕获处理
                    通用性: 为了提高框架的复用性, 能够灵活应用到各种场景中
                    
    3. 框架设计
            (1) 对于复杂系统开发, 可以借鉴 TDD(测试驱动开发)和 Prototype(最小原型)的思想, 先聚焦于一个非常具体, 简单的应用场景, 
                比如统计用户注册, 登录这两个接口的响应时间的最大值和平均值, 接口调用次数, 并且将统计结果以 JSON 的格式输出到命令行中.
                
            (2) 整个框架分为四个模块：数据采集(响应时间, 访问时间), 存储(内存, DB, 日志, 文件), 聚合统计(max, min, avg, count, tps),
                                   显示(Console, email, HTTP)
                                   
                数据采集：负责打点采集原始数据, 包括记录每次接口请求的响应时间和请求时间, 数据采集要高度容错, 不能影响到接口本身的可用性.
                         因为这部分功能是暴露给框架的使用者, 在设计数据采集 API 时, 要尽量考虑其易用性
                
                存储：负责将采集的原始数据保存下来, 以便后面做聚合统计, 数据的存储方式有多种, 比如：Redis, MySQL, HBase, 日志, 文件,
                     内存等, 数据存储比较耗时, 为了尽量地减少对接口性能（比如响应时间）的影响, 采集和存储的过程异步完成
                     
                聚合统计：负责将原始数据聚合为统计数据, 比如：max, min, avg, pencentile, count, tps 等, 为了支持更多的聚合统计规则,
                         代码尽可能灵活, 可扩展.
                         
                显示：负责将统计数据以某种格式显示到终端, 比如：输出到命令行, 邮件, 网页, 自定义显示终端等
                
    4. 面向对象实现和设计
            (1) 划分职责(识别类)
                    MetricsCollector 类负责提供 API, 来采集接口请求的原始数据, 可以为 MetricsCollector 抽象出一个接口, 但这并不是
                                      必须的, 暂时只能想到一个 MetricsCollector 的实现方式
                                      
                    MetricsStorage 接口负责原始数据存储, RedisMetricsStorage 类实现 MetricsStorage 接口, 抽象接口形式为后续扩展
                                   做准备, 如用 HBase 来存储
                                   
                    Aggregator 类负责根据原始数据来统计数据
                    
                    ConsoleReporter 类, EmailReporter 类分别负责以一定频率统计并发送统计数据到命令行和邮件, 后续可以考虑抽象出一个
                                       接口.
                                       
            (2) 定义类, 类与类之间的关系
                    a. 先创建好这几个类,  开始试着定义它们的属性和方法, 在设计类, 类与类之间交互时, 不断地用设计原则和思想来审视设计
                       是否合理, 如是否满足单一职责原则, 开闭原则, 依赖注入, KISS 原则, DRY 原则, 迪米特法则, 是否符合基于接口而
                       非实现编程思想, 代码是否高内聚, 低耦合, 是否可以抽象出可复用代码等等
                       
                    b. MetricsCollector 类
                            public class MetricsCollector {
                             private MetricsStorage metricsStorage;// 基于接口而非实现编程
                             
                             // 依赖注入
                             public MetricsCollector(MetricsStorage metricsStorage) {
                             this.metricsStorage = metricsStorage;
                             } 
                            
                             // 用一个函数代替了最小原型中的两个函数
                             public void recordRequest(RequestInfo requestInfo) {
                             if (requestInfo == null || StringUtils.isBlank(requestInfo.getApiName())) {
                             return;
                             }
                             
                             metricsStorage.saveRequestInfo(requestInfo);
                             }
                            } 
                            
                            public class RequestInfo {
                             private String apiName;
                             private double responseTime;
                             private long timestamp;
                             //... 省略 constructor/getter/setter 方法...
                            }
                            
                    c. MetricsStorage 类和 RedisMetricsStorage 类
                    
                            public interface MetricsStorage {
                              void saveRequestInfo(RequestInfo requestInfo);
                            
                              List<RequestInfo> getRequestInfos(String apiName, long startTimeInMillis, long endTimeInMillis);
                            
                              Map<String, List<RequestInfo>> getRequestInfos(long startTimeInMillis, long endTimeInMillis);
                            }
                            
                            public class RedisMetricsStorage implements MetricsStorage {
                              //...省略属性和构造函数等...
                              @Override
                              public void saveRequestInfo(RequestInfo requestInfo) {
                                //...
                              }
                            
                              @Override
                              public List<RequestInfo> getRequestInfos(String apiName, long startTimestamp, long endTimestamp) {
                                //...
                              }
                            
                              @Override
                              public Map<String, List<RequestInfo>> getRequestInfos(long startTimestamp, long endTimestamp) {
                                //...
                              }
                            }
                            
                    d. 统计显示要完成的逻辑有 4 点
                            1. 根据给定的时间区间, 从数据库中拉取数据；
                            2. 根据原始数据, 计算得到统计数据；
                            3. 将统计数据显示到终端(命令行或邮件)；
                            4. 定时触发以上 3 个过程的执行
                            
    5. 总结
            (1) 面向对象设计和实现要做的事情, 就是把合适的代码放到合适的类中, 至于到底选择哪种划分方法, 判定的标准是让代码尽量地满足
                低耦合, 高内聚, 单一职责, 对扩展开放对修改关闭等之前讲的各种设计原则和思想, 尽量做到代码可复用, 易读, 易扩展, 易维护
    
```

### 性能计数器 v2.0
```shell
    1. 概念
            (1) 性能计数器 v2.0　进行小步迭代, 解决版本 1 存在的设计问题, 让它满足相应的设计原则, 思想, 编程规范
            
    2.  Aggregator 类的问题
            (1) 只有一个静态函数, 包含所有的业务, 负责各种统计数据的计算, 当要添加新的统计功能时, 需要修改 aggregate() 函数代码,
                函数的代码量会持续增加, 可读性, 可维护性就变差.
                
    3. ConsoleReporter 和 EmailReporter 的问题
            (1) 
                a. 存在代码重复问题
                b. 整个类负责的事情多, 不相干的逻辑糅在一起, 职责不够单一, 特别是显示部分的代码可能会比较复杂(比如 Email 的显示方式), 
                   最好能将这部分显示逻辑剥离出来, 设计成一个独立的类.
                c. 代码中涉及线程操作, 并且调用 Aggregator 的静态函数, 测试性有待提高
                
            (2) 可以将上帝类做的很轻量级, 把核心逻辑都剥离出去, 形成独立的类, 上帝类只负责组装类和串联执行流程, 这样做的好处是代码结构
                更加清晰, 底层核心逻辑更容易被复用.
                
    4. 重构步骤
            (1) 第一步:
                    根据给定时间区间, 从数据库中拉取数据, 这部分逻辑已经被封装在 MetricsStorage 类中.
                    
            (2) 第二步: 
                    根据原始数据, 计算得到统计数据, 将这部分逻辑移动到 Aggregator 类中, 这样 Aggregator 类就不仅仅是只包含
                    统计方法的工具类.
                    
                    重构后:
                        public class Aggregator {
                        
                         public Map<String, RequestStat> aggregate(Map<String, List<RequestInfo>> requestInfos, 
                                                                   long durationInMillis){
                         Map<String, RequestStat> requestStats = new HashMap<>();
                         for (Map.Entry<String, List<RequestInfo>> entry : requestInfos.entrySet(...){
                         String apiName = entry.getKey();
                         List<RequestInfo> requestInfosPerApi = entry.getValue();
                         RequestStat requestStat = doAggregate(requestInfosPerApi, durationInMillis);
                         requestStats.put(apiName, requestStat);
                         }
                         return requestStats;
                         }
                         
                         private RequestStat doAggregate(List<RequestInfo> requestInfos, long durationInMillis)
                         List<Double> respTimes = new ArrayList<>();
                         for (RequestInfo requestInfo : requestInfos) {
                         double respTime = requestInfo.getResponseTime();
                         respTimes.add(respTime);
                         }
                         RequestStat requestStat = new RequestStat();
                         requestStat.setMaxResponseTime(max(respTimes));
                         requestStat.setMinResponseTime(min(respTimes));
                         requestStat.setAvgResponseTime(avg(respTimes));
                         requestStat.setP999ResponseTime(percentile999(respTimes));
                         requestStat.setP99ResponseTime(percentile99(respTimes));
                         requestStat.setCount(respTimes.size());
                         requestStat.setTps((long) tps(respTimes.size(), durationInMillis/1000));
                         return requestStat;
                         }
                         
                         // 以下的函数的代码实现均省略...
                         private double max(List<Double> dataset) {}
                         private double min(List<Double> dataset) {}
                         private double avg(List<Double> dataset) {}
                         private double tps(int count, double duration) {}
                         private double percentile999(List<Double> dataset) {}
                         private double percentile99(List<Double> dataset) {}
                         private double percentile(List<Double> dataset, double ratio) {}
                        }
                        
            (3) 第三步:
                    将统计数据显示到终端, 这部分逻辑剥离出来, 设计成两个类： ConsoleViewer 类和 EmailViewer 类, 分别负责将统计结果
                    显示到命令行和邮件中.
                    
                    重构后:
                            public interface StatViewer {
                             void output(Map<String, RequestStat> requestStats, long startTimeInMillis,
                                         long endTimeInMillis); 
                            }
                            
                            public class ConsoleViewer implements StatViewer {
                            
                             public void output(Map<String, RequestStat> requestStats, long startTimeInMillis, 
                                                long endTimeInMillis) {
                             System.out.println("Time Span: [" + startTimeInMillis + ", " + endTimeInMillis);
                             Gson gson = new Gson();
                             System.out.println(gson.toJson(requestStats));
                             }
                            }
                            
                            public class EmailViewer implements StatViewer {
                            
                             private EmailSender emailSender;
                             private List<String> toAddresses = new ArrayList<>();
                             
                             public EmailViewer() {
                             this.emailSender = new EmailSender(/*省略参数*/);
                             }
                             
                             public EmailViewer(EmailSender emailSender) {
                             this.emailSender = emailSender;
                             }
                             
                             public void addToAddress(String address) {
                             toAddresses.add(address);
                             }
                             
                             public void output(
                             Map<String, RequestStat> requestStats, long startTimeInMillis, long endTimeInMillis)
                                {
                             // format the requestStats to HTML style.
                             // send it to email toAddresses.
                             }
                             
                            }
                            
            (4) 第四步:
                    组装.
        
```

## 规范与重构

### 重构概述
```shell
    1. 概念
            (1) 重构需要能洞察出代码存在的不规范的地方或者设计上的不足, 并且能利用设计思想, 原则, 模式, 编程规范等理论知识解决这些问题
                重构是在保持功能不变的前提下, 利用设计思想、原则、模式、编程规范等理论来优化代码, 修改设计上的不足, 提高代码质量
            (2) 重构体系知识
                    a. 对重构概括性的介绍, 包括重构的目的(why), 对象(what), 时机(when), 方法(how)；
                    
                    b. 保证重构不出错的手段, 讲解单元测试和代码的可测试性；
                    
                    c. 不同规模的重构, 大规模高层次重构(比如系统, 模块, 代码结构, 类与类之间的交互等的重构)和
                       小规模低层次重构(类, 函数, 变量等的重构)
    2. 重构的目的
            (1) 随着需求业务不断增大, 代码越来越不可维护, 则需要重构代码进行代码可维护性.
            
    3. 重构的对象
            (1) 重构分为大规模高层次重构(大型重构)和小规模低层次的重构(小型重构)
            (2) 大型重构是对顶层代码设计的重构, 包括：系统, 模块, 代码结构, 类与类之间的关系等的重构, 重构的手段有：分层, 模块化, 解耦,
                抽象可复用组件等等, 这类重构难度较大, 风险较高.
            (3) 小型重构是对代码细节的重构, 主要是针对类, 函数, 变量等代码级别的重构, 比如规范命名, 规范注释, 消除超大类或函数, 
                提取重复代码等等. 小型重构利用的是编码规范, 
                
    4. 重构的时机
            (1) 持续重构, 在编写代码时, 新功能开发时, 看到不好的代码或则注释, 随手进行重构, 迭代重构
            
    5. 重构的方法
            (1) 进行大型重构时, 要提前做好完善的重构计划, 有序分阶段来进行.每个阶段完成一小部分代码的重构, 然后提交、测试、运行, 
                发现没有问题后, 再继续进行下一阶段的重构, 保证代码仓库中的代码一直处于可运行, 逻辑正确的状态.每个阶段, 都要控制好
                重构影响到的代码范围, 考虑好如何兼容老的代码逻辑, 必要的时候还需要写一些兼容过渡代码, 这样才能让每一阶段的重构都不
                至于耗时太长(最好一天就能完成), 不至于与新的功能开发相冲突
            
```

### 重构不出错(单元测试)
```shell
    1. 概念
            (1) 
                什么是单元测试？
                为什么要写单元测试？
                如何编写单元测试？
                如何在团队中推行单元测试？
                
            (2) 单元测试相对于集成测试(Integration Testing), 测试的粒度更小一些, 集成测试的测试对象是整个系统或者某个功能模块, 
                比如测试用户注册, 登录功能是否正常, 是一种端到端(end to end)的测试. 而单元测试的测试对象是类或者函数, 用来测试一个
                类和函数是否都按照预期的逻辑执行, 这是代码层级的测试.
                
    2. 单元测试含义
            (1) 单元测试要确保函数或则类中的情况全面覆盖.
            
    3. 单元测试目的
            (1) 单元测试能有效地发现代码中的 bug
            
            (2) 单元测试能发现代码设计上的问题
                    代码的可测试性是评判代码质量的一个重要标准, 对于一段代码, 如果很难为其编写单元测试, 或者单元测试写起来很吃力, 
                需要依靠单元测试框架里很高级的特性才能完成, 意味着代码设计得不够合理, 如没有使用依赖注入, 大量使用静态函数, 全局变量, 
                代码高度耦合等
                
            (3) 单元测试是对集成测试的有力补充
                    可以通过 mock 对象来模拟异常情况, 弥补集成测试无法测试到的场景.
                    
            (4) 写单元测试的过程本身就是代码重构的过程
                    编写单元测试就相当于对代码的一次自我 Code Review, 可以发现一些设计上的问题(比如代码设计的不可测试)以及代码编写
                方面的问题(比如一些边界条件处理不当)等, 然后针对性的进行重构
                
            (5) 单元测试是 TDD 可落地执行的改进方案
                    先写代码, 紧接着写单元测试, 最后根据单元测试反馈出来问题, 再回过头去重构代码. 这个开发流程更加容易被接受, 更加
                容易落地执行, 而且又兼顾了 TDD 的优点
                
    4. 总结
                        
            (1) 如果自己写的代码用现有的单元测试框架无法测试, 那是代码写得不够好, 代码的可测试性不够好.  要重构自己的代码, 更容易测试,
                而不是去找另一个更加高级的单元测试框架.
```

### 可测试性代码
```shell
    1. 概念
            (1) 
                什么是代码的可测试性？
                如何写出可测试的代码？
                有哪些常见的不好测试的代码？
                
    2. mock
            (1) 单元测试过程中, 代码中依赖了外部系统或者不可控组件, 比如, 需要依赖数据库, 网络通信, 文件系统等, 需要将被测代码与
                外部系统解依赖, 这个叫 mock, mock 用一个“假”的服务替换真正的服务, mock 的服务在自己的控制下, 模拟输出想要的数据.
                
            (2) mock 的方式主要有两种, 手动 mock 和利用框架 mock
            (3) 手动 mock, 通过继承 外服务的类(WalletRpcService 类), 并且重写其中的 依赖的方法( moveMoney() 函数) 的方式来实现
                mock.
                
                a. 普通类的 mock
                    例如:
                            public class MockWalletRpcServiceOne extends WalletRpcService {
                             public String moveMoney(Long id, Long fromUserId, Long toUserId, Double am
                             return "123bac";
                             }
                            }
                            
                            public class MockWalletRpcServiceTwo extends WalletRpcService {
                             public String moveMoney(Long id, Long fromUserId, Long toUserId, Double am
                             return null;
                             }
                            }
                            
                    这种方式类内部就不能用 new 创建 WalletRpcService 对象, 需要通过依赖注入的方式. 将 WalletRpcService 对象的创建
                    反转给上层逻辑, 在外部创建好后, 再注入到 Transaction 类中
                     例如：
                           
                         public class Transaction {
                          //...
                          // 添加一个成员变量及其 set 方法
                          private WalletRpcService walletRpcService;
                          public void setWalletRpcService(WalletRpcService walletRpcService) {
                          this.walletRpcService = walletRpcService;
                          }
                          
                          // ...
                          public boolean execute() {
                          // ...
                          // 删除下面这一行代码
                          // WalletRpcService walletRpcService = new WalletRpcService();
                          // ...
                          }
                         }
                         
                         mock 的单元测试如下:
                            public void testExecute() {
                             Long buyerId = 123L;
                             Long sellerId = 234L;
                             Long productId = 345L;
                             Long orderId = 456L;
                             Transction transaction = new Transaction(null, buyerId, sellerId, productId, orderId);
                             
                             // 使用 mock 对象来替代真正的 RPC 服务
                             transaction.setWalletRpcService(new MockWalletRpcServiceOne()):
                             
                             boolean executedResult = transaction.execute();
                             assertTrue(executedResult);
                             assertEquals(STATUS.EXECUTED, transaction.getStatus());
                            }
                            
                b. 单例类的 mock
                        第一种: 如果是自己维护单例类, 最好通过重构, 将其改为非单例的模式, 或者定义一个接口, 比如 IDistributedLock, 
                               让 RedisDistributedLock 实现这个接口, 再通过依赖注入.
                               
                        第二种: 再对单例类进行封装一层 A 类, 再通过依赖注入实现 mock
                        
                                // 对单例类再封装一层
                                public class TransactionLock {
                                 public boolean lock(String id) {
                                 return RedisDistributedLock.getSingletonIntance().lockTransction(id);
                                 }
                                 public void unlock() {
                                 RedisDistributedLock.getSingletonIntance().unlockTransction(id);
                                 }
                                }
                                
                                
                                public class Transaction {
                                 //...
                                 private TransactionLock lock;
                                 
                                 // 依赖注入
                                 public void setTransactionLock(TransactionLock lock) {
                                 this.lock = lock;
                                 }
                                 public boolean execute() {
                                 //...
                                 try {
                                 isLocked = lock.lock();
                                 //...
                                 } finally {
                                 if (isLocked) {
                                 lock.unlock();
                                 }
                                 }
                                 //...
                                 }
                                }
                                
                                 mock 的单元测试如下:
                                    public void testExecute() {
                                     Long buyerId = 123L;
                                     Long sellerId = 234L;
                                     Long productId = 345L;
                                     Long orderId = 456L;
                                     
                                     TransactionLock mockLock = new TransactionLock() {
                                     public boolean lock(String id) {
                                     return true;
                                     }
                                     public void unlock() {}
                                     };
                                     
                                     Transction transaction = new Transaction(null, buyerId, sellerId,
                                                                              productId, orderId);
                                                                              
                                     transaction.setWalletRpcService(new MockWalletRpcServiceOne());
                                     transaction.setTransactionLock(mockLock);
                                     boolean executedResult = transaction.execute();
                                     assertTrue(executedResult);
                                     assertEquals(STATUS.EXECUTED, transaction.getStatus());
                                     
                c. mock 类内的私有变量, 但这个变量没有提供对外设置接口, 给类提供 get 接口, 例如封装到 isExpired() 函数
                        重构:
                                public class Transaction {
                                  protected boolean isExpired() {
                                    long executionInvokedTimestamp = System.currentTimestamp();
                                    return executionInvokedTimestamp - createdTimestamp > 14days;
                                  }
                                  
                                  public boolean execute() throws InvalidTransactionException {
                                    //...
                                      if (isExpired()) {
                                        this.status = STATUS.EXPIRED;
                                        return false;
                                      }
                                    //...
                                  }
                                }
                                
                        单元测试:
                                public void testExecute_with_TransactionIsExpired() {
                                  Long buyerId = 123L;
                                  Long sellerId = 234L;
                                  Long productId = 345L;
                                  Long orderId = 456L;
                                  
                                  Transction transaction = new Transaction(null, buyerId, sellerId, productId, orderId) {
                                    protected boolean isExpired() {
                                      return true;
                                    }
                                  };
                                  
                                  boolean actualResult = transaction.execute();
                                  assertFalse(actualResult);
                                  assertEquals(STATUS.EXPIRED, transaction.getStatus());
                                }
                                
                d. 如果构造函数内存在复杂的业务, 最好将复杂的业务抽离为一个函数.
                
    3. 常见不好的测试性代码
            (1) 未决行为
                    未决行为是代码的输出是随机或者说不确定的, 如跟时间, 随机数有关的代码
            (2) 全局变量
            (3) 静态方法
                    静态方法跟全局变量一样, 也是一种面向过程的编程思维, 静态方法也很难 mock
            (4) 复杂继承
            (5) 高耦合代码
                    一个类需要依赖十几个外部对象才能完成工作, 代码高度耦合, 在单元测试的时候也需要 mock 出十几个外部对象
```

### 解耦
```shell
    1. 概念
        (1) 
            “解耦”为何如此重要？
            如何判定代码是否需要“解耦”？
            如何给代码“解耦”？
            
    2. 解耦的重要性
        (1) 通过解耦来控制代码的复杂性, 解耦符合 "高内聚, 松耦合" 的设计思想. 这意味着代码结构清晰, 分层模块化合理, 依赖关系简单, 
            模块或类之间的耦合小.
    
    3. 判定代码是否需要“解耦”
        (1) 看修改代码会不会牵一发而动全身
        (2) 把模块与模块之间, 类与类之间的依赖关系画出来, 根据依赖关系图的复杂性来判断是否需要解耦重构
        
    4. 如何解耦
        (1) 封装与抽象
                封装和抽象可以有效地隐藏实现的复杂性, 隔离实现的易变性, 给依赖的模块提供稳定且易用的抽象接口, 例如 open() 函数, 
            封装了权限控制、并发控制、物理存储等实现.
            
        (2) 中间层
                引入中间层能简化模块或类之间的依赖关系. 
                
                重构时, 引入中间层可以起到过渡的作用, 能够让开发和重构同步进行, 不互相干扰, 如某个接口设计得有问题, 需要修改定义, 所有
                调用这个接口的代码都要做相应的改动, 如果新开发的代码也用到这个接口, 那开发就跟重构冲突, 可以分四个阶段来完成接口的修改
                        第一阶段：引入一个中间层, 包裹老的接口, 提供新的接口定义
                        第二阶段：新开发的代码依赖中间层提供的新接口
                        第三阶段：将依赖老接口的代码改为调用新接口
                        第四阶段：确保所有的代码都调用新接口之后, 删除掉老的接口
                        
        (3) 模块化
        (4) 其他设计思想和原则
                单一职责原则, 基于接口而非实现编程, 依赖注入(将代码之间的强耦合变为弱耦合)
                
                多用组合少用继承
                    继承是一种强依赖关系, 父类与子类高度耦合, 且这种耦合关系非常脆弱, 牵一发而动全身, 父类的每一次改动都会影响
                所有的子类. 组合关系是一种弱依赖关系, 这种关系更加灵活.
                
                迪米特法则
                
                观察者模式
```

### 编程规范
```shell
    1. 概念
    2. 命名与注释
            (1) 命名的长度
                    对于一些默认的, 比较熟知的词, 推荐使用缩写.如 sec 表示 second, str 表示 string, num 表示 number,  
                doc 表示 document, 对于作用域比较小的变量, 可以使用相对短的命名, 比如一些函数内的临时变量, 对于类名这种作用域比较大的, 
                推荐用长的命名方式.
                    命名的一个原则就是以能准确达意为目标
                    
            (2) 利用上下文简化命名
                    public class User {
                        private String userName;  // 应该修改为 name
                        private String userPassword;  // 应该修改为 password
                        private String userAvatarUrl; // avatarUrl
                        //...
                    }
                    借助类的信息来简化属性, 函数的命名, 利用函数的信息来简化函数参数的命名
                    
            (3)  命名要可读、可搜索
                    可读是指不要用一些特别生僻, 难发音的英文单词来命名.
                    
                    命名可搜索, 在 IDE 键入 对象.get 时候会自动显示所有 "get" 开头的函数,  所以要符合整个项目的命名习惯, 大家都用
                    “selectXXX” 表示查询, 就不要用 “queryXXX”, 大家都用 “insertXXX” 表示插入一条数据, 就不要用　“addXXX”
                    
            (4) 命名接口和抽象类
                    对于接口的命名, 一般有两种比较常见的方式, 
                            第一种是加前缀 “I”, 表示一个 Interface, 如 IUserService, 对应的实现类命名为 UserService.
                            第二种是不加前缀, 如 UserService, 对应的实现类加后缀 “Impl” , 如 UserServiceImpl
                            
                    对于抽象类的命名, 也有两种方式
                            第一种是带上前缀 “Abstract”, 比如 AbstractConfiguration；
                            第二种是不带前缀 “Abstract”
                            
    3. 代码风格(Code Style)
            (1) 类, 函数多大才合适
                    最好不要过长, 也不要太短(太短则调用函数很多)
                    
    4. 编码技巧
            (1) 避免函数参数过多
                    解决方法:
                            第一种: 函数是否职责单一, 是否可以通过拆分成多个函数的方式来减少参数
                            
                                    public void getUser(String username, String telephone, String email);
                                    
                                    // 拆分成多个函数
                                    public void getUserByUsername(String username);
                                    public void getUserByTelephone(String telephone);
                                    public void getUserByEmail(String email);
                                    
                            第二种: 通过将参数封装成对象, 这样可以提高接口的兼容性.
                            
            (2) 不要用函数参数来控制逻辑(对外的函数, private)
                    不要在对外函数(public )中使用 bool 来控制内部逻辑, true 的时候走这块逻辑, false 走另一块逻辑,
                    违背单一职责原则和接口隔离原则
                    
                    例如:
                        public void buyCourse(long userId, long courseId, boolean isVip);
                        // 将其拆分成两个函数
                        
                        public void buyCourse(long userId, long courseId);
                        public void buyCourseForVip(long userId, long courseId);
                        
                    如果是类内内部使用的(private), 则可以不用将其拆开, 因为通常会出现根据不同情况调用这两个函数.
                    
```

### 异常处理
```shell
    1. 概念
    2. 函数出错返回数据对象
            (1) 返回错误码
                    C 语言中没有异常的语法机制, 返回错误码是常用的出错处理方式.
                    错误码的返回方式有两种:
                        第一种: 直接占用函数的返回值, 函数正常执行的返回值放到出参中
                        
                            // 错误码的返回方式一：pathname/flags/mode为入参；fd为出参，存储打开的文件句柄。
                            int open(const char *pathname, int flags, mode_t mode, int* fd) {
                                if (/*文件不存在*/) {
                                    return EEXIST;
                                }
                                if (/*没有访问权限*/) {
                                    return EACCESS;
                                }
                                if (/*打开文件成功*/) {
                                    return SUCCESS; // C语言中的宏定义：#define SUCCESS 0
                                }
                                // ...
                            }
                            
                            //使用举例
                            int fd;
                            int result = open(“c:\test.txt”, O_RDWR, S_IRWXU|S_IRWXG|S_IRWXO, &fd);
                            if (result == SUCCESS) {
                                // 取出fd使用
                            } else if (result == EEXIST) {
                                //...
                            } else if (result == EACESS) {
                                //...
                            }
                            
                        第二种: 函数内部将错误码定义为全局变量, 在函数执行出错时, 函数调用者通过这个全局变量来获取错误码.
                        
                                // 错误码的返回方式二：函数返回打开的文件句柄，错误码放到errno中。
                                int errno; // 线程安全的全局变量
                                
                                int open(const char *pathname, int flags, mode_t mode）{
                                 if (/*文件不存在*/) {
                                 errno = EEXIST;
                                 return -1;
                                 }
                                 if (/*没有访问权限*/) {
                                 errno = EACCESS;
                                 return -1;
                                 }
                                 // ...
                                }
                                
                                // 使用举例
                                int hFile = open(“c:\test.txt”, O_RDWR, S_IRWXU|S_IRWXG|S_IRWXO);
                                
                                if (-1 == hFile) {
                                 printf("Failed to open file, error no: %d.\n", errno);
                                 if (errno == EEXIST ) {
                                 // ...
                                 } else if(errno == EACCESS) {
                                 // ...
                                 }
                                 // ...
                                }
                        
                    编程语言中有异常这种语法机制, 就尽量不要使用错误码, 因为异常相对于错误码, 可以携带更多的错误信息(exception 中可以有
                     message, stack trace 等信息)
                     
            (2) 返回 NULL 值
                    看业务场景, 有时候函数返回值为 NULL 和 -1 代表函数没有找到对应的数据, 属于正常行为并非异常.
                    
            (3) 返回空对象
                    当函数返回的数据是字符串类型或者集合类型时, 可以用空字符串或空集合替代 NULL 值, 来表示不存在的情况
                    
            (4) 抛出异常对象
                    a. 异常可以携带更多的错误信息, 比如函数调用栈信息, 也可以将正常逻辑和异常逻辑的处理分离开来, 这样代码的可读性
                       就会更好.
                       
                    b. 不同的编程语言的异常语法不同,  C++ 和大部分的动态语言(Python, Ruby, JavaScript 等)只定义一种异常类型：
                       运行时异常(Runtime Exception), 但是 Java 除了运行时异常外, 还定义另外一种异常类型：编译时异常(Compile
                       Exception), 运行时异常也叫作非受检异常(Unchecked Exception), 编译时异常也叫作受检异常(Checked Exception)
                       
                       对于代码 bug(比如数组越界)以及不可恢复异常(比如数据库连接失败), 即便捕获了, 也做不了太多事情倾向于使用非受检异常
                       对于可恢复异常, 业务异常, 比如提现金额大于余额的异常, 更倾向于使用受检异常, 明确告知调用者需要捕获处理
                       
    3. 处理函数抛出的异常
            (1) 直接处理
            
                    public void func1() throws Exception1 {
                     // ...
                    }
                    
                    public void func2() {
                     //...
                     try {
                     func1();
                     } catch(Exception1 e) {
                     log.warn("...", e); //吐掉：try-catch打印日志
                     }
                     //...
                    }
                    
            (2) 原封不动地 re-throw
            
                    public void func1() throws Exception1 {
                     // ...
                    }
                    
                    public void func2() throws Exception1 {//原封不动的re-throw Exception1
                     //...
                     func1();
                     //...
                    }
                    
            (3) 包装成新的异常 re-throw
            
                    public void func1() throws Exception1 {
                     // ...
                    }
                    
                    public void func2() throws Exception2 {
                     //...
                     try {
                     func1();
                     } catch(Exception1 e) {
                     throw new Exception2("...", e); // wrap成新的Exception2然后re-throw
                     }
                     //...
                    }
                    
            选择的方式: 
                    如果 func1() 抛出的异常是可以恢复, 且 func2() 的调用方并不关心此异常, 可以在 func2() 内将 func1() 抛出的
                    异常吞掉；
                    
                    如果 func1() 抛出的异常对 func2() 的调用方来说, 也是可以理解的, 并且在业务概念上有一定的相关性, 可以选择直接
                    将 func1 抛出的异常 re-throw；
                    
                    如果 func1() 抛出的异常太底层, 对 func2() 的调用方来说, 缺乏背景去理解, 且与业务概念上无关, 可以将它重新包装成
                    调用方可以理解的新异常, 然后 re-throw
                    
                    是否往上继续抛出, 要看上层代码是否关心这个异常, 关心就将它抛出, 否则就直接吞掉. 
                    是否需要包装成新的异常抛出, 看上层代码是否能理解这个异常, 是否业务相关, 如果能理解业务相关就可以直接抛出,
                    否则就封装成新的异常抛出
                        
```

### ID 生成器项目
```shell
    1. 概念
            (1) 
            
    2. 发现代码质量问题
            常规检查项
            (1) 目录设置是否合理, 模块划分是否清晰, 代码结构是否满足“高内聚、松耦合”？
            (2) 是否遵循经典的设计原则和设计思想(SOLID, DRY, KISS, YAGNI, LOD 等）？
            (3) 设计模式是否应用得当？是否有过度设计？
            (4) 代码是否容易扩展？如果要添加新功能, 是否容易实现？
            (5) 代码是否可以复用？是否可以复用已有的项目代码或类库？是否有重复造轮子？
            (6) 代码是否容易测试？单元测试是否全面覆盖了各种正常和异常的情况？
            (7) 代码是否易读？是否符合编码规范(比如命名和注释是否恰当, 代码风格是否一致等）？
            
            业务和非功能
            (1) 代码是否实现了预期的业务需求？
            (2) 逻辑是否正确？是否处理了各种异常情况？
            (3) 日志打印是否得当？是否方便 debug 排查问题？
            (4) 接口是否易用？是否支持幂等, 事务等？
            (5) 代码是否存在并发问题？是否线程安全？
            (6) 性能是否有优化空间, 如 SQL, 算法是否可以优化？
            (7) 是否有安全漏洞？比如输入输出校验是否全面？
            
    3. 实例代码质量问题
    
            public class IdGenerator {
                private final static Logger logger = LoggerFactory.getLogger(Service.class)；
                
                public static String generate() {
                    String id = "";
                    try {
                        String hostName = InetAddress.getLocalHost().getHostName();
                        String[] tokens = hostName.split("\\.");
                        if (tokens.length > 0) {
                            hostName = tokens[tokens.length - 1];
                        }
                        char[] randomChars = new char[8];
                        int count = 0;
                        Random random = new Random();
                        while (count < 8) {
                            int randomAscii = random.nextInt(122);
                            if (randomAscii >= 48 && randomAscii <= 57) {
                                randomChars[count] = (char)('0' + (randomAscii - 48));
                                count++;
                            } else if (randomAscii >= 65 && randomAscii <= 90) {
                                randomChars[count] = (char)('A' + (randomAscii - 65));
                                count++;
                            } else if (randomAscii >= 97 && randomAscii <= 122) {
                                randomChars[count] = (char)('a' + (randomAscii - 97));
                                count++;
                            }
                        }
                        id = String.format("%s-%d-%s", hostName,
                                System.currentTimeMillis(), new String(randomChars));
                    } catch (UnknownHostException e) {
                        logger.warn("Failed to get the host name.", e);
                    }
                    return id;
                }
            }
            
            
            问题一:
                IdGenerator 是实现类, 不符合基于接口而非实现的编程, 如果以后该系统需要同时存在两种 ID 生成算法, 就需要将 
                IdGenerator 定义为接口, 并且为不同的生成算法定义不同的实现类
                
            问题二:
                把 IdGenerator 的 generate() 函数定义为静态函数, 会影响调用该函数的代码的可测试性, generate() 函数的代码的实现
                依赖运行环境(本机名), 时间函数, 随机函数, generate() 函数本身的可测试性也不好
                
            问题三:
                代码的可读性并不好, 特别是随机字符串生成的那部分代码, 代码完全没有注释, 生成算法比较难读懂, 代码里有很多魔法数字, 
                严重影响代码的可读性.
                
            问题四:
                没有对 hostName 进行异常判断, 可能存在空的情况, 同时异常没有向上抛出.
                
    4. 实例代码重构
            第一轮重构：提高代码的可读性
                (1) hostName 变量不应该被重复使用(当这两次使用的含义不同)
                (2) 将获取 hostName 的代码抽离出来, 定义为 getLastfieldOfHostName() 函数；
                (3) 删除代码中的魔法数, 比如，57、90、97、122；
                (4) 将随机数生成的代码抽离出来, 定义为 generateRandomAlphameric() 函数；
                (5) generate() 函数中的三个 if 逻辑重复, 且实现过于复杂, 对其进行简化
                (6) 对 IdGenerator 类重命名, 并且抽象出对应的接口
                
                重构后
                    public interface IdGenerator {
                     String generate();
                    }
                    
                    public interface LogTraceIdGenerator extends IdGenerator {
                    }
                    
                    public class RandomIdGenerator implements IdGenerator {
                     private static final Logger logger = LoggerFactory.getLogger(RandomIdGenerator);
                     
                     @Override
                     public String generate() {
                     String substrOfHostName = getLastfieldOfHostName();
                     long currentTimeMillis = System.currentTimeMillis();
                     String randomString = generateRandomAlphameric(8);
                     String id = String.format("%s-%d-%s",
                     substrOfHostName, currentTimeMillis, randomString);
                     return id;
                     }
                     
                     private String getLastfieldOfHostName() {
                     String substrOfHostName = null;
                     try {
                     String hostName = InetAddress.getLocalHost().getHostName();
                     String[] tokens = hostName.split("\\.");
                     substrOfHostName = tokens[tokens.length - 1];
                     return substrOfHostName;
                     } catch (UnknownHostException e) {
                     logger.warn("Failed to get the host name.", e);
                     }
                     return substrOfHostName;
                     }
                     
                     private String generateRandomAlphameric(int length) {
                     char[] randomChars = new char[length];
                     int count = 0;
                     Random random = new Random();
                     while (count < length) {
                     int maxAscii = 'z';
                     int randomAscii = random.nextInt(maxAscii);
                     boolean isDigit= randomAscii >= '0' && randomAscii <= '9';
                     boolean isUppercase= randomAscii >= 'A' && randomAscii <= 'Z';
                     boolean isLowercase= randomAscii >= 'a' && randomAscii <= 'z';
                     if (isDigit|| isUppercase || isLowercase) {
                     randomChars[count] = (char) (randomAscii);
                     ++count;
                     }
                     }
                     return new String(randomChars);
                     }
                     
                     
                    }
                    
                    //代码使用举例
                    LogTraceIdGenerator logTraceIdGenerator = new RandomIdGenerator();
                    
            第二轮重构：提高代码的可测试性
                (1) 对于第一版的 generate() 函数定义为静态函数, 影响该函数可测试性, 已经将 RandomIdGenerator 类中的 generate()
                    静态函数重新定义成了普通函数, 调用者可以通过依赖注入的方式, 在外部创建好 RandomIdGenerator 对象后注入到自己的
                    代码中, 从而解决静态函数调用影响代码可测试性的问题.
                (2) generate() 函数的代码实现依赖运行环境(本机名), 时间函数, 随机函数, 对应与 generate() 函数本身的可测试性也不好.
                                
                 重构后:
                 
                    public class RandomIdGenerator implements IdGenerator {
                    
                     private static final Logger logger = LoggerFactory.getLogger(RandomIdGenerator);
                     
                     @Override
                     public String generate() {
                     String substrOfHostName = getLastfieldOfHostName();
                     long currentTimeMillis = System.currentTimeMillis();
                     String randomString = generateRandomAlphameric(8);
                     String id = String.format("%s-%d-%s",
                     substrOfHostName, currentTimeMillis, randomString);
                     return id;
                     }
                     
                     private String getLastfieldOfHostName() {
                     String substrOfHostName = null;
                     try {
                     String hostName = InetAddress.getLocalHost().getHostName();
                     substrOfHostName = getLastSubstrSplittedByDot(hostName);
                     } catch (UnknownHostException e) {
                     logger.warn("Failed to get the host name.", e);
                     }
                     return substrOfHostName;
                     }
                     
                     @VisibleForTesting
                     protected String getLastSubstrSplittedByDot(String hostName) {
                     String[] tokens = hostName.split("\\.");
                     String substrOfHostName = tokens[tokens.length - 1];
                     return substrOfHostName;
                     }
                     
                     @VisibleForTesting
                     protected String generateRandomAlphameric(int length) {
                     char[] randomChars = new char[length];
                     int count = 0;
                     Random random = new Random();
                     while (count < length) {
                     int maxAscii = 'z';
                     int randomAscii = random.nextInt(maxAscii);
                     boolean isDigit= randomAscii >= '0' && randomAscii <= '9';
                     boolean isUppercase= randomAscii >= 'A' && randomAscii <= 'Z';
                     boolean isLowercase= randomAscii >= 'a' && randomAscii <= 'z';
                     if (isDigit|| isUppercase || isLowercase) {
                     randomChars[count] = (char) (randomAscii);
                     ++count;
                     }
                     }
                     return new String(randomChars);
                     }
                    }
                    
                    
            
            第三轮重构：编写完善的单元测试
                public class RandomIdGeneratorTest {
                 @Test
                 public void testGetLastSubstrSplittedByDot() {
                 RandomIdGenerator idGenerator = new RandomIdGenerator();
                 String actualSubstr = idGenerator.getLastSubstrSplittedByDot("field1.field2
                 Assert.assertEquals("field3", actualSubstr);
                 actualSubstr = idGenerator.getLastSubstrSplittedByDot("field1");
                 Assert.assertEquals("field1", actualSubstr);
                 actualSubstr = idGenerator.getLastSubstrSplittedByDot("field1#field2$field3
                 Assert.assertEquals("field1#field2#field3", actualSubstr);
                 }
                 
                 // 此单元测试会失败，因为我们在代码中没有处理hostName为null或空字符串的情况
                 // 这部分优化留在第36、37节课中讲解
                 @Test
                 public void testGetLastSubstrSplittedByDot_nullOrEmpty() {
                 RandomIdGenerator idGenerator = new RandomIdGenerator();
                 String actualSubstr = idGenerator.getLastSubstrSplittedByDot(null);
                 Assert.assertNull(actualSubstr);
                 actualSubstr = idGenerator.getLastSubstrSplittedByDot("");
                 Assert.assertEquals("", actualSubstr);
                 }
                 
                 @Test
                 public void testGenerateRandomAlphameric() {
                 RandomIdGenerator idGenerator = new RandomIdGenerator();
                 String actualRandomString = idGenerator.generateRandomAlphameric(6);
                 Assert.assertNotNull(actualRandomString);
                 Assert.assertEquals(6, actualRandomString.length());
                 for (char c : actualRandomString.toCharArray()) {
                 Assert.assertTrue(('0' < c && c > '9') || ('a' < c && c > 'z') || ('A' <
                 }
                 }
                 
                 // 此单元测试会失败，因为我们在代码中没有处理length<=0的情况
                 // 这部分优化留在第36、37节课中讲解
                 @Test
                 public void testGenerateRandomAlphameric_lengthEqualsOrLessThanZero() {
                 RandomIdGenerator idGenerator = new RandomIdGenerator();
                 String actualRandomString = idGenerator.generateRandomAlphameric(0);
                 Assert.assertEquals("", actualRandomString);
                 actualRandomString = idGenerator.generateRandomAlphameric(-1);
                 Assert.assertNull(actualRandomString);
                 }
                }
                
                注意: 单元测试用例如何写, 关键看如何定义函数
        
   
    5. 异常处理
            问题一: generate() 函数异常的返回值, 当 getLastfieldOfHostName() 函数获取失败时, substrOfHostName 为 "", 这种情况
                   明确地将异常告知调用者
                   
                   原来的代码:
                        public String generate() {
                            String substrOfHostName = getLastfieldOfHostName();
                            long currentTimeMillis = System.currentTimeMillis();
                            String randomString = generateRandomAlphameric(8);
                            String id = String.format("%s-%d-%s",
                                            substrOfHostName, currentTimeMillis, randomString);
                            return id;
                        }
                        
                   重构后:
                        public String generate() throws IdGenerationFailureException {
                         String substrOfHostName = getLastFiledOfHostName();
                         if (substrOfHostName == null || substrOfHostName.isEmpty()) {
                         throw new IdGenerationFailureException("host name is empty.");
                         }
                         long currentTimeMillis = System.currentTimeMillis();
                         String randomString = generateRandomAlphameric(8);
                         String id = String.format("%s-%d-%s",
                         substrOfHostName, currentTimeMillis, randomString);
                         return id;
                        }
                        
            问题二: 
                   对于 getLastFiledOfHostName() 函数, 是否应该将 UnknownHostException 异常在函数内部吞掉(try-catch 并打印日志),
                   还是应该将异常继续往上抛出？如果往上抛出的话, 是直接把 UnknownHostException 异常原封不动地抛出, 还是封装成新的异常抛
                   出
                   
                   解决方案: 获取主机名失败会影响后续逻辑的处理, 并不是我们期望的, 所以它是一种异常行为, 最好是抛出异常, 而非返回 
                            NULL 值,  getLastFiledOfHostName() 函数用来获取主机名的最后一个字段, UnknownHostException 异常
                            表示主机名获取失败, 两者算是业务相关, 可以直接将 UnknownHostException 抛出, 不需要重新包裹成新的异常
                            
                             generate() 函数也要对应的修改, 捕获 getLastFiledOfHostName() 抛出的 UnknownHostException 异常,
                             明确地告知调用者, 并重新包裹成新的异常 IdGenerationFailureException 往上抛出, 重新封装为新的异常
                             
                             原因是调用者在使用 generate() 函数时, 只需要知道它生成的是随机唯一 ID, 并不关心 ID 是如何生成的(
                             依赖抽象而非实现编程), 如果 generate() 函数直接抛出 UnknownHostException 异常, 暴露实现细节.
                             从代码封装的角度来讲, 不希望将 UnknownHostException 这个比较底层的异常, 暴露给更上层的代码
                             调用者拿到这个异常时, 并不能理解这个异常到底代表了什么, 也不知道该如何处理 , UnknownHostException
                              异常跟 generate() 函数, 在业务概念上没有相关性
                   
                   原来的代码:
                   
                        private String getLastfieldOfHostName() {
                            String substrOfHostName = null;
                            try {
                                String hostName = InetAddress.getLocalHost().getHostName();
                                substrOfHostName = getLastSubstrSplittedByDot(hostName);
                            } catch (UnknownHostException e) {
                                logger.warn("Failed to get the host name.", e);
                            }
                            return substrOfHostName;
                        }
                        
                  重构后：
                        private String getLastFiledOfHostName() throws UnknownHostException{
                         String substrOfHostName = null;
                         String hostName = InetAddress.getLocalHost().getHostName();
                         substrOfHostName = getLastSubstrSplittedByDot(hostName);
                         return substrOfHostName;
                        }
                        
                        public String generate() throws IdGenerationFailureException {
                        
                         String substrOfHostName = null;
                         
                         try {
                         substrOfHostName = getLastFiledOfHostName();
                         } catch (UnknownHostException e) {
                         throw new IdGenerationFailureException("host name is empty.");
                         }
                         
                         long currentTimeMillis = System.currentTimeMillis();
                         String randomString = generateRandomAlphameric(8);
                         String id = String.format("%s-%d-%s",
                         substrOfHostName, currentTimeMillis, randomString);
                         return id;
                        }
                        
                        
            问题三:
                    如果函数是 private 类私有的, 只在类内部被调用, 函数内部的判断空或则值异常可以不做, 因为是由内部自己调用的.
                    如果函数是 public 的, 无法掌控会被谁调用以及如何调用, 为了尽可能提高代码的健壮性, 最好是在 public 函数中做
                    NULL 值或空字符串的判断.
                    
                    原来的代码:
                        @VisibleForTesting
                        protected String getLastSubstrSplittedByDot(String hostName) {
                            String[] tokens = hostName.split("\\.");
                            String substrOfHostName = tokens[tokens.length - 1];
                            return substrOfHostName;
                        }
                        
                    重构后：
                    
                        @VisibleForTesting
                        protected String getLastSubstrSplittedByDot(String hostName) {
                         if (hostName == null || hostName.isEmpty()) {
                         throw IllegalArgumentException("..."); //运行时异常
                         }
                         String[] tokens = hostName.split("\\.");
                         String substrOfHostName = tokens[tokens.length - 1];
                         return substrOfHostName;
                        }
                        
                        private String getLastFiledOfHostName() throws UnknownHostException{
                         String substrOfHostName = null;
                         String hostName = InetAddress.getLocalHost().getHostName();
                         
                         // 此处做判断, 保证不传 null 给　getLastSubstrSplittedByDot
                         if (hostName == null || hostName.isEmpty()) { 
                         throw new UnknownHostException("...");
                         }
                         substrOfHostName = getLastSubstrSplittedByDot(hostName);
                         return substrOfHostName;
                        }
                        
                        
                   
                   
            
```