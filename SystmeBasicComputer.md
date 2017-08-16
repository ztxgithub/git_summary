
- system page size

``` c
    A page, memory page, or virtual page是跟虚拟内存有关,它是一段连续的虚拟内存,
    一个page size 大小与处理器架构有关，一般是4K bytes
    
	
```

- [字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)


- 可重入函数

``` c
    多个线程可以同时调用该函数,该函数不会有操作静态变量
    
	
```

- 一个block的大小为512 bytes

- CRC16 = X16+X15+X2+X0

- free 内存分布

``` c

  应用程序(process)实际需要使用时,kernel仍会自动从memory釋放cache給process使用
    
  #释放pagecache
  echo 1 > /proc/sys/vm/drop_caches
	
  #釋放dentries與inodes
  echo 2 > /proc/sys/vm/drop_caches
  
    
  echo 3 > /proc/sys/vm/drop_caches
  3 是指釋放pagecache、dentries與inodes，也就是釋放所有的cache

```


