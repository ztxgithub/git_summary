# window 操作

## windows 调试工具
```shell
  1. Debug Diagnostics Tool: 可以排除 win 下的内存不足，cpu 过高情况
  2. 操作流程
        （1） 下载 DebugDiagx86.msi 或则 DebugDiagx64.msi
         (2)  在 64 位系统上抓取 32 位进程的 dump, 运行 C:\Windows\SysWOW64\taskmgr.exe, 可以看到进程名为
              taskmgr.exe*32, 则证明任务管理器进程为 32 位
         (3)  在要抓取的进程中右键 -> 创建转储文件
         (4) 然后在保存下来的 dump 文件右键打开方式进行 Debug Diagnostics tool 打开
```
## 查看动态链接库 dll 依赖关系
```shell
    1. 使用 depends.exe 
```

## windows vs2015 调试
```shell
    1.拿到编译出来的 *.pdb 文件，和 *.dump 文件
    2. 以 vs2015 打开 *.dump -> Debug with Native Only -> Break
    3. 在 vs2015 工具栏 -> Debug -> Windows -> Call Stack, 出现 Call Stack 视图，点击最上层
    4. 
```