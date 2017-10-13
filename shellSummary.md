# 平台相关FSU

- 将FSU恢复成出厂配置

``` shell

    1.$ rm -rf /media/sd/opt/*
    2.$ cp /media/sd/CASS/.OS.bin /media/sd/CASS/UPDATE
    3.$ mv /media/sd/CASS/UPDATE/.OS.bin /media/sd/CASS/UPDATE/OS.bin 
    4.$ reboot
    
    将云平台的升级回FSU铁塔
    1.$ rm -rf /media/sd/opt/*
    2.$ mv /media/sd/CASS/UPDATE/OS.bin /media/sd/CASS/UPDATE/
    3.$ reboot
	
```

- 查看平台的FSU有没有升级成功

``` shell
    
    失败：产生 /media/sd/CASS/.SD_update_fail　文件
	
```

- dbclient 远程登陆
``` shell
    
   跟ssh一样
   $ dbclient root@192.0.0.65
	
```


# BBDS,FSU 命令专用

- 在BBDS中显示内核的打印信息

``` shell

	$ echo 7 > /sys/module/Netlink_HAL/parameters/debug
	
	$ tail -f /var/log/messages 
	
```

- iostat 看linux系统的io操作(读写sd卡，磁盘等s)

``` shell

	$ iostat [ 选项 ] [ <时间间隔> [ <次数> ] ]

	$ iostat -k 1    (以每秒KB为单位显示,一秒刷新一次)
	
	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.99    0.00    1.98    0.00    0.00   97.03

	Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
	mmcblk0           6.93         0.00        27.72          0         28
	mmcblk0p1         6.93         0.00        27.72          0         28
	
	$ iostat --help
	
```

- lsof

``` shell

   1. $ lsof
	
	PID          程序名						打开文件
	391     /opt/CASS/BIN/battery   /media/sd/CASS/battery_data.db
	
	lsof 可以看出某个进程打开的文件和动态链接库等
	
   2. 查询指定的进程PID(23295)打开的文件
   
    $ lsof -p 23295
    
   3. 查询指定的进程(可执行二进制文件)打开的文件
   
    $ lsof -c sengine
          
   4. 查询port端口现在运行什么程序
         (1)
            $ lsof -i :port   得出程序的 pid
            $ ps aux | grep pid  得出具体的程序名
         (2)
            $ netstat -tnulnp | grep port
         
```


- 时刻监测一个命令的运行结果

``` shell

	$ watch[参数][命令]
	
	$ watch -n 1 ls -l  (1秒钟刷新 ls -l 命令的结果)
	
```


- FSU，BBDS中获取linux系统的版本

``` shell

	$ cat /proc/version
	
```

- 查看 Centos的版本号

``` shell

	$ cat /etc/redhat-release
	
```

- 

- FSU中 udhcpc 进程

``` shell

	$ udhcpc -S -i eth1 -p /var/run/udhcpc.pid
	
	参数:
	    -S,--syslog             Log to syslog too
	    -i,--interface IFACE    Interface to use (default eth0)
	    -p,--pidfile FILE       Create pidfile
	
```

- 将后台的程序打印到显示器上 

``` shell

    console_redirect
	
```

- 查看文件夹下的大小 du(disk usage)

``` shell

	$ du -h folder     :以人类可读的方式递归显示该folder及子目录的大小
	
	$ du -sh folder    :只显示folder文件夹包含的大小
	
```

- 查看linux系统是64还是32位

``` shell

	$ file /bin/ls
	/bin/ls: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, 
	interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, 
	BuildID[sha1]=eca98eeadafddff44caf37ae3d4b227132861218, stripped
	
```

- 查看Core文件是由哪个程序生成

``` shell

	$ file core-fault_seg
	结果:
    core-fault_seg: ELF 64-bit LSB core file x86-64, version 1 (SYSV), SVR4-style, from './fault_seg'
	
```

- linux 获取主机开机时的运行时间 uptime

``` shell

	$ uptime
	15:20:50(开机时的开始时间) up 1 min(运行了多少时间:1分钟),  1 users,  load average: 1.61, 0.48, 0.16
	15:15:53 up  1:40(运行1个小时40分钟),  3 users,  load average: 3.32, 3.50, 3.43
	
	$ cat /proc/uptime
	6135.47(系统开机时总共运行了多少秒) 4043.31(开机时系统空闲多少秒)
```

- 将某个文件设置内容为0

``` shell

	$ cat /dev/null > filename   ( >:代表全部替换， >> :代表追加 )
	
	或者
	
	$ > filename
```

- 通过 sz 命令传输有问题可以用 scp

``` shell

	$ scp CASSFSUService_Cloud_2017-04-27.log zhangtx@192.168.0.2:~/
	
```

- ulimit:获取和设置该用户权限下的限制(比如一个程序能够打开的最多的文件描述符的个数，一个栈最大空间等)

``` shell

    描述内容:
        ulimit 用于限制 shell 启动进程所占用的资源,作为临时限制，ulimit 可以作用于通过使用其命令登录的 shell 会话，
        在会话终止时便结束限制，并不影响于其他 shell 会话。而对于长期的固定限制，
        ulimit 命令语句又可以被添加到由登录 shell 读取的文件中，作用于特定的 shell 用户。
        ulimit 限制的是当前 shell 进程以及其派生的子进程。举例来说，如果用户同时运行了两个 shell 终端进程，
        只在其中一个环境中执行了 ulimit – s 100，则该 shell 进程里创建文件的大小收到相应的限制，
        而同时另一个 shell 终端包括其上运行的子程序都不会受其影响：
        ulimit 修改不加指定的(-H和-S),则会同时修改 hard” and “soft” 的值
        
        一般情况下,修改完ulimit设置后都要重启一下进程,可以通过 $ cat /proc/pid/limits 看是否有变化
    

	$ ulimit -a (显示所有的信息)
	
	$ ulimit -s (显示The maximum stack size)
	
	$ ulimit -Hu (用户最大的可用进程数(一旦设置不能增加.))
	
```

# linux 命令专用

- 搜索某个命令的安装包

``` shell

	$ sudo yum search rz
	
	lrzsz.x86_64 : The lrz and lsz modem communications programs
	
```

- 安装rz,sz命令,使SercureCRT能够使用托文件

``` shell

	$ sudo yum install lrzsz.x86_64
	
```

- 内存占用率

``` shell

	$ ps -e -o 'rsz,pid,comm,args,pcpu,vsz,stime,user,uid' | sort -nr
	
	参数：
	    -e:等于-A,　Select all processes：选择所有的进程
	    -o:  -o format:指定输出的格式及内容
	    
	rsz:实际的物理内存
	
```

- telnet 功能

``` shell

    查看某个ip的port是否打开
    
    telnet ip port
    例如:
    telnet 223.93.172.225 7000 (什么都没显示则该port正常打开)
    
     telnet 223.93.172.225 7009 
     
     该端口没有打开提示(telnet: can't connect to remote host (223.93.172.225): Connection refused)
	
```

- grep 

``` shell

    grep命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来。
    grep全称是Global Regular Expression Print
    
    没有加任何参数只能使用基本正则表达式,像 ? , +, {3,4}无法使用
    要使用扩展正则表达式--> grep -E "[0-9]{3,4}" filename.txt
	
```

### 备用命令

 - man页面所属的分类标识(常用的是分类1和分类3)

``` shell

    分类1: shell 命令帮助文档
           $ man 1 ulimit
    
    分类3: 一些常用函数申明
           $ man 3 ulimit
          
	
```

- whereis :查看shell命令(二进制可执行文件)的位置

``` shell

    $ whereis supervisord
    
    结果:
        supervisord: /usr/bin/supervisord /etc/supervisord.conf
         
```

- ls 

``` shell

    1. 按文件的修改时间排序
        $ ls -lt 
            -l : 每一个file一行
            -t: 按文件修改日期
            
    2. 给每一个文件前面加序号
        $ ll | cat -n
         
```

- 给文件增加别名 (ln)

``` shell

    1.创建一个软连接,删除源文件，另一个无法使用
    
               源文件    软连接名字
    $ ln -s src_file  dst_symlink
    
    2.创建一个硬链接,删除一个,另一个继续能用
    $ ln src_file dst_hardlink
    
         
```

- Linux 输入输出重定向

``` shell

    在shell中,文件描述符 0:标准输入stdin 
             文件描述符 1:标准输出stdout 
             文件描述符 2:标准错误stderr
          
    test.sh 内容:
        t
        date
    
    1. $./test.sh > test1.log
         date的执行结果被重定向到log文件中了，而t无法执行的错误则只打印在屏幕上
         
    2. $ ./test.sh > test2.log 2>&1  == $ ./test.sh 1> test2.log 2>&1 
        stderr和stdout的内容都被重定向到log文件中
    
    > 就相当于 1> 也就是重定向标准输出,不包括标准错误.通过2>&1,就将标准错误重定向到标准输出了，
    那么再使用>重定向就会将标准输出和标准错误信息一同重定向.如果只想重定向标准错误到文件中，则可以使用2> file。
    
    
    cmd >a 2>a 和 cmd >a 2>&1 为什么不同？
    cmd >a 2>a ：stdout和stderr都直接送往文件 a ，a文件会被打开两遍，由此导致stdout和stderr互相覆盖。
    cmd >a 2>&1 ：stdout直接送往文件a ，stderr是继承了FD1的管道之后，再被送往文件a 。a文件只被打开一遍，就是FD1将其打开.
         
```

- bash 快捷键

``` shell

    Ctl-w   删除当前光标到前边的最近一个空格之间的字符
    Ctl-r   查找历史shell命令
         
```

- find 查找

``` shell

    1.只查找文件夹名为 "mqtt*"
        find / -name "mqtt*" -type d
        
         
```

- top

``` shell

    P：根据CPU使用百分比大小进行排序。
    M：根据驻留内存大小进行排序.
    1 : 可监控每个逻辑CPU的状况
    b : 对进程状态为R进行加亮显示
    k: 终止一个进程
        按'k',再输入pid
    s: 改变两次刷新之间的延迟时间（单位为s）
    l: 切换显示平均负载和启动时间信息
    m 切换显示内存信息
    t 切换显示进程和CPU状态信息
    c 切换显示命令名称和完整命令行
    T 根据时间/累计时间进行排序
    
    
    用top查看单一的进程情况
    $ top -p `pidof sengine`
    
    结果:
            
     PID   USER  PR   NI    VIRT      RES     SHR   S  %CPU   %MEM     TIME+     COMMAND 
    15629  root  20   0    1266924   49668    4732  S   0.0   1.3     21:20.74    sengine 
    
    参数:
        -c:显示完整command
        -d<秒数> : 设置更新时间
        -u<用户名> :显示指定用户名
        -n<次数> : 显示2次
        
        
    显示内容:
       第三行  %Cpu(s):  0.4 us,  0.9 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
         
         5.9%us — 用户空间占用CPU的百分比。
         3.4% sy — 内核空间占用CPU的百分比。
         0.0% ni — 改变过优先级的进程占用CPU的百分比
         90.4% id — 空闲CPU百分比
         0.0% wa — IO等待占用CPU的百分比
         0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
         0.2% si — 软中断（Software Interrupts）占用CPU的百分比
         
         
       第四行 KiB Mem:  32949016 total,  14411180 used, 18537836 free, 169884 buffers
       
             32949016k total — 物理内存总量（32GB）
             14411180k used — 使用中的内存总量（14GB）
             18537836k free — 空闲内存总量（18GB）
             169884k buffers — 缓存的内存量 （169M）
             
       第五行 KiB Swap:  0 total, 0 used,  0 free.  1644772 cached Mem
            0 total — 交换区总量
            0k used — 使用的交换区总量（0K）
            0 free — 空闲交换区总量 
            1644772k cached — 缓冲的交换区总量（1.6GB）
            
       第七行以下：各进程（任务）的状态监控，项目列信息说明如下：
            PID — 进程id
            USER — 进程所有者
            PR — 进程优先级 ,PR越小优先级越高
            NI — nice值。负值表示高优先级，正值表示低优先级, NI越小优先级越高
            VIRT — 进程使用的虚拟内存总量，单位kb。
            RES — 进程使用的、未被换出的物理内存大小，单位kb
            SHR — 共享内存大小，单位kb
            S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
            %CPU — 上次更新到现在的CPU时间占用百分比
            %MEM — 进程使用的物理内存百分比
            TIME+ — 进程使用的CPU时间总计，单位1/100秒
            COMMAND — 进程名称（命令名/命令行）
             
```

- pidof

``` shell

    查看进程名对应的pid
    
    $ pidof sengine
    
    结果:
        16541
    
         
```

- pmap : 进程用了多少内存(虚拟内存)

``` shell

    pmap pid
    
    结果:
         total  612040K   (虚拟内存)
         
```

- exec 

``` shell

    作为 find 命令的一个选项, -exec 选项通常要以';'结尾
    (1)在当前目录下(包含子目录),查找所有txt文件并找出含有字符串”bin”的行
      find ./ -name “*.txt” -exec grep “bin” {} \;
      
    (2)在当前目录下(包含子目录),删除所有txt文件
      find ./ -name "*ls" -exec rm {} \;   ('{}'与'\' 之间要有空格,'\'紧接加';')
      
    3. 要在找到结果中进行2次 exec 
      find ./ -name "ba*" -exec cat {} \; -exec grep "test" {} \;
      
```

- ps 

``` shell

    ps: process status,该命令可以确定有哪些进程正在运行和运行的状态,进程是否结束,进程有没有僵死,哪些进程占用了过多的资源等
    
    进程的状态:
       1. 运行(正在运行或在运行队列中等待) ---> R (runnable) 
       2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)   ---> S  (sleeping)
       3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)  ---> D (uninterruptible sleep)
       4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放) ----> Z (zombie)
       5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行)   -----> T  (traced or stopped)
       
    参数:
        1. -A: 显示所有进程信息(进程名不是全路径)
            $ ps -A
            
        2. -x : 显示没有控制终端的进程
           -a : 所有进程(all)
        
            (1)列出目前所有的正在内存当中的程序
                $ ps aux
                结果:
                USER   PID  %CPU  %MEM    VSZ     RSS   TTY  STAT  START   TIME    COMMAND
                root  23015  0.0   0.1   762996  9800    ?    Sl   Sep26   0:24    /home/yytd/sengine/sengine
                
                %CPU：该 process 使用掉的 CPU 资源百分比
                %MEM：该 process 所占用的物理内存百分比
                VSZ ：该 process 使用掉的虚拟内存量 (Kbytes),它包含可执行二进制(binary),动态链接库,堆栈总大小
                RSS ：该 process 实际的物理内存 (Kbytes),包含实际在内存中的可执行二进制(binary),动态链接库,堆栈总大小
                      RSS一般会小于VSZ
                TTY ：该 process 是在那个终端机上面运作,若与终端机无关，则显示 ?
                      tty1-tty6 是本机上面的登入者程序,若为 pts/0 等等的，则表示为由网络连接进主机的程序.
                      
                START：该 process 被触发启动的时间
                TIME ：该 process 实际使用 CPU 运作的时间
                COMMAND：该程序的实际指令
                
            (2) 按cpu利用率排序(--sort 后面跟着'-'是降序)
                $ ps aux --sort -pcpu (降序)
                $ ps aux --sort +pcpu (升序)
                
            (3) 按内存利用率排序(--sort 后面跟着'-'是降序)
                  $ ps aux --sort -pmem (降序)
                  $ ps aux --sort +pmem (升序)
                  
        3. -L :显示进程中的线程(-L pid)
            $ ps -L 23015
            
              PID   LWP TTY      STAT   TIME COMMAND
            23015 23015 ?        Sl     0:00 /home/yytd/sengine/sengine
            23015 23016 ?        Sl     0:01 /home/yytd/sengine/sengine
            23015 23017 ?        Sl     0:04 /home/yytd/sengine/sengine
            23015 23018 ?        Sl     0:08 /home/yytd/sengine/sengine
            23015 23019 ?        Sl     0:00 /home/yytd/sengine/sengine
            23015 23021 ?        Sl     0:00 /home/yytd/sengine/sengine
            23015 23022 ?        Sl     0:03 /home/yytd/sengine/sengine
            23015 23023 ?        Sl     0:00 /home/yytd/sengine/sengine
            23015 23024 ?        Sl     0:02 /home/yytd/sengine/sengine
            23015 23025 ?        Sl     0:02 /home/yytd/sengine/sengine
            
            LWP:(thread ID)
            
            可以用pstack 更加详细
            
         4. -o format : 输出指定的选项
                format的值: User, PID, %CPU, %MEM, VSZ, RSS, TTY, 
                           STAT, START, TIME 和 COMMAND, lstart(详细的时间信息)
                $ ps aux -o pid
                
         5. u :用来决定以针对用户的格式输出，由User, PID, %CPU, %MEM, VSZ, RSS,
               TTY, STAT, START, TIME 和 COMMAND这几列组成。
               
         6. -u username : 指定某个用户的进程情况
                $ ps -u yytd u (显示yytd用户进程情况,按用户的格式输出)
                结果:
                USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
                yytd      2498  0.0  0.0   4296   588 ?        Ss   Sep20   0:02  memsup
                yytd      2499  0.0  0.0   4296   360 ?        Ss   Sep20   0:00  cpu_sup
           
         7. 不加任何参数的ps 是 只显示控制终端的进程
            
```

- cmp filename1 filename2 :二进制文件的比较
     

### 用户管理

- 用户

``` shell

    1.添加用户
        $ useradd -m username
            -d： 指定用户的主目录
            -m： 如果存在不再创建，但是此目录并不属于新创建用户；如果主目录不存在，则强制创建； -m和-d一块使用。
            -s： 指定用户登录时的shell版本
            
    2.设置用户密码
        $ passwd username
        
    3.删除用户
        $userdel -r username
            -r: 同时删除用户的主目录 /home/username
         
```

- 用户组

``` shell

    1.添加用户到特定的组中
        $ usermod -G groupNmame username
            
    2.修改用户所属于的组
        $ usermod -g groupName username
        
    3.删除用户
        $userdel -r username
            -r: 同时删除用户的主目录 /home/username
         
```

### linux 环境变量

``` shell

    /etc/profile: 用来设置系统环境参数,比如$PATH. 这里面的环境变量是对系统内所有用户生效的.
    /etc/bashrc:  这个文件设置系统bash shell相关的东西，对系统内所有用户生效。只要用户运行bash命令，那么这里面的东西就在起作用.
    ~/.bash_profile 是交互式、login 方式进入 bash 运行的，意思是只有用户登录时才会生效。
    ~/.bashrc 是交互式 non-login 方式进入 bash 运行的，用户不一定登录，只要以该用户身份运行命令行就会读取该文件。
         
```

