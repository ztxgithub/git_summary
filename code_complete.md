# 代码大全

- 表驱动技术(很重要)
  
    
## 不常见的数据类型

### 指针

- 在释放指针后要记得将指针的值为NULL(对应在其他地方使用该指针时要记得判断是否为NULL)
```c
    free(ptr);
    ptr = NULL;
```

- 如果在函数(子程序)传入要改变的值,最好传该对象的指针
- 如果传入的参数为只读的,可以定义const 对象的引用(这样可以减少拷贝对象的资源和时间的开销)
```c++

    int set_multi_threshold_proc(const MultiThresholdSetReq &req);
    
    
```
- 引用指针时,要检查指针是否为空

### 结构体

```c++

    如果传入的变量太多,可以考虑用结构体来组织相关的数据结构
    
```

### 全局变量

- 使用访问器子程序来包含全局变量

```c++

    1.对全局变量创建好特定的规则,如g_var等
    
    
```

## 组织直线型代码

- 将代码相关的语句组织在一起

```c++

    SalesData saleData;
    saleData.ComputeQuarterly();
    saleData.ComputeAnnual();
    saleData.Print();
    
    代码可读性较强,易于封装成一个函数.
    
```

## 使用条件语句

- if-else if - else if 先判断最常见的情况

- 如果 if(exp) 中 exp太过复杂可以考虑封装一个布尔函数