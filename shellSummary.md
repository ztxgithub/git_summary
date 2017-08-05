# 平台相关FSU

- 将FSU恢复成出厂配置

``` shell

    1.$ rm -rf /media/sd/opt/*
    2.$ cp /media/sd/CASS/.OS.bin /media/sd/CASS/UPDATE
    3.$ mv /media/sd/CASS/UPDATE/.OS.bin /media/sd/CASS/UPDATE/OS.bin 
    4.$ reboot
	
```

- 查看平台的FSU有没有升级成功

``` shell
    
    失败：产生 /media/sd/CASS/.SD_update_fail　文件
	
```

- dbclient 远程登陆
``` shell
    
   跟ssh一样
   $ dbclient root@192.0.0.65
	
```


# BBDS,FSU 命令专用

- 在BBDS中显示内核的打印信息

``` shell

	$ echo 7 > /sys/module/Netlink_HAL/parameters/debug
	
	$ tail -f /var/log/messages 
	
```

- iostat 看linux系统的io操作(读写sd卡，磁盘等s)

``` shell

	$ iostat [ 选项 ] [ <时间间隔> [ <次数> ] ]

	$ iostat -k 1    (以每秒KB为单位显示,一秒刷新一次)
	
	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.99    0.00    1.98    0.00    0.00   97.03

	Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
	mmcblk0           6.93         0.00        27.72          0         28
	mmcblk0p1         6.93         0.00        27.72          0         28
	
	$ iostat --help
	
```

- lsof: 看某个程序打开那些文件

``` shell

	$ lsof
	
	PID          程序名						打开文件
	391     /opt/CASS/BIN/battery   /media/sd/CASS/battery_data.db
	
```

- 时刻监测一个命令的运行结果

``` shell

	$ watch[参数][命令]
	
	$ watch -n 1 ls -l  (1秒钟刷新 ls -l 命令的结果)
	
```


- FSU，BBDS中获取linux系统的版本

``` shell

	$ cat /proc/version
	
```

- 

- FSU中 udhcpc 进程

``` shell

	$ udhcpc -S -i eth1 -p /var/run/udhcpc.pid
	
	参数:
	    -S,--syslog             Log to syslog too
	    -i,--interface IFACE    Interface to use (default eth0)
	    -p,--pidfile FILE       Create pidfile
	
```

- 将后台的程序打印到显示器上 

``` shell

    console_redirect
	
```

- 查看文件夹下的大小 du(disk usage)

``` shell

	$ du -h folder     :以人类可读的方式递归显示该folder及子目录的大小
	
	$ du -sh folder    :只显示folder文件夹包含的大小
	
```

- 查看linux系统是64还是32位

``` shell

	$ file /bin/ls
	/bin/ls: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, 
	interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, 
	BuildID[sha1]=eca98eeadafddff44caf37ae3d4b227132861218, stripped
	
```

- linux 获取主机开机时的运行时间 uptime

``` shell

	$ uptime
	15:20:50(开机时的开始时间) up 1 min(运行了多少时间:1分钟),  1 users,  load average: 1.61, 0.48, 0.16
	15:15:53 up  1:40(运行1个小时40分钟),  3 users,  load average: 3.32, 3.50, 3.43
	
	$ cat /proc/uptime
	6135.47(系统开机时总共运行了多少秒) 4043.31(开机时系统空闲多少秒)
```

- 将某个文件设置内容为0

``` shell

	$ cat /dev/null > filename   ( >:代表全部替换， >> :代表追加 )
	
	或者
	
	$ > filename
```

- 通过 sz 命令传输有问题可以用 scp

``` shell

	$ scp CASSFSUService_Cloud_2017-04-27.log zhangtx@192.168.0.2:~/
	
```

- ulimit:获取和设置该用户权限下的限制(比如一个程序能够打开的最多的文件描述符的个数，一个栈最大空间等)

``` shell

    描述内容:
        ulimit 用于限制 shell 启动进程所占用的资源,作为临时限制，ulimit 可以作用于通过使用其命令登录的 shell 会话，
        在会话终止时便结束限制，并不影响于其他 shell 会话。而对于长期的固定限制，
        ulimit 命令语句又可以被添加到由登录 shell 读取的文件中，作用于特定的 shell 用户。
        ulimit 限制的是当前 shell 进程以及其派生的子进程。举例来说，如果用户同时运行了两个 shell 终端进程，
        只在其中一个环境中执行了 ulimit – s 100，则该 shell 进程里创建文件的大小收到相应的限制，
        而同时另一个 shell 终端包括其上运行的子程序都不会受其影响：

	$ ulimit -a (显示所有的信息)
	
	$ ulimit -s (显示The maximum stack size)
	
	$ ulimit -Hu (用户最大的可用进程数(一旦设置不能增加.))
	
```

# linux 命令专用

- 搜索某个命令的安装包

``` shell

	$ sudo yum search rz
	
	lrzsz.x86_64 : The lrz and lsz modem communications programs
	
```

- 安装rz,sz命令,使SercureCRT能够使用托文件

``` shell

	$ sudo yum install lrzsz.x86_64
	
```

- 内存占用率

``` shell

	$ ps -e -o 'rsz,pid,comm,args,pcpu,vsz,stime,user,uid' | sort -nr
	
	参数：
	    -e:等于-A,　Select all processes：选择所有的进程
	    -o:  -o format:指定输出的格式及内容
	    
	rsz:实际的物理内存
	
```

