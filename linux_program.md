# linux 编程

## linux 动态加载动态库
```shell
    1. 动态加载动态库,其动态库相当于插件的形式,在程序动态执行时才加载库.动态链接库和动态加载库文件没有任何区别,
    2. 接口函数
        (1) void *dlopen(const char *filename, int flag);
            描述:
                在 dlopen的() 函数以指定模式打开指定的动态连接库文件，并返回一个句柄给调用进程。
                使用 dlclose（）来卸载打开的库
            参数:
                filename: 动态链接库路劲
                flag:
                    RTLD_LAZY 暂缓决定，等有需要时再解出符号 
                    RTLD_NOW 立即决定，返回前解除所有未决定的符号
            返回值:
                打开错误返回 NULL, 成功，返回库引用 
            注意: 要调用 dlopen 程序编译时要加入 -ldl (指定 dl 库)
            例如:
                void *handle = dlopen(LIB_CACULATE_PATH, RTLD_LAZY);
                
        (2) void* dlsym(void* handle,const char* symbol)
            描述: dlsym 根据动态链接库操作句柄 handle 与符号 symbol，返回符号对应的地址
                 使用这个函数不但可以获取函数地址，也可以获取变量地址
            参数:
                handle: dlopen 打开的指针
                symbol: 获取的函数或全局变量的名称
            返回:
                返回在内存中的地址
                
        (3) int dlclose(void *handle);
            描述:
                关闭 handle , 将对该动态链接库引用减一,只有当该动态链接库引用为 0 时,才会从系统中卸载掉该动态链接库
    3. 实例代码:
        typedef HPR_INT32(*pInit)();    
        typedef struct
        {
            pInit pfnInit;
        }dymal, *dymal_t;
        
    4. https://blog.csdn.net/bailyzheng/article/details/17613847
    5. 生成动态库的头文件有相应的规则
            #ifndef _DagServerExport_h_
            #define _DagServerExport_h_ 
            
            #if (defined _WIN32 || defined _WIN64)
            #   ifdef DAGSERVER_EXPORTS
            #       define IVMS_EXTERN extern "C" __declspec(dllexport)
            #   else
            #       define IVMS_EXTERN extern "C" __declspec(dllimport)
            #   endif
            #   define IVMS_API __stdcall
            #else
            #   ifdef __linux__
            #       define IVMS_EXTERN extern "C"
            #   else
            #       define IVMS_EXTERN
            #   endif
            #   define IVMS_API
            #endif
            
            /** @fn IVMS_EXTERN int IVMS_API IVMS_DAG_Init()
             *  @brief 初始化。
             *  @param void
             *  @return 成功返回0，否则返回其他值。
             */
            IVMS_EXTERN int IVMS_API IVMS_DAG_Init(const char* pszCmd);
           
            
            typedef int (IVMS_API *NotifyCallBack)(int msgType, void* pUserData);
            IVMS_EXTERN void IVMS_API IVMS_DAG_SetNotifyCallBack(NotifyCallBack fpMsgCB, void* pUserData);
            
            #endif // _DagServerExport_h_
                 
        
```
