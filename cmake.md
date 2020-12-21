# cmake 使用

``` shell
1.实例代码
#支持CMAKE最低版本
cmake_minimum_required(VERSION 2.8)

#项目名称
project(example_test)

set(CMAKE_SYSTEM_NAME Linux)

MESSAGE(STATUS "This is BINARY dir " ${example_test_BINARY_DIR})
MESSAGE(STATUS "This is SOURCE dir " ${example_test_SOURCE_DIR})

set(LIBRARY_OUTPUT_PATH ${example_test_BINARY_DIR}/lib)
set(EXECUTABLE_OUTPUT_PATH ${example_test_BINARY_DIR}/bin)

set(CMAKE_CROSSCOMPILING TRUE)
set(CMAKE_C_COMPILER "gcc")
set(CMAKE_CXX_COMPILER "g++")
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

SET(CMAKE_BUILD_TYPE Release)
#SET(CMAKE_BUILD_TYPE Debug)

#配置编译选项
SET(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -g -ggdb -Wall")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fpermissive")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -fpermissive -std=c++11")

# 添加宏定义
# add_definitions(""-Wall -lpthread -g")

OPTION(UDP "OFF to close UDP_SWITCH ON to open" OFF) 
IF(UDP)
ADD_DEFINITIONS(-DUDP_COM)
ENDIF(UDP)

#配置头文件目录
include_directories(
include
)

# 设置环境变量 LIB_CPP
set(LIB_CPP
src/c_string.cpp
)

set(SOURCE_FILES
main.cpp
)

# 动态链接库的路径
link_directories(

)

# 动态链接库的名字 libicuuc.so
set(LIBRARIES

)

# 生成可执行文件
add_executable(example_test ${SOURCE_FILES} ${LIB_CPP})

# 生成动态链接库
add_library(example_test SHARED ${LIB_CPP})

#共享库链接
target_link_libraries(example_test ${LIBRARIES})
```

## cmake 用法
```shell
  1. CMakeList.txt 中定义宏定义
         add_definitions(-DPPCONSUL_USE_CPPNETLIB)
  2. 打印信息 MESSAGE 
         message([<mode>] "message to display" ...)
         mode 的值：
            (1) STATUS: 这个是信息类型，指明用户所感兴趣的信息。
            (2) FATAL_ERROR: CMake Error, stop processing and generation
            (3) SEND_ERROR: continue processing, but skip generation
         例如:
            MESSAGE(STATUS "This is BINARY dir " ${example_test_BINARY_DIR})
  3. 变量
        (1) ${PROJECT_NAME} : 代表项目名
            例如 project(json11), 则 ${PROJECT_NAME} 值为 "json11"
        (2)  ${CMAKE_SOURCE_DIR} : 当前 CMakeList.txt 所处的绝对路径
  4. 生成链接库
        add_library( <name>  [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] [source1] [source2 ...])
        描述：生成一个库(library target)名且 name, 通过 source files 进行编译，如果这些 source files
             在后面被 target_sources() 使用，则 [source1] [source2 ...] 可以不写
             
  5. 增加一个子目录到编译系统
        add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
        其中 source_dir 子目录包含下一级的 CMakeLists.txt
  6. add_executable
        add_executable(<name> [WIN32] [MACOSX_BUNDLE] [EXCLUDE_FROM_ALL] source1 [source2 ...])
        定义一个名字为 name 可执行程序输出目标,其中 source1 [source2 ...]是源文件，比如C语言是.C的源文件，
        .H的源文件不需要指定，cmake会自动去搜索
        
  7. target_sources
        target_sources(<target> <INTERFACE|PUBLIC|PRIVATE> [items1...]
                        [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
        往一个目标里面添加源文件，这个目标名字 target 是在 add_executable() 或者add_library() 中定义的 name
        
  8. 指定编译参数(为add_executable() 或者 add_library() 中定义的输出目标指定编译选项)
        (1) target_include_directories(<target> [SYSTEM] [BEFORE] <INTERFACE|PUBLIC|PRIVATE> [items1...] 
                                       [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
            Include 的头文件的查找目录，也就是Gcc的[-I dir...]选项
            实例:
                target_include_directories(test_elf
                                                PRIVATE
                                                ${CMAKE_SOURCE_DIR}
                                                ${CMAKE_SOURCE_DIR}/common
                                            )
                                            
        (2) target_compile_definitions(<target> <INTERFACE|PUBLIC|PRIVATE> [items1...]
                                       [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
            通过命令行定义的宏变量，也就是 gcc 的[-Dmacro[=defn]...]选项
            set(MONITOR_OMIT_BSS_INIT      "0")
            target_compile_definitions(test_elf
                PRIVATE
                MONITOR_OMIT_BSS_INIT=${MONITOR_OMIT_BSS_INIT}                                    
            )
            
        (3) target_compile_options(<target> [BEFORE] <INTERFACE|PUBLIC|PRIVATE> [items1...] 
                                   [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...]
            gcc其他的一些编译选项指定，比如-fPIC
            
            target_compile_options(test_elf
                                        PRIVATE
                                        -std=c99 
                                        -Wall 
                                        -Wextra 
                                        -Werror
                                    )
   9. 链接参数(编译生成输出目标需要连接其他的库文件)
          (1) target_link_libraries(<target> <PRIVATE|PUBLIC|INTERFACE> <item>... 
                                     [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
             

              # 链接选项设置
              target_link_libraries(test_elf
                  PRIVATE
                  -msoft-float 
                  -static 
                  -nostdlib
              )

              #设置链接的标准路径的库
              target_link_libraries(test_elf PRIVATE gcc)

              #设置链接的本 project 自己输出的目标库 libkernel.a
              target_link_libraries(test_elf PRIVATE kernel)

              #设置链接非标准路径中别人编译好的库
              target_link_libraries(test_elf 
                                    PRIVATE 
                                    $ENV{HOME}/lib/libtest.a
                                    )
         
```

## 综合使用
```shell
  1. 定义的输出目标(可以是二进制文件，静态库，动态库)是全局的，比如你在根目录通过 add_executable 定义了一个二进制文件，
     可以在子目录中用 target_sources 命令往这个输出目标里面加源文件，cmake 在输出这个目标之前会在
     当前 project 中搜集所有添加到该目标的源文件再去生成一个编译链接的规则
     
     #定义一个mytool 可执行程序输出目标，目前只有一个源文件  mytool.cpp
      add_executable(mytool mytool.cpp)

      #往mytool可执行程序输出目标添加源文件，现在mytool目标里面包含两个源文件
      target_sources(mytool  PRIVATE mytool2.cpp)

      #定义一个动态库archive 输出目标，文件有三个源文件
      add_library(archive SHARED archive.cpp zip.cpp lzma.cpp)

      #定义一个静态库archive 输出目标，也可以不指定STATIC 因为add_library默认输出目标是#静态库
      add_library(archive STATIC archive.cpp zip.cpp lzma.cpp)

      #从给出的源文件直接生成object文件，比如linux下C语言的.o文件
      add_library(archive OBJECT archive.cpp zip.cpp lzma.cpp)
      
  2. 
      PRIVATE:   只给自己用，不给依赖者用
      PUBLIC:    自己和依赖者都可以用
      INTERFACE: 自己不用，给依赖着用
```

## 使用规范
```shell
    1. 在 CMakeList.txt 中一定要指名是 debug(CMAKE_C_FLAGS_DEBUG) 还是 release(CMAKE_C_FLAGS_RELEASE), 
       二者选其一
            option(DEBUG "ON for debug or OFF for release" OFF)
            if(DEBUG)
                message(STATUS "Debug mode")
                set(CMAKE_BUILD_TYPE Debug)
                set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g -Wall -fprofile-arcs -ftest-coverage")
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g -Wall -std=c++11 -fprofile-arcs -ftest-coverage")
            else(DEBUG)
                set(CMAKE_BUILD_TYPE Release)
                set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -s -Wall -fprofile-arcs -ftest-coverage")
                set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -s -Wall -std=c++11 -fprofile-arcs -ftest-coverage")
            endif(DEBUG)
            
    2. 添加编译选项
            (1) 第一种方式: 
                    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g -Wall -fprofile-arcs -ftest-coverage")
            (2) 第二种方式:
                    add_definitions(-fprofile-arcs -ftest-coverage)
                    
    3. 增加编译链接选项
            (1) 第一种方式:
                    SET(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} -fprofile-arcs -ftest-coverage -lgcov")
```

## 安装
```shell
    1. 去官网下载可执行的二进制文件
    2. > sudo vim /etc/profile
        export PATH=$PATH:/home/user/cmake/bin
    https://www.cnblogs.com/weiqinglan/p/5681539.html
```

## 错误排查

```shell
  1.  出现 target_compile_features specified unknown feature cxx_std_11 for target... 
      原因是 cmake 版本太低.
```
## 操作使用

```shell
  1. 使用 cmake 安装到指定目录 cmake -DCMAKE_INSTALL_PREFIX=/usr ..
  2. 如果是 CMakeList.txt 内部定义的宏定义，则 cmake -DBUILD_STATIC_LIB=ON .
```