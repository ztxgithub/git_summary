
#使用 sshpass 进行脚本免交互密码

- 安装sshpass

```shell
         下载sshpass  下载地址https://sourceforge.net/projects/sshpass/files/sshpass/
         软件在/

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




