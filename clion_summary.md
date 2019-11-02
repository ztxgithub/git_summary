# clions 使用

## windows clion 使用
```shell
    1. 配置好 Toolchains
        在 windows 环境下进行 linux 开发, settings -> Build、Execution、Deployment -> Toolchains -> 
       Environment -> Remote Host -> Credentials 
       
    2. 配置好 CMake
        settings -> Build, Execution, Deployment -> CMake
            Build type: Debug (这个 Debug 是必须的)
            Toolchains: Remote Host(上一步设置好的 Remote Host)
            CMake options: -DCMAKE_BUILD_TYPE=Debug
            Generation path: cmake-build-debug
    3. Deployment -> Connection
            Root path:/
       
    4. Deployment -> Mappings
            Local path: E:\clion_example\untitled
            Deployment path:/tmp/unititled
            
```