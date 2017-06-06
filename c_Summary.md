# c用法

- ? : 三目运算符,　运算符优先级从右到左

``` c

   printf("Detach state        = %s\n", 
           (i == PTHREAD_CREATE_DETACHED) ? "PTHREAD_CREATE_DETACHED" :
           (i == PTHREAD_CREATE_JOINABLE) ? "PTHREAD_CREATE_JOINABLE" :
           "???");
           
   如果 i == PTHREAD_CREATE_JOINABLE 则输出：
   
   Detach state        = PTHREAD_CREATE_JOINABLE
	
```

- 将字符串转化为 unsigned long

``` c
    
    unsigned long int strtoul (const char* str, char** endptr, int base);
    
    参数:
        str:字符串的起始地址
        endptr:指向下一个要转化的起始地址,一般为NULL
        base:进制
            0:由str序列的格式决定的,"0x12"-->十六进制的0x12
            
      char szNumbers[] = "2001 60c0c0 0x6fffff";
      char * pEnd;
      long int li1, li2, li3;
      li1 = strtoul (szNumbers,&pEnd,10);
      li2 = strtoul (pEnd,&pEnd,16);
      li3 = strtoul (pEnd,NULL,0);
      printf ("The decimal equivalents are: %ld, %ld, %ld.\n", li1, li2, li3)
      
     结果:
         The decimal equivalents are: 2001, 6340800, 734003
        
```

- 字节对齐

``` c
    
    受于硬件的限制,一般要进行自然对齐(字节对齐),例如一个uint32的变量,其地址是4的倍数.
        
```

- pragma pack(1)

``` c
    
    在数据传输中数据结构有时候不需要自动补全,默认是自动填充.
    
    #pragma pack(1)
    typedef struct _EventDataHeader {
    	uint8_t version;       ///< 版本号 默认是0
    	uint8_t flag; 	       ///<  0 – 没数据 1 – 有数据 2 – last 结束
    	uint8_t data_number;   ///< 时间点的个数
    	uint8_t reverse;	   ///< 保留
    	uint32_t event_id;     ///< 事件的序号
    	uint64_t device_code;  ///< 蓄电池组设备ID(33010400010070001)
    	uint32_t bat_num;      ///< 该电池组设备的电池数
    }EventDataHeader;
    
    #pragma pack()
        
```

- 获取当前运行系统的配置信息

``` c
    
   #include <unistd.h>
   
   long sysconf(int name);
   
        参数:
            _SC_NPROCESSORS_CONF:处理器的个数
            _SC_PAGESIZE:pagesize(页大小)
            _SC_PHYS_PAGES：总共的页数量
            _SC_AVPHYS_PAGES:当前可利用的页的数量
            _SC_OPEN_MAX:最大的打开的文件描述符
        
```

