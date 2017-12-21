
## undefined reference问题总结

- 链接时缺失了相关目标文件（.o）

- 链接时缺少相关的库文件（.a/.so）

- 多个库文件链接顺序问题

``` shell
    
    依赖其他库的库一定要放到被依赖库的前面，这样才能真正避免undefined reference的错误，完成编译链接

```

- 在c++代码中链接c语言的库

``` shell
    
    解决方法：
    方法一:
        即在main.cpp中，把与c语言库test.a相关的头文件包含添加一个extern "C"的声明即可
        
        在 main.cpp 中
        
        extern "C" 
        {
            #include"test.h"
        }
        
     方法二:
        在test.h,test.c中,test.c中的函数实现是用C语言编写的,
        那么在test.h
        
            #ifdef __cplusplus
            extern "C" {
            #endif
            
            /** start a socket server (socket, bind and listen)
             *  parameters:
             *          bind_ipaddr: the ip address to bind
             *          port: the port to bind
             *          err_no: store the error no
             *  return: >= 0 server socket, < 0 fail
            */
            int socketServer(const char *bind_ipaddr, const int port, int *err_no);
            
            函数的申明
            .....
            
            #ifdef __cplusplus
            }
            #endif
            
        这时在main.cpp就可以直接 #include"test.h"了

```
[参考资料](http://ticktick.blog.51cto.com/823160/431329)

- 设置好相关动态链接库

``` shell
    
    1.修改 /etc/ld.so.conf文件
    新增内容： /usr/local/lib/
	
    2.输入linux命令行: ldconfig

```


- 交叉编译依赖库 libpcap.so.1.8.1 

``` shell
    
    源代码libpcap-1.8.1 路径: /home/jame/soft/libpcap-1.8.1
    1.确保安装 flex和bison (sudo apt-get install bison，sudo apt-get install flex)
    2. $ ./configure --host=arm-linux --with-pcap=linux
    3. $ make  (这个时候就生成 libpcap.so.1.8.1)
    4.cp ~/soft/libpcap-1.8.1/libpcap.so.1.8.1 /home/jame/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/lib
    5.ln -s libpcap.so.1.8.1 libpcap.so 
   
```

- Makefile

``` shell
    
    更改编译工具
    make CC=arm-linux-gcc 
    
    引入外部的.h 
    
    make CFLAGS+= -I/home/jame/soft/libpcap-1.8.1/include (路径名)
   
```

## 交叉编译 xl2tpd

``` shell
    
    在 xl2tpd-1.3.10 目录下
    1.在Makefile中这行 all: $(EXEC) pfc $(CONTROL_EXEC) 删除 "pfc"
    2.make CC=arm-linux-gcc
    
```

## 交叉编译 动态链接库 libpaho-mqtt3a.so

``` shell
    
    在 mqtt_client 目录下
    1.make CC=arm-linux-gcc
    2.在./mqtt_client/build/output 目录下找到对应的动态链接库
    
```




