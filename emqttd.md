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
   
			
```

- 要使emqtt客户端连接用上用户名和密码

``` shell

    1. vim /yytd/emqttd/etc/plugins/emq_auth_username.conf
       auth.user.1.username = yytd
       auth.user.1.password = yytd
       
    2. 网页上dashboard的plugins的Authentication with Username/Password 部分开启
   
			
```