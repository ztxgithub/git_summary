
- FSU，BBDS中获取linux系统的版本

``` shell

	cat /proc/version
	
```

- 查看文件夹下的大小 du(disk usage)

``` shell

	du -h folder     :以人类可读的方式递归显示该folder及子目录的大小
	
	du -sh folder    :只显示folder文件夹包含的大小
	
```

- 查看linux系统是64还是32位

``` shell

	> file /bin/ls
	/bin/ls: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=eca98eeadafddff44caf37ae3d4b227132861218, stripped
	
```