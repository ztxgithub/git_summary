# 基础库的使用

## 概念
```shell
  1. 使用 volatile, 避免代码优化,强制每次从内存中读取
        
```

## 原子变量
```shell
    (1) windows
            InterLockedIncrement -> 原子自加 value += 1
            InterLockedDecrement -> 原子自减 value -= 1
            InterLockedExchangeAdd -> 原子加
            InterLockedExchangeSet  -> 原子赋值
            InterLockedCompareExchangeSet  -> 原子交换
```

## 事件
```shell
    (1) windows
            CreateEvent -> 创建事件
            CloseHandle -> 销毁事件
            WaitForSingleObject -> 等待单个事件
            WaitForMultipleObject -> 等待多个事件
            ResetEvent -> 重置事件
            SetEvent -> 触发事件
```

## 动态库操作
```shell
    (1) windows
            LoadLibraryEx, FreeLibrary, GetProcAddress
            
    (2) linux
            dlopen, dlclose, dlsym
            
    注意:
        a.
            linux 动态库符号默认是导出(函数,全局变量), 有可能导致多个模块间符号冲突,调用错乱.
            -Wl,-Bsymbolic: 优先使用库内的符号
            -Wl,--retain-symbols-file: 导出符号集合
            -Wl,--version-script : 限制导出符号
            编译时通过 -fvisibility=hidden 参数来隐藏符号,在需要导出的函数和变量前加上 
            __attribute__((visibility("default"))) 来导出符号
        b.
            dlopen 中 RTLD_LOCAL 能防止动态库中定义的符号被其后打开的库重定位
        c.
            如果依赖的库未限制符号导出,需要注意库链接的顺序,以免使用错误的符号.
            例如: LIBS = -lhpr -lhlog -lutils
            这个时候链接时先链接了 libhlog.so,而 libhlog.so 引用了版本 A 的 get_memory_used(utils.a), 
            get_memory_used(utils.a 版本 A)   -->  libhlog.so  --> isa.so   <-- get_memory_used(utils.a 版本 B)
            isa.so 先链接了 libhlog.so, 这时候直接链接版本 A 的 get_memory_used, 版本 B 被忽略
        d. 
            如果多个库导出向相同的符号,例如一个程序链接两个库(一个静态库,一个动态库), 先链接的库的符号会加入到全局符号表中,
            并被所有模块调用(除非其他动态链接库编译时加上  -Wl,-Bsymbolic,以优先使用自己库内的符号), 这时如果想要调用静态库中
            符号(get_memory_used), 则优先链接这个静态链接库
```

## 文件系统
```shell
    1. windows
            (1) 创建目录 CreateDirectory
            (2) 删除目录 RemoveDirectory
            (3) 获取目录内文件 FindFirstFileExW, FindNextFileW
            (4) 打开文件 CreateFile
            (5) 关闭文件 CloseHandle
            (6) 删除文件 DeleteFile
            (7) 读文件 ReadFile
            (8) 写文件 WriteFile
            (9) 移动读写位置 SetFilePointer
            (10) 文件移到结尾 ERROR_HANDLE_EOF == GetLastError()
            (11) 统计文件信息 GetFileInformationByHandle
            (12) 刷新文件缓冲区 FlushFileBuffers
            (13) 获取当前执行程序路劲 GetModuleFileName
            (14) 确定文件或则文件夹的访问权限 _access    (AccessFile)
            (15) 复制文件 CopyFile
            (16) 重命名文件 MoveFile    (rename file)        

```

## 网络传输

```shell
    1. UDP 最大能传输 64 KB, UDP中的总长度字段为2字节 所能表示的最大数为65535 UDP协议头本身占据了8字节
       所以所能发送的最大数据长度是65535-8=65527
```