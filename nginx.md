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
            
        # 在客户端或代理服务器连接上的两个连续的读取或写入操作之间设置timeout.如果在此时间内没有数据传输,则连接被关闭
            proxy_timeout 10m
            
        # 对应于upstream 的名称    
            proxy_pass AAA
            
       http协议特有的
        #  该指令设置与代理服务器的读超时时间,它决定了nginx会等待多长时间来获得请求的响应,
            这个时间不是获得整个response的时间,而是两次reading操作的时间,默认60s
           proxy_read_timeout 60
           
        # 说明这个指定设置了发送请求给upstream服务器的超时时间.超时设置不是为了整个发送期间,而是在两次write操作期间.
          如果超时后,upstream没有收到新的数据,nginx会关闭连接,默认60s
            proxy_send_timeout 60
            
            
    upstram 模块: 负载均衡模块,通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡
    
        适用于http协议:
        Nginx的负载均衡模块目前支持4种调度算法:
        
            1.weight 轮询（默认）
                每个请求按时间顺序逐一分配到不同的后端服务器,如果后端某台服务器宕机,故障系统被自动剔除，使用户访问不受影响.weight指定
                轮询权值,weight值越大,分配到的访问机率越高,主要用于后端每个服务器性能不均的情况下.
            2.ip_hash
                每个请求按访问IP的hash结果分配,这样来自同一个IP的访客固定访问一个后端服务器,有效解决了动态网页存在的session共享问题
            3.fair
                比上面两个更加智能的负载均衡算法.此种算法可以依据页面大小和加载时间长短智能地进行负载均衡,
                也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持fair的,
                如果需要使用这种调度算法，必须下载Nginx的upstream_fair模块。
            4.url_hash
                按访问url的hash结果来分配请求,使每个url定向到同一个后端服务器,可以进一步提高后端缓存服务器的效率.
                Nginx本身是不支持url_hash的,如果需要使用这种调度算法,必须安装Nginx 的hash软件包.
                
        通过server指令指定后端服务器的IP地址和端口，,同时还可以设定每个后端服务器在负载均衡调度中的状态.常用的状态有
            1.down,表示当前的server暂时不参与负载均衡。
            2.backup,预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。
            3.max_fails,允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。
            4.fail_timeout，在经历了max_fails次失败后，暂停服务的时间.max_fails可以和fail_timeout一起使用.
            
        注意 当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup
        
        upstream emqttd{
            ip_hash;
            server 192.168.12.1:80;
            server 192.168.12.2:80 down;
            server 192.168.12.3:8080  max_fails=3  fail_timeout=20s;
            server 192.168.12.4:8080;
        }
        
        适用于tcp协议:
            tcp也支持几种负载均衡方式 round-robin, least_conn , hash等
            
                upstream stream_backend {
                     zone tcp_servers 64k;
                    hash $remote_addr; // hash $remote_addr consistent (consistent指定一致性哈希算法)
                    server 192.168.0.3:1883 max_fails=2 fail_timeout=30s max_conns=5000; 在这里添加要代理的mqtt
                    server 10.0.0.250:1883 max_fails=2 fail_timeout=30s;    
            }        
```

## 用途

### 静态HTTP服务器

```shell
    1.Nginx是一个HTTP服务器，可以将服务器上的静态文件（如HTML、图片）通过HTTP协议展现给客户端
    2.实例：
        server {
        	listen 80;
        	location {
        		root /usr/share/ngnix/html;  #静态文件路径
        	}
        }
```

### 反向代理服务器

```shell
    1.客户端本来可以直接通过HTTP协议访问某网站应用服务器,如果网站管理员在中间加上一个Nginx,
      客户端请求Nginx,Nginx请求应用服务器,然后将结果返回给客户端,此时Nginx就是反向代理服务器.
      
    2.
        server {
        	listen 80;
        	location {
        		proxy_pass http://192.168.20.1:8080;  #应用服务器http地址
        	}
        }
```

### 负载均衡

```shell
    upstream deal_porc{
    	server 192.168.20.1:8080； #应用服务器1
    	server 192.168.20.2:8080； #应用服务器2
    }
    
    server {
    	listen 80;
    	location {
    		proxy_pass http://deal_porc；
    	}
    }
```

### 虚拟主机

```shell
    1.有的网站访问量大,需要负载均衡,然而并不是所有网站都如此出色，有的网站，由于访问量太小，需要节省成本，
      需要将多个网站部署在同一台服务器上。
      例如将www.aaa.com和www.bbb.com两个网站部署在同一台服务器上，两个域名解析到同一个IP地址，
      但是用户通过两个域名却可以打开两个完全不同的网站,互相不影响,就像访问两个服务器一样,所以叫两个虚拟主机.
      
    2.
        server {
        	listen 80 default_servere;
        	server_name _;
        	return 444;   # 过滤其他域名的请求，返回444状态码
        }
        
        server {
        	listen 80;
        	server_name www.aaa.com; #www.aaa.com 域名
        	location {
        		proxy_pass http://localhost:8080；  #对应端口号8080
        	}
        }
        server {
        	listen 80;
        	server_name www.bbb.com; #www.bbb.com 域名
        	location {
        		proxy_pass http://localhost:8081；  #对应端口号8081
        	}
        }
```