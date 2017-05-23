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

- 查看文件夹下的大小 du(disk usage)

``` shell

	$ du -h folder     :以人类可读的方式递归显示该folder及子目录的大小
	
	$ du -sh folder    :只显示folder文件夹包含的大小
	
```

- 查看linux系统是64还是32位

``` shell

	$ file /bin/ls
	/bin/ls: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=eca98eeadafddff44caf37ae3d4b227132861218, stripped
	
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

	$ ulimit -a (显示所有的信息)
	
	$ ulimit -s (显示The maximum stack size)
	
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

- 安装rz,sz命令,使SercureCRT能够使用托文件

``` shell

	$ sudo yum install lrzsz.x86_64
	
```