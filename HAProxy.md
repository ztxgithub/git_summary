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
             
             
             
      
        defaults # 默认部分的定义
            # mode语法：mode {http|tcp|health}  http是七层模式,tcp是四层模式,health是健康检测
             mode http
         
            # 使用127.0.0.1上的syslog服务的local3设备记录错误信息
             log 127.0.0.1 local3 err
           
            # 定义连接后端服务器的失败重连次数,连接失败次数超过此值后将会将对应后端服务器标记为不可用
             retries 3

            # 启用日志记录HTTP请求，默认haproxy日志记录是不记录HTTP请求的,只记录“时间[Jan 5 13:23:46] 
              日志服务器[127.0.0.1] 实例名已经pid[haproxy[25218]] 信息[Proxy http_80_in stopped.]”
             option httplog

            # 当使用了cookie时,haproxy将其请求的后端服务器的serverID插入到cookie中,以保证会话的SESSION持久性；
              此时如果后端的服务器宕掉了,客户端的cookie是不会刷新的,如果设置此参数,会将客户的请求强制定向到另外一个
              后端server上,以保证服务的正常
             option redispatch

            # 当服务器负载很高的时候,自动结束掉当前队列处理比较久的链接
             option abortonclose
             
            # 启用该项，日志中将不会记录空连接。所谓空连接就是在上游的负载均衡器或者监控系统为了探测该服务是否存活可用时,
              需要定期的连接或者获取某一固定的组件或页面,或者探测扫描端口是否在监听或开放等动作被称为空连接；
              如果该服务上游没有其他的负载均衡器的话，建议不要使用该参数，因为互联网上的恶意扫描或其他动作就不会被记录下来
             option dontlognull

             # 使用该参数,每处理完一个request时,haproxy都会去检查http头中的Connection的值,如果该值不是close,
             haproxy将其***，如果该值为空将会添加为：Connection: close.使每个客户端和服务器端在完成一次传输后都会主动
             关闭TCP连接。与该参数类似的另外一个参数是“option forceclose”,该参数的作用是强制关闭对外的服务通道
             因为有的服务器端收到Connection: close时,也不会自动关闭TCP连接,如果客户端也不关闭,连接就会一直处于打开,
             直到超时.
             option httpclose

             # 设置成功连接到一台服务器的最长等待时间,默认单位是毫秒,新版本的haproxy使用timeout connect替代,
                该参数向后兼容
             contimeout 5000 或则 timeout connect 50s

             # 设置连接客户端发送数据时的成功连接最长等待时间,默认单位是毫秒，新版本haproxy使用,timeout client替代.
               该参数向后兼容
             clitimeout 3000 或则 timeout client 60s

             # 设置服务器端回应客户度数据发送的最长等待时间,默认单位是毫秒,新版本haproxy使用timeout server替代.
                该参数向后兼容   
             srvtimeout 3000 或则 timeout server 50s
             
        listen status 
             # 定义一个名为status的部分,可以在listen指令指定的区域中定义匹配规则和后端服务器ip，
             #相当于需要在其中配置frontend,backend的功能。一般做tcp转发比较合适,不用太多的规则匹配.
             
             # 定义监听的套接字
             bind 0.0.0.0:1080
             
              # 定义为HTTP模式
             mode http
            
            # 继承global中log的定义
             log global
             
            # stats是haproxy的一个统计页面的套接字,该参数设置统计页面的刷新间隔为30s
             stats refresh 30s
             
            # 设置统计页面的uri为/admin?stats
             stats uri /admin?stats
             
            # 设置统计页面认证时的提示内容
             stats realm Private lands
             
            # 设置统计页面认证的用户和密码，如果要设置多个，另起一行写入即可 
             stats auth admin:password
             
            # 隐藏统计页面上的haproxy版本信息 
             stats hide-version
             
        frontend http_80_in    # 定义一个名为http_80_in的前端部分,haproxy会监听bind的端口
        
             # http_80_in定义前端部分监听的套接字
             bind 0.0.0.0:80
             
             # 定义为HTTP模式
             mode http
             
             # 继承global中log的定义
             log global
             
             # 启用X-Forwarded-For,在requests头部插入客户端IP发送给后端的server,使后端server获取到客户端的真实IP
             option forwardfor
           
             # 定义一个名叫static_down的acl,当backend static_sever中存活机器数小于1时会被匹配到
             acl static_down nbsrv(static_server) lt 1
             
             #acl php_web path_end .php 定义一个名叫php_web的acl,当请求的url末尾是以.php结尾的,将会被匹配到,下面两种写
              法任选其一
             acl php_web url_reg /*.php$
             
             # 定义一个名叫static_web的acl，当请求的url末尾是以.css、.jpg、.png、.jpeg、.js、.gif
               结尾的,将会被匹配到，下面两种写法任选其一
             acl static_web url_reg /*.(css|jpg|png|jpeg|js|gif)$
             
             acl static_web path_end .gif .png .jpg .css .js .jpeg
            
             # 如果满足策略static_down时，就将请求交予backend php_server
             use_backend php_server if static_down
            
             # 如果满足策略php_web时，就将请求交予backend php_server
             use_backend php_server if php_web
             
             # 如果满足策略static_web时，就将请求交予backend static_server
             use_backend static_server if static_web
             
             
 
                	
```