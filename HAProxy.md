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
        1.global   # 全局参数的设置 
                     
             # log语法：,指定使用127.0.0.1上的syslog服务中的local0日志设备,        
                        记录日志等级为info的日志 
                        log [address] [device] [maxlevel] [minlevel]：日志输出配置,如log 127.0.0.1 local0 info warning,
                        即向本机rsyslog或syslog的local0输出info到warning级别的日志
                        HAProxy的日志共有8个级别，从高到低为emerg/alert/crit/err/warning/notice/info/debug
                        
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
             
             
             
      
        2.defaults # 默认部分的定义
            # mode语法：mode {http|tcp|health}  http是七层模式,tcp是四层模式,health是健康检测
                mode http
         
            # 使用127.0.0.1上的syslog服务的local3设备记录错误信息
                log 127.0.0.1 local3 error
           
            # 定义连接后端服务器的失败重连次数,连接失败次数超过此值后将会将对应后端服务器标记为不可用
                retries 3

            # 启用日志记录HTTP请求，默认haproxy日志记录是不记录HTTP请求的,只记录“时间[Jan 5 13:23:46] 
              日志服务器[127.0.0.1] 实例名已经pid[haproxy[25218]] 信息[Proxy http_80_in stopped.]”
                option httplog
             
            #开启tcplog
                option tcplog            

            # 当使用了cookie时,haproxy将其请求的后端服务器的serverID插入到cookie中,以保证会话的SESSION持久性；
              此时如果后端的服务器宕掉了,客户端的cookie是不会刷新的,如果设置此参数,会将客户的请求强制定向到另外一个
              后端server上,以保证服务的正常
                option redispatch

            # 当服务器负载很高的时候,自动结束掉当前队列处理比较久的链接
                option abortonclose
             
            # 启用该项，日志中将不会记录空连接。所谓空连接就是在上游的负载均衡器或者监控系统为了探测该服务是否存活可用时,
              需要定期的连接或者获取某一固定的组件或页面,或者探测扫描端口是否在监听或开放等动作被称为空连接；
              也就是不记录健康检查日志信息
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
             
             #TCP模式下，应将timeout client和timeout server设置为一样的值，以防止出现问题
             
        3.listen status 
             # 定义一个名为status的部分,可以在listen指令指定的区域中定义匹配规则和后端服务器ip，
             #相当于需要在其中配置frontend,backend的功能。一般做tcp转发比较合适,不用太多的规则匹配.
             
             # 定义监听的套接字
                    bind 0.0.0.0:1080
             
              # 定义为HTTP模式
                    mode http
            
            # 继承global中log的定义
                    log global
             
            #  stats refresh [time]：监控数据刷新周期
               stats是haproxy的一个统计页面的套接字,该参数设置统计页面的刷新间隔为30s
                    stats refresh 30s
             
            # stats uri [uri] 设置统计页面的uri为/admin?stats
                stats uri /admin?stats
             
            # 设置统计页面认证时的提示内容
                stats realm Private lands
             
            # stats auth [user]:[password]：监控页面的认证用户名密码
              设置统计页面认证的用户和密码，如果要设置多个，另起一行写入即可 
                stats auth admin:password
             
            # 隐藏统计页面上的haproxy版本信息 
                stats hide-version
             
        4.frontend http_80_in    # 定义一个名为http_80_in的前端部分,haproxy会监听bind的端口
        
             # http_80_in定义前端部分监听的套接字
                    bind 0.0.0.0:80
             
             # mode: 此frontend的工作模式,主要有http和tcp两种,对应L7和L4两种负载均衡模式
               定义为HTTP模式
                    mode http
             
             # log 同global域的log配置,仅应用于此frontend。如果要沿用global域的log配置
                    log global
                    
             # maxconn：同global域的maxconn设置方式一样,仅应用于此frontend
             
             # 启用X-Forwarded-For,在requests头部插入客户端IP发送给后端的server,使后端server获取到客户端的真实IP
                    option forwardfor
                    
             # 以KeepAlive模式提供服务
                    option http-keep-alive
                    
             # 与http-keep-alive对应,关闭KeepAlive模式,如果HAProxy主要提供的是接口类型的服务,
               可以考虑采用httpclose模式,以节省连接数资源.但如果这样做了,接口的调用端将不能使用HTTP连接池
                    option httpclose
                    
             # 开启httplog,HAProxy将会以类似Apache HTTP或Nginx的格式来记录请求日志
                    option httplog
                    
             # 开启tcplog,HAProxy将会在日志中记录数据包在传输层的更多属性
                    option tcplog
                    
             # timeout client [time]：指连接创建后，客户端持续不发送数据的超时时间
             
             # timeout http-request [time]：指连接创建后，客户端没能发送完整HTTP请求的超时时间,主要用于防止DoS类攻击,
               即创建连接后,以非常缓慢的速度发送请求包,导致HAProxy连接被长时间占用
               
             
                           
             #acl [name] [criterion] [flags] [operator] [value]：定义一条ACL,
             ACL是根据数据包的指定属性以指定表达式计算出的true/false值.如"acl url_ms1 path_beg -i /ms1/"
             定义了名为url_ms1的ACL,该ACL在请求uri以/ms1/开头（忽略大小写）时为true
             # 定义一个名叫static_down的acl,当backend static_sever中存活机器数小于1时会被匹配到
                    acl static_down nbsrv(static_server) lt 1
             
             #acl php_web path_end .php 定义一个名叫php_web的acl,当请求的url末尾是以.php结尾的,将会被匹配到,下面两种写
              法任选其一
                    acl php_web url_reg /*.php$
             
             # 定义一个名叫static_web的acl，当请求的url末尾是以.css、.jpg、.png、.jpeg、.js、.gif
               结尾的,将会被匹配到，下面两种写法任选其一
                    acl static_web url_reg /*.(css|jpg|png|jpeg|js|gif)$
             
                    acl static_web path_end .gif .png .jpg .css .js .jpeg
            
             # use_backend [backend] if|unless [acl]：与ACL搭配使用,在满足/不满足ACL时转发至指定的backend
             # 如果满足策略static_down时，就将请求交予backend php_server
                    use_backend php_server if static_down
            
             # 如果满足策略php_web时，就将请求交予backend php_server
                    use_backend php_server if php_web
             
             # 如果满足策略static_web时，就将请求交予backend static_server
                    use_backend static_server if static_web
                    
             # acl后面是规则名称,-i为忽略大小写，后面跟的是要访问的域名，如果访问www.abc.com这个域名，就触发web规则
                    acl web hdr(host) -i www.abc.com  
                    
             #如果上面定义的web规则被触发，即访问www.abc.com，就将请求分发到php_server这个作用域。
                    use_backend php_server if web   
                    
             #default_backend [name]：frontend对应的默认backend,不满足acl规则响应backend的默认页面
              请求定向至后端服务群php_server
                    default_backend php_server 
             
             #禁用此frontend
                    disabled
             
             # http-request [operation] [condition]：对所有到达此frontend的HTTP请求应用的策略,
               例如可以拒绝、要求认证、添加header、替换header、定义ACL等等。
               
             # http-response [operation] [condition]：对所有从此frontend返回的HTTP响应应用的策略
            
             
        5.backend php_server #定义一个名为php_server的后端部分，frontend定义的请求会到到这里处理
        
             # 设置为http模式
                    mode http
            
            # 设置haproxy的调度算法为源地址hash
                    balance source
             
            # 允许向cookie插入SERVERID，每台服务器的SERVERID可在下面使用cookie关键字定义
                    cookie SERVERID
             
            # option httpchk [METHOD] [URL] [VERSION]：定义以http方式进行的健康检查策略 
              开启对后端服务器的健康检测，通过GET /test/index.php来判断后端服务器的健康情况
                     option httpchk GET /test/index.php
            
            # server语法：server [:port] [param*]
             使用server关键字来设置后端服务器；为后端服务器所设置的内部名称[php_server_1],该名称将会呈现在日志或警报中、
               后端服务器的IP地址,支持端口映射[10.12.25.68:80]、指定该服务器的SERVERID为1[cookie 1],接受健康监测[check],
               监测的间隔时长,单位毫秒[inter 2000]、监测正常多少次后被认为后端服务器是可用的[rise 3]、
               监测失败多少次后被认为后端服务器是不可用的[fall 3]、最为备份用的后端服务器，
               当正常的服务器全部都宕机后，才会启用备份服务器[backup],后端服务器最大连接数 [maxconn 3000]
               maxqueue：等待队列的长度，当队列已满后，后续请求将会发至此backend下的其他server，默认为0，即无限
               分发的权重[weight 2] weight：server的权重,0-256,权重越大,分给这个server的请求就越多.
               weight为0的server将不会被分配任何新的连接,所有server默认weight为1
               
                    server php_server_1 10.12.25.68:80 cookie 1 check inter 2000 rise 3 fall 3 weight 2
                    server php_server_2 10.12.25.72:80 cookie 2 check inter 2000 rise 3 fall 3 weight 1
                    server php_server_bak 10.12.25.79:80 cookie 3 check inter 1500 rise 3 fall 3 backup
                          	
```

## 健康监测
```shell
    1、通过监听端口进行健康检测
    这种检测方式,haproxy只会去检查后端server的端口,并不能保证服务的真正可用. 
    listen http_proxy 0.0.0.0:80 
            mode http 
            cookie SERVERID 
            balance roundrobin 
            option httpchk 
            server web1 192.168.1.1:80 cookie server01 check 
            server web2 192.168.1.2:80 cookie server02 check inter 500 rise 1 fall 2 
     
    2、通过URI获取进行健康检测
    这种检测方式，是用过去GET后端server的的web页面，基本上可以代表后端服务的可用性。
    listen http_proxy 0.0.0.0:80 
            mode http 
            cookie SERVERID 
            balance roundrobin 
            option httpchk GET /index.html 
            server web1 192.168.1.1:80 cookie server01 check 
            server web2 192.168.1.2:80 cookie server02 check inter 500 rise 1 fall 2 
     
    3、通过request获取的头部信息进行匹配进行健康检测
    这种检测方式则是基于高级，精细的一些监测需求。通过对后端服务访问的头部信息进行匹配检测。 
    listen http_proxy 0.0.0.0:80 
            mode http 
            cookie SERVERID 
            balance roundrobin 
            option httpchk HEAD /index.jsp HTTP/1.1\r\nHost:\ www.xxx.com 
            server web1 192.168.1.1:80 cookie server01 check 
            server web2 192.168.1.2:80 cookie server02 check inter 500 rise 1 fall 2 

```

## ACL 规则
```shell
    ########ACL策略定义#########################
    1、#如果请求的域名满足正则表达式返回true -i是忽略大小写
    acl denali_policy hdr_reg(host) -i ^(www.inbank.com|image.inbank.com)$
    
    2、#如果请求域名满足www.inbank.com 返回 true -i是忽略大小写
    acl tm_policy hdr_dom(host) -i www.inbank.com
    
    3、#在请求url中包含sip_apiname=，则此控制策略返回true,否则为false
    acl invalid_req url_sub -i sip_apiname=#定义一个名为invalid_req的策略
    
    4、#在请求url中存在timetask作为部分地址路径，则此控制策略返回true,否则返回false
    acl timetask_req url_dir -i timetask
    
    5、#当请求的header中Content-length等于0时返回 true
    acl missing_cl hdr_cnt(Content-length) eq 0
    
    #########acl策略匹配相应###################
    1、#当请求中header中Content-length等于0 阻止请求返回403
    block if missing_cl
    
    2、#block表示阻止请求，返回403错误，当前表示如果不满足策略invalid_req，或者满足策略timetask_req，则阻止请求。
    block if !invalid_req || timetask_req
    
    3、#当满足denali_policy的策略时使用denali_server的backend
    use_backend denali_server if denali_policy
    
    4、#当满足tm_policy的策略时使用tm_server的backend
    use_backend tm_server if tm_policy
    
    5、#reqisetbe关键字定义，根据定义的关键字选择backend
    reqisetbe ^Host:\ img dynamic
    reqisetbe ^[^\ ]*\ /(img|css)/ dynamic
    reqisetbe ^[^\ ]*\ /admin/stats stats
    
    6、#以上都不满足的时候使用默认mms_server的backend
    default_backend mms

```

[参考资料](http://www.jianshu.com/p/c9f6d55288c0)