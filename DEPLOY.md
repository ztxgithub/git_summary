各个系统环境部署
==================================================

--------------------------------------
> ## 子系统部署

```bash

	SEngine依赖子系统数据库mongoDB,broker服务器Emqtt.所有子系统及SEngine由进程管理工具Supervisor统一管理

```

> ### 操作系统环境

```bash
	CentOs 7

```

> ### 新建系统用户 yytd 统一管理SEngine,Emqtt.

```bash

	useradd yytd -m 

```

- 关掉防火墙

```
	systemctl stop firewalld (关闭防火墙,跟连接不了mongodb有关)
	systemctl disable firewalld （禁用防火墙,开机不启动）
```

- 关闭 SELINUX 服务(能正常执行mongodb服务和 systemctl restart 等操作)

> vi  /etc/selinux/config

```bash

	SELINUX=disabled   （需重启系统）
	
```

> ## mongoDB
> ### mongoDB版本号

```bash

	mongoDB-3.2
   
```
        
> ### 安装步骤

> #### 步骤一：安装mongodb

> ##### 方法一：在线安装mongodb

> ###### 创建yum源

```bash

	cd /etc/yum.repos.d/
	vi mongodb-org-3.2.repo

```

- mongodb-org-3.2.repo内容

```bash

	[mongodb-org-3.2]
	name=MongoDB Repository
	baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
	gpgcheck=0
	enabled=1

```

> ###### yum 安装mongoDB

```bash

	sudo yum install -y mongodb-org

```

> ##### 方法二：离线安装mongodb

> ###### 创建一个目录

```bash

	sudo mkdir /tmp/mongodb

```

> ###### 将安装包拷贝到/tmp/mongodb目录下,再用以下命令安装

```bash

	sudo rpm -ivh mongodb-org-3.2.11-1.el7.x86_64.rpm mongodb-org-shell-3.2.11-1.el7.x86_64.rpm mongodb-org-mongos-3.2.11-1.el7.x86_64.rpm mongodb-org-tools-3.2.11-1.el7.x86_64.rpm mongodb-org-server-3.2.11-1.el7.x86_64.rpm 

```
> #### 步骤二：设置配置信息

- MongoDB服务开机自启

```bash

	sudo chkconfig mongod on
	
```

- 修改MongoDB的配置文件/etc/mongod.conf

```

	# network interfaces
	net:
	port: 27017
	#  bindIp: 127.0.0.1  # Listen to local interface only, comment to listen on all interfaces.

```

参考:https://docs.mongodb.com/master/tutorial/install-mongodb-on-red-hat/


> ## Emqttd

- [Emqttd安装包下载](http://emqtt.com/downloads)

- 下载完成后解压至/home/yytd/

```

	cd /home/yytd
	unzip emqttd-centos7-v2.0.7.zip

```

- 修改/home/yytd/emqttd/etc/emq.conf 配置文件

```
	
	log.console.level = info
	mqtt.max_packet_size = 64MB             （解决emqttd日志中出现 Framing error - invalid_mqtt_frame_len（数据包的大小太小了））
	mqtt.session.max_inflight = 0
	mqtt.pubsub.pool_size = 64
	mqtt.listener.tcp.acceptors = 64
	mqtt.listener.tcp.max_clients = 10000
	mqtt.listener.ssl.max_clients = 1024
```

> ## SEngine 

> ### 创建sengine运行环境

> #### 创建sengine运行目录

```bash

	mkdir -p /home/yytd/sengine/
	
```

- 将可执行文件拷贝到 /home/yytd/sengine/目录下

```bash

	chmod +x /home/yytd/sengine/sengine
	
```

> #### 创建sengine运行时配置文件

```bash

    vi /home/yytd/sengine/sengine.config
	
	<?xml version="1.0" encoding="utf-8" ?>
    <Config>
	<BROKERIP>127.0.0.1:1883</BROKERIP>
	<SENGINEID>Sengine_test</SENGINEID>
	<SCLOUDID>Scloud_test</SCLOUDID>
	<DBURI>mongodb://test:YYtd_test@192.168.0.4:27017/scloud-test</DBURI>
	<DATABASENAME>scloud-test</DATABASENAME>
	<POLLING_UNIT_INTERVAL>60</POLLING_UNIT_INTERVAL>
	<LOGOUT_OVERTIME_THRESHOLD>180</LOGOUT_OVERTIME_THRESHOLD>
	<MQTT_KEEP_ALIVE_INTERVAL>60</MQTT_KEEP_ALIVE_INTERVAL>
	<MIN_LOG_LEVEL>ELOG_LVL_VERBOSE</MIN_LOG_LEVEL>
    </Config>

```
> #### 准备sengine运行时需要的动态链接库

- 动态库列表

```bash

        libpaho-mqtt3a.so
        libboost_serialization.so
        libbsoncxx.so
        libmongocxx.so
        libgtest.a

```

- 将上述动态链接库的压缩包SEngine_BBDS_V1.0_lib.tar.gz拷到 /usr/local/lib/ 目录下,并解压


```bash

    tar -xvf SEngine_BBDS_V1.0_lib.tar.gz

```

- 设置好相关动态链接库

```bash
    1.修改 /etc/ld.so.conf文件
    新增内容： /usr/local/lib/
	
    2.输入linux命令行: ldconfig
	
```

> ## Supervisor

> ### 下载并安装supervisor

- [Supervisor安装包下载](https://pypi.python.org/pypi/supervisor)

- 安装supervisor,将安装包拷贝到/tmp/

```bash

	tar -xvf supervisor-3.3.1.tar.gz
	cd supervisor-3.3.1/
	sudo python setup.py install   
	
```

> ### 配置supervisor为系统服务

> ### 生成supervisord.conf

```bash

	echo_supervisord_conf > /etc/supervisord.conf
	
	可以考虑一下 supervisord.conf里 
	 [supervisord]section:
          minfds=500000 (设置supervisord管理的各个进程的打开文件描述符数)

```

> ### 修改supervisord.conf

```bash

    [inet_http_server]         ; inet (TCP) server disabled by default
    port=0.0.0.0:9001        ; (ip_address:port specifier, *:port for all iface)
    username=yytd              ; (default is no username (open server))
    password=YYtd#1234               ; (default is no password (open server))

	[include]
	files = /etc/supervisor/*.conf

```

> ### supervisord 注意事项

```bash

    supervisor还要求管理的程序是非daemon程序,supervisord会帮你把它转成daemon程序

```

> ### supeivisor配置各子系统

> #### supeivisor配置emqttd

```bash
	mkdir -p /home/yytd/logs/emqttd/
	
	mkdir /etc/supervisor
	cd /etc/supervisor
	vi yytd.conf

```
  
```bash
	yytd.conf内容(所有内容开头不能有空格):
	 
	[program:emqttd]
	directory = /home/yytd/emqttd/bin/
	command = /home/yytd/emqttd/bin/emqttd console
	priority = 9
	autostart = true
	startsecs = 3
	autorestart = true
	startretries = 3
	user = yytd 
	redirect_stderr = true
	stdout_logfile_maxbytes = 100MB
	stdout_logfile_backups = 20
	stdout_logfile = /home/yytd/logs/emqttd/emqttd_stdout.log
	environment = HOME=/home/yytd
	

```

> #### supeivisor配置sengine

```bash

	mkdir -p /home/yytd/logs/sengine/
	
	mkdir /etc/supervisor
	cd /etc/supervisor
	vi yytd.conf

```
  
```bash

	yytd.conf内容(所有内容开头不能有空格):
	 
	[program:sengine]
	directory = /home/yytd/sengine/ ; 程序的启动目录
	command = /home/yytd/sengine/sengine  ; 启动命令，可以看出与手动在命令行启动的命令是一样的
	priority = 8
	autostart = true     ; 在 supervisord 启动的时候也自动启动
	startsecs = 3        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
	autorestart = true   ; 程序异常退出后自动重启
    ;autorestart = false   ; 程序异常退出后自动重启
	startretries = 3     ; 启动失败自动重试次数，默认是 3
	user = yytd          ; 用哪个用户启动
	redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
	stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
	stdout_logfile_backups = 20     ; stdout 日志文件备份数
    ; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
	stdout_logfile = /home/yytd/logs/sengine/sc_stdout.log
	

```


> #### 设置supervisord开机自启

- sudo vim /lib/systemd/system/supervisord.service

```bash

[Unit]
Description=Process Monitoring and Control Daemon
After=rc-local.service

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
SysVStartPriority=99

LimitCORE=infinity
LimitNOFILE= 65535
LimitNPROC= 65535

[Install]
WantedBy=multi-user.target
	
```

(在Windows上编辑这内容有出现^M回车符，linux脚本识别不了该命令，可以用sed -i -e 's/\r//g' /etc/init.d/supervisord 命令去掉^M回车符)

- 开机重启前改变文件所有者

```bash

    chown yytd:yytd /home/yytd
    chown -R yytd:yytd /home/yytd/*

```

- 执行以下命令

```bash

  systemctl enable supervisord.service
  systemctl start supervisord.service

```

> #### 部署完成

# 扩展命令

> #### supervisor开启子系统

```bash
	
	supervisorctl start/stop/restart sengine
	supervisorctl start/stop/restart emqttd

```


> ## 数据库索引添加
将数据库中企业相关的集合加入索引

- 企业识别码_fsu　集合  
    索引１　　{code:1}
    
- 企业识别码_device　集合  
    索引１　　{code:1}

- 企业识别码_signal　集合  
    索引１　　{code:1}

- 企业识别码_history　集合  
    联合索引１　　{signalId:1,max:1}
    
- 企业识别码_alarm　集合  
    联合索引１　　{serialNo:1,signal.strId:1}
    
- 企业识别码_event_info　集合  
    联合索引１　　{eventId:1,deviceCode:1}