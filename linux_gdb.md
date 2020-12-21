# linxu gdb 使用
```shell
    1. attach 到已经运行的 pid 上
        > gdb attach pid
        实例:
             > gdb attach 125240
             
    2. 打印当前程序的所有线程的堆栈信息
        (gdb) thread apply all bt
        结果:
            Thread 1 (Thread 0x7fbb922b37c0 (LWP 13041)):
            #0  0x00007fbb8ae53f90 in __nanosleep_nocancel () from /lib64/libc.so.6
            #1  0x00007fbb8ae84884 in usleep () from /lib64/libc.so.6
            #2  0x00007fbb8cf018c1 in ?? ()
            #3  0x0000000000000000 in ?? ()
            
    3. 切换到对应的线程
        (gdb) thread 1
        结果:
            [Switching to thread 1 (Thread 0x7fbb922b37c0 (LWP 13041))]
            #0  0x00007fbb8ae53f90 in __nanosleep_nocancel () from /lib64/libc.so.6
       打印这个线程堆栈
        (gdb) bt
        结果:
            #0  0x00007fbb8ae53f90 in __nanosleep_nocancel () from /lib64/libc.so.6
            #1  0x00007fbb8ae84884 in usleep () from /lib64/libc.so.6
            #2  0x00007fbb8cf018c1 in ?? ()
            #3  0x0000000000000000 in ?? ()
            
    4. 切换到对应的函数(堆栈)
        (gdb) fr 1      (Fram : 当前堆栈帧)
        或则
        (gdb) f 1
        
    5. 打印出对应的信息 
        (gdb) p *pMutex
        结果:
            $2 = {..... _owner = 8684}  // 其中锁被占用的线程号为 8684
            
    6. 打印出所有线程的序号(这个可以知道当前的线程)
        (gdb) info thread
        结果:
              Id   Target Id         Frame 
            * 1    Thread 0x7fbb922b37c0 (LWP 13041) "nb_stab_test" 0x00007fbb8ae84884 in usleep ()　　// 当前运行到线程数
              2    Thread 0x7fbb922b37c0 (LWP 8684) "nb_stab_test" 0x00007fbb8ae84884 in usleep ()
              线程号 8684 对应的就是序号为 2 
    7. 切换到对应的线程序号为 2
        (gdb) thread 2
        在进行堆栈的打印，变量的打印
```

## gdb 概要
```shell
    1. 进行 gdb 程序调试时,会有三种可执行文件
            (1) gcc 编译时带有 -g 编译选项，可执行程序带有额外的调试信息, 这一般是拥有程序源码.
                例如:
                    (gdb) bt
                    #0  0x00007f3d3a20cf47 in pthread_join () from /lib64/libpthread.so.0
                    #1  0x000000000042a20a in main (argc=1, argv=0x7fff3e077618)    // 这边 argc, argv 都有对应的值
                        at /home/zhangtx/work_area/demo/nb_stab_test/src/main.cpp:141
                        
            (2) gcc 编译时不带有 -g 编译选项，可执行程序不带有额外的调试信息,但有符号信息
                例如:
                    (gdb) bt
                    #0  0x00007f7ce5af4f47 in pthread_join () from /lib64/libpthread.so.0
                    #1  0x000000000042a20a in main ()  // 参数名和对应的参数值都没有
                    
            (3) 可执行程序编译后进行 strip 操作(> strip 可执行文件)，既不带有额外调试信息，也无符号信息
                    例如:
                        (gdb) bt
                        #0  0x00007f3180ee2fad in nanosleep () from /lib64/libc.so.6
                        #1  0x00007f3180ee2e44 in sleep () from /lib64/libc.so.6
                        #2  0x000000000042a105 in ?? 
                        #3  0x00007f31811f2dd5 in start_thread () from /lib64/libpthread.so.0
                        #4  0x00007f3180f1c02d in clone () from /lib64/libc.so.6\
                        
    2. 针对第二种可执行文件(不带有额外的调试信息,但有符号信息),可以采用寄存器的方向
            %rax 作为函数返回值使用。
            %rsp 栈指针寄存器，指向栈顶。
            %rbp 栈指针寄存器，指向栈底。
            %rdi、%rsi、%rdx、%rcx、%r8、%r9用作保存函数参数保存，依次对应第一个参数、第二个参数、第三个参数…..
            从上面寄存器原理来看，就算是没有额外调试信息，函数的前 6 个参数我们还是可以直接从参数寄存器中读取，操作如下：
            
            (gdb) bt
            #0  0x0000000000428740 in foo(int, int, int, int, int, int, int, int) ()
            #1  0x00000000004287e8 in main ()
            (gdb) fr 0
            #0  0x0000000000428740 in foo(int, int, int, int, int, int, int, int) ()
            (gdb) i r
            rax            0x4287a5 4360101
            rbx            0x0      0
            rcx            0x1      1
            rdx            0x1      1
            rsi            0x1      1
            rdi            0x1      1
            rbp            0x7fffffffe370   0x7fffffffe370
            rsp            0x7fffffffe370   0x7fffffffe370
            r8             0x1      1
            r9             0x1      1
            r10            0x7fffffffdd60   140737488346464
            r11            0x7ffff0d2cef0   140737233735408
            r12            0x412ca0 4271264
            r13            0x7fffffffe480   140737488348288
            r14            0x0      0
            r15            0x0      0
            rip            0x428740 0x428740 <foo(int, int, int, int, int, int, int, int)+4>
            eflags         0x202    [ IF ]
            cs             0x33     51
            ss             0x2b     43
            ds             0x0      0
            es             0x0      0
            fs             0x0      0
            gs             0x0      0
            
            如果想要看超过 6 其他的参数,通过反编译:
            (gdb) set disassembly-flavor intel
            (gdb) disassemble main  // 反编译 main 函数(也可以反编译其他的函数)
            Dump of assembler code for function main:
               0x00000000004287a5 <+0>:     push   rbp
               0x00000000004287a6 <+1>:     mov    rbp,rsp
               0x00000000004287a9 <+4>:     sub    rsp,0x20
               0x00000000004287ad <+8>:     mov    DWORD PTR [rbp-0x4],edi
               0x00000000004287b0 <+11>:    mov    QWORD PTR [rbp-0x10],rsi
               0x00000000004287b4 <+15>:    mov    DWORD PTR [rsp+0x8],0x1
               0x00000000004287bc <+23>:    mov    DWORD PTR [rsp],0x1
               0x00000000004287c3 <+30>:    mov    r9d,0x1
               0x00000000004287c9 <+36>:    mov    r8d,0x1
               0x00000000004287cf <+42>:    mov    ecx,0x1
               0x00000000004287d4 <+47>:    mov    edx,0x1
               0x00000000004287d9 <+52>:    mov    esi,0x1
               0x00000000004287de <+57>:    mov    edi,0x1
               0x00000000004287e3 <+62>:    call   0x42873c <_Z3fooiiiiiiii>
               0x00000000004287e8 <+67>:    mov    edi,0x2710
               0x00000000004287ed <+72>:    call   0x412200 <HPR_Sleep@plt>
               0x00000000004287f2 <+77>:    jmp    0x4287e8 <main+67>
            End of assembler dump.
            
        
        https://www.cnblogs.com/findumars/p/7545818.html

```

## 注意
```shell
    1. 使用时要进行 -g 操作
```

## 常用命令

```shell
    1. (gdb) break 10 thread 2 当线程 2 到达源代码的 10 行的时候，停止执行
       (gdb) break 10 thread 2 if x == y 当线程 2 到达源代码 10 行，且变量 x 和变量 y 相等的时候，停止执行
       
    2. (gdb) thread apply tid/all args 向指定的线程发送指定的指令。常用的指定是查看所有线程的调用堆栈 thread apply all bt ，
       这个指令与 pstack 命令有些相似

```

## 查看线程死锁的问题
```shell
    1. 第一步打印出所有线程的堆栈信息 bt
        (gdb) thread apply all bt
```
