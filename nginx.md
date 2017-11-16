# Nginx

## Nginx 安装步骤

``` shell

nginx依赖以下模块：
    1. gzip模块需要 zlib 库
    2. rewrite模块需要 pcre 库

mkdir nginx在home目录下创建nginx目录,作为nginx的安装目录.

    1.安装pcre
        (1) 获取pcre编译安装包，在http://www.pcre.org/上可以获取当前最新的版本,也可使用附件中的软件包。
        (2) 解压缩pcre-xx.zip包。
        (3) 进入解压缩目录，执行./configure。
        (4) make & make install
        
    2.安装zlib
        (1) 获取zlib编译安装包，在http://www.zlib.net/上可以获取当前最新的版本,也可使用附件中的软件包。
        (2) 解压缩openssl-xx.tar.gz包。
        (3) 进入解压缩目录，执行./configure。
        (4) make & make install
        
    3.安装nginx
        (1) 获取nginx，在http://nginx.org/en/download.html上可以获取当前最新的版本,也可使用附件中的软件包。
        (2) 解压缩nginx-xx.tar.gz包。
        (3) 进入解压缩目录，执行./configure --prefix=/home/nginx --with-stream
        (4) make & make install			
```
[Nginx安装](https://segmentfault.com/a/1190000007116797)

## Nginx 概要
    Nginx可以基于TCP协议的代理转发以及负载均衡.
    Nginx可以在线服务状态下平滑升级
    
## Nginx 配置

``` shell

    基于http协议:3个部分:main(全局设置),events(nginx工作模式),
                       http(sever(主机设置 包含location(URL匹配)子子配置),upstream(负载均衡服务器设置) 等子配置)
                       
    基于tcp协议:3个部分:main(全局设置),events(nginx工作模式),
                      stream(sever(主机设置),upstream(负载均衡服务器设置) 等子配置)

    main模块:
    
    #定义Nginx运行的用户和用户组
        #user  nobody; 
    
    #nginx进程数，建议设置为等于CPU总核心数。
        worker_processes  1; 
    
    #全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
        #error_log  logs/error.log;
        #error_log  logs/error.log  notice;
        #error_log  logs/error.log  info;
    
    #进程文件
        #pid        logs/nginx.pid;
    
    #一个nginx进程打开的最多文件描述符数目,理论值应该是最多打开文件数(系统的值ulimit -n)与nginx进程数相除,
    但是nginx分配请求并不均匀,所以建议与ulimit -n的值保持一致.
        worker_rlimit_nofile 65535;
    
    events 模块:
    
    #工作模式与连接数上限
        events {
        
            #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; 
             epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，就用kqueue模型。
                use epoll;
            #单个进程最大连接数（最大连接数=连接数*进程数 worker_processes*worker_connections）
                worker_connections  1024;
        }
        
        
    server 子模块: 它用来定一个虚拟主机
    
        # 用于指定虚拟主机的服务端口(listen)
            listen 192.168.0.8:2222
            
        # 后端服务器连接的超时时间_发起握手等候响应超时时间,默认是60s,超时通常不能超过75秒 
           proxy_connect_timeout 60
         
        #  该指令设置与代理服务器的读超时时间,它决定了nginx会等待多长时间来获得请求的响应,
            这个时间不是获得整个response的时间,而是两次reading操作的时间,默认60s
           proxy_read_timeout 60
           
        # 说明这个指定设置了发送请求给upstream服务器的超时时间.超时设置不是为了整个发送期间,而是在两次write操作期间.
          如果超时后,upstream没有收到新的数据,nginx会关闭连接,默认60s
            proxy_send_timeout 60
            
        # 在客户端或代理服务器连接上的两个连续的读取或写入操作之间设置timeout.如果在此时间内没有数据传输,则连接被关闭
            proxy_timeout 10m
            
        # 对应于upstream 的名称    
            proxy_pass AAA
           
           
```