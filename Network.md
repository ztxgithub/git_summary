### 网络方面

- 在配置ip时要注意mac地址会不会冲突

- 网段表示 172.16.82.0/25

``` shell
    
   172.16.82.0是子网号,25是子网掩码的位数(代表前面25位是子网号,后面7位是主机号)
	
```

##基于SCClient 程序

- 一个网卡增加多个虚拟ip

``` shell
    
    $ ifconfig eth0:0 192.0.0.65 netmask 255.255.255.0
	
```

- 删除虚拟ip

``` shell
    
    $ ifconfig eth0:0 down
	
```
- arp协议

``` shell
    
   描述：
        必须要在同一个物理局域网内 
        $ arping 192.0.0.65
        
        输入arping 命令的主机必须跟　192.0.0.65　在同一个物理网段,此时就得到 mac地址
	
```

- 判断系统有没有eth0等网络接口

``` c
    
    看有没有　/sys/class/net/eth0 文件
	
```

- shell 命令 lsusb 其实从文件 /proc/bus/usb/devices 中读取

- 获取 network interface Name 对应的 ip

``` c
    
    string ConnectVPN::GetIfIpAddr(const string &interfaceName) {
        string rec = "";
        struct ifreq ifr;
        int inetSock = socket(AF_INET, SOCK_DGRAM, 0);
        memset(&ifr, 0, sizeof(ifr));
        ifr.ifr_addr.sa_family = AF_INET;
        strcpy(ifr.ifr_name, interfaceName.c_str());
        if(!(ioctl(inetSock, SIOCGIFADDR, &ifr) < 0)){
            rec =  inet_ntoa(((struct sockaddr_in*)&(ifr.ifr_addr))->sin_addr);
        }
        close(inetSock);
        return rec;
    
    }
	
```

- 在路由表中获取目的ip对应的网关(根据/proc/net/route)

``` c
    
    string ConnectVPN::GetGateway(const string &destAddr) {
        char devName[64];
        unsigned long _destAddr, gateway, mask;
        int flgs, ref, use, metric, mtu, win, ir;
        struct in_addr gw;
    
        FILE *fp = fopen("/proc/net/route", "r");
        if(!fp) return "";
    
        if(fscanf(fp, "%*[^\n]\n") < 0) {
            fclose(fp);
            return "";
        }
    
        int r = 0;
        while(r >= 0 && !feof(fp)) {
            r = fscanf(fp, "%63s%lx%lx%X%d%d%d%lx%d%d%d\n",
                       devName, &_destAddr, &gateway, &flgs, &ref, &use, &metric, &mask,
                       &mtu, &win, &ir);
    
            if((r == 11) && (_destAddr == inet_addr(destAddr.c_str()))) {
                gw.s_addr =  gateway;
                fclose(fp);
                return inet_ntoa(gw);
            }
        }
        fclose(fp);
        return "";
    }
	
```

- libpcap 动态链接库

``` c
    
    主要用于网络接口程序:获取网络接口,释放网络接口, 打开网络接口, 获取数据包
	
```

### iptables 防火墙

#### 概述:
    当我们用iptables添加规则，这些规则保存在/etc/sysconfig/iptables 文件中.
    iptables的结构：
        iptables -> Tables -> Chains -> Rules. tables由chains组成，而chains又由rules组成的.
        
    iptables具有Filter, NAT, Mangle, Raw四种内建表:
        1. Filter表
        Filter表示iptables的默认表,因此如果你没有自定义表,那么就默认使用filter表,它具有以下三种内建链：
            INPUT链 – 处理来自外部的数据。
            OUTPUT链 – 处理向外发送的数据。
            FORWARD链 – 将数据转发到本机的其他网卡设备上.
            
        2. NAT表
        NAT表有三种内建链：
            PREROUTING链 – 处理刚到达本机并在路由转发前的数据包.它会转换数据包中的目标IP地址（destination ip address）,
                            通常用于DNAT(destination NAT)。
            POSTROUTING链 – 处理即将离开本机的数据包.它会转换数据包中的源IP地址（source ip address）,通常用于SNAT(source NAT)
            OUTPUT链 – 处理本机产生的数据包.
            
        3. Mangle表
        Mangle表用于指定如何处理数据包.它能改变TCP头中的QoS位。Mangle表具有5个内建链：
            PREROUTING
            OUTPUT
            FORWARD
            INPUT
            POSTROUTING
            
        4. Raw表
        Raw表用于处理异常，它具有2个内建链：
            PREROUTING chain
            OUTPUT chain
            
    IPTABLES 规则(Rules):
        Rules包括一个条件和一个目标(target)
        如果满足条件，就执行目标(target)中的规则或者特定值
        如果不满足条件，就判断下一条Rules
        
    目标值（Target Values）
    下面是你可以在target里指定的特殊值：
        ACCEPT – 允许防火墙接收数据包
        DROP – 防火墙丢弃包
        QUEUE – 防火墙将数据包移交到用户空间
        RETURN – 防火墙停止执行当前链中的后续Rules，并返回到调用链(the calling chain)中
    
#### 防火墙命令:
    
- 查看防火墙表的规则

``` shell
    
    iptables -t table --list
    
        -L == --list
        --line-numbers : 显示规则的行号
        -n：以数字的方式显示ip，它会将ip直接显示出来，如果不加-n，则会将ip反向解析成主机名。
        -v：显示详细信息
    
    例如:查看filter表的规则
    $ iptables -t filter --list
       
```

- 清空所有iptables规则

``` shell
    
    iptables -F
    
    清空后仍然需要查看规则,有的linux并不会清空 NAT 表,此时需要手动清除(iptables -t NAT -F)
       
```

- iptables规则

``` shell
    
    iptables [-t table] COMMAND chain CRETIRIA -j ACTION
        
        table: filter, nat, mangle
        
        COMMAND:
            1.链管理命令
            
                (1) -P :设置默认策略的（设定默认门是关着的还是开着的）
                    iptables -P INPUT DROP 这就把默认规则给拒绝了.并且没有定义哪个动作，
                    所以关于外界连接的所有规则包括Xshell连接之类的，远程连接都被拒绝了
                    
                (2) -F: FLASH,清空规则链的
                        iptables -t nat -F PREROUTING  (清空nat表的PREROUTING链)
                        iptables -t nat -F (清空nat表的所有链)
                        
                (3) -N: 支持用户新建一个链
                        iptables -N inbound_tcp_web 表示附在tcp表上用于检查web的.
                 
                (4) -X: 用于删除用户自定义的空链
                        iptables -X inbound_tcp_web, 但是在删除之前必须要将里面的链给清空昂了
                        
                (5) -E：用来Rename chain主要是用来给用户自定义的链重命名
                        iptables -E oldname newname
                        
             2. 规则管理命令
             
                (1) -A：追加,在当前链的最后新增一个规则
                
                (2) -I num : 插入，把当前规则插入为第几条
                
                (3) -R num：Replays替换/修改第几条规则
                
                (4) -D num：删除，明确指定删除第几条规则
                
         chain: 
            filter表的链: INPUT链, OUTPUT链, FORWARD链
            nat表的链: PREROUTING链, POSTROUTING链, OUTPUT链
            mangle表的链: filter表的链 和  nat表的链
            
        CRETIRIA:
                这些描述是对规则的基本描述。
                      -p 协议（protocol）
                          指定规则的协议，如tcp, udp, icmp等，可以使用all来指定所有协议。
                          如果不指定-p参数，则默认是all值。这并不明智,最好要指定协议
                          可以使用协议名(如tcp)，或者是协议值（比如6代表tcp）来指定协议,映射关系请查看/etc/protocols
                          还可以使用–protocol参数代替-p参数
                      -s 源地址（source）
                          指定数据包的源地址
                          参数可以使IP地址、网络地址、主机名
                          例如：-s 192.168.1.101指定IP地址
                          例如：-s 192.168.1.10/24指定网络地址
                          如果不指定-s参数，就代表所有地址
                          还可以使用–src或者–source
                      -d 目的地址（destination）
                          指定目的地址
                          参数和-s相同
                          还可以使用–dst或者–destination
                      -j 执行目标（jump to target）
                           代表”jump to target”
                           ACCEPT, DROP, QUEUE, RETURN
                          还可以指定其他链（Chain）作为目标
                      -i 输入接口（input interface）
                          -i代表输入接口(input interface),指定了要处理来自哪个接口的数据包,
                          这些数据包即将进入INPUT, FORWARD, PREROUTE链
                          例如：-i eth0指定了要处理经由eth0进入的数据包
                          如果不指定-i参数，那么将处理进入所有接口的数据包
                          如果出现! -i eth0，那么将处理所有经由eth0以外的接口进入的数据包
                          如果出现-i eth+，那么将处理所有经由eth开头的接口进入的数据包
                          还可以使用–in-interface参数
                      -o 输出（out interface）
                      -o eth0
                      -o指定了数据包由哪个接口输出
                        这些数据包即将进入FORWARD, OUTPUT, POSTROUTING链, 如果不指定-o选项,
                        那么系统上的所有接口都可以作为输出接口
                        如果出现! -o eth0，那么将从eth0以外的接口输出, 如果出现-i eth+,那么将仅从eth开头的接口输出
                        还可以使用–out-interface参数
       
                 描述规则的扩展参数:
                     --sport 源端口（source port）针对 -p tcp 或者 -p udp
                         缺省情况下，将匹配所有端口, 可以指定端口号或者端口名称，例如”–sport 22″与”–sport ssh”, 
                         /etc/services文件描述了上述映射关系, 从性能上讲，使用端口号更好,使用冒号可以匹配端口范围,
                         如”--sport 22:100″(--sport 22-100) 
                         还可以使用”–source-port”
                         
                     --dport 目的端口（destination port）针对-p tcp 或者 -p udp
                     参数和–sport类似还可以使用”–destination-port”
                     
                     显式扩展（-m）:扩展各种模块
                     -m multiport：表示启用多端口扩展,之后我们就可以启用比如 --dports 21,23,80
                     
                     --tcp-flags TCP标志 针对-p tcp
                     可以指定由逗号分隔的多个参数,有效值可以是：SYN, ACK, FIN, RST, URG, PSH,可以使用ALL或者NONE
                     
                     -–icmp-type ICMP类型 针对-p icmp
                     –icmp-type 0 表示Echo Reply
                     –icmp-type 8 表示Echo
                     
                     我们允许自己ping别人，但是别人ping自己ping不通
                        iptables -A INPUT -p icmp --icmp-type 0 -j ACCEPT
                        iptables -A OUTPUT -p icmp --icmp-type 8 -j ACCEPT
                        
                        iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT
                        iptables -A OUTPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT
                     
         -j ACTION:
                 常用的ACTION：
                    DROP：悄悄丢弃,一般我们多用DROP来隐藏我们的身份，以及隐藏我们的链表
                    ACCEPT：接受
                    REJECT：明示拒绝
                    custom_chain：转向一个自定义的链
                    DNAT
                    SNAT
                    MASQUERADE：源地址伪装
                    REDIRECT：重定向：主要用于实现端口重定向
                    MARK：打防火墙标记的
                    RETURN：返回
        
        1.添加新规则
        
         新的规则将追加到链尾,一般而言,最后一条规则用于丢弃(DROP)所有数据包.如果你已经有这样的规则了,
                并且使用 -A参数添加新规则，那么就是无用功
             iptables -A chain firewall-rule
                -A chain – 指定要追加规则的链(INPUT链, OUTPUT链等)
                firewall-rule – 具体的规则参数
                
        2.允许已建立的或相关连的通行
        
          $ iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT 
          
         
```

- iptable 配置文件保存

``` shell
    设置完防火墙配置后,要保存到 /etc/sysconfig/iptables文件中,
    
    方法一:
        service iptables save
        
    方法二:
        iptables-save > /etc/sysconfig/iptables
       
```

- SNAT和DNAT的实现(IP地址转化)

``` shell

    要进行地址转发,首先要开启Linux系统中的地址转发功能
        1.永久改变
        (1): 修改/etc/sysctl.conf配置文件件，将ip_forward的值设置为1
             vim /etc/sysctl.conf   net.ipv4.ip_forwaed=1  
             
        (2): sysctl -p     //重新读取修改后的配置
        
        2.临时改变,重启恢复
        # echo "1" > /proc/sys/net/ipv4/ip_forward
        
    iptables -t nat 过滤条件(筛选条件) DNAT/SNAT 执行动作

    1.DNAT目标地址转换
        处理刚到达本机并在路由转发前的数据包.它会转换数据包中的目标IP地址（destination ip address）
        DNAT只能用在nat表的PREROUTING链和OUTPUT链中
        
      1) 公司内部局域网内搭建了一台web服务器，IP地址为192.168.1.7，现在需要将其发布到互联网上,
          希望通过互联网访问web服务器。那么我们可以执行如下操作
          
            在iptables的PREROUTING中编写DNAT规则。
            # iptables -t nat -A PREROUTING -i eth0 -d 218.29.30.31(外网ip) -p tcp --dport 80 -j DNAT 
              --to-destination 192.168.1.7:80
            
      2) 公司的web服务器192.168.1.7需要通过互联网远程管理,由于考虑到安全问题,管理员不希望使用默认端口进行访问,
          这时我们可以使用DNAT修改服务的默认端口。操作如下：
            #iptables -t nat -A PREROUTING -i eth0 -d 218.29.30.31 -p tcp --dport 2346 -j DNAT 
             --to-destination 192.168.1.7:22
             
      3）在外网客户端浏览器中访问网关服务器的外网接口，可以发现访问的居然是内网192.168.1.7的web服务器的网页。
        而在使用sshd连接2346端口时,居然可以远程连接到192.168.1.7服务器上。
        
    2.SNAT基于源地址的转换
        处理即将离开本机的数据包.它会转换数据包中的源IP地址（source ip address）,通常用于将内网段的ip转化为一个外网ip地址.
        SNAT只能用在nat表的POSTROUTING链
        
        将所有192.168.10.0网段的IP在发送的时候全都转换成172.16.100.1这个外网地址
            iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -j SNAT --to-source 172.16.100.1
            
        如果外网的ip不固定,外网地址换成 MASQUERADE(动态伪装):它可以实现自动寻找到外网地址,而自动将其改为正确的外网地址
        对于ADSL宽带连接来说，连接名称通常为ppp0，ppp1等。操作如下
        # iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o ppp0 -j MASQUERADE
        
        
    具体实例:
        FSU有2个网卡,对应2个ip,一个是fsu_vpn_ip,一个是192.0.0.65,现在在同一个vpn域的pc,要对FSU的 fsu_vpn_ip:8088操作,实际
        是对连接在fsu下ip为192.0.0.64:80的摄像头进行操作.
        
        1.在FSU内部只要目的端口是8088,将dst_ip(应该是fsu_vpn_ip)改为摄像头ip(192.0.0.64):80
        
                                         |-----筛选条件-----------|         |--执行动作,将目的ip改成 192.0.0.64:80|
        iptables -t nat -A PREROUTING  -m tcp -p tcp --dport 8088 -j DNAT --to-destination 192.0.0.64:80
        
        2.将对应的数据包的源地址修改一下,改成FSU的本地ip(192.0.0.65)
        
                                       |------------筛选条件------------------|
        iptables -t nat -A POSTROUTING -m tcp -p tcp --dport 80 -d 192.0.0.64 -j SNAT --to-source 192.0.0.65
           
```
[参考资料](http://blog.51yip.com/linux/1404.html)