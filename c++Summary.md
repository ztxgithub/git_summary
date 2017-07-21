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