
#使用 sshpass 进行脚本免交互密码

- 安装sshpass

```shell
         下载sshpass  下载地址https://sourceforge.net/projects/sshpass/files/sshpass/
         软件在 D:\云平台项目\Scloud\日志备份

         #tar -xvf sshpass文件名.tar.gz
         #cd sshpass文件夹名
         #./configure
         # make && make install
         查看安装成功与否：
         #which sshpass
         /usr/local/bin/sshpass   --安装成功。

```

- 编写脚本

```shell
    #!/bin/sh
    systime=`date +%Y-%m-%d`     #get current time from system
    
    objectaddress=root@120.55.195.98  #object host's name and ip address
    accesspassword=aliplat07@YYTD.Ltd       #access to remote host needed a password
    
    #sengine log
    localpath_sengine=/home/yytd/product_log/sengine_log/ #local path with conserve logs
    bakname_sengine="sengine_"$systime  #backuped folder's name
    remotepath_sengine=/yytd/logs/sengine/ #remote folder include logs
    
    sshpass -p $accesspassword scp -r -p $objectaddress:$remotepath_sengine $localpath_sengine 
    #copy logs from remote host to local folder
    cd $localpath_sengine
    mv sengine $bakname_sengine           #rename folder's name

```

- 用crontab命令进行定期执行脚本

```shell

    1.crontab -e
    
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
    50 10 * * * /home/yytd/product_log/backup_log.sh
    
   
```

## crontab 命令解析

### 基本概念

```shell

    主要用来定期的执行一些脚本和任务,crontab命令通常需要配置环境变量.
    PATH=/sbin:/bin:/usr/sbin:/usr/bin 用来指定系统执行命令的路劲
    
    一般有2中类型,系统任务调度(/etc)和用户任务调度(用户自定义的定时任务)
    在系统任务调度中,cat /etc/crontab
    在用户任务调度中,用户定义的crontab 文件都被保存在 /var/spool/cron目录中,其文件名与用户名一致.
    
    规定 某些用户禁止使用crontab, vim /etc/cron.deny
    
   
```



