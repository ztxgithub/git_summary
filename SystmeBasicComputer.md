
- system page size

``` c
    A page, memory page, or virtual page是跟虚拟内存有关,它是一段连续的虚拟内存,
    一个page size 大小与处理器架构有关，一般是4K bytes
    
	
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
    
  #释放pagecache
  echo 1 > /proc/sys/vm/drop_caches
	
  #释放dentries與inodes
  echo 2 > /proc/sys/vm/drop_caches
  
    
  echo 3 > /proc/sys/vm/drop_caches
  3 是指释放pagecache、dentries與inodes，也就是釋放所有的cache

```

## 系统资源参数

### 系统最多的文件描述符

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
    
         net.ipv4.ip_forward 地址转发, 值为0禁止数据包转发,值为1允许数据包转发.
           临时修改地址转发功能, echo "1" > /proc/sys/net/ipv4/ip_forward 
           
         kernel.shmmax (单位:字节)
            用于定义单个共享内存段的最大值. 32位linux系统：可取最大值为4294967296 - 1 == 4294967295(bytes)
            64位linux 系统,可取的最大值为 物理内存值-1 byte
            
         kernel.shmall
            控制可以使用的共享内存的总页数.
            
         kernel.shmmni
            该参数是共享内存段的最大数量。shmmni缺省值4096，一般肯定是够用了。
            
         fs.file-max:
            决定了当前内核可以打开的最大的文件句柄数.这个是系统限制,限制所有用户打开文件描述符的总和.
            
         net.ipv4.ip_local_port_range
            net.ipv4.ip_local_port_range = 32768 59000   表示应用程序可使用的IPv4端口范围.
            改为1024到65000
            
         net.core.rmem_default：(单位字节)
            表示套接字接收缓冲区大小的缺省值 (net.core.rmem_default = 212992 ->208KB)
            
         net.core.rmem_max：(单位字节)
            表示套接字接收缓冲区大小的最大值 ( net.core.rmem_max = 212992  ->208 KB)
          
         net.core.wmem_default：(单位字节)
            表示套接字发送缓冲区大小的缺省值(net.core.wmem_default = 212992 ->208 KB) 
         
         net.core.wmem_max：(单位字节)
            表示套接字发送缓冲区大小的最大值(net.core.wmem_max = 212992 ->208 KB)
           
         net.ipv4.tcp_tw_reuse(默认是 0)
                这个参数设置为1,表示允许将TIME-WAIT状态的socket重新用于新的TCP链接.这个对服务器来说很有意义,
                因为服务器上总会有大量TIME-WAIT状态的连接
             
         net.ipv4.tcp_tw_recycle
            开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭
            
         net.ipv4.tcp_keepalive_time(单位 秒)
                这个参数表示当keepalive启用时,TCP发送keepalive消息的频度.默认是7200 seconds,
                意思是如果某个TCP连接在idle 2小时后,内核才发起probe.若将其设置得小一点,可以更快地清理无效的连接.
            
         net.ipv4.tcp_fin_timeout (单位秒)
                这个参数表示当服务器主动关闭连接时,socket保持在FIN-WAIT-2状态的最大时间,默认是60秒
                
         net.ipv4.tcp_max_tw_buckets
                这个参数表示操作系统允许TIME_WAIT套接字数量的最大值,如果超过这个数字,
                TIME_WAIT套接字将立刻被清除并打印警告信息.默认是 65536,过多TIME_WAIT套接字会使Web服务器变慢.
                
         net.ipv4.tcp_max_syn_backlog
                这个参数表示TCP三次握手建立阶段接受WYN请求队列的最大长度
                将其设置大一些可以使出现Nginx繁忙来不及accept新连接的情况时,Linux不至于丢失客户端发起的连接请求.
                加大队列长度为8192
                
         net.ipv4.tcp_rmem
                (net.ipv4.tcp_rmem =4096 32768 262142)
                这个参数定义了用于TCP接收滑动窗口的最小值，默认值，最大值
                
         net.ipv4.tcp_wmem
                (net.ipv4.tcp_wmem =4096 32768 262142)
                这个参数定义了用于TCP发送滑动窗口的最小值，默认值，最大值
                
         net.core.netdev_max_backlog
                当网卡接收数据包的速度大于内核处理的速度时，会有一个队列保存这些数据包。这个参数表示该队列的最大值
                
              
            
           
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
     
     
### 系统内存信息



