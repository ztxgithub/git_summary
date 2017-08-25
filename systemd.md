## systemd 概述

    字母d是守护进程（daemon）的缩写。 Systemd 这个名字的含义,就是它要守护整个系统.
    Systemd 并不是一个命令，而是一组命令，涉及到系统管理的方方面面
    
## 系统管理
    
### systemctl 

    Service unit：系统服务 , 每一个Unit都有一个配置文件, 通常是在/lib/systemd/system/目录下
    例如(/lib/systemd/system/supervisord.service),如果配置文件里面设置了开机启动,
    systemctl enable命令相当于激活开机启动。
    
- 显示单个 Service Unit 的状态

```bash
    
    $ sysystemctl status supervisord.service 

```

- 对一个服务操作

```bash
    
    $ sudo systemctl start/stop/restart/kill/reload supervisord.service
    
    kill: 杀死一个服务的所有子进程
    
    reload : 重新加载一个服务的配置文件

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
            Documentation：文档地址
            Requires：当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败
            Wants：与当前 Unit 配合的其他 Unit，如果它们没有运行，当前 Unit 不会启动失败
            BindsTo：与Requires类似，它指定的 Unit 如果退出，会导致当前 Unit 停止运行
            Before：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之后启动
            After：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之前启动
            Conflicts：这里指定的 Unit 不能与当前 Unit 同时运行
            Condition...：当前 Unit 运行必须满足的条件，否则不会运行
            Assert...：当前 Unit 运行必须满足的条件，否则会报启动失败
            
            
    [Service]区块用来 Service 的配置，只有 Service 类型的 Unit 才有这个区块。它的主要字段如下:
    
  
```



