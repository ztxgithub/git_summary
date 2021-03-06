# c++用法

- 字符串的截取

``` c++

	istringstream parse(str);
	string token;

	for(int i = 0; getline(parse, token, ',') && i < this->cell_number; i++ ) {
        this->cell_capacity[i] = (uint32_t) atol(token.c_str());
    }
	
	截取完成条件：当parse都截取完了，parse内容会改变，token每次调用会被替换
	
```

- switch-case,case下面一定要加{}

``` c++

      case FAST: // 冲放电事件情况下
            {
                fastSampleSend();
                loopSampleData();
                bool isEvent = BatMgrCheckAlarmAndEvent();
                if (isEvent) {
                    changeState(FAST);
                } else if (this->sampleState == FAST) {
                    changeState(SLOW);
                }
            }
	
```

- 类型转化要进行强制转换

``` c++

	int64_t cap = this->temp_used_capacity;
	int32_t current = data.getCurrent();
	uint32_t diff_in_ms = timeDiffInMS(&this->pre_time, current_time);
	cap += ((int64_t)current * (int64_t)diff_in_ms);
	
	注意：如果没有强制转换：cap += (current * diff_in_ms);会溢出
	
```

- list::erase

``` c++

	iterator erase (const_iterator position);
	
	iterator erase (const_iterator first, const_iterator last);
	删除的范围是 [first,last)，不包括last
	
	返回值：指向下一个可用的iterator
	
```

- vector 内存释放

``` c++

	1.当 vector 定义为局部变量时，不需要使用swap
	2.当 vector 定义为static 变量时，要使用
	string_vector.clear();
	vector<string>().swap(string_vector);
	
```

- 变量定义的时候才能用{0,0}之类的

``` c++

	typedef struct strSerialTime{
		uint16_t serial;
		uint16_t num;
	}SerialTime;
	
	SerialTime lstSerialTime = {0, 0};  //初始化可以用这种方式赋值
	
```

- 调试打印**太多**会占用程序执行的时间

- C++类的对象中开启多个线程

``` c++

	void *handleSmplDataQueueProxy(void *data) {
		SampleBattery *h = static_cast<SampleBattery*>(data);
		h->handleSmplDataQueue();
	}
	
	void SampleBattery::Start() {
		pthread_t thSmplDataQueue;
		pthread_create(&thSmplDataQueue, 0, handleSmplDataQueueProxy, static_cast<void*>(this));

	} 
	
```

- 如果.cpp 文件中要用到 .c中的函数

``` c++
    
    在.cpp 中
   extern "C" 
    {
        #include"test.h"
    }
	
```
- 当一个结构体的字节存放在string 中时,一定要考虑到显式的构造大小(默认按/0结尾)

``` c++
    
    mqtt_content.payload = string((char*)message->payload, message->payloadlen);
	
```

- 段错误不能用try,catch来捕获

- STL 特性
``` c++
    
    1.不管是list,map,vector 都可以重复删除相同的键名
        map<int, int > repeat_test;
        repeat_test[1] = 1;
        repeat_test.erase(1);
        repeat_test.erase(1);
        
    2.可以erase(删除)空值
        map<int, int > repeat_test;
        repeat_test.erase("");
        
    3.可以erase(删除)未存在的值
	     map<int, int > repeat_test;
         repeat_test.erase("11111");
       
```

- 临时变量的使用

``` c++

    string foo( );
    void bar(string&s)

    bar(foo( ));  //错误
    bar("hello world");  //错误
    
    foo( )和"hello world"字符串都会产生一个临时对象,而在C++中,这些临时对象都是const类型的.
	
```

- goto编译错误

``` c++

    编译时出现 erro: jump to label xxx
    主要原因是 在 第一个goto以后还有其他变量的定义.
    解决方法: 将所有变量的定义放在 第一个goto 的前面
	
```

- 当函数需要内存保存相关的值

```shell
    1.推荐使用
      　　让用户传入一块他自己的内存地址(buff和buff_length),而在函数中把要返回的内存放到这块内存中,
      　　好处就是由函数外部的程序来维护这块内存，比较简显直观
      
    2.不推荐使用
            (1) 在函数内部通过malloc或new在堆上分配内存,然后把这块内存返回(因为在堆上分配的内存是全局可见的).
                 这样带来的问题就是潜在的内存问题。因为，如果返回出去的内存不释放，那么就是memory Leak。
                 或者是被多次释放，从而造成程序的crash
                 
            (2) 利用了static的特性,static的栈内存一旦分配,那这块内存不会随着函数的返回而释放，而且，
                 它是全局可见的（只要你有这块内存的地址）.例如　char *inet_ntoa(struct in_addr in);
                 它存在的问题是　fprintf(fp, "源IP地址<%s>/t目的IP地址<%s>/n", inet_ntoa(src),   inet_ntoa(des));
                 其static 的内容会被覆盖,则2个值一样,
                 而且　if ( strcmp( inet_ntoa(ip1), inet_ntoa(ip2) )==0 ) {　// strcmp 返回值永远为 0
                    …. ….
                    }　
                    
                 还会导致线程不安全.
```

- 外部访问　类的枚举数据

```shell
    class Functor
    {
    
        public:
            enum Type{Plus,Sub};

    };
    
    cout << Functor<int>::Sub << endl;

```