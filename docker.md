# docker 使用

## 概念
```shell
    1. 简化程序: 打包应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，便可以实现虚拟化.
    2. Docker 容器通过 Docker 镜像来创建. 有点像 Docker 镜像是"类", Docker 容器是对象.
    3. 
        (1) Docker 镜像(Images): Docker 镜像是用于创建 Docker 容器的模板, 可以通过 docker images 来查看本机
         　　　　　　　　　　　　　　的镜像, REPOSITORY:TAG 代表一个唯一的镜像(REPOSITORY 是仓库源，　TAG 代表版本号)
        (2) Docker 容器(Container) : 容器是独立运行的一个或一组应用
        (3) Docker 客户端(Client) : Docker 客户端通过命令行或者其他工具使用 
                                   Docker API (https://docs.docker.com/reference/api/docker_remote_api) 与 
                                   Docker 的守护进程通信。
        (4) Docker Machine : Docker Machine是一个简化 Docker 安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装
                                     Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。
        (5) Docker 主机(Host) : 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。
        (6) Docker 仓库(Registry) : Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。
                                   Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用
        
```

## 命令

```shell
    1. docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
        参数:
            -P :是容器内部端口随机映射到主机的高端口
            -p : 是容器内部端口绑定到指定的主机端口
            --name: 给容器进行指定的名字
            -d: 以后台的形式运行
            -v : 将 host 的路劲挂载到容器中某个路劲
    
        启动容器(后台模式)
        > docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
        
        指定 docker 容器内端口号与主机端口号的映射关系(主机 port 为 8012, 容器内的 port 5000)
            > docker run -d -p 8012:5000 training/webapp python app.py
            training/webapp : 代表镜像名
             python app.py　：　镜像内的任务
             
        > docker run -d -P --name runoob training/webapp python app.py
          将该容器的名字命名为 runoob
          
        > docker run -p 6379:6379 -v $PWD/data:/data  -d redis:3.2 redis-server --appendonly yes
          将主机中当前目录下的 data 挂载到容器的 /data
        
    2. 查看容器的执行情况
        > docker ps
        输出:
        CONTAINER ID    IMAGE   COMMAND    CREATED   STATUS  PORTS 
        
    3. 查看 docker 日志
        > docker log container_id(容器编号)
        > docker log container_name(容器名字)
    4. 停止容器
        > docker stop + container_id(容器编号) 或则 container_name(容器名字)
    5. 查看 docker 容器内端口号与主机端口号的映射关系
        > docker port + container_id(容器编号) 或则 container_name(容器名字)
        输出结果:
            5001/tcp -> 0.0.0.0:5000    容器中的 5001 映射到主机的 5000     
        
    6. 检查容器的底层信息
        > docker inspect + container_id(容器编号) 或则 container_name(容器名字)
    7. 下载镜像
        > docker pull REPOSITORY:TAG 
    8. 查找镜像
        (1) > docker search + 镜像名
            例如:
                > docker seach httpd
                
        (2) > docker images
    9. 使用镜像
        > docker run httpd
    10. 创建镜像
        (1) 在已有的容器的基础上更改配置后，以次将该容器作为新的镜像
            runoob@runoob:~$ docker run -t -i ubuntu:15.10 /bin/bash
            root@e218edb10161:/#
            其中　e218edb10161 为容器 ID
            > docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
            参数:
                -m:提交的描述信息
                -a:指定镜像作者
                e218edb10161：容器ID
                runoob/ubuntu:v2 : REPOSITORY:TAG 
                
        (2) 使用 Dockerfile 指令来创建一个新的镜像
            a. 创建 Dockerfile 文件
                > cat Dockerfile 
                # 第一行必须指定基础镜像信息
                  FROM    centos:6.7    
                  
                # 维护者的信息
                  MAINTAINER      Fisher "fisher@sudops.com"　　
                  
                  # 镜像操作指令
                  RUN     /bin/echo 'root:123456' |chpasswd
                  RUN     useradd runoob
                  RUN     /bin/echo 'runoob:123456' |chpasswd
                  RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
                  EXPOSE  22
                  EXPOSE  80
                  
                  # 容器启动执行命令
                  CMD     /usr/sbin/sshd -D
                  
            b. 构建镜像
                > docker build [OPTIONS] PATH
                > docker build -t runoob/centos:6.7 /dockfile_dir
                参数:
                    -t : 创建的目标镜像名(tag)
                    PATH: 代表 Dockerfile 的绝对路径
                    
    11. 对容器进行操作
        docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
        参数:
            -i : 互动, Keep STDIN open even if not attached
            -t: --tty
        (1) 进入容器中
            > docker exec -it docker_nginx_v1(容器名)  /bin/bash
            
    12. 管理 docker 容器运行
        > docker start/stop/restart [OPTIONS] CONTAINER [CONTAINER...]
        例如:
        > docker stop 容器名
        
    13. 删除容器
        > docker rm 容器名
    
    14. 删除镜像
        > docker rmi REPOSITORY:TAG 
        
```

## dockerfile

```shell
    1. Dockerfile 是一个文本文件，内容包含了一条条的指令(Instruction)，每一条指令构建一层，
       有了 Dockerfile，当我们需要定制自己额外的需求时，只需在 Dockerfile 上添加或者修改指令，重新生成 image ，
       省去了敲命令的麻烦
    2. Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令、容器启动执行指令。一开始必须要指明所基于的镜像名称，
       接下来一般会说明维护者信息；后面则是镜像操作指令，例如 RUN 指令。每执行一条RUN 指令，镜像添加新的一层，并提交；
       最后是 CMD 指令，来指明运行容器时的操作命令
    2.1 FROM 
        FROM 可以在一个 Dockerfile 中出现多次，以便于创建混合的 images. 如果没有指定 tag , 
        latest 将会被指定为要使用的基础镜像版本,对于 linux_c++ 
        FROM  docker.hikvision.com.cn/library/centos:7.4.1708
        
    3. Docker 守护进程会一条一条的执行 Dockerfile 中的指令，而且会在每一步提交并生成一个新镜像，最后会输出最终镜像的 ID。
       生成完成后，Docker 守护进程会自动清理你发送的上下文
    4. RUN 
        (1) RUN command para1 para2 ...  : 其中 command 是 shell 终端的命令
        (2) RUN ["command", "para1", "para2"]  // 推荐使用
        
        RUN 指令缓存不会在下个命令执行时自动失效。比如 RUN apt-get dist-upgrade -y 的缓存就可能被用于下一个指令.
        --no-cache 标志可以被用于强制取消缓存使用
    5. CMD(指定容器开始时运行的命令)
        Dockerfile 只能有一个 CMD 指令, 提供默认的执行容器, 这些默认值可以包括可执行文件.
        例如:
            CMD ["ls",''-l"]  // 推荐使用
    6. EXPOSE ( 暴露指定的端口列表)
        EXPOSE 80 443 8080
    7. ENV (可以设置容器环境变量)
        ENV <key> <value>
    8. ADD
        添加指定的 src 到 dest 中，src 可以是文件、目录、tar
        ADD src dest
        ADD 命令自带解压功能，如果需要拷贝并解压一个文件到镜像中，可以使用 ADD 命令
        ADD 1.1.1.100:1234/jdk-8u74-linux-x64.tar.gz /usr/local/
    9. COPY
        复制本机 src 到容器 dest
        COPY src dest
        
        COPY 将文件从路径 <src> 复制添加到容器内部路径 <dest>. 
        <src> 必须是想对于源文件夹的一个文件或目录，也可以是一个远程的url，<dest> 是目标容器中的绝对路径
    10. VOLUME
        创建数据卷，用于挂载本地目录或用作数据卷容器,一般用来存放数据库和需要保持的数据等。
        VOLUME ["/data"]
    11. USER
        指定容器运行时的用户
        USER username
    12. WORKDIR
            WORKDIR PATH
            WORKDIR 用来切换工作目录的。Docker 默认的工作目录是/，只有 RUN 能执行 cd 命令切换目录，而且还只作用在当下下的 RUN，
            也就是说每一个 RUN 都是独立进行的。如果想让其他指令在指定的目录下执行，就得靠 WORKDIR。
            WORKDIR 动作的目录改变是持久的，不用每个指令前都使用一次 WORKDIR
           
```

## 使用
```shell
    1. 对于容器内进程修改的文件,可以存放在一个目录中,再 host 到容器内进行挂载
```
