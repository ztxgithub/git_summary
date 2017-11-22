# Keepalived

## Keepalived 基本原理

``` shell

     KeepAlived是一个高可用方案,通过VIP(即虚拟IP)和心跳检测来实现高可用.其原理是存在一组（两台）服务器,分别赋予Master,Backup两个角色,
     默认情况下Master会绑定VIP到自己的网卡上,对外提供服务.Master,Backup会在一定的时间间隔向对方发送心跳数据包来检测对方的状态,
     这个时间间隔一般为2秒钟,如果Backup发现Master宕机,那么Backup会发送ARP包到网关,把VIP绑定到自己的网卡,此时Backup对外提供服务,
     实现自动化的故障转移,当Master恢复的时候会重新接管服务.
     
     Keepalived监控A、B上的HAproxy,利用Keepalived的VIP漂移技术,若A、B上的HAprox都工作正常,
     则VIP与优先级别高的服务器（主服务器）绑定,当主服务器当掉时,则与从服务器绑定,而VIP则是暴露给外部访问的ip.	
     	
     在两台HAProxy的主机上分别运行着一个Keepalived实例,这两个Keepalived争抢同一个虚IP地址,
     两个HAProxy也尝试去绑定这同一个虚IP地址上的端口.
     同时只能有一个Keepalived抢到这个虚IP,抢到了这个虚IP的Keepalived主机上的HAProxy便是当前的MASTER。
     Keepalived内部维护一个权重值,权重值最高的Keepalived实例能够抢到虚IP。同时Keepalived会定期check本主机上的HAProxy状态,
     状态OK时权重值增加。
     
```

## Keepalived 下载安装

``` shell
   下载地址： http://www.keepalived.org/software/
   
    1.在安装Keepalived之前,先安装openssl
        1.> wget http://www.openssl.org/source/openssl-1.0.2f.tar.gz
        2.> tar -xzf openssl-1.0.2f.tar.gz
        3.> cd openssl-1.0.2f
        4.> mkdir /usr/local/openssl
        5.>  ./config --prefix=/usr/local/(最好是这个目录,如果是/usr/local/openssl编译时会有问题)
        6.> make
        7.> make install
        
        另外一种方法：
            > yum -y install openssl-devel
            
    2. 安装 libnl/libnl-3 库
            > yum install libnl*
            
    3. 
        yum install libnfnetlink-devel
            
    4.> ./configure --prefix=/home/yytd/keepalived
    5.> make
    6.> make install
        
    
```

[参考资料](http://www.cnblogs.com/xl-yin/articles/Keepalived.html)

## 配置Keepalived

``` shell

    1.注册为系统服务
        (1) > cp /home/yytd/keepalived/sbin/keepalived /usr/sbin/
        (2) > cp /home/yytd/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
        (3) > mkdir -p /etc/keepalived && vim /etc/keepalived/keepalived.conf 
        
                global_defs {
                   router_id LVS_DEVEL   #虚拟路由名称
                }
                
                vrrp_script chk_haproxy {
                        script "killall -0 haproxy"  #使用killall -0检查haproxy实例是否存在，性能高于ps命令
                        interval 2    #脚本运行周期
                        weight 2      #检查的加权权重值
                        fall 2        #命令/脚本执行失败多少次才算真的失败,即当出现2次shell脚本返回值为非0时，才priority减2
                        rise 1        #命令/脚本执行成功多少次才算真的成功  
                }　　　
                每次检测对应的priority不会每次都增加或减少,当keepalived成功启动后,检查到haproxy成功运行,priority加2,而后不再增加，
                如果这时后检测到haproxy异常断开,则priority减2,而后不再减少．
                #虚拟路由配置
                vrrp_instance VI_1 {
                    state MASTER　　　#本机实例状态，MASTER/BACKUP，备机配置文件中请写BACKUP
                    interface eno16780032　　 #本机网卡名称，使用ifconfig命令查看
                    virtual_router_id 51　　　#虚拟路由编号，主备机保持一致
                    priority 120　　　　　 #本机初始权重,备机请填写小于主机的值（例如100）
                    advert_int 1          #争抢虚地址的周期，秒
                    virtual_ipaddress {
                        192.168.0.121     #虚地址IP，主备机保持一致
                    }
                    track_script {
                        chk_haproxy  #对应的健康检查配置
                    }
                }
                
        (4) touch /etc/init.d/keepalived
        (5) chmod +x /etc/init.d/keepalived
        (6) vi /etc/init.d/keepalived
                #!/bin/sh
                #
                # Startup script for the Keepalived daemon
                #
                # processname: keepalived
                # pidfile: /var/run/keepalived.pid
                # config: /etc/keepalived/keepalived.conf
                # chkconfig: - 21 79
                # description: Start and stop Keepalived
                
                # Source function library
                . /etc/rc.d/init.d/functions
                
                # Source configuration file (we set KEEPALIVED_OPTIONS there)
                . /etc/sysconfig/keepalived
                
                RETVAL=0
                
                prog="keepalived"
                
                start() {
                    echo -n $"Starting $prog: "
                    daemon keepalived ${KEEPALIVED_OPTIONS}
                    RETVAL=$?
                    echo
                    [ $RETVAL -eq 0 ] && touch /var/lock/subsys/$prog
                }
                
                stop() {
                    echo -n $"Stopping $prog: "
                    killproc keepalived
                    RETVAL=$?
                    echo
                    [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$prog
                }
                
                reload() {
                    echo -n $"Reloading $prog: "
                    killproc keepalived -1
                    RETVAL=$?
                    echo
                }
                
                # See how we were called.
                case "$1" in
                    start)
                        start
                        ;;
                    stop)
                        stop
                        ;;
                    reload)
                        reload
                        ;;
                    restart)
                        stop
                        start
                        ;;
                    condrestart)
                        if [ -f /var/lock/subsys/$prog ]; then
                            stop
                            start
                        fi
                        ;;
                    status)
                        status keepalived
                        RETVAL=$?
                        ;;
                    *)
                        echo "Usage: $0 {start|stop|reload|restart|condrestart|status}"
                        RETVAL=1
                esac
                
                exit $RETVAL

  
    2.service keepalived start
    
    3.查看虚拟 IP 持有者
        > ip addr sh eno16780032
        结果：
            2: eno16780032: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
                link/ether 00:0c:29:24:fe:f4 brd ff:ff:ff:ff:ff:ff
                inet 192.168.0.4/24 brd 192.168.0.255 scope global eno16780032
                   valid_lft forever preferred_lft forever
                inet 192.168.0.121/32 scope global eno16780032
                   valid_lft forever preferred_lft forever
                inet6 fe80::20c:29ff:fe24:fef4/64 scope link 
                   valid_lft forever preferred_lft forever
    
```

[参考资料](http://www.cpper.cn/2016/09/15/architecture/keepalived-for-master-backup/)

## 注意事项

```shell
    1.通过虚IP无法访问到HAProxy，通过主机 IP 可以访问
        解决方法：配置 HAProxy 时,bind 的配置,写成了固定IP:PORT的格式,此时HAProxy无法绑定虚拟 IP .
        将改行配置改成 bind *：port,问题解决
        
```
