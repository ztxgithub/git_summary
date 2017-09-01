
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
    
    -e trace=file 只跟踪有关文件操作的系统调用. 
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
    
```

## ltrace 跟踪进程调用库函数的情况

### ltrace简介

``` shell
    ltrace: A library call tracer 
    
         
```