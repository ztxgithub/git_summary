
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

## 交叉编译 tcmalloc.so

```shell
    在　gperftools-2.7　中(https://github.com/gperftools/gperftools/releases)
    1. ./configure --host arm-none-linux-gnueabi CXX=arm-none-linux-gnueabi-g++ CC=arm-none-linux-gnueabi-gcc
    2. make V=1 CXX=arm-none-linux-gnueabi-g++ CC=arm-none-linux-gnueabi-gcc 
    3. 在　gperftools-2.7/.libs 进行查看
    
    4. 重新编译 make clean
    
    动态库有没有使用成功可使用这个命令：lsof -n | grep tcmalloc
    
    (1) 注意：这里为了进行内存泄漏分析，一定要将 libtcmalloc.so 放在最后
            CXXLFLAGS =  -g -Wall -L../bin -lBaseCode -lpthread  -lprotobuf -rdynamic -ltcmalloc(链接放在最后)
            
    (2) 进行 gdb 调试查找内存问题
            > env TCMALLOC_PAGE_FENCE=1 gdb CASSFSUService
            
    (3) 内存泄露
        env HEAPCHECK=normal ./server
```

## 编译安装 gcc-7

```shell
	1. 下载 gcc-7.1.0 的源码包
	2. > tar zxvf gcc-7.1.0.tar.gz
	3. 建立构建文件夹
	        > mkdir gcc-7.1.0-build
	4. 下载　gmp-6.1.0.tar.gz，mpfr-3.1.4.tar.gz，mpc-1.0.3.tar.gz 到 gcc-7.1.0目录下
	     > cd gcc-7.1.0
         > tar zxvf gmp-6.1.0.tar.gz
         > tar zxvf mpfr-3.1.4.tar.gz
         > tar zxvf mpc-1.0.3.tar.gz
         > ln -s  gmp-6.1.0 gmp
         > ln -s  mpfr-3.1.4 mpfr
         > ln -s  mpc-1.0.3 mpc
    5. > cd gcc-7.1.0-build
       > sudo ../gcc-7.1.0/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
       > sudo make -j2
       > sudo make install (默认位置 /usr/local,头文件在/usr/local/include目录，可执行文件在/usr/local/bin目录，
                                   库文件在/usr/local/lib目录)
    6. 使用update-alternatives命令配置增加最新版本编译器
       update-alternatives --install <链接> <名称> <路径> <优先级>
    
        > sudo update-alternatives \
            --install /usr/bin/gcc gcc /usr/local/bin/gcc 70 \
            --slave /usr/bin/gcc-ar gcc-ar /usr/local/bin/gcc-ar \
            --slave /usr/bin/gcc-nm gcc-nm /usr/local/bin/gcc-nm \
            --slave /usr/bin/gcc-ranlib gcc-ranlib /usr/local/bin/gcc-ranlib
            
        > sudo update-alternatives --install /usr/bin/g++ g++ /usr/local/bin/g++ 70
        
        (1) # 查询本机已有GCC编译器情况
            > sudo update-alternatives --query gcc
            # 查询本机已有G++编译器情况
            > sudo update-alternatives --query g++
            
        (2) 选择默认使用的GCC编译器版本：
            
            # 交互配置GCC编译器
            > sudo update-alternatives --config gcc
            # 交互配置G++编译器
            > sudo update-alternatives --config g++
            
    7. 使用G++7.3.0构建多线程程序，运行程序时出现类似“./main: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: 
    　　version `GLIBCXX_3.4.22’ not found (required by ./main)”
    
        解决方法：　在 /usr/lib/x86_64-linux-gnu 中将
        > cd /usr/lib/x86_64-linux-gnu
        > sudo rm -f libstdc++.so.6
        > sudo ln -s /usr/local/lib64/libstdc++.so.6
        
    参考资料: https://blog.csdn.net/davidhopper/article/details/79681695

```




