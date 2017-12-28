# 网络攻防

##  远程登陆


### 修改ssh服务的端口号

``` shell
    通过iptables进行端口映射,或则修改ssh服务的配置信息,将默认的端口号22改为不常见的端口号
			
```

### 限制sshd服务的访问来源

``` shell
    通过iptables进行编写防火墙规则,只允许特定的范围的ip进行访问(比如在同一个vpn内和同一个局域网192.0.0.65/24)
			
```

### 使用证书而不是密码来远程登录

``` shell
   在公司的服务器生成一对秘钥(公钥和私钥),同时在FSU上传入对应的公钥,这时只有当公司的这台服务器才能登陆到FSU
			
```

### 其他远程登陆方式

``` shell
   1.使用denyhosts解决ssh暴力破解
			
```

## 对开放的服务进行攻击

### 服务拒绝攻击（DoS denial-of-service）

``` shell
    限制同时打开的Syn半连接数目、缩短Syn半连接的time out 时间
			
```

#### netwox工具

``` shell

        Ubuntu上安装 netwox
            > apt-get install netwox
            
        
		netwox工具是集结了许多功能的软件箱.
			1.netwox 76 号工具  TCP SYN Flood攻击
```

#### TCP SYN Flood 攻击

``` shell
    当服务器收到一个TCP SYN报文后,马上为该连接请求分配缓冲区,然后返回一个SYN＋ACK报文,这时形成一个半连接。
    SYN Flood正是利用了这一点,发送大量的伪造源地址的SYN连接请求，而不完成连接。这样就大量的消耗的服务器的资源，消耗目标主机的连接队列资源,
    从而不能正常为用户提供服务
    
    TCP SYN Flood 攻击类型：
        1． Direct Attack 攻击方使用固定的源地址发起攻击，这种方法对攻击方的消耗最小
        2． Spoofing Attack 攻击方使用变化的源地址发起攻击，这种方法需要攻击方不停地修改源地址，实际上消耗也不大
        3． Distributed Direct Attack 这种攻击主要是使用僵尸网络(控制其他电脑)进行固定源地址的攻击
    
      
    使用netwox进行TCP SYN Flood 攻击
        > sudo netwox 76 -i "10.0.6.251" -p 22
        
    防御措施:
        (1)修改linux内核参数
            1.增大tcp_max_syn_backlog
            2.减小tcp_synack_retries
            3.启用tcp_syncookies
                根据这个CentOS SYN包计算出一个cookie值.在收到TCP ACK包时,TCP服务器在根据那个cookie值检查这个TCP ACK包的合法性。
                如果合法，再分配专门的数据区进行处理未来的TCP连接
            
        (2)
			
```

## 其他

``` shell
    1.关闭掉不需要的服务(端口号)
    2.rootkit入侵
        rootkit程序有以下几个功能:
            1.可以获取网络传输的用户名和密码
            2.可以给攻击者提供远程登陆的权限
            3.可以隐藏rootkit安装目录和运行的进程,例如重写ps、netstat命令
            4.清理日志文件,例如wtmp、utmp和lastlog
        
        攻击原理：
            攻击者首先找出目标系统上的现有漏洞(漏洞可能包括：开放的网络端口、未打补丁的系统或者具有脆弱的管理员密码的系统),
            再安装rootkit软件
            
        检验是否被攻击：
            与原先备份的ps,netstat的md5sum进行比较
            
        解决方法：
            从一个同版本可信操作系统下拷贝所有命令到这个入侵服务器下某个路径,然后在执行命令的时候指定此命令的完整路径即可.
            尝试查看/var/log/messages,/var/log/wtmp,/var/log/secure等日志.也可以通过/etc/shadow查看
			
```
