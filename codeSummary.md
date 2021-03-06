# 项目过程要用到的编码技巧

- 当要进行批量的转化一组数据时

``` c++

enum CurrentProbeRangeBDM {
    CURRENT_PROBE_RANGE_UNSET = 0,
    CURRENT_PROBE_RANGE_50A = 1,
    CURRENT_PROBE_RANGE_100A = 2,
    CURRENT_PROBE_RANGE_200A = 3,
    CURRENT_PROBE_RANGE_500A = 4,
};

BDMIter.currentProbeRange = BDM2currentProbeRange((CurrentProbeRangeBDM) range);

inline uint32_t BDM2currentProbeRange(CurrentProbeRangeBDM currentProbeRange) {
    switch (currentProbeRange) {
        case CURRENT_PROBE_RANGE_50A:
            return 50;
        case CURRENT_PROBE_RANGE_100A:
            return 100;
        case CURRENT_PROBE_RANGE_200A:
            return 200;
        case CURRENT_PROBE_RANGE_500A:
            return 500;
        default:
            log_w("bdm probeRange %d is UNKNOWN", currentProbeRange);
    }
    return 0;
}

```

- 创建ADT（抽象数据类型），将一个抽象具有的数据和操作放在一个文件内

- 在定义一个结构体时，不知道该结构体内有多少数据

``` c++

	struct EventData {
		EventDataHeader header;
		EventDataBody body[0];   //定义一个大小为0的数据结构
	};

	
```

- C++类的释放最好有析构函数，这样释放申请的内存空间就不用太复杂

``` c++

	delete group;  //自动调用 ~BatteryGroup()的析构函数
	
	BatteryGroup::~BatteryGroup(){
		log_i("~BatteryGroup");
		if(this->alarmSignal) {
			delete this->alarmSignal;
			this->alarmSignal = NULL;
		}
		if(this->valueSignal) {
			delete this->valueSignal;
			this->valueSignal = NULL;
		}
		if(this->capacity) {
			delete this->capacity;
			capacity = NULL;
		}
	}

	delete this->alarmSignal;  //而它又自主得调用~BatteryAlarmSignal()析构函数
	之后一层一层往下调

	
```

- C++声明函数的时候可以在形参上有一个默认值

``` c++

	static string U64ToString(uint64_t value, size_t length = 0, int frombase=10);
  
    string sid = TypeTransfer::U64ToString(*iterSucId, 17);  //一定要按顺序赋值，其实是 value==*iterSucId， length==17， frombase==10
	
	string sid = TypeTransfer::U64ToString(*iterSucId);		
	
```
- 宏定义的表示方式

``` c++

        #define DOSOMETHING() \
                do{ \
                  foo1();\
                  foo2();\
                }while(0)\
        	
```
- 循环最好用for

``` c++

        for(int i = 0; i < 10; i++) {
            printf("i[%d]\n", i);
            if(erro)
                continue;
                
            do something;
        }

        continue之后i++也会执行,用while则想要继续循环下去还得写一次i++
        while(i < 10)
        {
            if(erro)
            {
                i++;
                continue;
            }
            do something;
            i++
        }
        	
```

- 对文件写内容

``` c++
    比较安全的做法是:　先对filename.tmp进行写文件,并fsync(fd),在rename(filename.tmp,filename)
        	
```

- 对已经打开文件清空或则 从文件头重新写

``` c++
    用ftruncate可以清空文件，如：
                             (1) FILE *fp = fopen("/etc/passwd", "r");
                                 int fd = fileno(fp);
                                 ftruncate(fileno(fp), 0);   
                                     或则 
                             (2) ftruncate(open(filename, O_RDWR | O_CREAT, 0644)), 0)
                              
                             清空文件后，需要使用rewind或fseek或则lseek将文件指针移到文件头
        	
```

- 获取文件大小

``` c++
    1.  file_size = lseek(fd, 0, SEEK_END)
    2.  stat(const char *file_name, struct stat *buf);
        	
```

- 回调函数

``` c++
    项目的组织结构是A文件调用B文件里的函数,如果B文件中有可能要用到A中的资源变量(A中的函数逻辑),则可考虑回调函数
    
    A文件
        static void modbus_connect_accept_handler(struct modbus_connect *connect)
        {
            
        }
        
        modbus_set_accept_handler(modbus_connect_accept_handler);
        
    B文件
    
        static void (*modbus_connect_accept)(struct modbus_connect *); //函数指针
    
        void modbus_set_accept_handler(void (*handler)(struct modbus_connect *))
        {
            if(handler == NULL) return;
            modbus_connect_accept = handler;
        }
        
        modbus_connect_accept(modbus_connection);  //在B文件某个地方 调用函数指针modbus_connect_accept
        	
```