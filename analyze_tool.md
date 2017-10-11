
## pstack 显示每个进程的各个线程的栈情况

### pstack简介

``` shell

   打印运行程序的线程堆栈信息,实际上通过 $ whereis pstack,可以得到安装路劲为 /usr/bin/pstack,
   通过 ll /usr/bin/pstack,结果为 /usr/bin/pstack -> gstack, 它是gstack的符号链接,而 /usr/bin/gstack是
   一个shell 脚本.
         
```

### pstack 用法

``` shell

    $ pstack pid(已经存在的进程id)
    
    这个命令在排查进程问题时非常有用,比如我们发现一个服务一直处于work状态（如假死状态,好似死循环）,
    使用这个命令就能轻松定位问题所在；可以在一段时间内，多执行几次pstack,若发现代码栈总是停在同一个位置,
    那个位置就需要重点关注，很可能就是出问题的地方
         
```

# 跟踪调试程序

- strace和ltrace相同的命令行参数:

``` shell
    
    -f: 除了跟踪当前进程外,还跟踪其子进程(跟踪由fork调用所产生的子进程)
    -F 尝试跟踪vfork调用.在-f时,vfork不被跟踪. 
    -o file: 将输出信息写到文件file中，而不是显示到标准错误输出（stderr）
    -p PID: 绑定到一个由PID对应的正在运行的进程,此参数常用来调试后台进程（守护进程）
    
    以strace为例,每一行都是一条系统调用（ltrace为库函数）,等号左边是系统调用的函数名及其参数,右边是该调用的返回值  
```

## strace 跟踪进程中的系统调用

### strace简介

``` shell

    strace: Trace system calls and signals 
    strace常用来跟踪进程执行时的系统调用和所接收的信号. Linux进程不能直接访问硬件(设备读取磁盘文件,
    接收网络数据等等),需要由用户态模式切换至内核态模式,通过系统调用访问硬件设备.strace可以跟踪到一个
    进程产生的系统调用,包括参数，返回值，执行消耗的时间.一般比较适用于调试不明的故障,用strace找出系统
    调用中出错的地方,通常能得到故障发生的线索,特别是与文件有关的错误、参数错误等.
    注意：
        使用strace能够有效地发现系统调用失败有关的故障,但无法发现用户写出的程序或共享库中发生的错误。
         
```

### strace 用法

#### strace 基本用法

``` c

    #include<stdio.h>  
    #include<stdlib.h>  
      
    int main()  
    {  
        FILE *fp;  
        fp = fopen("/etc/shadow", "r");   
        if (fp == NULL)  
        {  
            printf("Error!\n");  
            return EXIT_FAILURE;  
        }  
        return EXIT_SUCCESS;  
    }  
         
```

``` shell

    # gcc -Wall -g st1.c -o st1 
    
    # ./st1
    结果:
        Error!
        
    # strace ./str1
    结果:
        open("/etc/shadow", O_RDONLY)           = -1 EACCES (Permission denied)  ****
        fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 18), ...}) = 0
        write(1, "Error!\n", 7Error!
        )                 = 7
        exit_group(1)                           = ?
        +++ exited with 1 +++
        
    更快的定位出问题可以采用从后往前查找,从标注的位置可以发现,最后即为在界面上显示错误信息的系统调用,再往前看,
    系统调用open()失败，而且立即可以得知程序在试图打开/etc/shadow时发生了Permission denied错误（EACCES）
    从而直到要以root用户运行该程序.
         
```

#### strace 各个选项

-  -i (找到地址方便GDB详细调试)

``` shell

        [00007f55c18fe460] open("/etc/shadow", O_RDONLY) = -1 EACCES (Permission denied)
        [00007f55c18fe054] fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 18), ...}) = 0
        [00007f55c18fe6e0] write(1, "Error!\n", 7Error!) = 7
        [00007f55c18d3b98] exit_group(1)        = ?
        [????????????????] +++ exited with 1 +++
        
各行开头[]中的数字就是执行系统调用的代码的地址。在GDB中可以指定该地址并显示backstrace.
暂时不知道怎么用.

```

- -p PID (或 -p `pidof ProcName`): attach到进程上，调试后台程序
 
``` shell
 
    此选项主要用于查看运行中的进程（如守护进程）的行为
    
```
 
- -t / -tt ——显示系统调用的执行时刻
 
``` shell
  
     此选项主要用于查看运行中的进程（如守护进程）的行为
     -t    精确到s, 显示的是运行系统调用的当前时间(10:34:04)
     -tt   精确到微秒 (10:30:15.568853)
     -ttt  时间戳(精确到微妙) 1504233246.223193
     
     -T  显示每一调用所耗的时间. 
                                                                            |--调用耗的时间--|
     open("/etc/shadow", O_RDONLY)           = -1 EACCES (Permission denied) <0.000221>
  
```

- -e expr 指定一个表达式,用来控制如何跟踪
  
``` shell
  
    -e trace=set 只跟踪指定的系统调用
    例如:-e trace=open,close,rean,write表示只跟踪这四个系统调用.默认的为set=all. 
     $ strace -e poll,select,connect,recvfrom,sendto nc www.news.com 80
    
    -e trace=file 只跟踪有关文件操作的系统调用. 
        相当于 -e trace=open.stat,chmod,unlink...
    -e trace=process 只跟踪有关进程控制的系统调用. 
    -e trace=network 跟踪与网络有关的所有系统调用. 
    -e strace=signal 跟踪所有与系统信号有关的系统调用 
    -e trace=ipc 跟踪所有与进程通讯有关的系统调用 
    -e abbrev=set 设定strace输出的系统调用的结果集.-v 等与 abbrev=none.默认为abbrev=all. 
    -e raw=set 将指定的系统调用的参数以十六进制显示. 
    -e signal=set 指定跟踪的系统信号.默认为all.如 signal=!SIGIO(或者signal=!io),表示不跟踪SIGIO信号. 
    -e read=set 输出从指定文件中读出 的数据.例如: -e read=3,5 -e write=set 输出写入到指定文件中的数据.
    
```
 
-  -s strsize 
  
``` shell

  指定系统调用参数(传入)的字符串的最大长度.默认为32, 如果传入参数的字符串是文件名,则字符串全部输出
  要尽量大一点.  -s 300
    
```

- -c 统计每一系统调用的所执行的时间,次数和出错的次数等. 

``` shell

    启用strace -c -p命令后，在你按ctrl-c退出前程序的调用时间将会打印出来
    
```

#### strace 应用

- 通过已知的fd,找到对应的文件位置

  
``` shell

    #include<stdio.h>  
    #include<unistd.h>  
    #include<sys/types.h>  
    #include<sys/stat.h>  
    #include<fcntl.h>  
      
    int main()  
    {  
        open("wcdj", O_CREAT|O_RDONLY);// open file foo  
        sleep(1200);// sleep 20 mins 方便调试  
      
        return 0;  
    }  
    
    $ gcc -Wall -g -o testlsof testlsof.c            
    $ ./testlsof &  
    [1] 12371  
    
    # strace -s 300 lsof -p 117573 2>&1 | grep "wcdj"
    结果:
        readlink("/proc/118630/fd/3", "/home/jame/c++/test123/test_example/wcdj", 4096) = 40
  
    可以利用对 lsof -p pid 进行strace,得到核心的系统调用.lsof利用了/proc/pid/fd目录,
    Linux内核会为每一个进程在/proc建立一个以其pid为名的目录用来保存进程的相关信息,而其子目录fd保存的是该进程打开的
    所有文件的fd。
    进入/proc/pid/fd目录下，发现每一个fd文件都是符号链接，而此链接就指向被该进程打开的一个文件.
    我们只要用readlink()系统调用就可以获取某个fd对应的文件了.
    
    # 
    
```

- 可以通过 strace -s 300 -o log ./exe 发现Segment Fault的原因

- 可以只看特定的系统函数 $ strace -e poll,select,connect,recvfrom,sendto ./exe

- 你能找到被一个程序读取的配置文件,通过 open("/proc/filesystems", O_RDONLY) = 3 这些系统函数

## ltrace 跟踪进程调用库函数的情况

### ltrace简介

``` shell

    ltrace: A library call tracer 
             
```

## nm 目标文件格式分析

### nm简介

``` shell

    nm 命令显示关于指定 File 中符号的信息, File(文件)可以是对象文件、可执行文件或对象文件库。
    如果文件没有包含符号信息,nm 命令报告该情况,但不把它解释为出错条件.
    作用:
    （1）判断指定程序中有没有定义指定的符号 (比较常用的方式：nm -C proc | grep symbol)
    （2）解决程序编译时undefined reference的错误，以及mutiple definition的错误
        (会发生在库的缺失或企图链接一个错误版本的库的时候)
    （3）查看某个符号的地址，以及在进程空间的大概位置（bss, data, text区，具体可以通过第二列的类型来判断）
    
    $ nm cpu.o
    
        00000024 T cleanup_before_linux
        00000018 T cpu_init
        00000060 T dcache_disable
        00000054 T dcache_enable
        0000006c T dcache_status
        00000000 T do_reset
        0000003c T icache_disable
        00000030 T icache_enable
        00000048 T icache_status
             
    第一列(00000024)是偏移量, 第二列(T) 代表符号类型, 第三行(cleanup_before_linux)代表名字
    
    包含可执行代码的段称为正文段,数据段包含了不可执行的信息或数据
    
```

### 选项

``` shell

    -C 输出demangle过了的符号名称(有利于阅读,适用于C++)
    -A：每个符号前显示文件名
    -D：显示动态符号,该选项仅对于动态目标(例如特定类型的共享库)有意义
    -g：仅显示外部符号
    -r：反序显示符号表
    -l 使用对象文件中的调试信息打印出所在源文件及行号(编译时要加 -g)
    -n 按照地址/符号值来排序,未定义的符号永远排在最开始
    -u 打印出那些未定义的符号；
    -p 按目标文件中遇到的符号顺序显示，不排序
    --size-sort :对符号的大小进行排序
    
```

### 符号类型

``` shell

    对于每一个符号来说，其类型如果是小写的，则表明该符号是local的；大写则表明该符号是global(external)的.
常用:

   A 该符号的值在今后的链接中将不再改变,这样的符号值常常出现在中断向量表中,例如用符号来表示各个中断向量函数在中断向量表中的位置.
   B 该符号放在BSS段中,通常是那些未初始化的全局变量,例如，在一个文件中定义全局static int test,则该符号test的类型为B
     位于bss section中,其值表示该符号在bss段中的偏移。一般而言,bss段分配于RAM中
   D 该符号放在普通的数据段中，通常是那些已经初始化的全局变量,分配到data section中.
     例如定义全局int baud_table[5] = {9600, 19200, 38400, 57600, 115200},则会分配于初始化数据段中。
   T 该符号放在代码段中，通常是那些全局非静态函数；
   U 该符号未定义过，该符号的定义在别的文件中.当前文件调用另一个文件中定义的函数,在当前文件中被调用的函数是未定义的；
     但是在定义它的文件中类型是T.但是对于全局变量来说,在定义它的文件中，其符号类型为C，在使用它的文件中，其类型为U。
   
不常用:

   W 未明确指定的弱链接符号；同链接的其他对象文件中有它的定义就用上，否则就用一个系统特别指定的默认值
   C 该符号为common,common symbol是未初始化数据段.该符号没有包含于一个普通section中.只有在链接过程中才进行分配.
     符号的值表示该符号需要的字节数。例如在一个c文件中，定义int test，并且该符号在别的地方会被引用，则该符号类型即为C。
     否则其类型为B.
     
   G 该符号也位于初始化数据段中。主要用于small object提高访问small data object的一种方式.
   I 该符号是对另一个符号的间接引用
   N 该符号是一个debugging符号
   R 该符号位于只读数据区.例如定义全局const int test[] = {123, 123};则test就是一个只读数据区的符号.
   V 该符号是一个weak object
   
   
```

### 示例

- 列出 a.out 对象文件的静态和外部符

``` shell

    nm -g
    
```

- 解决编译时undefined reference的错误

``` shell

   主要有2个方面
    1.动态链接库本身没有问题,工程链接的时候出问题.没有添加头文件,没有指定动态链接库.so的路劲等
    2.可能动态链接库本身有问题,这时可 
           nm -A libXXX.so |grep "CHttpParser"(undefined reference 的函数名)
    
```

- 显示可执行文件中所有Undefined Symbols

``` shell

    $ nm -u ./exe
    
```

- 显示可执行文件中所有 Symbols

``` shell

    $ nm -n ./exe
    
```

- 搜索某个特定的Symbols并显示其大小

``` shell

    $ nm -S ./exe | grep "CHttpParser"
    
```


## 程序映像

##### 简介

``` shell

    程序映射到内存中,从低地址到高地址依次为下列段:
    
        代码段： 只读，可共享; 代码段（code segment/text segment ）通常是指用来存放程序执行代码的一块内存区域.
                这部分区域的大小在程序运行前就已经确定,并且内存区域通常属于只读, 某些架构也允许代码段为可写，即允许修改程序。
                在代码段中，也有可能包含一些只读的常数变量，例如字符串常量等.
                
        数据段： 储存已被初始化了的静态数据.数据段（data segment ）通常是指用来存放程序中已初始化的全局变量的一块内存区域。
                数据段属于静态内存分配.
                
        BSS 段：未初始化的数据段. BSS 段（bss segment ）通常是指用来存放程序中未初始化的全局变量的一块内存区域.
                BSS 是英文Block Started by Symbol 的简称。BSS 段属于静态内存分配.
                
        堆（heap ）： 堆是用于存放进程运行中被动态分配的内存段，它的大小并不固定，可动态扩张或缩减。
                    当进程调用malloc 等函数分配内存时，新分配的内存就被动态添加到堆上（堆被扩张）；
                    当利用free 等函数释放内存时，被释放的内存从堆中被剔除（堆被缩减）
                    
        栈(stack) ：栈又称堆栈，是用户存放程序临时创建的局部变量，也就是说我们函数括弧“{} ”中定义的变量
                    （但不包括static 声明的变量，static 意味着在数据段中存放变量）。
                     在函数被调用时，其参数也会被压入发起调用的进程栈中，并且待到调用结束后，函数的返回值也会被存放回栈中。
                    由于栈的先进先出特点，所以栈特别方便用来保存/ 恢复调用现场。从这个意义上讲，
                    我们可以把堆栈看成一个寄存、交换临时数据的内存区.
                    
    高地址还储存了命令行参数及环境变量.
    
    (1) 程序被执行时, 操作系统将可执行模块(可执行程序)拷贝到内存的程序映像(program image)中去.
    (2) 正在执行的程序实例被称为进程: 当操作系统向内核数据结构中添加了适当的信息, 并为运行程序代码分配了必要的资源之后, 
        程序就变成了进程. 这里所说的资源就包括分配给进程的地址空间和至少一个被称为线程(thread)的控制流. 
        
    可执行文件段(磁盘映像)
    
     内存程序映像  	进程地址空间    可执行文件段
      code(text)     code(text)    code(text)
        data	       data	         data 
        bss  	       data	         bss
       heap	           data           - 
       stack          stack           - 
         
    内存程序映像和进程地址空间之比较:
        1. 它们的代码段(code)和栈(stack)相互一致
        2. 而内存程序映像的data, bss, heap对应到进程地址空间的data段. 也就是说, data, bss, heap会位于一个连续的地址空间中, 
            code和stack可能位于另外的地址空间.
            
    正因为内存程序映像中的各段可能位于不同的地址空间中, 它们不一定位于连续的内存块中. 操作系统将程序映像映射到地址空间时, 
    通常将内存程序映像划分为大小相同的块(也就是page, 页). 只有该页被引用时, 它才被加载到内存中. 
    不过对于程序员来说, 可以视内存程序映像在逻辑上是连续的.
    
    内存程序映像和可执行文件段之比较:
    
        1. 明显, 前者位于内存中, 后者位于磁盘中.
        2. 内存程序映像中的code, data, bss段分别对应于可执行文件段中的code, data, bss段.
        3. 堆栈在可执行文件段中是没有的, 因为只有程序被加载到内存中运行时才会被分配堆栈.
        4. 虽然可执行文件段中包含了bss, 但bss并不被储存在位于磁盘中的可执行文件中
        
    只有程序被加载到内存中时, 被初始化为全0的静态数据所对应的内存空间才被分配, 同时被赋予0值.可以通过ls命令来查看.
```

## size 查看程序内存映像大小

### size简介

``` shell

    size 查看的是内存映像信息,但无法查看堆栈的大小.
    ls   查看的是磁盘的程序大小,包含code(text)和data,不包含bss 
    
    (1) 如果静态变量被初始化为非0, 磁盘映像(ls)要大于内存映像(size).
    (2) 若静态变量被初始化为全0或未初始化, 磁盘映像要小于内存映像.因为初始化为全0或未初始化的静态变量都保存在bss段,
        磁盘映像是不包含bss段的.
    
    位于磁盘中的可执行程序中不关包含磁盘映像的内容(code, data, bss),还包括:符号表,调试信息,针对动态库的链接表等内容. 
    但这些内容在程序被执行时是不会被加载到内存中的.
    使用file命令查看可执行文件的信息. 使用strip命令删除可执行程序中的符号表.
    
```

### size 使用

``` shell

    $ size ./fault_seg   (显示存储在磁盘可执行文件的信息)
    结果:
                            
     text    data     bss     dec     hex filename
     1115     544       8    1667     683 fault_seg
     
     dec: text+data+bss十进制表示的总和
     hex: text+data+bss十六进制表示的总和
     
    由于堆栈是在程序执行时动态分配的, size无法显示它们的大小.
    
    不管静态数据有没有被初始化, 加载到内存中的程序映像大小(text+data+bss)是不变的. 
       它们之间的区别只是data和bss段大小的不同( 影响磁盘文件的大小通过ls命令体现)
    
```

## fuser 显示文件使用者

### fuser 概述

``` shell

    fuser 可以找出正在使用文件和网络套接字的进程.和lsof有相同的功能
    fuser 还可以一次性杀掉那些正在访问指定文件的进程
    
```

### fuser 参数

``` shell

    -v: 列出详细信息
    -u: 进程所属的用户
    -k: kill 所有跟特定文件相关的进程
    
    
```

### fuser 示例

- 得到正在使用指定文件的进程

``` shell

    $  fuser ./sengine
    
    结果:
      /yytd/sengine/sengine: 20114e
      
    每个进程号后面都跟随一个字母，该字母指示进程如何使用文件.
     c：进程的工作目录.
     e：该文件为进程的可执行文件(即进程由该文件拉起)
     f：该文件被进程打开，默认情况下f字符不显示。 
     F：该文件被进程打开进行写入，默认情况下F字符不显示。 
     r：该目录为进程的根目录。 
     m：进程使用该文件进行内存映射，抑或该文件为共享库文件，被进程映射进内存。
    
```

- 列出进程的详细信息

``` shell

    $  fuser -v  /yytd/logs/sengine/sengine_stdout.log
    
    结果:
                                              USER    PID    ACCESS      COMMAND
    /yytd/logs/sengine/sengine_stdout.log:    root   2872     F....   supervisord
      
```

- 列出进程所属的用户

``` shell

    $  fuser -u  /yytd/logs/sengine/sengine_stdout.log
    
    结果:
    /yytd/logs/sengine/sc_stdout.log:  2872(root)
      
```

- 杀死所有正在访问指定文件的进程 

``` shell

    $  fuser -k  /yytd/logs/sengine/sengine_stdout.log
    
    结果:
    /yytd/logs/sengine/sc_stdout.log:  2872(root)
      
```

- 列出所有正在访问端口的进程 

``` shell

    $  fuser -v 1883/tcp
    
    结果:
                         USER        PID ACCESS COMMAND
    1883/tcp:            root       1010 F.... beam.smp
      
```

## lsof

### lsof 简介

``` shell
    
    lsof: list open files
    lsof [option] [filename]
    在linux环境下,任何事物都以文件的形式存在,通过文件不仅仅可以访问常规数据,还可以访问网络连接和硬件.
    如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等
    在终端输入lsof来显示系统打开的文件,需要访问核心内存和各种系统文件,必须以 root 用户运行.
      
```

### lsof 参数

``` shell
    
    -a 各个参数的条件都必须满足(AND)
        缺省的情况是显示匹配任何一个参数 (OR) 的文件
    -r 重复执行lsof操作(不管该文件是否打开)
        # lsof -r 2 sc_stdout.log   (2秒执行一次)
    -c<进程名> 列出指定进程所打开的文件
    -g  列出GID号进程详情
    -d<文件号> 列出占用该文件号的进程
        # lsof -d 2-3   ->根据文件描述范围(2到3)列出文件信息
    +d<目录>  列出目录下被打开的文件
    +D<目录>  递归列出目录下被打开的文件
    -i<网络条件>  列出符合网络条件的进程。（4、6、协议、:端口、 @ip ）
        lsof -i[46] [protocol][@hostname|@hostaddr][:service|:port]
          4 6 --> IPv4 or IPv6
          protocol --> TCP or UDP
          hostname --> Internet host name
          hostaddr --> IPv4地址
          service --> /etc/service中的 service name (可以不止一个)
          port --> 端口号 (可以不止一个)
    -p<进程号> 列出指定进程号所打开的文件
        # lsof -p 1,2,3
    -u  列出某个用户打开的文件信息
    -h 显示帮助信息
    -v 显示版本信息
      
```

### lsof示例

- 查找某个文件相关的进程

``` shell
    
    # lsof /yytd/logs/sengine/sc_stdout.log
    结果:
      COMMAND    PID  USER    FD    TYPE  DEVICE  SIZE/OFF     NODE                  NAME
    supervisord  2872  root   20w   REG    253,1  2658184    74470608   /yytd/logs/sengine/sc_stdout.log 
      
    COMMAND：进程的名称
    PID：进程号
    USER：进程所有者
    FD：文件描述符
        1.fd+文件状态模式 例如(3u)
        文件状态模式:
            （1）u：表示该文件被打开并处于读取/写入模式
            （2）r：表示该文件被打开并处于只读模式
            （3）w：表示该文件被打开并处于
            （4）空格：表示该文件的状态模式为unknow，且没有锁定
            （5）-：表示该文件的状态模式为unknow，且被锁定
            
         2.cwd:表示应用程序的当前工作目录,这是该应用程序启动的目录,除非它本身对这个目录进行更改.
         3.txt: 程序代码,如应用程序(可执行二进制文件本身)或共享库
         
    TYPE:
        REG: 文件
        DIR: 目录
        CHR : 字符
        BLK: 块设备
        UNIX: UNIX 域套接字
        
        
    DEVICE：指定磁盘的名称
    SIZE：文件的大小
    NODE：索引节点（文件在磁盘上的标识）
    NAME：打开文件的确切名称
    
```

- 列出目录下被打开的文件的信息

``` shell
    
    1.非递归,只显示当前目录的被进程打开的文件
        # lsof +d /yytd/logs/sengine/
        结果:
          COMMAND    PID  USER    FD    TYPE  DEVICE  SIZE/OFF     NODE                  NAME
        supervisord  2872  root   20w   REG    253,1  2658184    74470608   /yytd/logs/sengine/sc_stdout.log
         
    2.递归显示指定目录下所有被进程打开的文件
        # lsof +D /yytd/logs/
        结果:
            COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
            superviso 16229 root    6w   REG  253,1 12269096 798680 /yytd/logs/sengine/sengine_stdout.log
            superviso 16229 root    9w   REG  253,1 59042262 798671 /yytd/logs/emqttd/emqttd_stdout.log
      
    
```

- 查看端口22号相关的文件

``` shell
    
    # lsof -i :22
    结果:
        COMMAND   PID USER    FD   TYPE  DEVICE SIZE/OFF NODE      NAME
        sshd     1533 root    3u  IPv4   18803      0t0  TCP      *:ssh (LISTEN)
        sshd     1533 root    4u  IPv6   18805      0t0  TCP      *:ssh (LISTEN)
        sshd    11768 root    3u  IPv4 6250323      0t0  TCP     localhost.localdomain:ssh->10.0.7.140:53090 (ESTABLISHED)
        sshd    14433 root    3u  IPv4  217905      0t0  TCP     localhost.localdomain:ssh->10.0.5.98:50689 (ESTABLISHED)
    
```

- 恢复删除的文件

``` shell
    
    1. 用lsof 命令 加入 删除文件的参数,得到对应的 /proc目录下进程号目录的信息
        # lsof /var/log/message
        结果:
            COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF      NODE NAME
            abrt-watc  831 root    4r   REG  253,1    79925 149712343 /var/log/messages
            rsyslogd  1174 root    6w   REG  253,1    79925 149712343 /var/log/messages
            
    2.再通过查看 /proc/PID/fd/FD 信息
        # cat /proc/831/fd/6 > /var/log/message
```

- 列出某个用户打开的文件信息

``` shell
    
    # lsof -u username
    
    显示出除了某个用户以外,所有的文件信息
    # lsof -u ^usename
```

- 列出目前连接主机peida.linux上端口为：20，21，22，25，53，80相关的所有文件信息，且每隔3秒不断的执行lsof指令
``` shell
 
  # lsof -i @peida.linux:20,21,22,25,53,80  -r  3
```