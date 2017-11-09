# emqttd 简介

``` shell
    1.mqtt协议: MQTT是一个轻量的发布订阅模式消息传输协议,专门针对低带宽和不稳定网络环境的物联网应用设计.
               特点:
                    (1) 发布订阅模式,一对多消息发布
                    (2) 基于TCP/IP网络连接
                    (3) 1字节固定报头,2字节心跳报文,报文结构紧凑
                    (4) 消息QoS支持,可靠传输保证
                    
    2.mqtt主题: 主题(Topic)通过’/’分割层级,支持’+’, ‘#’通配符
    
                '+': 表示通配一个层级,例如a/+，匹配a/x, a/y
                '#': 表示通配多个层级,(主题)topic的最后一个字符,例如a/#,匹配a/x, a/b/c/d
                注意:
                    订阅者可以订阅含通配符主题,但发布者不允许向含通配符主题发布消息.
                    
    3.报文类型:
    
        类型名称--类型值--报文说明
        CONNECT	--1--   发起连接
            Client(FSU_Client_JGsafety33010400010001@10.0.6.185:57197):
             RECV CONNECT(Q0, R0, D0, ClientId=FSU_Client_JGsafety33010400010001, ProtoName=MQTT, 
                          ProtoVsn=4, CleanSess=true, KeepAlive=20, Username=undefined, Password=undefined,
                           Will(Q2, R0, Topic=33010400010001/FSULogout/Sengine_test, Msg=JGsafety))
        CONNACK --2--   连接回执
            Client(FSU_Client_JGsafety33010400010001@10.0.6.185:57197): 
             SEND CONNACK(Q0, R0, D0, AckFlags=0, RetainCode=0)
        PUBLISH	--3--   发布消息
        PUBACK  --4--	发布回执
        PUBREC  --5--	QoS2消息回执
        PUBREL  --6--	QoS2消息释放
        PUBCOMP	--7--	QoS2消息完成
        SUBSCRIBE --8--	订阅主题
        SUBACK	--9--	订阅回执
        UNSUBSCRIBE	--10--	取消订阅
        UNSUBACK	--11--	取消订阅回执
        PINGREQ	--12--	PING请求
        PINGRESP --13--	PING响应
        DISCONNECT	--14--	断开连接
        
        发布消息流程:
            对于QoS2报文:PUBLISH->PUBREC->PUBREL->PUBCOMP
            对于QoS1报文:PUBLISH->PUBACK
            
        MQTT发布消息QoS保证不是端到端的，是客户端与服务器之间的
        
    4.MQTT会话(Clean Session)
        MQTT客户端向服务器发起CONNECT请求时,可以通过’Clean Session’标志设置会话.
            ‘Clean Session’设置为0,表示创建一个持久会话,在客户端断开连接时,会话仍然保持并保存离线消息,直到会话超时注销.
            ‘Clean Session’设置为1,表示创建一个新的临时会话,在客户端断开时,会话自动销毁.
            
    5.MQTT连接保活心跳
        MQTT客户端向服务器发起CONNECT请求时，通过KeepAlive参数设置保活周期。
        客户端在无报文发送时,按KeepAlive周期定时发送2字节的PINGREQ心跳报文,服务端收到PINGREQ报文后,回复2字节的PINGRESP报文.
        服务端在1.5个心跳周期内,既没有收到客户端发布订阅报文,也没有收到PINGREQ心跳报文时,主动心跳超时断开客户端TCP连接.
        (emqttd消息服务器按2.5心跳周期超时设计)
        
    6.MQTT遗愿消息(Last Will)
        MQTT客户端向服务器端CONNECT请求时,可以设置是否发送遗愿消息(Will Message)标志,和遗愿消息主题(Topic)与内容(Payload)
        MQTT客户端异常下线时(客户端断开前未向服务器发送DISCONNECT消息),MQTT消息服务器会发布遗愿消息给订阅该遗愿的客户端.
        
    7.MQTT保留消息(Retained Message)
        MQTT客户端向服务器发布(PUBLISH)消息时,可以设置保留消息(Retained Message)标志.
        保留消息(Retained Message)会驻留在消息服务器,后来的订阅者订阅主题时仍可以接收该消息.
        在Eclipse Paho中MQTTAsync_message结构体中,如果是发送消息时设置retained==1,那么后来的订阅者也会收到该消息.
        在接受消息的时候,可以根据retained的值来判断是实时发送的消息还是保留的消息,如果是retained==1则表明收到的消息
        是保留的消息不是实时的.
```



# emqttd 实际应用

- 进程的内部pid

``` shell

   进程名beam.smp
   进程名epmd(端口号4369)
   
   如果是自己启动emqttd
   1.# 启动emqttd
     ./bin/emqttd start  (作为服务程序)
     
   2.# 检查运行状态
     ./bin/emqttd_ctl status
     
   3.# 停止emqttd
     ./bin/emqttd stop
   		
```

- 如果emqtt 客户端没有连接到 emqttd服务端

``` shell

    1.在网页上dashboard的plugins的Authentication with Username/Password是否有开启
    2.Connection lost  cause[(null)] 考虑客户端的id是不是重复了
   
			
```

- 要使emqtt客户端连接用上用户名和密码

``` shell

    1. vim /yytd/emqttd/etc/plugins/emq_auth_username.conf
       auth.user.1.username = yytd
       auth.user.1.password = yytd
       
    2. 网页上dashboard的plugins的Authentication with Username/Password 部分开启
   
			
```

- 再加一个emqttd broker 服务器

``` shell

    1.将原先/yytd/emqttd 拷贝一份(要注意目录的拥有者)
    2.修改其/yytd/emqttd/etc/emq.conf 配置
        node.name = emqttd@127.0.0.1
        mqtt.listener.tcp = 1883
        mqtt.listener.ssl = 8883
        mqtt.listener.http = 8083
        mqtt.listener.https = 8084
     
    3.修改/yytd/emqttd/etc/plugins/emq_dashboard.conf 
        dashboard.listener.http = 18083
   		
```

## 参数配置

- Linux 操作系统参数

``` shell

    # 2M - 系统所有进程可打开的文件数量:

        sysctl -w fs.file-max=2097152
        sysctl -w fs.nr_open=2097152
        
    # 1M - 系统允许当前进程打开的文件数量:

        ulimit -n 1048576
   		
```

- TCP协议栈参数

``` shell

    # backlog - Socket 监听队列长度:
    sysctl -w net.core.somaxconn=65536
   		
```

- Erlang 虚拟机参数

``` shell
    ## Erlang Process Limit Erlang 虚拟机允许的最大进程数，EMQ 一个连接会消耗2个Erlang进程
    node.process_limit = 2097152

    ## Sets the maximum number of simultaneously existing ports for this system
    ## Erlang 虚拟机允许的最大 Port 数量，EMQ 一个连接消耗1个 Port
    ## Erlang 的 Port 非 TCP 端口，可以理解为文件句柄。
    node.max_ports = 1048576
   		
```

- EMQ 最大允许连接数

``` shell

    ## Size of acceptor pool 
    mqtt.listener.tcp.acceptors = 64
    
    ## Maximum number of concurrent clients
    mqtt.listener.tcp.max_clients = 10000
   		
```

- 客户端连接闲置时间
  
``` shell
  设置 MQTT 客户端最大允许闲置时间(Socket 连接建立,但未收到 CONNECT 报文):
  
  ## Client Idle Timeout (Second)
  mqtt.client.idle_timeout = 30

```

## 部署架构

``` shell

    基本部署结构:
        多个mqtt客户端->LB(负载均衡器)->多个mqtt服务器
        
  		
```