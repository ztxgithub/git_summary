## 开源协议

```shell
    1. GPL(General Public License) 明确规定，任何源码的衍生产品，如果对外发布，都必须保持同样的许可证,
       即如果是基于 GPL 源码开发基础上你也必须开源
    2. 选择更宽松的许可证，比如 BSD 和 Apache 许可证，LGPL(Lesser General Public License) , 
       不要求衍生代码开源, BSD 和 Apache 之间区别是 BSD 对于每一个修改后的文件，不需要都放置版权说明,
       而 Apache 则必须要
    3. 开源的策略: 整体软件，或者没有替代品的代码库，一定要使用 GPL 许可证； 
                 有替代品的代码库，可以使用限制较少的开源许可证(BSD 或则　Apache)，但是在取到足够市场份额之后，
                 也应该转为 GPL 许可证
                 
    总结:
         如果你在商业公司上班，最好不要使用GPL协议的开源软件，因为它具有“传染性”，并且强制开源，只要引入的某个模块是GPL的，
         它会一直扩展到最上层知道整个项目都强制GPL开源。
         用BSD、Apache或者MIT的开源项目则一般不会有问题，只需要开源项目本身的安全性或者健壮性等其他需求满足公司要求即可。
        
```

- system page size

``` c
    A page, memory page, or virtual page是跟虚拟内存有关,它是一段连续的虚拟内存,
    一个page size 大小与处理器架构有关，一般是4K bytes
    
	
```

- 时间单位

``` c
    毫秒: 单位ms 对应的英语millisecond  1秒= 10^3 毫秒
    微妙: 单位μs 对应的英语microsecond  1秒= 10^6 微秒   1毫秒 = 10^3 微妙
    纳秒: 单位ns 对应的英语nanosecond   1秒= 10^9 纳秒   1毫微妙 = 10^3 纳秒
    
	
```

## 内存问题

- malloc 问题

```shell

    1.如果要申请内存进行测试，只 malloc()　还不够，还要 memset(buf, 1, size) 对该内存区域进行操作，
    　操作系统才真正把内存给程序
    2. 调用 malloc_trim(0) 可以将进程中 free 的内存归还给系统(前提是系统使用 glibc 而不是 uclibc)
```


- [字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)


- 可重入函数

``` c
    多个线程可以同时调用该函数,该函数不会有操作静态变量
    
	
```

- 一个block的大小为512 bytes

- CRC16 = X16+X15+X2+X0

- free 内存分布

``` c

  应用程序(process)实际需要使用时,kernel仍会自动从memory釋放cache給process使用
    
  在释放之前 最好是使用命令 
    $ sync
    
  #释放pagecache
  echo 1 > /proc/sys/vm/drop_caches
	
  #释放dentries與inodes
  echo 2 > /proc/sys/vm/drop_caches
  
    
  echo 3 > /proc/sys/vm/drop_caches
  3 是指释放pagecache、dentries與inodes，也就是釋放所有的cache
  
  这些显式释放Cache虽然不会造成数据错误,但会影响性能,因为Cache被清空了,程序又需要读文件,只能通过IO磁盘操作重新加载Cache中.

```

## linux 32 位和 64 位

```shell
    1. 32 位和 64 位系统在 Linux下基本数据类型的大小都是一样的。只有指针的大小 和 long 不一样 
        sizeof(char):1
        sizeof(short):2
        sizeof(int):4
        
        32 位:
            sizeof(long):4
            sizeof(pointer):4
            
        64 位:
            sizeof(long):8
            sizeof(pointer):8

```

## 系统资源参数

### /proc/sys/fs (内核参数与文件系统有关)

#### 系统最多的文件描述符(这个很重要)

``` c

    1.Linux的/proc/sys/fs/file-max决定了当前内核可以打开的最大的文件句柄数.
    这个是系统限制,限制所有用户打开文件描述符的总和.
    
     一般为内存大小（KB）的10%来计算，如果使用shell，可以这样计算：
        grep -r MemTotal /proc/meminfo | awk '{printf("%d",$2/10)}'
        
      $ cat /proc/sys/fs/file-max
      运行结果:
        1048576
       
      $ sysctl -w fs.file-max=102400 (使用sysctl命令更改也是临时的)
      
      或者
      
      echo "65536" > /proc/sys/fs/file-max
      
      两者作用是相同的，前者改内核参数，后者直接作用于内核参数在虚拟文件系统（procfs, psuedo file system）上对应的文件而已。
      
      如果想要永久修改系统级的最大文件描述符限制
      1. $ vim /etc/sysctl.conf 内容为 fs.file-max = 1048576
      2. $ sysctl -p (使修改内容生效)
      
     
    2.查看当前kernel的打开的文件描述符的数量
    
        $ cat /proc/sys/fs/file-nr
        运行结果:
            (已经分配且在使用)   (已经分配但未使用)    (总数max,支持最大的文件描述符数)
                  2496                0                     1048576

```

#### /proc/sys/fs/epoll/max_user_watches

```shell
    1. 该用户打开的所有 epoll 实例总共能监控的事件的数目，而不是单个 epoll 实例监控的数量    
```

[相关资料](http://www.chengweiyang.cn/2015/11/14/how-to-enlarge-linux-open-files-upper-cell/)

### /etc/security/limits.conf

#### limits.conf文件描述

``` c

    limits.conf文件实际是Linux PAM（插入式认证模块，Pluggable Authentication Modules）中 pam_limits.so 的配置文件,
    突破用户资源的默认限制. limits.conf 和sysctl.conf区别在于limits.conf是针对用户,
    而sysctl.conf是针对整个系统参数配置(限制所有用户打开文件描述符的总和), limits.conf 不能超过 sysctl.conf内的设置大小.


```

#### limits.conf文件格式  <domain>  <type>  <item(resource)>  <value(limit )>

``` c

     domain : 
        username:被限制的用户名
        @groupname: 组名前面加@
        * : 通配符*来对所有用户的进行限制 (ubuntu系统不支持(只能写固定用户名如root),Centos系统支持)
        
     type: soft 的限制不能比har 限制高
        soft : 当前系统生效的设置值
        hard : 系统中所能设定的最大值
        
        -S, soft limit, 软限制，用户可以上调软限制到硬限制
        -H, hard limit, 硬限制，非 root 用户不能修改
        
        
     item(resource):要限制的资源
        core : 限制内核文件的大小 (KB)
            何谓core文件,当一个程序崩溃时，在进程当前工作目录的core文件中复制了该进程的存储图像。
            core文件仅仅是一个内存映象（同时加上调试信息），主要是用来调试的。
            core文件是个二进制文件，需要用相应的工具来分析程序崩溃时的内存映像，
            系统默认core文件的大小为0，所以没有被创建。可以用ulimit命令查看和修改core文件的大小，例如： 
            #ulimit –c 
            0 
            #ulimit -c 1000 
            -c 指定修改core文件的大小，1000指定了core文件大小。也可以对core文件的大小不做限制，如： ulimit -c unlimited 
            注意：如果想让修改永久生效，则需要修改配置文件，如 .bash_profile、/etc/profile或/etc/security/limits.conf 
            
        date : 最大数据大小(KB)
        
        fsize : 最大文件大小(KB)
        
        memlock : 最大锁定内存地址空间(KB)
        
        nofile : 打开文件的最大数目 (常用)
        
        stack - 最大栈大小 (KB)
        
        cpu - 以分钟为单位的 最多 CPU 时间 
        
        noproc - 进程的最大数目(常用)
        
        maxlogins - 此用户允许登录的最大数目 
        
     value(limit ) : limit的大小
            具体的数字
            unlimited(不限制)
        
        
     注意：
        要使 limits.conf 文件配置生效，必须要确保 pam_limits.so 文件被加入到启动文件中。
        修改 /etc/pam.d/login 文件中有：session required /lib/security/pam_limits.so 
        文件 /etc/ssh/sshd_config 中内容一定要有 "UsePAM yes" 这个字符串.
        在Centos 7 中, /etc/pam.d/login 文件中内容有：
        session    required     /usr/lib64/security/pam_limits.so


```

### 在 /etc/profile 文件中

``` c

    ulimit -n 65535  (设置用户最大文件描述符数为65535)

```

### sysctl

#### sysctl简介

``` shell

    sysctl是一个用来在系统运作中查看及调整系统参数的工具, /etc/sysctl.conf就是sysctl的配置文件
         
```

#### sysctl 使用

``` shell

    1. 临时改变某个指定参数的值
       sysctl -w net.ipv4.ip_forward=1  //开启NAT地址转发
       
    2. 显示所有的系统参数
       sysctl -a
       
    3.从指定的文件加载系统参数(使/etc/sysctl.conf文件改变的值立即生效)
       sysctl -p /etc/sysctl.conf
       
    /etc/sysctl.conf 内容参数含义(sysctl -a):
    
        常用参数:
         fs.file-max:
              决定了当前内核可以打开的最大的文件句柄数.这个是系统限制,限制所有用户打开文件描述符的总和.
              
         net.core.somaxconn:
              用来限制监听( LISTEN )队列最大数据包的数量,超过这个数量就会导致链接超时或者触发重传机制
              web 应用中 listen 函数的 backlog 默认会给我们内核参数的 net.core.somaxconn 限制到 128,
              而 nginx 定义的 NGX_LISTEN_BACKLOG 默认为 511，所以有必要调整这个值。对繁忙的服务器,
              增加该值有助于网络性能
              
         net.ipv4.tcp_max_syn_backlog
              这个参数表示 TCP 三次握手第一阶段(建立阶段)接受 SYN 请求队列的最大长度
              将其设置大一些可以使出现 Nginx 繁忙来不及 accept 新连接的情况时,Linux不至于丢失客户端发起的连接请求.
              加大队列长度为8192    
              
         net.ipv4.tcp_rmem
             (net.ipv4.tcp_rmem =4096 32768 262142)
             这个参数定义了用于TCP接收滑动窗口的最小值，默认值，最大值
             推荐使用 net.ipv4.tcp_rmem=4096 87380 16777216
                 
         net.ipv4.tcp_wmem
             (net.ipv4.tcp_wmem =4096 32768 262142)
             这个参数定义了用于TCP发送滑动窗口的最小值，默认值，最大值
             推荐使用 net.ipv4.tcp_wmem=4096 65536 16777216
             
         net.ipv4.tcp_syncookies
             表示是否打开 SYN Cookie.tcp_syncookies 是一个开关,该参数的功能有助于保护服务器免受 SyncFlood 攻击.
             默认值为0,这里设置为1.防止监听 socket 不断收到同一个 ip 的连接请求(SYNC)
              
         不常用参数:
    
         net.ipv4.ip_forward 地址转发, 值为0禁止数据包转发,值为1允许数据包转发.
           临时修改地址转发功能, echo "1" > /proc/sys/net/ipv4/ip_forward 
           
         net.ipv4.ip_local_port_range
                     net.ipv4.ip_local_port_range = 32768 59000   表示应用程序可使用的IPv4端口范围.
                     改为1024到65000
                      
         kernel.shmmax (单位:字节)
            用于定义单个共享内存段的最大值. 32位linux系统：可取最大值为4294967296 - 1 == 4294967295(bytes)
            64位linux 系统,可取的最大值为 物理内存值-1 byte
            
         kernel.shmall
            控制可以使用的共享内存的总页数.
            
         kernel.shmmni
            该参数是共享内存段的最大数量。shmmni缺省值4096，一般肯定是够用了。
                    
         net.core.rmem_default：(单位字节)
            表示套接字接收缓冲区大小的缺省值 (net.core.rmem_default = 212992 ->208KB)
            
         net.core.rmem_max：(单位字节)
            表示套接字接收缓冲区大小的最大值 ( net.core.rmem_max = 212992  ->208 KB)
            推荐使用net.core.rmem_max=16777216
          
         net.core.wmem_default：(单位字节)
            表示套接字发送缓冲区大小的缺省值(net.core.wmem_default = 212992 ->208 KB) 
         
         net.core.wmem_max：(单位字节)
            表示套接字发送缓冲区大小的最大值(net.core.wmem_max = 212992 ->208 KB)
            推荐使用 net.core.wmem_max=16777216
           
         net.ipv4.tcp_tw_reuse(默认是 0)
                这个参数设置为1,表示允许将TIME-WAIT状态的socket重新用于新的TCP链接.这个对服务器来说很有意义,
                因为服务器上总会有大量TIME-WAIT状态的连接
             
         net.ipv4.tcp_tw_recycle
            开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭
            (服务器不建议开启 tcp_tw_recycle 快速回收，会导致大局域网用户访问失败)
            
         net.ipv4.tcp_fin_timeout (单位秒)
                这个参数表示当服务器主动关闭连接时,socket保持在FIN-WAIT-2状态的最大时间,默认是60秒
                减少处于FIN-WAIT-2连接状态的时间,使系统可以处理更多的连接
                推荐使用 net.ipv4.tcp_fin_timeout = 10
                例如:
                    在一个tcp会话过程中,在会话结束时,A首先向B发送一个fin包,在A获得B的ack确认包后,
                    A就进入FIN-WAIT-2状态等待B的fin包,然后A给B发ack确认包.
                    net.ipv4.tcp_fin_timeout参数用来设置A进入FIN-WAIT-2状态等待对方fin包的超时时间.
                    如果时间到了仍未收到对方的fin包就主动释放该会话.
                
         net.ipv4.tcp_max_tw_buckets
                这个参数表示操作系统允许TIME_WAIT套接字数量的最大值,如果超过这个数字,
                TIME_WAIT套接字将立刻被清除并打印警告信息.默认是 65536,过多TIME_WAIT套接字会使Web服务器变慢.
                               
         net.core.netdev_max_backlog
                当网卡接收数据包的速度大于内核处理的速度时,会有一个队列保存这些数据包.这个参数表示该队列的最大值
                      
         net.ipv4.tcp_max_orphans
                系统所能处理不属于任何进程的TCP sockets最大数量.假如超过这个数量﹐那么不属于任何进程的连接会被立即reset,
                并同时显示警告信息
                表示系统中最多有多少TCP套接字不被关联到任何一个用户文件句柄上.如果超过这里设置的数字,
                连接就会复位并输出警告信息.这个限制仅仅是为了防止简单的DoS攻击.此值不能太小
         
         net.ipv4.tcp_synack_retries：
                这个参数用于设置内核放弃连接之前发送SYN+ACK包的数量,(主要是针对服务器而言,没有收到客户端最后一步的ack时,
                持续发送多少个SYN+ACK包)
                对于远端的连接请求SYN,内核会发送(SYN ＋ ACK)数据报,以确认收到上一个远端SYN连接请求包.
                这是所谓的三次握手(threewayhandshake)机制的第二个步骤.这里决定内核在放弃连接之前所送出的 SYN+ACK 数目.
                不应该大于255,默认值是5.对应于180秒左右时间.
                
         net.ipv4.tcp_syn_retries：
                此参数表示在内核放弃建立连接之前发送SYN包的数量.(主要针对客户端而言,再没有收到服务器SYN+ACK包时,持续发送
                多少个SYN包)
                对于一个新建连接,内核要发送多少个 SYN 连接请求才决定放弃.不应该大于255,默认值是5,对应于180秒左右时间.
                

  
         vm.swappiness
            swap分区的使用,减少对swap使用可以提高系统的性能,内存为512M对应vm.swappiness = 10,
            大内存服务器中我们需要设置这个值为0,尤其是在Mysql服务器上(0表示最大限度使用物理内存,然后才是 swap空间)
            
    tcp keepalive
        SO_KEEPALIVE 保持连接检测对方主机是否崩溃，避免（服务器）永远阻塞于TCP连接的输入。
        设置该选项后，当tcp发现有tcp_keepalive_time(7200)秒未收到对端数据后，
        开始以间隔tcp_keepalive_intvl(75)秒的频率发送的空心跳包，如果连续tcp_keepalive_probes(9)次以上未响应则
        对端已经down了,close连接
        
         net.ipv4.tcp_keepalive_intvl
            保活探测消息的发送频率,默认值为75s.发送频率tcp_keepalive_intvl乘以发送次数tcp_keepalive_probes，
            就得到了从开始探测直到放弃探测确定连接断开的时间，大约为11min
            
         net.ipv4.tcp_keepalive_probes
            TCP发送保活探测消息以确定连接是否已断开的次数。默认值为9（次）
            注意：只有设置了SO_KEEPALIVE套接口选项后才会发送保活探测消息
         
         net.ipv4.tcp_keepalive_time(单位 秒)
            在TCP保活打开的情况下,最后一次数据交换到TCP发送第一个保活探测消息的时间，即允许的持续空闲时间。默认值为7200s（2h）
             这个参数表示当keepalive启用时,TCP发送keepalive消息的频度.默认是7200 seconds,
             意思是如果某个TCP连接在idle 2小时后,内核才发起probe.若将其设置得小一点,可以更快地清理无效的连接.
             可以有效的防止空连接攻击.   
            
           
    4. 如果希望屏蔽别人 ping 你的主机,则加入以下代码：
     
        # Disable ping requests 
        
        net.ipv4.icmp_echo_ignore_all = 1 
        编辑完成后, 请执行以下命令使变动立即生效： 
        
        # sysctl -p /etc/sysctl.conf
        
        # sysctl -w net.ipv4.route.flush=1
      
```

[参考资料](http://linxucn.blog.51cto.com/1360306/740130)

#### /proc

##### /proc简介

``` shell

   procfs是Linux内核信息的抽象文件接口,大量内核中的信息以及可调参数都被作为常规文件映射到一个目录树中,
   这样我们就可以简单直接的通过echo或cat这样的文件操作命令对系统信息进行查取和调整.
   /proc 是一个目录.其中包含了反映内核和进程树的各种文件.这些文件和目录并不存在于磁盘中,
   因此当您对这些文件进行读取和写入时，实际上是在从操作系统本身获取相关信息.
         
```

#### 结构分布

``` shell

    1.进程相关部分(只读)
     以数字为名的子目录,这个数字就是相关进程的进程ID
     
    2.内核各子系统
        Linux内核的大部分默认可调参数都被放在了 /proc/sys目录下，这些参数都以常规文件的形式体现,
        并且可以用echo/cat等文件操作命令进行调整,调整的效果是即时的(马上生效),并且在系统运行的整个生命周期之间都有效
        (直到再次改变它们或者系统重启)
        也可以通过sysctl命令进行临时改变,要永久修改只要改/etc/sysctl.conf就行了
         
```

#### /proc/sys下内核文件与配置文件sysctl.conf中变量的对应关系

     由于可以修改的内核参数都在/proc/sys目录下，所以sysctl.conf的变量名省略了目录的前面部分（/proc/sys）
     即将/proc/sys中的文件转换成sysctl中的变量依据下面两个简单的规则：
        1．去掉前面部分/proc/sys
        2．将文件名中的斜杠变为点
     这两条规则可以将/proc/sys中的任一文件名转换成sysctl中的变量名
     
     例如：
     /proc/sys/net/ipv4/ip_forward ---> net.ipv4.ip_forward
     /proc/sys/kernel/hostname --> kernel.hostname
     
     可以使用下面命令查询所有可修改的变量名
     # sysctl –a
     
- /proc/pid/cmdline 查看某个进程的启动命令
     
     
### 系统内存信息

### linux 信号

``` shell

    1.当网络断开或终端窗口关闭后,控制进程(会话的首进程)收到SIGHUP信号退出，会导致该会话期内其他进程退出
       如果我们可以忽略SIGHUP信号，关掉窗口应该就不会影响程序的运行了.
       (1). nohup命令可以达到这个目的
               nohup命令只是使得程序忽略SIGHUP信号，还需要使用标记&把它放在后台运行
                > nohup <command> [argument…] &
                
       (2)  screen 则更加高级
             
```

- 信号概要

``` shell
    信号的生命周期要经过信号的注册、信号集操作、信号的发送和信号的处理四个阶段。
    1.信号的注册: 在应用程序中注册对此信号的处理方法
    2.信号集操作: 用于对指定的一个或多个信号进行信号屏蔽
    3.信号的发送: 可以通过硬件（如在终端上按下Ctrl-C）发送的信号和软件（如通过kill函数）发送的信号
    4.信号的处理: 操作系统对接收信号进程的处理,处理方法是先检查信号集操作函数是否对此信号进行屏蔽，如果没有屏蔽，
                操作系统将按信号安装函数中注册的处理函数完成对此进程的处理.
    
```

### 数据类型占的字节数

``` shell

    1. x86架构, 64位linux系统
        int   --> 4字节
        long  --> 8字节
        long long  --> 8字节
        float --> 4字节
         
```

## linux系统数据结构

``` shell

    size_t -> 无符号数据类型
    ssize_t -> 有符号数据类型
         
```

## 块设备和字符设备

``` shell

    块设备是可以进行随机(无序的)访问的,它使用缓冲区来存放暂时的数据,待条件成熟后,从缓存一次性写入设备,例如硬盘.
    字符设备只能进行顺序访问,如终端,键盘
         
```

## CPU 相关的参数

``` shell

    1.运行队列：负载
        每个cpu维持着线程的运行队列,系统负载(load)是由正在执行的进程和 CPU 运行队列中的进程的结合,
        如果有2个线程正在一个双核系统中执行且4个正在运行队列中,那么负载数即是6
         
```

## 普通进程(前台进程),后台进程,守护进程

``` shell

   默认情况下,进程是在前台运行的,这时就把shell给占据了,我们无法进行其它操作.对于那些没有交互的进程,很多时候,我们希望将其在后台启动,
   可以在启动参数的时候加一个'&'将其变为后台程序.
   后台进程也称为job.前台进程和后台进程都是受shell影响,而守护进程不受shell影响.
   后台进程的文件描述符也是继承于父进程,例如shell,所以它也可以在当前终端下显示输出数据.但是daemon进程自己变成了进程组长,
   其文件描述符号和控制终端没有关联，是控制台无关的。
   
   shell 进程启动的其他进程，由于复制了父进程的信息，因此也都同依附于这个控制终端。
   从终端启动的进程都依附于该终端，并受终端控制和影响。终端关闭，相应的进程都会自动关闭
   
   查看前台进程和后台进程
        > ps -a
        
   查看守护进程
        > ps -x
        
   1. & 最经常被用到
   
      这个用在一个命令的最后，可以把这个命令放到后台执行
   
   2. ctrl + z
   
        可以将一个正在前台执行的命令放到后台，并且暂停
   
   3. jobs
   
        查看当前有多少在后台运行的命令
   
   4. fg
   
        将后台中的命令调至前台继续运行  
   
   　　如果后台中有多个命令，可以用 fg %jobnumber将选中的命令调出，%jobnumber是通过jobs命令查到的后台正在执行的命令的序号(不是pid)
   
   5. bg 将一个在后台暂停的命令，变成继续执行
   
   如果后台中有多个命令，可以用bg %jobnumber将选中的命令调出，%jobnumber是通过jobs命令查到的后台正在执行的命令的序号(不是pid)
   
         
```

### C语言实现守护进程

``` shell

    将普通进程转化为守护进程需要以下步骤
        1.创建子进程，父进程退出
                if ((pid = fork()) == -1) {
                    printf("Fork error !\n");
                    exit(1);
                }
                if (pid != 0) {
                    exit(0);        // 父进程退出,fork函数如果是父进程返回值为非零(子进程的pid),
                                    //如果是子进程,则返回值是0
                }
                
            编写守护进程第一步，就是要使得进程独立于终端后台运行。为避免终端挂起将父进程退出，造成程序已经退出的假象，
            后面的工作都在子进程完成，这样控制终端也可以继续执行其他命令，从而在形式上脱离控制终端的控制.
            如果其终端关闭导致shell进程退出,先于子进程退出，子进程就变为孤儿进程，并由 init 进程作为其父进程收养.
            
        2.子进程创建新会话,禁止进程重新打开控制终端
        
            setsid();           // 子进程开启新会话，并成为会话首进程和组长进程
            if ((pid = fork()) == -1) {
                printf("Fork error !\n");
                exit(-1);
            }
            if (pid != 0) {
                exit(0);        // 结束第一子进程，第二子进程不再是会话首进程， 禁止进程重新打开控制终端
            }
                
            尽管父进程已经退出,但子进程的会话、进程组、控制终端的信息没有改变。为使子进程完全摆脱父进程的环境，需要调用 setsid 函数.
            
            进程组:
                进程组是一个或多个进程的集合,进程组id = 父进程id，即父进程为组长进程,组长进程可以创建一个进程组,创建该进程组中的进程，
                然后终止.只要进程组中有一个进程存在，进程组就存在，与组长进程是否终止无关.
                进程组生存期: 进程组创建到最后一个进程离开(终止或转移到另一个进程组)
                
                函数getpgrp返回调用当前进程的进程组ID
                    pid_t getpgrp(void);
                    返回值：调用进程的进程组ID
                        
                函数getpgid返回调用指定进程的进程组ID
                    pid_t getpgid(pid_t pid);
                    返回值：若成功则返回进程组ID，若出错则返回-1
                    
                进程可以通过调用setpgid来加入一个现有的组或者创建一个新进程组。
                    int setpgid(pid_t pid, pid_t pgid);
                    描述：
                        setpgid函数将pid进程的进程组ID设置为pgid。如果这两个参数相等，则由pid指定的进程变成进程组组长。
                        如果pid是0，则使用调用者的进程ID。另外，如果pgid是0，则由pid指定的进程ID将用作进程组ID。
                        一个进程只能为自己或它的子进程设置进程组ID
                    返回值：若成功则返回0，若出错则返回-1
                    
            会话：
                会话是一个或多个进程组的集合.会话开始于用户登录,终止于用户退出,期间的所有进程都属于这个会话。
                一个会话一般包含一个会话首进程(登陆shell)、一个前台进程组和多个后台进程组，控制终端可有可无；
                前台进程组只有一个，后台进程组可以有多个，这些进程组共享一个控制终端．
                
                前台进程组：
                    该进程组中的进程可以向终端设备进行读、写操作(属于该组的进程可以从终端获得输入scanf函数).
                    该进程组的 ID 等于控制终端进程组 ID，通常据此来判断前台进程组。
                
                后台进程组：
                    会话中除了会话首进程和前台进程组以外的所有进程，都属于后台进程组。
                    该进程组中的进程只能向终端设备进行写操作(printf函数)
                    
                如果调用进程是非组长进程，那么就能创建一个新会话：
                    该进程变成新会话的首进程
                    该进程成为一个新进程组的组长进程
                    该进程没有控制终端，如果之前有，则会被中断
                    
                通过调用 setsid 函数可以创建一个新会话，调用进程担任新会话的首进程，其作用有：
                    使当前进程脱离原会话的控制
                    使当前进程脱离原进程组的控制
                    使当前进程脱离原控制终端的控制
                    
                尽管进程变成无终端的会话首进程,但是它仍然可以重新申请打开一个控制终端。可以通过再次创建子进程结束当前进程，
                使进程不再是会话首进程来禁止进程重新打开控制终端,会话首进程(会话组长)可以申请打开一个控制终端但是其中的子进程
                没有权限打开控制终端.
                
        3.改变当前目录为根目录
            chdir("/");
            
            由于进程运行过程中,当前目录所在的文件系统（如：“/mnt/usb”）是不能卸载的,因此，一般需要守护进程的工作目录改变到其根目录.
            写运行日志的进程将工作目录改变到特定的目录如/tmp
            
        4.重设文件权限掩码
            umask(0);
            进程从创建它的父进程那里继承了文件创建掩模。它可能修改守护进程所创建的文件的存取权限,为防止这一点，将文件创建掩模清除
            
        5.关闭文件描述符
            同文件权限掩码一样,子进程可能继承了父进程打开的文件,而这些文件可能永远不会被用到,但它们一样消耗系统资源,
            而且可能导致所在的文件系统无法卸下,因此需要一一关闭它们.由于守护进程脱离了终端运行,因此标准输入、标准输出、
            标准错误输出这3个文件描述符也要关闭
                for(i=0; i<=2; i++)
                {
                    close(i);
                }  
                
        6.处理SIGCHLD信号(不是必须的,往往是一个进程)
            处理SIGCHLD信号并不是必须的。但对于某些进程,特别是服务器进程往往在请求到来时生成子进程处理请求。如果父进程不等待 子进程结束
            子进程先结束了,而且父进程没有函数调用 wait() 或 waitpid() 等待子进程结束
            子进程将成为僵尸进程（zombie）从而占用系统资源。如果父进程等待子进程结束，将增加父进程的负担，影响服务器进程的并发性能。
            在Linux下可以简单地将 SIGCHLD信号的操作设为SIG_IGN。
            signal(SIGCHLD,SIG_IGN); 
            这样，内核在子进程结束时不会产生僵尸进程
         
```

### 其他进程

``` shell
    孤儿进程：
        父进程先于子进程结束,这时的子进程应该称作“孤儿进程（Orphan）,它将被 1 号进程（init 进程）接管，init 进程成为其父进程
        
    僵尸进程:
        子进程先于父进程结束，而且父进程没有函数调用 wait() 或 waitpid() 等待子进程结束,也没有注册 SIGCHLD 信号处理函数,
        结果使得子进程的进程列表信息无法回收,就变成了僵尸进程（Zombie）.
            
```

## base64 编码

``` shell
    所谓Base64，就是说选出64个字符----小写字母a-z、大写字母A-Z、数字0-9、符号"+"、"/"（再加上作为垫字的"="，实际上是65个字符）
    作为一个基本字符集.然后，其他所有符号都转换成这个字符集中的字符.这个是标准的base64,如果要用于自己定义的base64加密的话,可以考虑
    将符号"+"、"/"和垫字(pad)的"="换成其他字符(不能是　A-Z a-z 0-9 这些值)．
    
    转换方式可以分为四步:
        第一步，将每三个字节作为一组，一共是24个二进制位
        第二步，将这24个二进制位分为四组，每个组有6个二进制位。
        第三步，在每组前面加两个00，扩展成32个二进制位，即四个字节．
        第四步，根据对应关系，得到扩展后的每个字节的对应符号，这就是Base64的编码值
        
    Base64将三个字节转化成四个字节，因此Base64编码后的文本，会比原文本大出三分之一左右.
    
    例如：
        Man的Base64编码是TWFu
        
        第一步，"M"、"a"、"n"的ASCII值分别是77、97、110，对应的二进制值是01001101、01100001、01101110，
              将它们连成一个24位的二进制字符串010011010110000101101110。
        第二步，将这个24位的二进制字符串分成4组，每组6个二进制位：010011、010110、000101、101110。
        第三步，在每组前面加两个00，扩展成32个二进制位，即四个字节：00010011、00010110、00000101、00101110。
              它们的十进制值分别是19、22、5、46。
        第四步，根据上表，得到每个值对应Base64编码，即T、W、F、u
    
    二个字节的情况
        当剩下的只有2个字节,Base64编码需要将这2个字节转化为4个字节,将这二个字节（16 bit)先转成三组,前面2组(每组6bit)在每组前面加
        两个00，
        最后一组除了前面加两个0以外，后面也要加两个0,这样得到一个三位的Base64编码，再在末尾补上一个"="号。
        比如，"Ma"这个字符串是两个字节，可以转化成三组00010011、00010110、00010000以后，对应Base64值分别为T、W、E，
        再补上一个"="号，因此"Ma"的Base64编码就是TWE=
    
    一个字节的情况：
        将这一个字节的8个二进制位，按照上面的规则转成二组，最后一组除了前面加二个0以外，后面再加4个0。这样得到一个二位的Base64编码，
        再在末尾补上两个"="号。
        比如，"M"这个字母是一个字节，可以转化为二组00010011、00010000，对应的Base64值分别为T、Q，再补上二个"="号，
        因此"M"的Base64编码就是TQ==。
    

    base64扩展
         在base64编码时可以考虑在编码过程中加上分隔字符串(不能是　A-Z a-z 0-9 + /这些值),每隔nLineLength(必须是4的整数倍)
         字节加上一字符串.
         例如源字符串"this is a example",先通过标准base64转化后是dGhpcyBpcyBhIGV4YW1wbGU=,在每隔8必须是4的整数倍)字节加上
         "ccc",就得到dGhpcyBpccccyBhIGV4cccYW1wbGU=
            
```

## 密码学知识

``` shell
     公钥和私钥是一一对应的关系，有一把公钥就必然有一把与之对应的、独一无二的私钥，反之亦成立.公钥用来加密信息，私钥用来数字签名.
     因为任何人都可以生成自己的（公钥,私钥）对,所以为了防止有人散布伪造的公钥骗取信任,就需要一个可靠的第三方机构来生成经过认证的
     （公钥，私钥）对.
     
     首先公钥是用来加密用的,而且公钥是可以提供给任何人进行使用的,私钥则是用来解密(将加密的内容解密为明文),私钥很重要不能泄露.
      
      数字签名:是针对拥有私钥的一方而言的,拥有私钥将明文通过hash函数生成digest(摘要),再用私钥加密为数字签名(signature)
      拥有公钥的一方可以用公钥对数字签名进行解密来确定来源是不是安全的.
      
      公钥加密,私钥解密.
      私钥数字签名,公钥验证.
```

## ssh原理及运用

``` shell
    SSH是一种网络协议,用于计算机之间的加密登录.
    
    没有用证书的ssh实现的方式：
        （1）远程主机收到用户的登录请求,把自己的公钥发给用户.
        （2）用户使用这个公钥,将登录密码加密后，发送回来。
        （3）远程主机用自己的私钥，解密登录密码，如果密码正确，就同意用户登录.
        
    注意：
        这种简单的ssh协议存在风险,即中间人攻击,如果有人截获了登录请求,然后冒充远程主机,将伪造的公钥发给用户
        这时就可以获取用户的登录密码,再用这个密码登录远程主机,那么SSH的安全机制就荡然无存了
        
    1.口令登录(要输入用户名和密码)
        > ssh user@host
        
      如果是第一次ssh连接的时候,因为不确定是不是收到了中间人攻击,侵入者会将自己的公钥传给客户端,诱导客户端输入远程服务器的用户名和密码,
      所以要确认远程服务器的公钥,当远程主机的公钥被接受以后,它就会被保存在文件$HOME/.ssh/known_hosts之中.
      下次再连接这台主机,系统就会认出它的公钥已经保存在本地了,从而跳过警告部分,直接提示输入密码
      
    2.公钥登录(不用需要重复输入用户名和密码)
        就是用户将自己的公钥储存在远程主机上.登录的时候,远程主机会向用户发送一段随机字符串,用户用自己的私钥加密后,再发回来。
        远程主机用事先储存的公钥进行解密,如果成功,就证明用户是可信的,直接允许登录shell,不再要求密码。
        
        1.ssh客户端生成自己的一对秘钥
            > ssh-keygen -t rsa
                -t:type
        在$HOME/.ssh/目录下，会新生成两个文件：id_rsa.pub(公钥)和id_rsa(私钥)
        
        2.将公钥传送到远程主机host上面
            > ssh-copy-id -i ~/.ssh/id_rsa.pub root@xxx.xx.xxx.xx
            
                -i:指定的公钥
            实际上是将id_rsa.pub里面的内容拷贝到远程服务器的 $HOME/.ssh/authorized_keys中
            
            其中要对远程服务器进行配置/etc/ssh/sshd_config,去掉注释
            
                　　RSAAuthentication yes
                　　PubkeyAuthentication yes
                　　AuthorizedKeysFile .ssh/authorized_keys
            重启远程主机ssh服务
                > /etc/init.d/ssh restart        
```

## Protobuf 简介

```shell
    1.
        (1) protobuf 是可扩展的序列化结构数据格式,很适合用做数据存储和作为不同应用，不同语言之间相互通信的数据交换格式
            只要实现相同的协议格式即同一proto文件被编译成不同的语言版本，加入到各自的工程中去。
            这样不同语言就可以解析其他语言通过protobuf序列化的数据。
           
        (2) 使用 Protocol buffers，你需要写一个 .proto 文件，用于描述你所希望存储的数据结构。利用 .proto 文件，
        　　protocol buffer 编译器可以创建一个类，用于实现对高效的二进制格式的 protocol buffer 数据的自动化编码和解码
        
        (3) 网络编程中使用 Protobuf 需要解决问题
                A. 长度, Protobuf 打包的数据没有自带长度信息或终结符，需要应用程序自己在发送和接受的时候自己切分
                    解决方案:
                        可以在每一个消息内容的头部加个固定的 length header 
                B. 类型，Protobuf 打包的数据没有自带类型信息，需要由发送方把类型信息传给接收方，接收方创建具体的
                   Protobuf Message 对象，再做反序列化
                   解决方案:
                        根据 type name 反射自动创建 Message 对象．
                            (A) 用 DescriptorPool::generated_pool() 创建一个 DescriptorPool 对象，它包含程序编译时
                            　　链接的全部 Protobuf Message types
                            (B) 根据 type name 用　DescriptorPool::FindMessageTypeByName() 找到指定的 Descriptor 对象
                            (C) 再用 MessageFactory::generated_factory() 创建一个 MessageFactory 对象，它能创建
                            　　程序编译时链接的所有 Protobuf Message types
                            (D) 根据 Descriptor 对象用 MessageFactory::GetPrototype() 找到具体 Message type 的
                            　　 default instance( google::protobuf::Message* prototype)
                            (E) 最后用 prototype->new() 创建对象
                            
        (4) Protobuf 传输格式
                struct protobuf_message{
                   int32_t len;  // 包含 4(name_len) + type_name_len + proto_data + 4(check_sum)
                   int32_t name_len;  // 包含 '\0' 
                   char typename[name_len]; // 以 '\0' 结束
                   char proto_data[len - name_len - 8];
                   int32_t check_sum;  // 采用 adler32 算法进行数据校验
                };
                
                A. 采用 int32_t 而不是 uint_32, 考虑到跨语言的移植性，java 语言没有 unsigned int 类型
                B. check_sum 　adler32 算法计算量小，速度快， zlib 和 java.unit.zip 支持该算法，不需要自己实现
                C. type_name 以 '\0' 结尾方便进行调试，根据 tcpdump 抓取下来的包很容易看出 type_name,
                
         (5) 只有在使用 TCP 长连接，且在一个连接上使用多种类型的消息时(时间同步和心跳等不同业务)，才使用
         　　 protobuf_message(加上消息类型和长度)，这个时候需要一个分发器 dispatcher,把不同的消息类型分发到各自的消息处理函数
             否则使用默认的(Protobuf 打包的数据不带长度和消息类型),使用默认的
         　　Protobuf 传输格式(不带长度和类型)的情况很多，有在某一个 port 上只接受一个消息类型．
         
         (6) protobuf 是精心设计的协议格式，客户端可以先升级，服务端则可以安全的跳过未识别的字段
      
    2. protobuf 语法定义
            (1) 用 protobuf 语法 定义 proto 文件，内容如下:
            
                    message Article {
                        required int32 article_id=1;
                        optional string article_excerpt=2;
                        repeated string article_picture=3;
                    }
                    
                    message:	    是消息定义的关键字
                    required:	    表示这个字段必须的，必须在序列化的时候被赋值
                    optional:	    代表这个字段是可选的，可以为0个或1个但不能大于1个。
                    repeated:	    代表此字段可以被重复任意多次包括0次
                    int32/string	字段的类型
                    
                    最后的1，2，3则是代表每个字段的一个唯一的编号标签，在同一个消息里不可以重复。
                    这些编号标签用与在消息二进制格式中标识你的字段，并且消息一旦定义就不能更改。
                    需要说明的是标签在1到15范围的采用一个字节进行编码。所以通常将标签1到15用于频繁发生的消息字段。
                    编号标签大小的范围是1到229。此外不能使用protobuf系统预留的编号标签（19000 －19999）
                    
            (2) 当一个proto文件需要另一个proto文件的时候，我们可以通过import导入
                    import "article.proto";
                    message Book {
                    //定义消息体
                    }
                    
            (3) 更新 proto 文件要求
                    1. 不能改变已有的任何编号标签。
                    2. 只能添加 optional 和 repeated 的字段。这样旧代码能够解析新的消息，只是那些新添加的字段会被忽略。
                        但是序列化的时候还是会包含哪些新字段。而新代码无论是旧消息还是新消息都可以解析。
                    3. 非 required 的字段可以被删除，但是编号标签不可以再次被使用，应该把它标记到reserved中去
                    4. 非 required 可以被转换为扩展字段，只要字段类型和编号标签保持一致
                    5. 相互兼容的类型，可以从一个类型修改为另一个类型，譬如 int32 的字段可以修改为 int64
                    
            (4)  包管理
                    .proto 文件以一个 package 声明开始，这可以避免不同项目的命名冲突。
                    在 C++，你生成的类会被置于与 package 名字一样的命名空间。
                    
                    
    3. 实际例子
    
            syntax = "proto2";
            message Article {
              required int32 article_id = 1;　// 消息的字段编号标签为 1 
              optional string article_excerpt = 2;
              repeated string article_picture = 3;
              optional int32  article_pagecount = 4 [default = 0];　// 给字段设置默认值
              // 可以定义了enum枚举类型
              enum ArticleType {
                NOVEL = 0;
                PROSE = 1;
                PAPER = 2;
                POETRY = 3;
              }
              optional ArticleType article_type = 5 [default = NOVEL];
              // 还可以嵌套消息
              message Author {
                required string name = 1; //作者的名字
                optional string phone = 2;
              }
              optional Author author = 6;
              
              // packed=true可以更加有效率的encode
              // packed只能用于 repeated && 数值类型 的字段。不能用于string类型的字段。
              repeated int32 article_numberofwords = 7 [packed=true];
              
              //reserved关键字主要用于保留相关编号标签,用reserved标记这些编号标签以保证不会被使用
              reserved  9, 10, 12 to 15;
              
              // 用extensions关键字来定义一些编号标签供第三方扩展。这样的好处是不需要修改原来的消息格式。
              // 
              extensions 100 to 1000;
            }
            
            //extend关键字来扩展。只要扩展的字段编号标签在extensions定义的范围里
            extend Article {
              optional int32 followers_count = 101;
              optional int32 likes_count= 102;
            }
            message Other {
              optional string other_info = 1;
              
              // oneof 关键字相当于 union, 当你设置了oneof里某个成员值时，
              // 它会自动清除掉oneof里的其他成员，也就是说同一时刻oneof里只有一个成员有效
              // oneof里的字段不能用 required，optional，repeted关键字
              oneof test_oneof {
                string code1 = 2;
                string code2 = 3;
              }
            }
                    
```