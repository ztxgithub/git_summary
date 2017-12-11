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
    
    unsigned long int strtoul(const char* str, char** endptr, int base);
    long int strtol(const char *str, char **endptr, int base)
    
    参数:
        str:字符串的起始地址
        endptr:指向下一个要转化的起始地址,一般是NULL
        base:进制
            0:由str序列的格式决定的,"0x12"-->十六进制的0x12
            10: 十进制
            16: 十六进制
            
      char szNumbers[] = "2001 60c0c0 0x6fffff";
      char *pEnd;
      long int li1, li2, li3;
      li1 = strtoul (szNumbers,&pEnd,10);
      li2 = strtoul (pEnd,&pEnd,16);
      li3 = strtoul (pEnd,NULL,0);
      printf ("The decimal equivalents are: %ld, %ld, %ld.\n", li1, li2, li3)
      
     结果:
         The decimal equivalents are: 2001, 6340800, 734003
         
    注意：
        strtol()函数与sscanf()函数差别是,strtol()不仅将字符串中的数字转化为int型,还可以根据字符串中的单位再进行进一步的业务处理
        例如"16MB",通过strtol(szNumbers,&pEnd,10)可以得到16,之后pEnd为非空还可以进行判断
        
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

    int sscanf(const char * src, const char * format, ...);
    描述:
        the format 其实有一系列的指令集(directives),它将用于如何处理源字符串(src),即源字符串
        要与format一一匹配
        (1):如果源字符串有"abc",则format 中位置要一致且有"abc"
        (2):Each conversion specification in format begins with 
        either the character '%' or the character sequence "%n$"
            I: %*:代表滤过对应的值,不需要指针指向的内存来保存.同时sscanf返回值不计数在内
            II:m 只适用于%s,%c,%[
                 char *s; 
                 scanf("%ms", &s);
                 printf("%s", s);
                 free(s);
                 
    参数:
        format:相关的格式,conversion specifications,将源字符串中按format格式进行
               匹配,将对应的数据保存在指针指向的内存中(注意数据类型要一一对应),数量也要一一对应
    返回值:
        成功:则返回全部匹配的值
        
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
            
    10.     %n: 看总共读取的字节数(不包含'\0'), sscanf 返回值中不进行计数
            char str[]="192.168.0.5:8080";
            unsigned int a, b, c, d, port = 0;
            int len = 0;
            int s_ret_len = 0;
            s_ret_len = sscanf(str, "%u.%u.%u.%u:%u%n", &a, &b, &c, &d, &port, &len);
            
            printf("a[%u], b[%u], c[%u], d[%u], port[%u], len[%d], s_ret_len[%d]\n",
            a, b, c, d, port, len, s_ret_len);

            结果:a[192], b[168], c[0], d[5], port[8080], len[16], s_ret_len[5]
            注意:只有当s_ret_len==5时,len的长度才有效
        
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
        2.可以在程序中用变量指定width或者.precision
     
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
    
         (1):对于字符串 %.7s :代表取源字符串中的头7个字符,之后再加上'\0',形成一个字符串 
            
          char dig[8];
          memset(dig, 1, 8);
          int valid_len = sprintf(dig, "%.1s", "123");
          
          
          dig[0] == '1'   //只输出一个字符
          dig[1] == '\0'　//后面紧接着'\0'
          dig[2] == 1
          
          (2)
            　char a1[] = {'A', 'B', 'C', 'D', 'E', 'F', 'G'};
            　char a2[] = {'H', 'I', 'J', 'K', 'L', 'M', 'N'};
              sprintf(s, "%.7s%.7s", a1, a2);//产生："ABCDEFGHIJKLMN"
          
    5.使用sprintf 的 % 时.一定要注意 %f 对应的是浮点数, %d 对应的是 int
        char dig[8];
        memset(dig, 1, 8);
        int i_value  = 10;
        float f_value = 10.3;
        double d_value = 10.3;
        int valid_len = sprintf(dig, "%.2f", i_value);  //错误用法
        结果dig打印出来0.00(结果错误)
        
        int valid_len = sprintf(dig, "%d", i_value);  //正确用法
          
    6. 用变量指定相应的字段宽度(width)或者精度(.precision)  %*中 *代替变量
    
            char dig[8];
            memset(dig, 1, 8);
            int i_value  = 1;
            int width = 4;
            int valid_len = sprintf(dig, "%0*d", width, i_value);
            
            结果:
             dig[0] == '0'   
             dig[1] == '0'　
             dig[2] == '0'　
             dig[3] == '1'　
             dig[4] == '\0'　
             dig[5] == 1　
                          
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

- static 变量在函数内只会被申明一次(同时有初始化则只初始化一次)

```c
    void static_inc()
    {
        static int i = 0;  //只初始化一次
        printf("i[%d]\n", i++);
    }
    
```

- fcntl系统调用(根据文件描述词来操作文件的特性)
```c

    int fcntl(int fd, int cmd, long arg); 
    
    参数: 
        fd:文件描述符
        cmd:命令操作
            F_SETFD:设置文件描述词标志
          
     返回值:
        成功则返回0
    
    1.fcntl(fd, F_SETFD, FD_CLOEXEC) 
        It sets the close-on-exec flag for the file descriptor, which causes the file descriptor to be automatically
         (and atomically) closed when any of the exec-family functions succeed.This is useful to keep from 
         leaking your file descriptors to random programs run by e.g. system().
         
    2.  设置文件非阻塞模式
        int flags = fcntl(sock, F_GETFL, 0);
        fcntl(sock, F_SETFL, flags | O_NONBLOCK);
    
```

- Error: free(): invalid next size (fast) 原因

```c
    注意:
        程序在写请求的内存时,一定不能写越界,要不然free会出问题
  1.free 不是由malloc或则calloc得来的指针
  2.多次free内存,其中包括free 重叠的内存
  You may be overflowing a buffer or 
  otherwise writing to memory to which you shouldn't be writing, 
  causing heap corruption.
       
```

- sizeof struct数据结构

```c
    
    struct ctl_msg {
        void (*mg_event_handler_t)(struct mg_connection *nc, int ev,
                                   void *ev_data);  //函数指针
    };
    
    1.在32位linux上,
    sizeof(struct ctl_msg) == 4 
    sizeof(void *) == 4
    
    2.在64位linux上
     sizeof(struct ctl_msg) == 8
     sizeof(void *) == 8   
    
    (1)考虑字节对齐
     在64位linux上:
        struct ctl_msg {
            void (*mg_event_handler_t)(struct mg_connection *nc, int ev,
                                       void *ev_data);  //函数指针
            char size[100];
        };
        
        sizeof(struct ctl_msg) == 8 + 13*8(考虑到字节对齐 char[100] 不能被8整除) == 112
```



- main主函数 argc 参数的含义

```c
    > ./sample
    结果:
        argc = 1
        argv[0] = "./sample"
```

- 产生随机函数

```c
     void srand(unsigned  int  seed)
     描述:
         初始化随机数发生器,srand()用来设置rand()产生随机数时的随机数种子, 如果每次seed都设相同值,紧接着调用第一次rand()所产生的
         随机数值会一样
     参数:
         seed必须是个整数，通常可以利用time(0)的返回值或NULL来当做seed
         
         
     int rand(void)
     描述:
        rand()的内部实现是用线性同余法做的，它不是真的随机数，因其周期特别长，故在一定 的范围里可看成是随机的。
        rand()返回一随机数值的范围在0至RAND_MAX 间。RAND_MAX的范围最少是在32767之间(int).
        
     注意:
        1.如果调用rand()函数之前没有调用srand()函数,那么默认随机数种子为1(srand()函数中seed参数为1),当这个程序没有重启时,多次调用
        rand()函数,没有关系会产生不一样的随机数,但是当程序重启时跟第一次程序产生的随机数是一样的.
        例如
           for (i=0; i<3; i++)      //产生10个随机数
            {
                printf("%d\n", rand());
            }
         > ./test         第一次运行
                1932553610
                245590780
                1780233405 
                
         > ./test         第二次运行
               1932553610
               245590780
               1780233405 
               
        2.正确使用方式:每次调用时先调用一次srand()函数,seed参数为time(NULL),确保seed唯一,之后就可以多次调用rand()函数,只有当rand()
        函数调用65535后,随机数才会重复.
     
     产生一定范围随机数的通用表示公式
           要取得[a,b)的随机整数，使用(rand() % (b-a))+ a （结果值含a不含b）。
           要取得[a,b]的随机整数，使用(rand() % (b-a+1))+ a （结果值含a和b）。
           要取得(a,b]的随机整数，使用(rand() % (b-a))+ a + 1 （结果值不含a含b）。
           （总的来说，通用公式：a + rand() % n ；其中的a是起始值，n是整数的范围）
           要取得a到b之间的随机整数，另一种表示：a + (int)b * rand() / (RAND_MAX + 1)。
           要取得0～1之间的浮点数，可以使用rand() / double(RAND_MAX)
```


- 打印程序中错误的信息error

```c
     printf("%s\n", strerror(errno));
```

- 文件内容的特殊字符

```c

     ./zdfs_providerd^@provider.conf^@restart^@
     
     其中通过 ssize_t read(int fd, void *buf, size_t count);将该文件里的内容读到buff中时,^@会转化为"\0",
     所以printf该buf时,只能打印出./zdfs_providerd(用strcmp函数也是一样的)
```

- 给进程发送信号kill()函数

```c
    #include <signal.h>
    
    int kill(pid_t pid, int sig);
    
    参数：
        pid:进程pid
                1、pid>0 将信号传给　进程识别码为pid 的进程.
                2、pid=0 将信号传给和目前进程相同进程组的所有进程
                3、pid=-1 将信号广播传送给系统内所有的进程
                4、pid<0 将信号传给进程组识别码为pid 绝对值的所有进程参数 
        sig:
            0: 通过返回值,可以判断pid进程是否正常运行
            SIGTERM：向pid进程发送终止操作
            
    返回值：
        成功：０
        失败:-1
            printf("%s\n", strerror(errno));  
```

## 排序查找

- qsort()快速排序

```c
    void qsort(void *base, size_t nitems, size_t size, int(*comp)(const void * , const void *));
    
    描述：
        升序：
        int cmp(const void *a, const void *b)
        如果a > b，返回>0
        如果a == b, 返回0
        如果a < b，返回<0
        
        降序相反,a和b对调一下
        
    参数：
        base：排序的数组起始地址
        nitems：数组中的元素数目
        size：每个数组元素占用内存空间，可使用sizeof获得
```

- 二分查找 

```c
    void *bsearch(const void *key, const void *base, size_t nitems, size_t size,
                  int (*compar)(const void *, const void *))
    
    描述：
        升序：
        int cmp(const void *a, const void *b)
        如果a > b，返回>0
        如果a == b, 返回0
        如果a < b，返回<0
        
        降序相反,a和b对调一下
        
    参数：
        key: 指向要查找的元素的指针，类型转换为 void*
        base：排序的数组起始地址
        nitems：数组中的元素数目
        size：每个数组元素占用内存空间，可使用sizeof获得
        
    返回值：
        成功：指向数组中匹配元素的指针
        失败：NULL
        
    注意：
        二分查找时,base数组元素的排序方向(升降序)和compar的实现的升降序要一致
```
## 系统资源接口函数

- realloc 的应用

```c
    void* realloc( void *ptr, size_t new_size );
    1.对ptr进行判断，如果ptr为NULL，则函数相当于malloc(new_size),
      试着分配一块大小为new_size的内存，如果成功将地址返回，否则返回NULL。
      如果ptr不为NULL，则进入2
    2.查看ptr是不是在堆中，如果不是的话会跑出异常错误，会发生realloc invalid pointer（具体原因在后面讲.
      如果ptr在堆中，则查看new_size大小.
      如果new_size大小为0，则相当于free(ptr)，将ptr指针释放，返回NULL.
      如果new_size小于原大小，则ptr中的数据可能会丢失，只有new_size大小的数据会保存（这里很重要）.
      如果size等于原大小，等于啥都没做.
      如果size大于原大小，则看ptr所在的位置还有没有足够的连续内存空间，如果有的话，分配更多的空间，返回的地址和ptr相同，
                        如果没有的话，在更大的空间内查找，如果找到size大小的空间，
                        将旧的内容拷贝到新的内存中，把旧的内存释放掉，则返回新地址，否则返回NULL。
    这里解释一下，为什么ptr要在堆中，学过编译原理的同学应该都知道C语言中的malloc，realloc，calloc()这种用户主动分配的内存放在堆区，
    而临时变量放在栈区，这是两块不同的区域。堆区的变量由用户通过free释放空间，而栈区的变量在函数退出后就自动释放，
    所以这两个区域的变量生存时间不同，不能相互间进行内存分配操作。
    所以要求ptr必须在堆区，即ptr必须是malloc，realloc或者calloc的返回值，要不然可以为NULL，把realloc变为malloc。
    所以使用realloc函数应该注意一下几点：
    1.ptr必须为NULL，或者为malloc，realloc或者calloc的返回值，否则发生realloc invalid pointer错误
    2.new_size如果小于old_size，只有new_size大小的数据会被保存，可能会发生数据丢失，慎重使用。
    3.如果new_size大于old_size，可能会分配一块新的内存，这时候ptr指向的内存会被释放，ptr成为野指针，再访问的时候会发生错误。
    4.最后不要将返回结果再赋值给ptr，即ptr=realloc(ptr,new_size)是不建议使用的，
    因为如果内存分配失败，ptr会变为NULL，如果之前没有将ptr所在地址赋给其他值的话，会发生无法访问旧内存空间的情况，
    所以建议使用temp=realloc(ptr,new_size)。
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

- 资源的限制 getrlimit/setrlimit

``` c

    struct rlimit {
    　　rlim_t rlim_cur;　　//软限制　RLIM_INFINITY　无限制
    　　rlim_t rlim_max;   //硬限制　RLIM_INFINITY
    };
    
    #include <sys/resource.h>
    int getrlimit(int resource, struct rlimit *rlim);
    int setrlimit(int resource, const struct rlimit *rlim);
    
    参数：
        resource：
            RLIMIT_NOFILE :指定进程可打开的最大文件描述符最大值，超出此值，将会产生EMFILE错误。
            RLIMIT_NPROC :用户可拥有的最大进程数。
            RLIMIT_AS :进程的最大虚内存空间，字节为单位。
            RLIMIT_CORE :内核转存文件的最大长度。
            RLIMIT_CPU :最大允许的CPU使用时间，秒为单位。当进程达到软限制，内核将给其发送SIGXCPU信号，
                        这一信号的默认行为是终止进程的执行。然而，可以捕捉信号，处理句柄可将控制返回给主程序。
                        如果进程继续耗费CPU时间，核心会以每秒一次的频率给其发送SIGXCPU信号，直到达到硬限制，
                        那时将给进程发送 SIGKILL信号终止其执行。
            RLIMIT_DATA :进程数据段的最大值。
            RLIMIT_FSIZE :进程可建立的文件的最大长度。如果进程试图超出这一限制时，核心会给其发送SIGXFSZ信号，
                          默认情况下将终止进程的执行。
            RLIMIT_LOCKS :进程可建立的锁和租赁的最大值。
            RLIMIT_MEMLOCK :进程可锁定在内存中的最大数据量，字节为单位。
            RLIMIT_MSGQUEUE :进程可为POSIX消息队列分配的最大字节数。
            RLIMIT_NICE :进程可通过setpriority() 或 nice()调用设置的最大完美值。
            RLIMIT_RTPRIO ：进程可通过sched_setscheduler 和 sched_setparam设置的最大实时优先级。
            RLIMIT_SIGPENDING ：用户可拥有的最大挂起信号数。
            RLIMIT_STACK ：最大的进程堆栈，以字节为单位
            
    返回值：
        0:成功
        -1:失败
        EFAULT：rlim指针指向的空间不可访问
        EINVAL：参数无效
        EPERM：增加资源限制值时，权能不允许
        printf("%s\n", strerror(errno));
        
    注意：
        一般是设置软限制rlimit.rlim_cur,且rlimit.rlim_cur <= rlimit.rlim_max
        
```

## 文件处理函数

- 移动文件的读写位置lseek()

```c
     off_t lseek(int fd, off_t offset, int whence);
     
     描述:
         每一个已打开的文件都有一个读写位置,当打开文件时通常其读写位置是指向文件开头, 若是以附加的方式打开文件(如O_APPEND), 
         则读写位置会指向文件尾. 当read()或write()时, 读写位置会随之增加,lseek()便是用来控制该文件的读写位置. 
         
     参数:
         fd:为已打开的文件描述词
         参数 offset 的含义取决于参数 whence
            1.whence 是 SEEK_SET, offset: 将文件偏移量设置为offset
            2.whence 是 SEEK_CUR，文件偏移量将被设置为 cfo(当前文件偏移量) 加上 offset，offset 可以为正也可以为负
            3.whence 是 SEEK_END，文件偏移量将被设置为文件长度加上 offset，offset 可以为正也可以为负
            
      返回值:
            成功时:返回目前的读写位置, 也就是距离文件开头多少个字节.
            失败:返回-1, errno 会存放错误代码,strerror(errno)查看错误信息
            
     应用:
        1) 将读写位置移到文件开头时:lseek(int fildes, 0, SEEK_SET);
        2) 将读写位置移到文件尾时:lseek(int fildes, 0, SEEK_END);
        3) 想要取得目前文件位置时:lseek(int fildes, 0, SEEK_CUR);
        
     常用用途:
        (1) 得到文件的大小
               file_size = lseek(int fildes, 0, SEEK_END)
     
```

- 读文件里面的内容read()函数

```c
    ssize_t read(int fd, void *buf, size_t count);
    
    描述:
        read()会把参数fd所指的文件传送count个字节到buf指针所指的内存中,若参数count为0，则read()不会有作用并返回0。
        此外文件读写位置会随读取到的字节移动.
        
    参数:
        fd:打开的文件描述符
        buf:内容保存的起始地址
        count:要读的字节数
        
    返回值
        实际读取到的字节数，
        返回0，表示已到达文件尾或是无可读取的数据
        返回-1:读取失败
                EINTR 此调用被信号所中断.
                EAGAIN 当使用非阻塞I/O 时(O_NONBLOCK), 若无数据读取则返回此值.
                EBADF 参数fd 非有效的文件描述词, 或该文件已关闭.
        
    注意:
        如果顺利read()会返回实际读到的字节数，最好能将返回值与参数count作比较，若返回的字节数比要求读取的字节数少，
        则有可能读到了文件尾、从管道(pipe)或终端机读取，或者是read()被信号中断了读取动作。
        当有错误发生时则返回-1，错误代码存入errno中，而文件读写位置则无法预期.
        其中count的值不能超过SSIZE_MAX((2147479552)bytes   
```

- readlink
   
```c
    #include <unistd.h>
    
    int readlink(const char *path, char *buf, size_t bufsiz);
    
    描述:
        获取参数path指向的符号连接内容,将其保存到buf中
    参数:
        path:源
        buf: 对应的符号连接内容
        bufsiz : 传入参数,内容存放的大小
        
    返回值:
        成功:字符串有效的字节数
        失败则返回-1, 错误代码存于errno
        错误代码：
            1、EACCESS 取文件时被拒绝, 权限不够。
            2、EINVAL 参数bufsiz 为负数。
            3、EIO I/O 存取错误。
            4、ELOOP 欲打开的文件有过多符号连接问题。
            5、ENAMETOOLONG 参数path 的路径名称太长。
            6、ENOENT 参数path 所指定的文件不存在。
            7、ENOMEM 核心内存不足。
            8、ENOTDIR 参数path 路径中的目录存在但却非真正的目录
            
    实例一
        获得程序自身的运行路劲: int cnt = readlink("/proc/self/exe", current_absolute_path, MAX_SIZE);
```

- 获取文件状态 stat()函数
   
```c

    struct stat
    {
        dev_t st_dev; //device 文件的设备编号
        ino_t st_ino; //inode 文件的i-node
        mode_t st_mode; // 文件的类型和存取的权限
        nlink_t st_nlink; //number of hard links 连到该文件的硬连接数目, 刚建立的文件值为1.
        uid_t st_uid; //user ID of owner 文件所有者的用户识别码
        gid_t st_gid; //group ID of owner 文件所有者的组识别码
        dev_t st_rdev; //device type 若此文件为装置设备文件, 则为其设备编号
        off_t st_size; //total size, in bytes 文件大小, 以字节计算
        unsigned long st_blksize; //blocksize for filesystem I/O 文件系统的I/O 缓冲区大小.
        unsigned long st_blocks; //number of blocks allocated 占用文件区块的个数, 每一区块大小为512 个字节.
        time_t st_atime; //time of lastaccess 文件最近一次被存取或被执行的时间, 一般只有在用mknod、utime、read、write 与tructate 时改变.
        time_t st_mtime; //time of last modification 文件最后一次被修改的时间, 一般只有在用mknod、utime 和write 时才会改变
        time_t st_ctime; //time of last change i-node 最近一次被更改的时间, 此参数会在文件所有者、组、权限被更改时更新
    };
    
    int stat(const char *file_name, struct stat *buf);
    
    描述:
        stat()用来将参数file_name 所指的文件状态, 复制到参数buf 所指的结构中
    参数:
        file_name:文件名
        buf: 文件状态数据结构
        
    返回值:
        成功:0
        失败则返回-1, 错误代码存于errno
        错误代码：
            printf("%s\n", strerror(errno));
            
            
    实例一
        S_ISDIR (buf.st_mode) 是否为目录,是放回true
```

- 获取运行程序的当前工作目录绝对路劲dir

```c
    char *getcwd(char *buf, size_t size);
    描述:
        将当前工作目录的绝对路劲拷贝到buf处
        
    参数:
        buf:申请内存空间起始地址
        size: 内存大小,单位是字节
        
    返回值:
        成功:返回的值等与buf
        失败:放回NULL
                ERANGE:参数size大于0,但小于绝对路劲的长度
                EINVAL: 参数size 等于 0
                EACCES: 该当前路劲不允许访问
                
    注意:
        传入参数 buf 可以为NULL,那么它就调用了malloc函数,需要我们自己显示的调用free函数,同时也要判断一下返回值,当参数buf为NULL时
        size可以为0,代表系统自动根据当前工作目录申请合适的大小,最好不要用buf 为NULL
                
```

- 改变当前的工作目录

```c
    int chdir(const char *path);
    描述:
        将当前的工作目录改变成以参数path所指的目录
        
    参数:
        path:工作目录
        
    返回值:
        成功: 0
        失败:-1
           错误代码：
              printf("%s\n", strerror(errno));        
```

- 改变文件创建掩模

```c
    mode_t umask(mode_t mask);
    
    描述:
        默认情况下的umask值是022(可以用umask命令查看), 此时你建立的文件权限是644(664&(~022)),
        建立的目录的默认权限是755(775&(~022))
        
    参数:
        mask:掩码参数,例如0026
        
    返回值:
        成功: 上一次umask的值        
```

##　字符串操作函数

- 字符串的拷贝赋值strdup()函数

```c
    char* strdup(const char *str)
    
    描述:
        该函数根据传入参数str,动态的请求内存malloc,并将源字符串进行拷贝,返回指向新分配字符串的指针．
        
    参数:
        src:源字符串,不能为空
        
    返回值
        指向新分配字符串的指针
        NULL：调用失败
        
    注意:
        在调用前要判断一下传入参数str是否为空,调用完后返回值是否为空,使用完后一定要将其释放内存free() 
```

- 复制内存内容（可以处理重叠的内存块）memmove()函数

```c
    void* memmove(void *dest, const void *src, size_t num);
    
    描述:
         memmove()与memcpy()类似都是用来复制src所指的内存内容前num个字节到dest所指的地址上。
         不同的是，memmove() 更为灵活,当src和dest 所指的内存区域重叠时，memmove() 仍然可以正确的处理
         
    返回值
            指向dest字符串的指针
```

- 字符查找

```c
    char* strrchr(char *str, int character);
    描述:
        查找最后一个匹配character的位置,失败返回NULL
        
    返回值:
        对应的最后匹配的位置,
        没有找到返回NULL
```

## 进程组和会话相关信息

- 创建子进程

```c
    pid_t fork(void);
    
    描述:
        通过系统调用创建一个与原来进程几乎完全相同的进程,也就是两个进程可以做完全相同的事，但如果通过返回值可以判断是子进程还是父进程,
        两个进程也可以做不同的事,子进程继承了父进程的资源,例如工作目录,最大打开文件描述符等等
        
    返回值:
        在父进程中,fork返回新创建子进程的进程ID
        在子进程中,fork返回0
        如果出现错误,fork返回一个负值
```

- 创建会话

```c
     pid_t setsid(void);
    
    描述:
        创建一个会话和设置进程组id,调用setsid函数的进程是非组长进程,那么调用完setsid函数后该进程变成该会话的leader
        ( session ID == process ID)
        同时该进程也是在这个新的会话中的进程组的组长进程,该进程没有控制终端
        
    返回值:
        成功：返回session ID 
        失败：-1
```