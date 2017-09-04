
## pstack 显示每个进程的栈跟踪

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
    strace常用来跟踪进程执行时的系统调用和所接收的信号. Linux进程不能直接访问硬件(设备读取磁盘文件，接收网络数据等等),需要
    由用户态模式切换至内核态模式,通过系统调用访问硬件设备.strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间.
    一般比较适用于调试不明的故障,用strace找出系统调用中出错的地方,通常能得到故障发生的线索,特别是与文件有关的错误、参数错误等.
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
    Linux内核会为每一个进程在/proc建立一个以其pid为名的目录用来保存进程的相关信息,而其子目录fd保存的是该进程打开的所有文件的fd。
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
    -n 按照地址/符号值来排序；
    -u 打印出那些未定义的符号；
    -p 按目标文件中遇到的符号顺序显示，不排序
    
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
     符号的值表示该符号需要的字节数。例如在一个c文件中，定义int test，并且该符号在别的地方会被引用，则该符号类型即为C。否则其类型为B.
     
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
           nm libXXX.so |grep CHttpParser(undefined reference 的函数名)
    
```