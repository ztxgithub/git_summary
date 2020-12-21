# c++ stl

## 字符串使用

```shell
1. 将字符串中某个 subString 替换为其他的字符串 replaceStr
newString = std::regex_replace(srcString, 
std::regex(substring), replaceStr);
2. 源字符串 str 剔除掉 subString, 拿到剩下的字符串 remainStr
std::string::size_type index = str.find(subString);
if (index == std::string::npos)
continue;
str.erase(index, subString.length());
这个时候 str 就是 remainStr

    3. string substr (size_t pos = 0, size_t len = npos)
    
         示例代码:
             std::string str="We think in generalities, but we live in details.";
                                                       // (quoting Alfred N. Whitehead)
             std::string str2 = str.substr (3,5);     // "think"
             std::size_t pos = str.find("live");      // position of "live" in str
             std::string str3 = str.substr (pos);     // (输出 "live in details")get from "live" to the end
             
    4. https://blog.csdn.net/jamesfancy/article/details/1543338
    5. 删除 string 中所有相同字符
        src_str.erase(std::remove(src_str.begin(), src_str.end(), '+'), src_str.end());
        
        其中　
       　　 auto iter = std::remove(src_str.begin(), src_str.end(), '+') 
           原理是查找 string 中所有等于字符'+', 再将后面第一个不等于字符'+', 替换掉字符'+', 最后返回新的 new_end 迭代器
           new_end 之前是有效数据, new_end ~ string.end() 是无效数据.
```

## 迭代器使用

```shell
1. 使用下标的方式访问标准容器
(1) auto it = my_map.begin();    // std::map has bidirectional iterators
std::advance(it, 4);         // for (auto i = 0; i < 4; ++i) ++it;

这个时候 advance 返回 void 类型，同时想要再次使用 advance 则 it 需要重新  my_map.begin(); 
(2) auto it = my_map.begin();
auto it4 = std::next(it, 4); // copies it, then advances and returns that copy
    2. 对于 vector 而言, erase 返回是下个迭代器
            iterator erase(iterator position);  // 返回下一个迭代器
            iterator erase(iterator first, iterator last);
       https://www.cnblogs.com/blueoverflow/p/4923523.html
    3. 对于关联容器(如map, set, multimap,multiset)，删除当前的iterator，仅仅会使当前的iterator失效，只要在erase时，
       递增当前iterator即可。这是因为map之类的容器，使用了红黑树来实现，插入、删除一个结点不会对其他结点造成影响
       
       对于序列式容器(如vector,deque)，删除当前的 iterator 会使后面所有元素的 iterator 都失效。这是因为 vetor,deque使用了
       连续分配的内存，删除一个元素导致后面所有的元素会向前移动一个位置。还好 erase 方法可以返回下一个有效的 iterator
```

# C 
```shell
    1. 对于字符数组,只打印指定的长度
        printf("server  message: %.*s\n",
                (int)m->reply_text.len, (char *)m->reply_text.bytes);
```

## c++ 用法
```shell
    1. 基类的析构函数应当为虚,
```

## 工具类

```shell
    1. string to int/long/double
    
     template <class Type>
     Type stringToNum(const string& str) {
     istringstream iss(str);
     Type num;
     iss >> num;
     return num;
     }
     
        long long lvalue = 0;
        string sValue = "1592197060000";
        lvalue = stringToNum<long long>(sValue);
        printf("lvalue[%lld]\n", lvalue * 1000);
        
    2. float to string
        // 保留 2 位有效小数
        std::string result;
        stringstream ss;
        ss << std::setiosflags(std::ios::fixed) << setprecision(2) << 13.666666666;
        ss >> result;
        
    3. 以指定的进制显示
                std::cout << "255(2进制): " << std::setbase(2) << 255 << std::endl;//二进制输出
                std::cout << "255(8进制): " << std::setbase(8) << 255 << std::endl;//八进制输出
                std::cout << "255(16进制): " << std::setbase(16) << 255 << std::endl;//十六进制输出
```

# c++ stl 解析

## 概要

```shell
1. STL 的实现版本有很多，出名的是 SGI STL (Silicon Graphics Computer System  Inc)
```

## STL 概要

```shell
    1. STL 不仅仅只是一个工具库，它更是一个抽象的概念库，
        有第一层(基础): Assignable, Default Constructible, Equality Comparable, LessThan Compareable, Regular
        第二层: Input Iterator, Output Iterator, Forward Iterator(单向迭代器), Bidirectional Iterator(双向迭代器)
               Random Access Iterator, Unary Function(一元函数), Binary Function(二元函数)
        第三层: sequence container(例如 vector), associative container(例如 map) 
        
    2. STL 是形成一套标准接口, 任何组件都有最大的独立性, 可以和 Iterator 结合, 也可以和 adapter 配接,
       也可以和仿函数(functor) 动态选择某种策略(policy 或则 strategy)
    3. STL 提供六大组件，彼此可以组合套用
            (1) 容器(Containers), 各种数据结构，存放数据, 例如 vector, list, map, 从实现角度看, stl 容器是一种 class template
            (2) 算法(algorithms), 例如: sort, search, erase, stl 算法是一种 function template
            (3) 迭代器(iterators), 是所谓的泛型指针，它是一个 class template, 是将 operator*, operator->, operator++,
                                  operator-- 等指针操作予以重载．
            (4) 仿函数(functors), 可作为算法的某种策略(policy), 仿函数是一种重载了 operator() 的 class
            (5) 适配器(adatpers), 一种用来修饰容器, 仿函数，迭代器接口的．例如 stack 和 queue 不是一个容器, 其实是一个适配器,
                                它们的底层操作完全借助与 deque, 其称之为 container adapters
            (6) 配置器(allocators), 主要组件，复杂内存空间的配置和管理．从实现上来看, 配置器就是一个实现动态空间配置, 空间管理,
                                   空间释放的 class template
    4. SGI STL 文件分布
            (1) STL 标准头文件(无扩展名 .h), 例如 algorithm, 这是对外接口文件, 真正实现的是 stl_algorithm.h
            (2) SGI STL 内部 private 使用文件, 例如 stl_vector.h 真正实现的是这个．
    5. 专业名词
            (1) argument deduction(template 参数推导), partial specialization(偏特化)
    6. c++ 语法
            (1) class template 是否支持 static member data, 主要是看编译器是否支持, gcc 和 vc6 支持, CB4 不支持.
            (2) __STL_CLASS_PARTIAL_SPECIALIZATION: 编译器是否支持偏特化, gcc 支持
            (3) __STL_MEMBER_TEMPLATES: class template 内可否再有 template (members)
            (4) __STL_LIMITED_DEFAULT_TEMPLATES
            (5) __STL_TEMPLATE_NULL : 将这个宏定义为 template <>, 主要用于模板的特化, 使用最好不用这个，阅读使用
    7. 临时对象产生
            (1) 临时对象产生方法： 只需要在类别加上(), 并指定初值.
            (2) 适用场景
                    a. 应用于仿函数(functor)与算法的搭配上.
                            template <typename T>
                            class print{
                            public:
                                void operator()(const T& elem){cout << elem << endl;}
                            };
                            
                            void fPrint(const int& elem)
                            {
                                cout << elem << endl;
                            }
                            
                            int main(int argc, char const *const *argv) {
                            
                                int ia[6] = {0, 1, 2, 3, 4, 5};
                                vector<int> iv(ia, ia + 6);
                                for_each(iv.begin(), iv.end(), print<int>());
                                for_each(iv.begin(), iv.end(), fPrint);
                                return 0;
                            }
                            
    8. 静态常量整数成员在定义的时候可以直接初始化
            template <typename T>
            class testClass{
            public:
                static const int datai_ = 5;   // 必须要 const 
                static const long datal_ = 3L;
                static const char datac_ = 'c';
            };
            
            cout << testClass<int>::datai_<< endl;
            cout << testClass<int>::datal_<< endl;
            cout << testClass<int>::datac_<< endl;
                        
    9. operator++ (前置和后置), dereference(引用)
          (1) 前置和后置 ++ 区别在于参数个数和类型，C++ 规定后缀形式有一个 int 类型参数，当函数被调用时，
              编译器传递一个 0 做为 int 参数的值给该函数
          (2) 这些操作符前缀与后缀形式返回值类型是不同的,前缀形式返回一个引用，后缀形式返回一个 const 类型
              后缀形式返回一个 const 类型
                        原因一: 要与内置类型行为(int i, i 是无法 i++++)一致, 
              　　　　　 原因二: 使用两次后缀 increment 所产生的结果与调用者期望的不一致。第二次调用 operator++ 改变的值是
                              第一次调用返回对象的值，而不是原始对象的值
          (3) 使用后缀++ 会产生临时变量，前置 ++ 则不会，如果想要提高效率则优先使用前置++
          (4) 操作符一般都要加 (), 例如 operator*() 等等．
            class INT{
                friend ostream& operator << (ostream& os, const INT& i);
            public :
                INT(int i):m_i(i){}
                // 先 ++ , 再返回, 前置 ++
                INT& operator++()
                {
                    ++(this->m_i);
                    return *this;
                }
            
                // postfix : fetch and then increment, 后置 ++
                // 如果你没有在函数里使用参数, 许多编译器会显示警告信息，为了避免这些警告信息,
                // 一种经常使用的方法时省略掉你不想使用的参数名称
                const INT operator++(int) // 
                {
                    INT tmp = *this;
                    ++(*this);
                    return tmp;
                }
            
                //引用
                int& operator*() const
                {
                    return (int&) m_i;
                }
            
            private:
                int m_i;
            };
            
            ostream& operator << (ostream& os, const INT& i)
            {
                os << '[' << i.m_i << ']';
                return os;
            }
            
            INT I(5);
            cout << I++ << endl;  // I.operator++();
            cout << ++I << endl;  // I.operator++(0);
            cout << *I << endl;    
              
    10. [first, last) : 前闭后开表示法
            (1) 任何一个 stl 算法，都需要由一对迭代器所标示的区间来标示操作范围. 其中 last 表示最后一个元素下一个.
    11. 所谓的仿函数(functor) 就是使用起来像函数一样的东西
            template<class T>
            struct Plus{
                T operator()(const T& x, const T& y) const  // 重载函数调用符 operator()
                {
                    return x + y;
                }
            };
            
            Plus<int> plusObj;
            cout << plusObj(3, 4) << endl;  // 输出 7, 使用函数调用符
            cout << Plus<int>()(3, 4) << endl;  // 输出 7 , 第一个 () 是产生临时变量, 第二个是函数调用 ()
```

## allocator (空间配置器)
```shell
    1. 对于 GCC 的 SGI 的空间配置器与标准规范不一致, 其名称为 alloc 而不是 allocator, 而且不接受任何参数
        例如:
            vector<int, std::alloc> iv;
    2. 
```
