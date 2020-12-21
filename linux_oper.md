# linux 操作

## 环境操作
```shell
1. 动态链接库的编译
(1) 如果有 autogen.sh 文件(首先先 apt-get install autoconf automake libtool)
> ./autogen.sh
(2) ./configure (指定某个路径 ./configure --prefix=$HOME/public )
(3) make
(4) make install
  
2. 编译第三方库
  (1). 安装 cmake 
      (1) $ tar xvf cmake-3.14.0.tar.gz
      (2) $ cd cmake-3.14.0/
      (3) $ ./configure
      (4) $ make
      (5) $ sudo make install
      https://blog.csdn.net/fxnawm/article/details/78489586

  (2). 安装 apr
      (1) > cd apr-1.4.6
      (2) > ./buildconf
      (3) > ./configure --prefix=/usr/local/apr/
      (4) > make
      (5) > sudo make install
      (6) > cd apr-util-1.6.1/
      (7) >  ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
      (8) > make
      (9) > sudo make
      
  (2.1) 安装 ActiveMQ (https://blog.csdn.net/wei_xiaox126/article/details/41874853)
          > ./configure --prefix=/usr/local/ActiveMQ-cpp --with-apr=/usr/local/apr/
            --with-apr-util=/usr/local/aprutil --with-cppunit=/usr/local/cppunit

          > make && make install

  (3).  编译 evpp (https://www.cnblogs.com/lsgxeva/p/10718979.html)
        需要 evpp-master.zip 和 concurrentqueue.zip
        在 evpp-master/3rdparty/concurrentqueue 替换为下载下来的源
         $ mkdir -p build && cd build
         $ cmake -DCMAKE_BUILD_TYPE=Debug  -DCMAKE_INSTALL_PREFIX=~/install ..
         $ make  (动态链接库在 lib 目录下)
         
  (4) 安装 boost
        > cd boost_1_59_0 
        > ./bootstrap.sh --prefix=/usr/local/boost
        > sudo ./b2 install
  (5) 安装编译 jsoncpp
        > mkdir -p build/debug
        > cd build/debug
        > cmake -DCMAKE_BUILD_TYPE=debug -DBUILD_STATIC_LIBS=ON -DBUILD_SHARED_LIBS=OFF -DARCHIVE_INSTALL_DIR=. -G "Unix Makefiles" ../..
        > make
  
  (6) 编译动态链接库 RabbitMQ
      > mkdir build && cd build
      > cmake -DENABLE_SSL_SUPPORT=OFF ..
      > cmake --build .
  
  (7) 编译安装 ppconsul(consul-client c++)
      > mkdir workspace
      > cd workspace
      > cmake  -DBOOST_ROOT=/home/zhangtx/install  -DCURL_ROOT=/home/zhangtx/install -DBUILD_STATIC_LIB=ON ..
      > make
      
  (8) 编译安装 curl
      > ./buildconf 
      > ./configure --prefix=/home/zhangtx/install
      > make
      > sudo make install
  (9) 编译安装 libxml2
        > ./autogen.sh
        > ./configure --prefix=/home/zhangtx/install
        > make 
            其中会遇到 fatal error: Python.h: No such file or directory 
            解决方法: > sudo yum install python-devel   # for python2.x installs
                     > sudo yum install python3-devel   # for python3.x installs
        > sudo make install
  (10) gflags
            > mkdir build && cd build
            > cmake .. -DGFLAGS_NAMESPACE=google -DCMAKE_CXX_FLAGS=-fPIC -DBUILD_SHARED_LIBS=ON ..
            > make 
            > sudo make install 
            > sudo ldconfig
            
  (11) glog
            > mkdir build && cd build
            > cmake -DGFLAGS_NAMESPACE=google -DCMAKE_CXX_FLAGS=-fPIC -DBUILD_SHARED_LIBS=ON ..
            > sudo make install
            > sudo ldconfig
            
  (12) libevent
            > cd libevent-2.1.8-stable/
            > ./configure --prefix=/usr/local
            > make
            > sudo make install

        
4. 给用户添加 sudo 权限
    (1) 
            
5. 第三方编译时引用到其他的第三方库(include, lib) 不在系统默认的搜索路径下
    vim /etc/profile
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/zhangtx/install/lib
    export LIBRARY_PATH=$LIBRARY_PATH:/home/zhangtx/install/lib
    export C_INCLUDE_PATH=$C_INCLUDE_PATH:/home/zhangtx/install/include
    export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/home/zhangtx/install/include
    
    (1) 编译时搜索的头文件的顺序 
            第一: 搜索项目自身加入的头文件路径, -I
            第二: 查找 gcc 的环境变量 C_INCLUDE_PATH，CPLUS_INCLUDE_PATH，OBJC_INCLUDE_PATH
            第三: 查找内定的目录 /usr/include ,/usr/local/include 等
    (2) 编译时搜索的库文件(动态链接库)的顺序
            第一: 搜索项目自身加入的库文件路径, -L
            第二: 查找 gcc 的环境变量 LIBRARY_PATH
            第三: 查找内定的目录  /lib /usr/lib /usr/local/lib
    (3) 运行时动态链接库的搜索顺序
            第一: 编译目标代码时指定的动态库搜索路径 -Wl, rpath ./
            第二: 环境变量 LD_LIBRARY_PATH 指定的动态库搜索路径
            第三: 配置文件/etc/ld.so.conf 中指定的动态库搜索路径
            第四: 默认的动态库搜索路径 /lib -> /usr/lib
    
6. 改变命令行前缀
     vim ~/.bashrc
     PS1="┌\[\u@\\h:\w]\n└->\$ "
```

## 创建用户并授权

```shell
    1. > adduser john(用户名)
    2. > passwd john  (设置密码)
    3. 查看 sudoer 文件
       > whereis sudoer
    4. 修改 /etc/sudoer 的可写权限
       > chmod +w /etc/sudoers
    5. vim /etc/sudoers
       root ALL=(ALL) ALL"在起下面添加"xxx ALL=(ALL) ALL  xxx 代表用户名
```

## Centos 操作
```shell
  1. 查看 centos 的版本号
        > cat /etc/redhat-release
        输出 CentOS Linux release 7.6.1810 (Core) 
  2. centos 更新 yum 源
        (1) 备份原镜像文件 mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
        (2) 下载新的CentOS-Base.repo 到/etc/yum.repos.d/
            > wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
        (3) 运行yum makecache生成缓存
            > yum makecache
  3. centos 安装 openssl
        > yum install openssl
        > yum install openssl-devel
     centos 安装 uuid
        > sudo yum install libuuid-devel
  3. 
    > sudo yum install autoconf automake libtool
    
  4. 安装软件
        第一步:　> yum list gdb   // 这个 gdb 可以模糊匹配
                gdb.x86_64  7.6.1-100.el7   
                
        第二步: yum  -y install gdb.x86_64   
        
  5. 防火墙使用
        > firewall-cmd --zone=block --add-port=80/tcp #添加端口；
           success 
        > firewall-cmd --zone=block --list-port #列出zone的端口；
        80/tcp
        > firewall-cmd --zone=block --remove-port=80/tcp
        
  6. 中文切换为英文
        1. > 先备份语言配置文件
           cp /etc/locale.conf /home/locale.conf.backup
        2  > 打开配置文件
           vim /etc/locale.conf
        3  > 把“zh_CN.UTF-8”修改为“en_US.UTF-8”  
        4、 保存并退出
        5 > shutdown -r now 重启
        
  7. rpm 安装
        > rpm -ivh lcov-1.13-1.noarch.rpm 
        
```

## configure 操作

```shell
    1. option 选项
          (1) --prefix=directory : 将成果物安装在哪个目录下
          (2) --libdir=DIR : 编译时指定第三方动态链接库
          (3) --includedir=DIR : 编译时指定第三方动态头文件搜索路径
```

## linux 编译
```shell
  1. 一般 g++/gcc 编译动态链接库时, -share -fPIC 通常都要带上 -fPIC, 即产生的动态链接库使用相对路径, 可以被加载器加载到内存的任意
     位置，都可以正确的执行。这正是共享库所要求的，共享库被加载时，在内存的位置不是固定的。
  2. gcc 编译选项
        (1) -m32: The -m32 option sets int, long and pointer to 32 bits and generates code that runs on any i386 system.
        (2) -m64: The -m64 option sets int to 32bits and long and pointer to 64 bits and generates code for AMD’s 
                  x86-64 architecture(64 位系统)
        (3) -mx32: The -mx32 option sets int, long and pointer to 32 bits and generates code for AMD’s x86-64 architecture.
  
  3. g++ 编译选项
        (1) -ftemplate-depth-n : 保证模板实例化的递归深度不超过整数n
        
  4. 动态链接库链接编译选项
      A: ldflags:
        (1) -Wl,-Bsymbolic : 主要是为了防止应用程序和动态链接库中定义了同一个全局变量名, 加了这个选项强制采用本地的全局变量定义，
                             就不会出现动态链接库的全局变量定义被应用程序的同名定义给覆盖
                             
                             http://www.php.cn/linux-368565.html  同名符号在linux下发生冲突的解决方案
                             
        (2) -Wl,--enable-new-dtags : 保持默认有限的行为, 防止采用绝对路径
        
        (3) -Wl,--hash-style=sysv : 在高版本的 linux 环境下编译，兼容在低版本的 linux 下运行
        (4) -Wl,--rpath= ./ : 在目标机器的下优先选择在当前目录下查找动态链接库
        
      B:ldlibs: 链接的动态链接库的名字
        (1) -lrt: 引用实时函数(lib real time), 例如 <time.h>中的函数 timer_create,timer_settime 等等
        (2) -lz : 压缩库(libz)
        (3) -lm : 数学库 (libmath)
        (4) -lc : 标准 c 库 (-libc)
        
      C:  -L/opt/  : 编译时查找动态链接库的位置
``` 

## linux 命令
```shell
    1. nc 命令
        (1) nc(netcat)
            nc [-hlnruz][-g<网关...>][-G<指向器数目>][-i<延迟秒数>][-o<输出文件>][-p<通信端口>][-s<来源位址>][-v...][-w<超时秒数>][主机名称][通信端口...]
            参　　数：
             -h  在线帮助。
             -l  使用监听模式，管控传入的资料。
             -n  直接使用IP地址，而不通过域名服务器。
             -r  乱数指定本地与远端主机的通信端口。
             -u  使用UDP传输协议。
             -z  使用0输入/输出模式，只在扫描通信端口时使用。
             -g<网关>  设置路由器跃程通信网关，最多可设置8个。
             -G<指向器数目>  设置来源路由指向器，其数值为4的倍数。
             -i<延迟秒数>  设置时间间隔，以便传送信息及扫描通信端口。
             -o<输出文件>  指定文件名称，把往来传输的数据以16进制字码倾倒成该文件保存。
             -s<来源位址>  设置本地主机送出数据包的IP地址。
             -v 详细输出--用两个-v可得到更详细的内容
             -w<超时秒数>  设置等待连线的时间。
             -p<通信端口>  设置本地主机使用的通信端口。
        (2) 端口扫描
            > nc -v -w 2 192.168.2.34 -z 21-24        
               
    2. curl
        (1) 查看 libcurl.so 用的是 NSS, 还是 openssl
            > curl -V
            修改引用的动态链接库路劲
            > vim etc/ld.so.conf
            > ldconfig
            
    3. iptable
        (1) 查看防火墙开发的端口
            > iptables -nL
            
    4. taskset
        (1) taskset 可以让某个程序运行在某个（或）某些 CPU 上
        (2) > taskset -p 21184 (显示进程运行的 CPU)
            显示结果：
                pid 21184's current affinity mask: ffffff
            
            注：21184 是 redis-server 运行的 pid
            显示结果的 ffffff 实际上是二进制 24 个位均为 1 的bitmask，每一个 1 对应于 1 个CPU，表示该进程在 24 个CPU上运行
            
        (3) > taskset -c 1 ./redis-server ../redis.conf
            进程启动时指定 CPU(从 0 开始计数)
             
```

## 动态链接库解释
```shell
    1. libexpat.so 用于解析 xml 
```