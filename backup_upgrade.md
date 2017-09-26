
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
    在用户任务调度中,用户定义的crontab 文件都被保存在 /var/spool/cron目录中,其文件名与用户名一致.当以某个用户名登陆到linux
    系统时,通过 $ crontab -e 进行任务的编辑,内容就会保存在/var/spool/cron目录下.
    
    规定 某些用户禁止使用crontab, vim /etc/cron.deny ,每一行一个用户名
    
```

### 内容含义

```shell

    用户所建立的crontab文件中,每一行都代表一项任务,每一行的各个字段都以空格隔开
    minute   hour   day   month   week   command
    其中：
        minute： 表示分钟，可以是从0到59之间的任何整数。
        hour：表示小时，可以是从0到23之间的任何整数。
        day：表示日期，可以是从1到31之间的任何整数。
        month：表示月份，可以是从1到12之间的任何整数。
        week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。
        command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。
        
    在以上各个字段中，还可以使用以下特殊字符：
        星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
        逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
        中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
        正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。
                   同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次.
    
```

### 命令详解

```shell

    各个参数:
    
        1.可以从文件中读取相关的任务命令
            $ crontab filename(文件中内容是  minute   hour   day   month   week   command)
        2.-e
            编辑某个用户的crontab文件内容.如果不指定用户,则表示编辑当前用户的crontab文件
        3.-l
            显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容
        4.-r
            从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件.
            
    注意:
        最好编写包含一组crontab命令的文件,再用 crontab filename来加载,这样可以保存历史crontab.
    
```

### 示例

```shell

    1.每隔2分钟执行一次命令
        */2 * * * * /home/yytd/test/bash_ls
        
    2.每小时的第3和第15分钟执行
        3,15 * * * * /home/yytd/test/bash_ls
        
    3.在上午8点到11点的第3和第15分钟执行
        3,15 8-11 * * * /home/yytd/test/bash_ls
        
    4.每隔两天的上午8点到11点的第3和第15分钟执行
        3,15 8-11 */2 * * /home/yytd/test/bash_ls
        
    5.每个星期一的上午8点到11点的第3和第15分钟执行
        3,15 8-11 * * 1 /home/yytd/test/bash_ls
      
    6. 每月1,10,22日的4:45 执行该脚本
        45 4 1,10,22 * * /home/yytd/test/bash_ls
    
```

### 注意事项

```shell

    1.环境变量问题
        手动执行脚本没有问题,但用crontab创建一个任务却执行不了,手动执行某个脚本任务时,是在当前shell环境下进行的,
        此时程序能找到环境变量.但是用crontab创建一个任务是不会加载任何的环境变量,
        因此,就需要在crontab文件中指定任务运行所需的所有环境变量.
        
        对于要执行的任务(脚本文件中和crontab文件)
            (1):文件路径要写全局路径
            (2):要用到环境变量,用source命令
            例如:
                crontab 文件内容:
                source /etc/profile
                0 * * * * . /etc/profile;/bin/sh /var/www/java/audit_no_count/bin/restart_audit.sh
                
    2.注意清理系统用户的邮件日志
        每条任务调度执行完毕,系统都会将任务输出信息通过电子邮件的形式发送给当前系统用户,长期下去会导致系统出问题.
         crontab 文件内容:
            0 */3 * * * /usr/local/apache2/apachectl restart >/dev/null 2>&1
            
    3.系统级任务调度与用户级任务调度
        可以将用户级任务调度放到系统级任务调度来完成（不建议这么做）,但是系统级的任务调度不能放在用户级调度上,
         root用户的任务调度操作可以通过“crontab -u root -e”来设置,也可以将调度任务直接写入/etc/crontab文件,
         如果要定义一个定时重启系统的任务,就必须将任务放到/etc/crontab文件,
         即使在root用户下创建一个定时重启系统的任务也是无效的.
         
    4.新创建的cron job,不会马上执行,至少要过2分钟才执行.如果重启cron(systemctl restart crond.service)则马上执行.
      在crontab 文件中%是有特殊含义的,表示换行的意思。如果要用的话必须进行转义\%，
      如经常用的date ‘+%Y%m%d’在crontab里是不会执行的,应该换成date ‘+\%Y\%m\%d’.最好还是写一个脚本文件
      例如 45 4 1,10,22 * * /home/yytd/test/bash_ls
    
```