# HAProxy

## HAProxy 安装步骤

``` shell
    1. tar -xvf haproxy-1.7.9.tar.gz 
    2. cd haproxy-1.7.9/
    3. make PREFIX=/home/yytd/HAProxy ARCH=x86_64 TARGET=linux2628
    
            其中TARGET的值根据当前操作系统内核版本指定(uname -a)
                - linux22     for Linux 2.2
                - linux24     for Linux 2.4 and above (default)
                - linux24e    for Linux 2.4 with support for a working epoll (> 0.21)
                - linux26     for Linux 2.6 and above
                - linux2628   for Linux 2.6.28, 和高于Linux2.6.28
                
            ARCH=x86_64 #系统位数
            PREFIX为指定的安装路径
            
    4. make install PREFIX=/home/ha/haproxy
            
    5.创建HAProxy配置文件
        mkdir -p /home/yytd/HAProxy/conf
        vi /home/yytd/HAProxy/conf/haproxy.cfg
        
    6.将HAProxy注册为系统服务
        在/etc/init.d目录下添加HAProxy服务的启停脚本:/etc/init.d/haproxy
                
			
```

## HAProxy负载均衡算法

``` shell
    ①roundrobin:表示简单的轮询是负载均衡基本算法；
    ②static-rr:表示根据权重，建议关注；
    ③leastconn:表示最少连接者先处理，建议关注；
    ④source:表示根据请求源IP，这个跟Nginx的IP_hash机制类似，我们用其作为解决session问题的一种方法
    ⑤ri:表示根据请求的URI；
    ⑥rl_param:表示根据请求的URl参数'balance url_param' requires an URL parameter name；
    ⑦hdr(name):表示根据HTTP请求头来锁定每一次HTTP请求；
    ⑧rdp-cookie(name):表示根据据cookie(name)来锁定并哈希每一次TCP请求
                	
```

## HAProxy配置文件详解

``` shell
    HAProxy配置中分五大部分：
        global：全局配置参数,进程级的,用来控制Haproxy启动前的一些进程及系统设置
        defaults：配置一些默认的参数,可以被frontend,backend,listen段继承使用
        frontend：用来匹配接收客户所请求的域名,uri等,并针对不同的匹配,做不同的请求处理
        backend：定义后端服务器集群,以及对后端服务器的一些权重、队列、连接数等选项的设置,我将其理解为Nginx中的upstream块
        listen：frontend和backend的组合体
        
    例子如下:
        global   # 全局参数的设置 
                     
             # log语法：log [max_level_1] 全局的日志配置,使用log关键字,指定使用127.0.0.1上的syslog服务中的local0日志设备,        
                        记录日志等级为info的日志 
                        
             log 127.0.0.1 local0 info   
                                           
             # 设置运行haproxy的用户和组，也可使用uid，gid关键字替代之                             
             user haproxy 
             group haproxy 
             
             # 以守护进程的方式运行 
             daemon 
             
            # 设置haproxy启动时的进程数,该值的设置应该和服务器的CPU核心数一致,即常见的2颗8核心CPU的服务器,即共有16核心,
              则可以将其值设置为：<=16 ,创建多个进程数,可以减少每个进程的任务队列,但是过多的进程数也可能会导致进程崩溃 
             nbproc 16
          
            # 定义每个haproxy进程的最大连接数,由于每个连接包括一个客户端和一个服务器端,所以单个进程的TCP会话最大数目将
             是该值的两倍。 
             maxconn 4096 
        
            # 设置最大打开的文件描述符数,在1.4的官方文档中提示，该值会自动计算，所以不建议进行设置 
             ulimit -n 65536 
        
            # 定义haproxy的pid的具体位置
             pidfile /var/run/haproxy.pid 
            
                	
```