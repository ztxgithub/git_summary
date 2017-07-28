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
        endptr:指向下一个要转化的起始地址,一般是NULL
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

- 在C语言中只要遇到位域的一定要用无符号

``` c
    
    struct adpt_ssbi_t
    {
        UINT8_T type;             ///< The signal point's type, AI, AO, DI, DO.
        UINT8_T dev_type;       ///< The device's type of the signal point belongs to.
        int addr;               ///< Relative offset of the register.
        int addr_len;           ///< The register's length.
        unsigned int bit_shift:4;        ///< If the signal point's type is DI,
        int scale;              ///< Decimal places of the point's value.
        union sig_val_t val;    ///< The signal point's value.
        char name[NAME_SIZE];   ///< The signal point's name.
        unsigned int valid:1;          ///< Whether this point is valid.
    };
        
        
```

- sscanf

    描述：
        1.可以进行字符串的分割
        2.可以进行 字符串->数字

``` c

    1. 取指定长度的字符串。如在下例中，取最大长度为4字节的字符串。
    　sscanf("123456 ", "%4s", buf);
    　printf("%s\n", buf);
    　结果为：1234
    
    2.遇到空格停止读取
        char buf[512] ={};
        sscanf("123456 789", "%s", buf);
        printf("%s\n", buf);
        
        结果为：123456
        
    3.取仅包含指定字符集的字符串。如在下例中，取仅包含1到9和小写字母的字符串。
      　　sscanf("123456abcdedfBCDEF", "%[1-9a-z]", buf);
      　　printf("%s\n", buf);
      　　结果为：123456abcdedf
      
      %*:表示跳过以下条件的字符
    4.给定一个字符串iios/12DDWDFF@122，获取 / 和 @ 之间的字符串，先将 "iios/"过滤掉，再将非'@'的一串内容送到buf中
      　　sscanf("iios/12DDWDFF@122", "%*[^/]/%[^@]", buf);
      　　printf("%s\n", buf);
      
    5.给定一个字符串iios/12DDWDFF@122,只读取iios到buf
            char buf[512] ={};
            sscanf("iios/12DDWDFF@122", "%[^/]", buf);
            printf("%s\n", buf);
            
            结果为：iios
            
    6.给定一个字符串iios/12DDWDFF@122,只读取12DDWDFF@122到buf
            char buf[512] ={};
            sscanf("iios/12DDWDFF@122", "%*[^/]/%s", buf);
            printf("%s\n", buf);
            
            结果为：12DDWDFF@122
            
    7.2006:03:18-2006:04:18 将日期分别存到2个字符串数组中
    format-type中有%[]这样的type field。如果读取的字符串，不是以空格来分隔的话，就可以使用%[]。
        char sztime1[16] = "", sztime2[16] = "";
        sscanf("2006:03:18-2006:04:18", "%[0-9,:]-%[0-9,:]", sztime1, sztime2);
        printf("sztime1:%s\n", sztime1);
        printf("sztime2:%s\n", sztime2);
        
    8.
        int no = -1;
        int type = -1;
        int no_three = -1;
        int no_four = -1;
        sscanf("1,2,3,,1,1,,,,,,,,,1,1,1,,1,aaaa,,bbbb,cccc", "%*d,%d,%d%*[,]%d", &type, &no_three, &no_four);
        printf("no[%d],type[%d],no_three[%d],no_four[%d]\n", no, type, no_three, no_four);
        
        1,2,3,,1 剔除多个,,取１
        
    9.
            int no = -1;
            int type = -1;
            int no_three = -1;
            int no_four = -1;
            char str1[16] = "", str2[16] = "", str3[16] = "";
            sscanf(",aaaa,,bbbb,cccc", "%*[,]%[^,]%*[,]%[^,]%*[,]%[^,]", str1, str2, str3);
            printf("str1:%s\n", str1);
            printf("str2:%s\n", str2);
            printf("str3:%s\n", str3);
            
            截取多个字符串
        
```

 [相关资料](http://www.cnblogs.com/lyq105/archive/2009/11/28/1612677.html)
 
 - printf输出格式
 
 ``` c
 	unsinged int,unsigned short,unsigned char --> %u
 	unsigned long int, unsigned long --> %lu
 	unsigned long long, unsigned long long int --> %llu
 	
 	int  --> %d
 	long int --> %ld
 	
 ```
 
 - scanf或者sscanf读取值转化为枚举的值要用 %u
 
  ``` c
  
     enum enable_c1
     {
         DISABLE_C1 = 0,
         ENABLE_C1 = 1
     };
     
     enum enable_c1 en;
     
     scanf(%u", &en);
                        
  ```
  
- strlen和printf("%s)的实现原理都是以'\0'为结尾的标准
  
-  strncpy(char *dst, char *src, int size)　会使dst剩余的值为 '\0'


- sprintf

 ``` c
  
     功能:
        1.数字(int,double)转为字符串
        2.
     
     int sprintf( char *buffer, const char *format [, argument] ... )
     
     参数：
        format：%[flags][width][.precision][length]specifier
        
                specifier:说明符
                          d/i:有符号的十进制整数
                          f:十进制浮点数
                          o:有符号八进制
                          s:字符串格式
                          u:无符号十进制
                          x:无符号十六进制(如果转化后有字母,则显示小写)
                          X:无符号十六进制(如果转化后有字母,则显示大写)
                          %:字符'%'
                          
                flags:标识
                        -:在给定的字段宽度内左对齐,默认是右对齐
                        +:强制在结果之前显示加号或减号（+ 或 -），即正数前面会显示 + 号。
                        默认情况下，只有负数前面会显示一个 - 号。
                        #:与 o、x 或 X 说明符一起使用时，非零值前面会分别显示 0、0x 或 0X。
                        0:在指定的固定宽度(width),有效数不足,用0补,而不是用默认的空格(' ')补
                        
                width:宽度
                        number(例如7):要输出的字符的最小数目。如果输出的值短于该数，
                        结果会用空格填充。如果输出的值长于该数，结果不会被截断。
                        
                .precision :精度
                        
                        .number: 
                            对于整数说明符（d、i、o、u、x、X）,效果与%0width 一样
                            对于浮点数(f),代表小数点保留number位
                            对于 s: 要输出的最大字符数。默认情况下，所有字符都会被输出，直到遇到末尾的空字符。
                                    不管怎么样,后面都会跟一个'\0'.
                                    
                length :长度
                        h:参数被解释为短整型或无符号短整型（仅适用于整数说明符：i、d、o、u、x 和 X）
                        hh: signed char (hhd),unsigned char(hhu)
                        l:参数被解释为长整型或无符号长整型，适用于整数说明符（i、d、o、u、x 和 X）
                        ll: long long int or unsigned long long int
                        L: 参数被解释为长双精度型
     
     返回值：
            实际的字符串的大小(不包括\0)
     
     
     
    1.将数字转化为字符串(不足位数左边补０)
    char dig[8];
    sprintf(dig, "%04d", 123);
    
    ---结果为 int(123)->char *("0123")-----
    
    for(int i = 0; i < 8; i++)
        printf("val[%d]\n", dig[i]);
    printf("dig[%s]\n", dig);
    
    2.sprintf格式化到字符数组中,除去有效的字符和'\0',剩余的目的字符数组中保存原来的值．
    
        char dig[8];
        memset(dig, ' ', 8);
        int valid_len = sprintf(dig, "%d", 12);
        
        
        ---结果为 dig[0] == '1'
                 dig[1] == '2'
                 dig[2] == '\0'
                 dig[3] == ' '
                 dig[4] == ' '
                 dig[5] == ' '
        
        for(int i = 0; i < 8; i++)
            printf("val[%d]\n", dig[i]);
        printf("dig[%s]\n", dig);
        printf("valid_len[%d]\n", valid_len);
        
    3.format中%flags标示的在给定的字段宽度(width)内对齐问题
        (1):flags默认是右对齐(给定的字段宽度(width))
        
            char dig[8];
            memset(dig, 1, 8);
            int valid_len = sprintf(dig, "%2d", 0x5);
            
            在起始地址dig,取字段宽度为2(取2个字节),再根据右对齐的原则,如果字段宽度充足,则没填充的字节
            按空格' '填充,
            dig[0] == ' '  //不足空格填充
            dig[1] == '5'  //右对齐,字段宽度为2
            dig[2] == '\0'
            dig[3] == 1  //原来初始化的值
            
            
         (2):flags 为　'-',左对齐
         
                 char dig[8];
                 memset(dig, 1, 8);
                 int valid_len = sprintf(dig, "%-3d", 0x5);
                 
                 在起始地址dig,取字段宽度为2(取2个字节),再根据右对齐的原则,如果字段宽度充足,则没填充的字节
                 按空格' '填充,
                 dig[0] == '5'  //左对齐
                 dig[1] == ' '  //不足空格填充
                 dig[2] == ' '  //字段宽度为3,不足空格填充
                 dig[3] == '\0'
                 dig[4] == 1
                 
    4. %.precision ,字符串的应用
    
          char dig[8];
          memset(dig, 1, 8);
          int valid_len = sprintf(dig, "%.1s", "123");
          
          
          dig[0] == '1'   //只输出一个字符
          dig[1] == '\0'　//后面紧接着'\0'
          dig[2] == 1
          
    5.使用sprintf 的 % 时.一定要注意 %f 对应的是浮点数, %d 对应的是 int
        char dig[8];
        memset(dig, 1, 8);
        int i_value  = 10;
        float f_value = 10.3;
        double d_value = 10.3;
        int valid_len = sprintf(dig, "%.2f", i_value);  //错误用法
        结果dig打印出来0.00(结果错误)
        
        int valid_len = sprintf(dig, "%d", i_value);  //正确用法
          
            
        
        
     
                          
```

- union 可以进行无差别赋值,但不能进行无差别比较

```c

 union sig_val_t
  {
      int ival;    ///< Integer value.
      float fval;  ///< Float value.
  };
  
      union sig_val_t l;
      union sig_val_t r;
      l.fval = 1.3;
      r = l;      //这个操作是可以的
      if(l.fval == r.fval)　　
          printf("l == r\n");
          
      if(l == r) //这个操作是不行的
    
```

   
  
    
