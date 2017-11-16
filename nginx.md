# Nginx

## Nginx 安装步骤

``` shell

nginx依赖以下模块：
    1. gzip模块需要 zlib 库
    2. rewrite模块需要 pcre 库

mkdir nginx在home目录下创建nginx目录,作为nginx的安装目录.

    1.安装pcre
        (1) 获取pcre编译安装包，在http://www.pcre.org/上可以获取当前最新的版本,也可使用附件中的软件包。
        (2) 解压缩pcre-xx.zip包。
        (3) 进入解压缩目录，执行./configure。
        (4) make & make install
        
    2.安装zlib
        (1) 获取zlib编译安装包，在http://www.zlib.net/上可以获取当前最新的版本,也可使用附件中的软件包。
        (2) 解压缩openssl-xx.tar.gz包。
        (3) 进入解压缩目录，执行./configure。
        (4) make & make install
        
    3.安装nginx
        (1) 获取nginx，在http://nginx.org/en/download.html上可以获取当前最新的版本,也可使用附件中的软件包。
        (2) 解压缩nginx-xx.tar.gz包。
        (3) 进入解压缩目录，执行./configure --prefix=/home/nginx --with-stream
        (4) make & make install			
```
[Nginx安装](https://segmentfault.com/a/1190000007116797)
