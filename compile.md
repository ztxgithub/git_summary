
## undefined reference问题总结

- 链接时缺失了相关目标文件（.o）

- 链接时缺少相关的库文件（.a/.so）

- 多个库文件链接顺序问题

``` shell
    
    依赖其他库的库一定要放到被依赖库的前面，这样才能真正避免undefined reference的错误，完成编译链接

```

- 在c++代码中链接c语言的库

``` shell
    
    解决方法：即在main.cpp中，把与c语言库test.a相关的头文件包含添加一个extern "C"的声明即可
    
    在 main.cpp 中
    
    extern "C" 
    {
        #include"test.h"
    }

```
[参考资料](http://ticktick.blog.51cto.com/823160/431329)

