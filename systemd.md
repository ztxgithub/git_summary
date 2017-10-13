## systemd 概述

    字母d是守护进程（daemon）的缩写。 Systemd 这个名字的含义,就是它要守护整个系统.
    Systemd 并不是一个命令，而是一组命令，涉及到系统管理的方方面面
    
## 系统管理
    
### systemctl 

    Service unit：系统服务 , 每一个Unit都有一个配置文件, 通常是在/lib/systemd/system/目录下
    例如(/lib/systemd/system/supervisord.service),如果配置文件里面设置了开机启动,
    systemctl enable命令相当于激活开机启动。
    
- 显示单个 Service Unit 的状态

```shell
    
    $ sysystemctl status supervisord.service 
    
    结果:
    ● supervisord.service - Process Monitoring and Control Daemon
       Loaded: loaded (/usr/lib/systemd/system/supervisord.service; enabled; vendor preset: disabled)
       Active: active (running) since 五 2017-08-25 16:05:44 CST; 1h 24min ago
      Process: 29335 ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf (code=exited, status=0/SUCCESS)
     Main PID: 29338 (supervisord)
       CGroup: /system.slice/supervisord.service
               ├─29338 /usr/bin/python2 /usr/bin/supervisord -c /etc/supervisord.conf
               ├─29339 /yytd/jdk1.8/bin/java -Djava.util.logging.config.file=/yytd/server/tomcat1/conf/log...
               ├─29340 nginx: master process /yytd/server/nginx/sbin/nginx
               ├─29346 nginx: worker process
               ├─29347 nginx: worker process
               ├─29348 nginx: worker process
               ├─29349 nginx: worker process
               ├─29428 /yytd/emqttd/erts-7.3/bin/epmd -daemon
               ├─29434 /yytd/emqttd/erts-7.3/bin/beam.smp -W w -e 256000 -Q 1048576 -P 2097152 -A 32 -K tr...
               ├─29570 sh -s disksup
               ├─29572 /yytd/emqttd/lib/os_mon-2.4/priv/bin/memsup
               └─29575 /yytd/emqttd/lib/os_mon-2.4/priv/bin/cpu_sup
               
     Loaded行：配置文件的位置，是否设为开机启动(第二个";"前面)
     Active行：表示正在运行
     Main PID行：主进程ID
     CGroup块：应用的所有子进程
 
```

- 对一个服务操作

```bash
    
    $ sudo systemctl start/stop/restart/kill/reload supervisord.service
    
    kill: 杀死一个服务的所有子进程,当 systemctl stop supervisord.service 没有用时,没有响应,服务停不下来,就
           用 systemctl kill supervisord.service
    
    reload : 重新加载一个服务的配置文件

```

- 对一个服务开机自启

```shell
    
    $ sudo systemctl enable supervisord.service
    
    该命令是将sshd.service的一个符号链接，放在/etc/systemd/system目录下面的multi-user.target.wants子目录之中,
    而 $ systemctl get-default, shell命令的结果是 multi-user.target(一组服务),在这个组里的所有服务，都将开机启动.
    设置开机启动以后,软件并不会立即启动，必须等到下一次开机。如果想现在就运行该软件，那么要执行systemctl start 命令

```

- 停止开机自启服务

```shell
    
    $ sudo systemctl disable supervisord.service

```

- 重载所有修改过的配置文件(针对所有的服务的配置文件)

```bash
    
   $ sudo systemctl daemon-reload
   
    之后的操作一般是 $ systemctl restart supervisord.service

```

- 配置文件的状态

```shell
    
   $ systemctl list-unit-files | grep supervisord.service

    结果显示:
        
        supervisord.service   enabled
         
    状态:
        enabled：已建立启动链接
        disabled：没建立启动链接
        static：该配置文件没有[Install]部分（无法执行），只能作为其他配置文件的依赖
        masked：该配置文件被禁止建立启动链接
```

- 配置文件的格式(supervisord.service)

```shell
    
    查看配置文件的内容
    
    $ systemctl cat supervisord.service  (不需要考虑当前处于什么pwd下)
    
    配置文件分成几个区块,每个区块的第一行，是用方括号表示的区别名，比如[Unit].配置文件的区块名和字段名,都是大小写敏感的.
    每个区块内部是一些等号连接的键值对.
    
    [Service]
    Type=forking (等号两边没有空格)
    
    区块类型:
    
    [Unit]区块通常是配置文件的第一个区块，用来定义 Unit 的元数据，以及配置与其他 Unit 的关系。它的主要字段如下:
    
            Description：简短描述
            Documentation：文档位置
            Requires：当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败(强依赖关系)
            Wants：与当前 Unit 配合的其他 Unit，如果它们没有运行，当前 Unit 不会启动失败(弱依赖关系)
            BindsTo：与Requires类似，它指定的 Unit 如果退出，会导致当前 Unit 停止运行
            Before：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之后启动(启动顺序,一定是按这个顺序启动的,至于成不成功不管)
            After：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之前启动(启动顺序)
            Conflicts：如果指定的 Unit 正在运行,那么当前 Unit 不能运行
            Condition...：当前 Unit 运行必须满足的条件，否则不会运行
                例如: ConditionFileIsExecutable=/etc/rc.local
            Assert...：当前 Unit 运行必须满足的条件，否则会报启动失败
            
            
     例如:
            sshd.service 的内容:
            [Unit]
            Description=OpenSSH server daemon
            Documentation=man:sshd(8) man:sshd_config(5)
            After=network.target sshd-keygen.service
            Wants=sshd-keygen.servic
                   
            Wants字段：表示sshd.service与sshd-keygen.service之间存在"弱依赖"关系,
                      即如果"sshd-keygen.service"启动失败或停止运行,不影响sshd.service继续执行。
                      
            Requires字段则表示"强依赖"关系，即如果该服务启动失败或异常退出，那么sshd.service也必须退出.
            Wants字段与Requires字段只涉及依赖关系，与启动顺序无关，默认情况下是同时启动的.
            
    
    [Service]区块用来 Service 的配置，只有 Service 类型的 Unit 才有这个区块。它的主要字段如下:
    
            Type：定义启动时的进程行为。它有以下几种值。
                Type=simple：默认值，执行ExecStart指定的命令，启动主进程
                Type=forking：以 fork 方式从父进程创建子进程，创建后父进程会立即退出
                Type=oneshot：一次性进程,Systemd 会等它执行完，才启动其他服务
                Type=dbus：类似于simple,但会等待 D-Bus 信号后启动
                Type=notify：当前服务启动完毕，会通知Systemd，再继续往下执行
                Type=idle：等其他任务执行完毕，当前服务才会运行,一种使用场合是为让该服务的输出，不与其他服务的输出相混合.
                
            ExecStart：启动当前服务的命令
            ExecStartPre：启动当前服务之前执行的命令
            ExecStartPost：启动当前服务之后执行的命令
            ExecReload：重启当前服务时执行的命令
            ExecStop：停止当前服务时执行的命令
            ExecStopPost：停止当其服务之后执行的命令
            RestartSec：表示 Systemd 重启服务之前，需要等待的秒数
            
            Restart：定义何种情况 Systemd 会自动重启当前服务,即当服务退出后,以何种方式重启服务.
                        no（默认值）：退出后不会重启
                        on-success：只有正常退出时（退出状态码为0），才会重启
                        on-failure：非正常退出时（退出状态码非0），包括被信号终止和超时，才会重启
                        on-abnormal：只有被信号终止和超时，才会重启
                        on-abort：只有在收到没有捕捉到的信号终止时，才会重启
                        on-watchdog：超时退出，才会重启
                        always：不管是什么退出原因，总是重启
                        
            TimeoutSec：定义 Systemd 停止当前服务之前等待的秒数
                            单位是秒，默认是0不限制
                            
            Environment：指定环境变量
            RemainAfterExit: yes/no (这个是针对Type = oneshot (一次性进程:执行完后,服务就结束了), 
                            当如果RemainAfterExit设置为yes,则表示进程退出以后，服务仍然保持执行
            KillMode : 定义 Systemd 如何停止 sshd 服务
                        control-group（默认值）：当前控制组里面的所有子进程，都会被杀掉
                        process：只杀主进程
                        mixed：主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号
                        
             LimitCORE=infinity  开启core dump
             LimitNOFILE=65536   设置文件描述符的数量为65536     
                     
                         
            关键字的对应关系:             
                Directive        ulimit equivalent     Unit
                LimitCPU=        ulimit -t             Seconds      
                LimitFSIZE=      ulimit -f             Bytes
                LimitDATA=       ulimit -d             Bytes
                LimitSTACK=      ulimit -s             Bytes
                LimitCORE=       ulimit -c             Bytes
                LimitRSS=        ulimit -m             Bytes
                LimitNOFILE=     ulimit -n             Number of File Descriptors 
                LimitAS=         ulimit -v             Bytes
                LimitNPROC=      ulimit -u             Number of Processes 
                LimitMEMLOCK=    ulimit -l             Bytes
                LimitLOCKS=      ulimit -x             Number of Locks 
                LimitSIGPENDING= ulimit -i             Number of Queued Signals 
                LimitMSGQUEUE=   ulimit -q             Bytes
                LimitNICE=       ulimit -e             Nice Level 
                LimitRTPRIO=     ulimit -r             Realtime Priority 
            
     例如:
           1. sshd.service:
            
            [Service]
            EnvironmentFile=/etc/sysconfig/sshd
            ExecStart=/usr/sbin/sshd -D $OPTIONS
            ExecReload=/bin/kill -HUP $MAINPID
            Type=simple
            KillMode=process
            Restart=on-failure
            RestartSec=42s
            
            EnvironmentFile字段：指定当前服务的环境参数文件(/etc/sysconfig/sshd).
                                该文件(/etc/sysconfig/sshd)内部的key=value键值对,
                                在当前配置文件(sshd.service)中可以用$key的形式来获取
                                
            ExecStart: 定义启动进程时执行的命令, 其中 $OPTIONS 来自于EnvironmentFile(/etc/sysconfig/sshd)
                        环境文件中的值
            将KillMode 设为process,表示只停止主进程，不停止任何sshd 子进程，即子进程打开的 SSH session 仍然保持连接。
            这个设置不太常见,但对 sshd 很重要,否则你停止服务的时候,会连自己打开的 SSH session 一起杀掉(那就会失去连接控制)
            
            Restart设为on-failure,表示任何意外的失败,就将重启sshd.如果 sshd 正常停止（比如执行systemctl stop命令）,它就不会重启。
                        
            
            2. [Service]
                 ExecStart=/bin/echo execstart1
                 ExecStart=
                 ExecStart=/bin/echo execstart2
                 ExecStartPost=/bin/echo post1
                 ExecStartPost=/bin/echo post2
                 
                上面这个配置文件，第二行ExecStart设为空值，等于取消了第一行的设置，运行结果如下。
                
                execstart2
                post1
                post2
                            
             3.
                所有的启动设置之前,都可以加上一个连词号（-），表示"抑制错误"，即发生错误的时候，不影响其他命令的执行。
                比如，EnvironmentFile=-/etc/sysconfig/sshd（注意等号后面的那个连词号），
                就表示即使/etc/sysconfig/sshd文件不存在，也不会抛出错误。
            
            
     [Install] 通常是配置文件的最后一个区块,用来定义如何启动，以及是否开机启动。它的主要字段如下:
     
        WantedBy：它的值是一个或多个 Target，当前 Unit 激活时（enable）符号链接会放入/etc/systemd/system目录下面以
                  Target 名 + .wants后缀构成的子目录中
        RequiredBy：它的值是一个或多个 Target，当前 Unit 激活时，
                    符号链接会放入/etc/systemd/system目录下面以 Target 名 + .required后缀构成的子目录中
        Alias：当前 Unit 可用于启动的别名
        Also：当前 Unit 激活（enable）时，会被同时激活的其他 Unit
    
  
```

- 修改一个服务的资源限制(cat /proc/pid/limit)

```shell
    
    $  systemctl cat supervisord.service (得到 supervisord.service的绝对路劲)
    
    $ vim  /usr/lib/systemd/system/supervisord.service  (修改该文件具体的内容)
    
    $ systemctl daemon-reload (重新加载所有服务(service)配置文件)
    
    $ systemctl restart supervisord.service 

```
### 日志管理

- 查看某个 Unit 的日志

```shell
    
    $ sudo journalctl -u nginx.service
    $ sudo journalctl -u nginx.service --since today
    $ sudo journalctl -u nginx.service -f (实时滚动显示某个 Unit 的最新日志) 

```
[参考资料](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)



