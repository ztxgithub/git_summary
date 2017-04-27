
- FSU，BBDS中获取linux系统的版本

``` shell

	> cat /proc/version
	
```

- 查看文件夹下的大小 du(disk usage)

``` shell

	> du -h folder     :以人类可读的方式递归显示该folder及子目录的大小
	
	> du -sh folder    :只显示folder文件夹包含的大小
	
```

- 查看linux系统是64还是32位

``` shell

	> file /bin/ls
	/bin/ls: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=eca98eeadafddff44caf37ae3d4b227132861218, stripped
	
```

- linux 获取主机开机时的运行时间 uptime

``` shell

	> uptime
	15:20:50(开机时的开始时间) up 1 min(运行了多少时间:1分钟),  1 users,  load average: 1.61, 0.48, 0.16
	15:15:53 up  1:40(运行1个小时40分钟),  3 users,  load average: 3.32, 3.50, 3.43
	
	> cat /proc/uptime
	6135.47(系统开机时总共运行了多少秒) 4043.31(开机时系统空闲多少秒)
```