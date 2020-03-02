# gtest 使用(c++ 单元测试框架)
```shell
    1. 编译安装
            > cmake -DCMAKE_INSTALL_PREFIX=/work/install -DBUILD_SHARED_LIBS=ON 
                    -Dgtest_build_tests=ON -Dgtest_build_samples=ON .
```

## main 使用

```shell
    1.     testing::InitGoogleTest(&argc, argv);  // gtest 相关初始化,也用于运行参数的传入
           RUN_ALL_TESTS();  // 运行所有测试用例
```

## TEST 宏

```shell
    1. TEST(test_case_name, test_name)
    
       //多个测试场景需要相同数据配置的情况，用TEST_F。TEST_F  test fixture，测试夹具，测试套，承担了一个注册的功能。
       利用了 C++ 继承类来实现对父类方法的测试
       TEST_F(test_fixture, test_name)  
```

## 断言(ASSERT_*/EXPECT_*)
```shell
    1. EXPECT_*
            (1) EXPECT_TRUE(condition);   condition 为 true 通过
            (2) EXPECT_FALSE(condition);  condition 为 false 通过
            (3) EXPECT_EQ(expected, actual);  expected == actual
            (4) EXPECT_NE(val1, val2);  val1 != val2
            (5) EXPECT_LT(val1, val2);  val1 < val2
            (6) EXPECT_LE(val1, val2);  val1 <= val2
            (7) EXPECT_GT(val1, val2);  val1 > val2
            (8) EXPECT_GE(val1, val2);  val1 >= val2
            (9) EXPECT_STREQ(expected_str, actual_str);   c 风格的字符串 2 个内容相同通过
            (10) EXPECT_STRNE(str1, str2);  // the two C strings have different content
            (11) EXPECT_STRCASEEQ(expected_str, actual_str);  // c 风格的字符串 2 个内容相同,忽略大小写
            (12) EXPECT_STRCASENE(str1, str2);  // the two C strings have different content, ignoring case
    2. 用户可以直接通过 “<<” 在这些断言宏后面跟上自己希望在断言命中时的输出信息
```

## 事件机制(test fixture 测试固件)

```shell
    1. gtest 的事件机制,是为方便我们在案例之前或则案例之后做一些操作., 主要继承 Test 类, 使用 TEST_F 宏, 
       事件机制就是测试固件(test fixture), 就是测试运行之前所需的稳定的、公共的可重复的运行环境，这个“环境”不仅可以是数据，
       也可以指对被测软件的准备，例如实例化被测方法所依赖的类、加载数据库等等。
       测试固件原则是每个 testinfo 运行，不管结果如何，下一次进行　test fixture 都不会有影响.
    2. gtest 的事件一共有 3 种：
           (1) 全局的, 所有案例执行前后
           (2) TestSuite 级别的，在某一批案例中第一个案例前, 最后一个案例执行后。
           (3) TestCase 级别的，每个 TestCase 前后。
    3. 全局事件
            要实现全局事件, 必须写一个类, 继承testing::Environment类, 实现里面的 SetUp 和 TearDown 方法。
            (1). SetUp()方法在所有案例执行前执行
            (2). TearDown()方法在所有案例执行后执行
            (3) 在 main 中　通过　testing::AddGlobalTestEnvironment(new FooEnvironment);将这些类注册上去
            
    4. TestSuite 事件
            需要写一个类，继承 testing::Test，然后实现两个静态方法
            (1). SetUpTestCase() 方法在第一个 TestCase 之前执行
            (2). TearDownTestCase() 方法在最后一个 TestCase 之后执行
            
            例如:
                class FooTest : public testing::Test {
                 protected:
                  static void SetUpTestCase() {
                    shared_resource_ = new ;
                  }
                  static void TearDownTestCase() {
                    delete shared_resource_;
                    shared_resource_ = NULL;
                  }
                  // Some expensive resource shared by all tests.
                  static T* shared_resource_;
                };
                
                在编写测试案例时，使用 TEST_F 这个宏，第一个参数必须是上面类的名字，代表一个 TestSuite
                
                TEST_F(FooTest, Test1)
                 {
                    // you can refer to shared_resource here 
                 }
                 
                TEST_F(FooTest, Test2)
                 {
                    // you can refer to shared_resource here 
                 }
    5. TestCase 事件
            挂在每个案例执行前后的，继承自  testing::Test, 需要实现的是 SetUp 方法和 TearDown 方法：
            (1) SetUp() 方法在每个 TestCase 之前执行
            (2) TearDown()方法在每个 TestCase 之后执行
            
            实例:
                class FooCalcTest:public testing::Test
                {
                protected:
                    virtual void SetUp()
                    {
                        m_foo.Init();
                    }
                    virtual void TearDown()
                    {
                        m_foo.Finalize();
                    }
                
                    FooCalc m_foo;
                };
                
                TEST_F(FooCalcTest, HandleNoneZeroInput)
                {
                    EXPECT_EQ(4, m_foo.Calc(12, 16));
                }
                
                TEST_F(FooCalcTest, HandleNoneZeroInput_Error)
                {
                    EXPECT_EQ(5, m_foo.Calc(12, 16));
                }
                
            其他:
                test fixture 可以进行多层继承, 注意 virtual void SetUp() 会进行重写覆盖，如果要用第一层的 SetUp() 得显式调用
                
                class QuickTest : public testing::Test {
                 protected:
                  // Remember that SetUp() is run immediately before a test starts.
                  // This is a good place to record the start time.
                  virtual void SetUp() {
                    start_time_ = time(NULL);
                  }
                
                  // TearDown() is invoked immediately after a test finishes.  Here we
                  // check if the test was too slow.
                  virtual void TearDown() {
                    const time_t end_time = time(NULL);
              
                    EXPECT_TRUE(end_time - start_time_ <= 5) << "The test took too long.";
                  }
               
                  time_t start_time_;
                };
                
                class QueueTest : public QuickTest {
                 protected:
                  virtual void SetUp() {
                    // First, we need to set up the super fixture (QuickTest).
                    QuickTest::SetUp();
                
                    // Second, some additional setup for this fixture.
                    q1_.Enqueue(1);
                  }
                
                  // By default, TearDown() inherits the behavior of
                  // QuickTest::TearDown().  As we have no additional cleaning work
                  // for QueueTest, we omit it here.
                  //
                  // virtual void TearDown() {
                  //   QuickTest::TearDown();
                  // }
                
                  Queue<int> q0_;
   
                };
```

## 参数化
```shell
    0. 对代码实现的功能使用不同的参数进行测试，比如使用大量随机值来检验算法实现的正确性, 或者比较同一个接口的不同实现之间的差别.
       gtest 把“集中输入测试参数”的需求抽象出来提供支持，称为值参数化测试(Value Parameterized Test)
       
    1. 主要是解决重复写 EXPECT_TRUE(IsPrime(3));
            TEST(IsPrimeTest, HandleTrueReturn)
            {
                EXPECT_TRUE(IsPrime(3));
                EXPECT_TRUE(IsPrime(5));
                EXPECT_TRUE(IsPrime(11));
                EXPECT_TRUE(IsPrime(23));
                EXPECT_TRUE(IsPrime(17));
            }
            
    2. 方案流程
            (1) 第一个添加一个类继承 testing::TestWithParam<T>, 这个类名就是 TEST_P 的第一个参数
                模板参数是需要输入的测试参数的类型.由于 TestWithParam 本身是从 Test 派生的，所以派生类就成一个测试固件类
                class IsPrimeParamTest : public::testing::TestWithParam<int>
                {
                
                };
                在 IsPrimeParamTest (派生类)中, 可以实现诸如 SetUp、TearDown等方法。
                测试参数(由外部显示输入)由 TestWithParam 实现的 GetParam() 方法依次返回。
            (2) 
                TEST_P(IsPrimeParamTest(类名), HandleTrueReturn)
                {
                    int n =  GetParam();  // IsPrimeParamTest 中的成员函数(继承自 TestWithParam)
                    EXPECT_TRUE(IsPrime(n));
                }
            (3)
                使用 INSTANTIATE_TEST_CASE_P 这宏来告诉 gtest 要测试的参数范围
                INSTANTIATE_TEST_CASE_P(TrueReturn, IsPrimeParamTest, testing::Values(3, 5, 11, 23, 17));
                    第一个参数是测试案例的前缀，可以任意取。 
                    第二个参数是测试案例的名称，需要和之前定义的参数化的类的名称相同，如：IsPrimeParamTest 
                    第三个参数是可以理解为参数生成器，上面的例子使用 test::Values 表示使用括号内的参数
                        Google提供了一系列的参数生成的函数:
                            (1) Range(begin, end[, step]) : 范围在 begin ~ end 之间, 步长为 step, 不包括 end
                            (2) Values(v1, v2, ..., vN) : v1,v2 到 vN 的值
                            (3) ValuesIn(container) 或则 ValuesIn(begin, end): 从一个 C 类型的数组或是 STL 容器，或是迭代器中取值
                            (4) Bool() : 取 false 和 true 两个值
                            (5) Combine(g1, g2, ..., gN) : 它将 g1,g2,...gN 进行排列组合, g1,g2,...gN 本身是一个参数生成器,
                                                           每次分别从 g1,g2,..gN 中各取出一个值，组合成一个元组(Tuple)作为一个参数。
                                                            说明：这个功能只在提供了 <tr1/tuple> 头的系统中有效。 gtest 会自动去
                                                            判断是否支持 tr/tuple, 如果你的系统确实支持，而 gtest 判断错误的话,
                                                            你可以重新定义宏 GTEST_HAS_TR1_TUPLE=1。
                                                            
            (4) 例子:
                    using testing::TestWithParam;
                    using testing::Values;
                    
                    // 累加方法
                    unsigned NaiveAddUpTo(unsigned n) {
                        unsigned sum = 0;
                        for(unsigned i = 1; i <= n; ++i) sum += i;
                        return sum;
                    }
                    
                    // 公式方法
                    unsigned FastAddUpTo(unsigned n) {
                        return n*(n+1)/2;
                    }
                    
                    class AddUpToTest : public testing::TestWithParam<unsigned>
                    {
                        public:
                            AddUpToTest() { n_ = GetParam(); }
                        protected:
                            unsigned n_;
                    };
                    
                    TEST_P(AddUpToTest, Calibration) {
                        EXPECT_EQ(NaiveAddUpTo(n_), FastAddUpTo(n_));
                    }
                    
                    INSTANTIATE_TEST_CASE_P(
                            NaiveAndFast, // prefix
                            AddUpToTest,   // test case name
                    //        testing::Range(1u, 1000u) // parameters
                    testing::Values(1, 2, 3);
                    );
                    
                    注意 TestWithParam 的模板参数设置为 unsigned 类型，而在代码倒数第 2 行，两个常量值都加了 u 后缀来指定为
                    unsigned 类型。模板函数在进行类型推断（deduction）时匹配相当严格，不像普通函数那样允许类型提升（promotion）。
                    如果上面省略 u 后缀，就会造成编译错误。当然还可以显式指定模板参数：testing::Range<unsigned>(1, 1000)
                        
```

## 模板测试

```shell
    1. https://blog.csdn.net/breaksoftware/article/details/51059584
    2. googletest/sample6_unittest.cc
            (1) 主要是针对已经知道所有类型的模板类
            (2)  不太确定以后类型是否有增加.
```


## 死亡测试
```shell
    1. 通常在测试过程中，需要考虑各种各样的输入，有的输入可能直接导致程序崩溃，这时就需要检查程序是否按照预期的方式挂掉，
       这也就是所谓的“死亡测试”. gtest 的死亡测试能做到在一个安全的环境下执行崩溃的测试案例，同时又对崩溃结果进行验证。
    2. *_DEATH(statement, regex)
        (1) 其中包括 ASSERT_DEATH 和 EXPECT_DEATH
        (2) 
            1. statement 是被测试的代码语句
            2. regex 是一个正则表达式，用来匹配异常时在 stderr 中输出的内容
            例如:
                void Foo()
                {
                    int *pInt = 0;
                    *pInt = 42 ;
                }
                
                TEST(FooDeathTest, Demo)
                {
                    EXPECT_DEATH(Foo(), "");
                }
                编写死亡测试案例时，TEST 的第一个参数，即 testcase_name，请使用 DeathTest 后缀。原因是 gtest 会优先运行
                死亡测试案例，应该是为线程安全考虑
                
    3. *_EXIT(statement, predicate, regex)
        (1) 
            1. statement是被测试的代码语句
            2. predicate 在这里必须是一个委托，接收 int 型参数，并返回 bool。只有当返回值为 true 时，死亡测试案例才算通过。
               gtest 提供了一些常用的 predicate：
                    testing::ExitedWithCode(exit_code) : 如果程序正常退出并且退出码与 exit_code 相同则返回 true
                    testing::KilledBySignal(signal_number) : 如果程序被 signal_number 信号 kill 的话就返回 true
                    
            3. *_DEATH 其实是对 *_EXIT 进行的一次包装, *_DEATH 的 predicate 判断进程是否以非 0 退出码退出或被一个信号杀死
                TEST(ExitDeathTest, Demo)
                {
                    EXPECT_EXIT(_exit(1),  testing::ExitedWithCode(1),  "");
                }
                
```

## 运行参数
```shell
    1. 设置运行参数的途径
            (1) 系统环境变量
            (2) 命令行参数
            (3) 代码中指定 FLAG
    2. 生效的优先级是，最后设置的那个会生效, 但最好采用 命令行参数 > 代码中指定 FLAG > 系统环境变量
    3. 如果需要在代码中指定 FLAG, 可以使用 testing::GTEST_FLAG 这个宏来设置。比如命令行参数 --gtest_output = xml,
       代码中指定 FLAG 使用 testing::GTEST_FLAG(output) = "xml:";不需要加 --gtest 前缀。推荐将这句代码中指定 FLAG放置
       InitGoogleTest 之前, 这样就可以使得对于同样的参数，命令行参数优先级高于代码中指定
            int _tmain(int argc, _TCHAR* argv[])
            {
                testing::GTEST_FLAG(output) = "xml:";
                testing::InitGoogleTest(&argc, argv);
                return RUN_ALL_TESTS();
            }
            
    4. 系统环境变量使用
            如果需要 gtest 的设置系统环境变量, 必须注意的是：
            1. 系统环境变量全大写，比如对于--gtest_output，响应的系统环境变量为：GTEST_OUTPUT
            2.  有一个命令行参数例外，那就是--gtest_list_tests，它是不接受系统环境变量的。（只是用来罗列测试案例名称）
    5. 一般是使用命令行参数
    
         a. 测试案例集合:  
            > output/x86_test --gtest_list_tests   // 只列出 case 个数
            结果:
                  Running main() from gtest_main.cc
                  FactorialTest.
                    Negative
                    Zero
                    Positive
                  IsPrimeTest.
                    Negative
                    Trivial
                    Positive
            (1) --gtest_list_tests : 使用这个参数时, 将不会执行里面的测试案例，而是输出一个案例的列表
            (2) --gtest_filter: 对执行的测试案例进行过滤，支持通配符
                                ?    单个字符
                                *    任意字符
                                -    排除，如，-a 表示除了a
                                :    取或，如，a:b 表示a或b
                             
                 > ./foo_test --gtest_filter = FooTest.* 运行所有“测试案例名称(testcase_name)”为 FooTest 的案例
                 > ./foo_test --gtest_filter=-*DeathTest.* 运行所有非死亡测试案例
                 > ./foo_test --gtest_filter=FooTest.*-FooTest.Bar 运行所有“测试案例名称(testcase_name)”为 FooTest 的案例,
                                                                   但是除了 FooTest.Bar 这个案例
                                                                   
            (3) --gtest_also_run_disabled_tests: 执行案例时，同时也执行被置为无效的测试案例
                    关于设置测试案例无效的方法为：在测试案例名称或测试名称中添加 DISABLED 前缀，比如
                    // Tests that Foo does Abc.
                    TEST(FooTest, DISABLED_DoesAbc) {  }
                    
                    class DISABLED_BarTest : public testing::Test {  };
                    // Tests that Bar does Xyz.
                    TEST_F(DISABLED_BarTest, DoesXyz) {  }
                    
            (4) --gtest_repeat=[COUNT]
                    设置案例重复运行次数,比如：
                    --gtest_repeat=1000      重复执行1000次，即使中途出现错误。
                    --gtest_repeat=-1        无限次数执行。。。。
                    --gtest_repeat=1000 --gtest_break_on_failure  重复执行1000次，并且在第一个错误发生时立即停止
                                                                  这个功能对调试非常有用。
                    --gtest_repeat=1000 --gtest_filter=FooBar     重复执行 1000 次测试案例名称为 FooBar 的案例
                
        b. 测试案例输出:
                --gtest_color=(yes|no|auto) : 输出命令行时是否使用一些五颜六色的颜色,默认是 auto
                --gtest_print_time: 输出命令行时是否打印每个测试案例的执行时间,默认是打印的
                --gtest_output=xml[:可以输出相对路劲] :  不指定输出路径时，默认为案例当前路径
                        > output/x86_test --gtest_output=xml:output/    (output/ 是相对于执行可执行文件的路劲而言)
                        > output/x86_test --gtest_output=xml:output/file.xml
        c. 对案例的异常处理
                --gtest_break_on_failure : 当案例失败时停止，方便调试
                --gtest_throw_on_failure : 当案例失败时以 C++ 异常的方式抛出
                --gtest_catch_exceptions : windows 才用到，防止异常窗口弹出.
```

## 总结

```shell
    1. EXPECT_*  失败时，案例继续往下执行. ASSERT_* 失败时，直接在当前函数中返回，当前函数中 ASSERT_* 后面的语句将不会执行
    2. gtest.h 文件中包含　EXPECT_EQ() 等宏定义.
    
    3. TEST 有 2 个参数, 第一个是 test case name(就是一个函数), 第二个是 test name(在这个函数不同情况 case)
        例如:
            TEST(FactorialTest, Negative){}
            TEST(FactorialTest, Zero) {}
            TEST(IsPrimeTest, Negative){}
       注意:
            1. 这 2 个参数的命名符合变量命名，并且不要用 "_" 字符．
            2. 2 个 TEST(...., ....) 之间不能保证顺序(谁先谁后), 能保证是都执行.
            
    4. 用　ASSERT_* 方便调试，因为失败了会将失败的信息打印出来．
        例如:
            ASSERT_EQ(0, Factorial(-1));
            打印信息:
                      Expected: 0
                To be equal to: Factorial(-1)
                      Which is: 1
                      
    5.  UnitTest 类负责所有测试案例的执行,管理
    
```

# gmock

## 简要

```shell
    1. Mock Objects (模拟对象)来模拟程序的一部分来构造测试场景。Mock Object 模拟了实际对象的接口，通过一些简单的代码模拟实际对象部分
       的逻辑, 实现起来简单很多. 通过 Mock object 的方式可以更好的提升项目的模块化程度, 隔离不同的程序逻辑或环境。
    2. 在实际工作中, 一个人不可能完成整条线的开发工作, 于是我们会在约定接口的前提下，各自完成各自的模块。自己的模块开发完之后，
      需要自测, 但是这个时候别人的模块可能还没完成，这时我们就可以定义 Mock 对象来模拟那些类的行为,
      就是自己实现一个假的依赖类，对这个类的方法你想要什么行为就可以有什么行为，你想让这个方法返回什么结果就可以返回怎么样的结果,
      同时 gmock 可以帮助比较方便、比较轻松地实现这些个假的依赖类.
      那么我们就需要模拟约定的接口进行自测, Gmock 就是一个强大的模拟接口的工具。
    3. 特性
            (1) 轻松地创建 mock 类
            (2) 支持丰富的匹配器（Matcher）和行为（Action）
            (3) 支持有序、无序、部分有序的期望行为的定义
            (4) 多平台的支持
```

## 具体流程
```shell
    0. 需要测试 CSubEventHandler::handleRead 的业务逻辑, 同时也想测试 CSubscriber 方法里的协议解析逻辑,,
       但是对于 CSubscriber 封装的读写包部分，希望可以 mock 成我想要的网络包
    1. 先 mock 一个 CSubscriber 类
            
        class MockCSubscriber : public CSubscriber
        {
        public:
            MockCSubscriber(int fd):CSubscriber(fd){}
            MOCK_METHOD1(readBuf, int(int len));
            MOCK_METHOD1(writeBuf, int(int len));
            MOCK_METHOD0(closeSock, void());
        };
        
        MOCK_METHOD#1(#2, #3(#4) )
        
        #1表示要 mock 的方法共有几个参数, #2是要 mock 的方法名称 #3表示这个方法的返回值类型, #4是这个方法具体的参数
     
        以下操作是 在模拟对象上设置你的预期(它们怎样被调用,应该怎样回应?).
    2. 如果只关心 mock 方法的返回值可以使用宏 ON_CALL.
            ON_CALL(subObj, readBuf(1000)).WillByDefault(Return(blen));
            
            ON_CALL(#1, #2(#3)).WillByDefault(Return(#4));
            
            #1表示 mock 对象。对 CSubscriber 定义了一个 Mock 类，那么就必须生成相应的 mock 对象，例如：
                MockCSubscriber subObj(5);
            
            #2表示想定义的那个方法名称。上例中我想定义 readBuf 这个方法的返回值。
            #3表示 readBuf 方法的参数。这里的 1000 表示，只有调用 CSubscriber::readBuf 同时传递参数为 1000 时，
              才会用到 ON_CALL 的定义。
            #4表示调用 CSubscriber::readBuf 同时传递参数为 1000时 ，返回值是 blen 。
            
    3. 自定义方法/成员函数的期望行为
            在单元测试/主程序中使用 Mock 类中的方法的时候最关键的就是对期望行为的定义
            EXPECT_CALL(mock_object, method(matcher1, matcher2, ...))  // mock_object 是 Mock 类的对象, matcher(匹配器)定义方法参数的类型
                .With(multi_argument_matcher)
                .Times(cardinality)
                .InSequence(sequences)  // 定义这个方法被执行顺序（优先级）
                .After(expectations)
                .WillOnce(action)  // 定义一次调用所产生的行为，比如定义该方法返回怎么样的值等等
                .WillRepeatedly(action) // 重复行为
                .RetiresOnSaturation();
                
            (1) Matcher（匹配器）: Matcher 用于定义 Mock 类中的方法的形参的值(你的方法不需要形参时，可以保持 match 为空)
                    a. 通配符
                            _	可以代表任意类型
                            A() or An()	可以是 type 类型的任意值
                    b. 一般比较
                            Eq(value) 或者 value:  argument == value，method 中的形参必须是 value
                            Ge(value)	          argument >= value，method 中的形参必须大于等于 value
                            Gt(value)	          argument > value
                            Le(value)	          argument <= value
                            Lt(value)	          argument < value
                            Ne(value)	          argument != value
                            IsNull()	          method 的形参必须是 NULL 指针
                            NotNull()	          argument is a non-null pointer
                            Ref(variable)	      形参是 variable 的引用
                            TypedEq(value)	      形参的类型必须是 type 类型，而且值必须是value
                    c. 字符串匹配
                            ContainsRegex(string)	形参匹配给定的正则表达式
                            EndsWith(suffix)	    形参以 suffix 截尾
                            HasSubstr(string)	    形参有 string 这个子串
                            MatchesRegex(string)	从第一个字符到最后一个字符都完全匹配给定的正则表达式.
                            StartsWith(prefix)	    形参以 prefix 开始
                            StrCaseEq(string)	    参数等于 string，并且忽略大小写
                            StrCaseNe(string)	    参数不是 string，并且忽略大小写
                            StrEq(string)	        参数等于 string
                            StrNe(string)	        参数不等于 string
                    d. 容器的匹配(很多 STL 的容器的比较都支持 == 这样的操作，对于这样的容器可以使用上述的 Eq(container) 来比较)
                            Contains(e)	    : 在 method 的 形参中，只要有其中一个元素等于 e
                            Each(e)	        : 参数各个元素都等于 e
                            ElementsAre(e0, e1, …, en)	: 形参有 n + 1 的元素，并且挨个匹配
                            ElementsAreArray(array) 或者 ElementsAreArray(array, count)	: 和 ElementsAre()类似, 除了预期值/匹配器来源于一个C风格数组
                            ContainerEq(container)	   : 类型 Eq(container), 就是输出结果有点不一样，这里输出结果会带上哪些个元素不被包含在另一个容器中
                     
                    示例:        
                            string value = "Hello World!";
                            MockFoo mockFoo;
                    
                            EXPECT_CALL(mockFoo, setValue(testing::_));  // 让 setValue 的形参可以传入任意参数
                            mockFoo.setValue(value);  // MockFoo 中的方法
                            
                    e. 成员匹配器
                            (1) Field(&class::field, m)	
                                    argument.field (或 argument->field, 当argument是一个指针时)与匹配器 m 匹配(例如匹配器Ge(0)),
                                    这里的 argument 是一个 class 类的实例.
                            (2) Key(e)	
                                    形参(argument) 比较 是一个类似 map 这样的容器，然后 argument.first 的值等于 e
                            (3) Pair(m1, m2)	
                                    形参(argument) 必须是一个 pair，并且 argument.first 等于 m1, argument.second等于m2.
                            (4) Property(&class::property, m)
                            	    argument.property()(或argument->property(),当argument是一个指针时)与匹配器 m 匹配, 
                            	    这里的 argument 是一个 class 类的实例.
                            示例:
                                   TEST(TestField, Simple) {
                                           MockFoo mockFoo;
                                           Bar bar;
                                           // 定义了一个 Field(&Bar::num, Ge(0))，以说明 Bar 的成员变量 num 必须大于等于 0
                                           EXPECT_CALL(mockFoo, get(Field(&Bar::num, Ge(0)))).Times(1);
                                           mockFoo.get(bar);
                                   }
                                   
                                   int main(int argc, char** argv) {
                                           ::testing::InitGoogleMock(&argc, argv);
                                           return RUN_ALL_TESTS();
                                   }
                    f. 匹配函数或函数对象的返回值
                            (1) ResultOf(f, m)
                                    f(argument) 与匹配器 m 匹配, 这里的 f 是一个函数或函数对象, argument 代表调用时传入参数.
                    g. 指针匹配器
                            (1) Pointee(m)	
                                    argument (不论是智能指针还是原始指针) 指向的值与匹配器 m 匹配.
                    h. 复合匹配器
                            (1) AllOf(m1, m2, …, mn)	
                                    argument 匹配所有的匹配器 m1 到 mn
                            (2) AnyOf(m1, m2, …, mn)	
                                    argument 至少匹配 m1 到 mn 中的一个
                            (3) Not(m)
                            	    argument 不与匹配器 m 匹配
                            示例:
                                // 传入的参数必须 >5 并且 <= 10
                                EXPECT_CALL(foo, DoThis(AllOf(Gt(5), Ne(10))));
                                // 第一个参数不包含“blah”这个子串
                                EXPECT_CALL(foo, DoThat(Not(HasSubstr("blah")), NULL));
                                
            (2) 基数(Cardinalities): 用于 Times() 中来指定模拟函数将被调用多少次
                    AnyNumber()	 : 函数可以被调用任意次.
                    AtLeast(n)	 : 预计至少调用 n 次.
                    AtMost(n)	 : 预计至多调用 n 次.
                    Between(m, n)	: 预计调用次数在 m 和 n (包括n) 之间.
                    Exactly(n) 或 n	: 预计精确调用 n 次. 当 n 为 0 时,函数应该永远不被调用.
                    
            (3) 行为(Actions) : 用于指定 Mock 类的方法所期望模拟的行为：比如返回什么样的值、对引用、指针赋上怎么样个值，等等
                    a. 定义值的返回
                            Return()	　　　: 让 Mock 方法返回一个 void 结果
                            Return(value)	 : 返回值 value
                            ReturnNull()	 : 返回一个 NULL 指针
                            ReturnRef(variable)	 : 返回 variable 的引用.
                            ReturnPointee(ptr)	 : 返回一个指向 ptr 的指针
                            
                    b. Assign(&variable, value)	将 value 分配给 variable
                    c. 使用函数或者函数对象（Functor）作为行为
                            (1) Invoke(f)	
                                    使用模拟函数的参数调用 f, 这里的 f 可以是全局/静态函数或函数对象.
                            (2) Invoke(object_pointer, &class::method)	
                                    使用模拟函数的参数调用 object_pointer 对象的 mothod 方法.
                    d. 复合动作
                            (1) DoAll(a1, a2, …, an)
                            	    每次发动时执行 a1 到 an 的所有动作.
                            (2) IgnoreResult(a)	
                                    执行动作 a 并忽略它的返回值. a 不能返回 void.
                                    
                    示例
                        TEST(SimpleTest, F1) {
                            std::string* a = new std::string("yes");
                            std::string* b = new std::string("hello");
                            MockIParameter mockIParameter;
                            EXPECT_CALL(mockIParameter, getParamter(testing::_, testing::_)).Times(1).\
                                WillOnce(testing::DoAll(testing::Assign(&a, b), testing::Return(1)));
                            mockIParameter.getParamter(a, b);
                        }
                        
            (4) 序列(Sequences)
                    (1) 默认情况下, 定义要的期望行为是无序（Unordered）的, 例如:
                             MockFoo mockFoo;
                             EXPECT_CALL(mockFoo, getSize()).WillOnce(Return(1));
                             EXPECT_CALL(mockFoo, getValue()).WillOnce(Return(string("Hello World")));
                             
                             进行上面 EXPECT_CALL() 调用, getSize() 调用和 getValue() 的调用没有先后区别
                       
                        
                    (2) 有时候我们需要定义有序的(Ordered)的调用方式(比如在调用 getValue() 时要先调用 getSize()),
                        即序列 (Sequences) 指定预期的顺序. 在同一序列里的所有预期调用必须按它们指定的顺序发生.
                        
                        还可以根据我定义期望行为(EXPECT_CALL)的顺序而自动地识别调用顺序，这种方式可能更为地通用
                        
                        using ::testing::InSequence;
                                InSequence dummy;
                                MockFoo mockFoo;
                                EXPECT_CALL(mockFoo, getSize()).WillOnce(Return(1));
                                EXPECT_CALL(mockFoo, getValue()).WillOnce(Return(string("Hello World")));
                        
                                cout << "First:\t" << mockFoo.getSize() << endl;
                                cout << "Second:\t" << mockFoo.getValue() << endl;
                                
                                这样的话,只有 getSize() 先调用, getValue() 后调用才正确.
                            
                        
       
       示例:
            EXPECT_CALL(subObj, readBuf(1000)).Times(1);
       
            最后的 Times 表示，只希望 readBuf 在传递参数为 1000 时，被调用且仅被调用一次。
       
            其实这些宏有很复杂的用法的，例如：
       
            EXPECT_CALL(subObj, readBuf(1000))
               .Times(5)
               .WillOnce(Return(100))
               .WillOnce(Return(150))
               .WillRepeatedly(Return(200));　　
       
            表示 readBuf 希望被调用五次，期望第一次返回值为 100，期望第二次返回 150，期望后三次返 回200。如果不满足，会报错
            Return(value) 表示返回值设置为 value
            
            MockFoo mockFoo;
            EXPECT_CALL(mockFoo, getArbitraryString()).Times(1).
                    WillOnce(Return(value));
            EXPECT_CALL 为　MockFoo 的 getArbitraryString()方法定义一个期望行为，其中 Times(1) 的意思是运行一次，
            WillOnce(Return(value))的意思是第一次运行时把 value 作为 getArbitraryString() 方法的返回值
      

```

# 参考资料

```shell
    1. https://www.cnblogs.com/coderzh/archive/2009/04/06/1430396.html
    2. https://www.cnblogs.com/welkinwalker/archive/2011/11/29/2267225.html
```