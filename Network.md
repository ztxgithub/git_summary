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

