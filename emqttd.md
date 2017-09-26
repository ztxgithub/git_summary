# emqttd

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