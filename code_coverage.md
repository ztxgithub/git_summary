# 代码覆盖率(gcov)

## 概念
```shell
    1. 代码覆盖率的度量方式
            (1) 语句覆盖(StatementCoverage)
                 又称行覆盖(LineCoverage), 段覆盖(SegmentCoverage), 基本块覆盖(BasicBlockCoverage)，
                 这是最常用的一种覆盖方式，就是度量被测代码中每个可执行语句是否被执行到了，不包括像 C＋＋ 的头文件声明，
                 代码注释，空行等等。只统计能够执行的代码被执行了多少行。需要注意的是，单独一行的花括号 {} 也常常被统计进去，
                 它只管覆盖代码中的执行语句, 却不考虑各种分支的组合等等。
            (2) 判定覆盖(DecisionCoverage)
                 又称分支覆盖(BranchCoverage), 它度量程序中每一个判定的分支是否都被测试到了，非常容易和条件覆盖混淆。
            (3) 条件覆盖(ConditionCoverage)
                 它度量判定中的每个子表达式的结果 true 和 false 是否被测试到了, 需要特别注意的是：条件覆盖不是将判定中的每个条件
                 表达式的结果进行排列组合，而是只要每个条件表达式的结果 true 和 false 测试到了就OK了。
                 可以这样推论：完全的条件覆盖并不能保证完全的判定覆盖
            (4) 路径覆盖(PathCoverage)
                 又称断言覆盖(PredicateCoverage)。它度量了是否函数的每一个分支都被执行了。 
                 所有可能的分支都执行一遍，有多个分支嵌套时，需要对多个分支进行排列组合，
                 测试路径随着分支的数量指数级别增加,它被很多人认为是“最强的覆盖”
    2. gcov 过程
            (1) 编译前，在编译器中加入编译器参数 -fprofile-arcs -ftest-coverage
            (2) 源码经过编译预处理, 然后编译成汇编文件，在生成汇编文件的同时完成插桩. 插桩是在生成汇编文件的阶段完成的，
                插桩是汇编时候的插桩，每个桩点插入 3 ~ 4 条汇编语句, 直接插入生成的 *.s 文件中，
                生成可执行文件, 并且生成关联 BB 和 ARC 的 .gcno 文件
            (3) 执行可执行文件, 在运行过程中桩点负责收集程序的执行信息( 桩点，其实就是一个变量，内存中的一个格子，对应的代码执行一次，
                则其值增加一次)
            (4) 生成.gcda文件, 其中有 BB 和 ARC 的执行统计次数等，由此经过加工可得到覆盖率。
            
    3. 编译参数:
            -ftest-coverage：在编译的时候产生 .gcno 文件，它包含了重建基本块图和相应的块的源码的行号的信息
            -fprofile-arcs： 在运行编译过的程序的时候，会产生.gcda文件，它包含了弧跳变的次数等信息
           
    4. 生成 gcov 代码覆盖率报告(*.gcov),(这一步在生成 lcov, genhtml 可以忽略)
            > gcov main.cpp.gcno
            main.cpp.gcov 里面的内容是:
                         -:    0:Source:/home/zhangtx/work_area/demo/test_demo/src/main.cpp
                         -:    0:Graph:main.cpp.gcno
                         -:    0:Data:main.cpp.gcda
                         -:    0:Runs:1
                         -:    0:Programs:1
                         -:    1:#include <string>
                         -:    2:#include <stdint.h>
                         -:    3:#include <stdio.h>
                         -:    4:#include <stdlib.h>
                         -:    5:#include <string.h>
                         -:    6:
                         -:    7:#include <stdio.h>
                         -:    8:#include <sys/types.h>
                         -:    9:#include <ifaddrs.h>
                         -:   10:#include <netinet/in.h>
                         -:   11:#include <string.h>
                         -:   12:#include <arpa/inet.h>
                         -:   13:#include <sys/ioctl.h>
                         -:   14:#include <net/if.h>
                         -:   15:
                         -:   16:using namespace std;
                         -:   17:
                     #####:   18:void NotUsed()
                         -:   19:{
                     #####:   20:    printf("not used \n");
                     #####:   21:}
                         1:   22:int main(int argc, char const *const *argv) {
                         -:   23:
                         -:   24://    printf("IP: %s\n", inet_ntoa(myaddr->sin_addr));
                         1:   25:    printf("test\n");
                         1:   26:}
                         
                         选项:
                            - : 代表这个语句不进行统计
                            ##### : 代表要统计但没有执行到
                            具体的数据: 例如 1 , 是指被执行的次数
                            
    5. 借助 lcov, genhtml 工具直接生成 html 报告
            (1) 安装 lcov 
                    > yum list lcov  # 查询 lcov 安装包
                      结果:
                        > yum list lcov
                            Loaded plugins: fastestmirror
                            Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
                            Determining fastest mirrors
                            Available Packages
                            lcov.noarch              1.13-1.el7                                       epel
                    > yum install -y  lcov.noarch
                    
           (2) 对当前目录下覆盖率的采集, 根据 *.gcda 文件并汇总到名为 outputfile.info 的文件中
                > lcov -b ./ -d ./ -c -o outputfile.info   
                参数选项:
                    -b directory : --base-directory directory : 使用 directory 主要解决 
                                   ERROR: could not read source file 的问题,
                    -d directory : 指定 *.gcda 文件的目录
                    -c: 捕获覆盖率数据
           (2.1) 将 ./ 目录下的 *.gcda 删除
                > lcov -d ./ -z
           (3) 通过 outputfile.info 生成图形化的 html 页面
                >  genhtml -o result_html outputfile.info 
                
           
           
    6. 配置
            1. 指定生成 gcda 文件到其它的路径了。解决的方法很简单，只要设一下 GCOV_PREFIX 和 GCOV_PREFIX_STRIP 
               这两个环境变量就可以。 GCOV_PREFIX 制定生成数据文件的前缀，GCOV_PREFIX_STRIP 表示需要在原来的路径上去掉多少层目录，
               通过指定这两个变量的值把数据文件生成到我们想要的地方
               export GCOV_PREFIX="/home/zhangtx/work_area"
               export GCOV_PREFIX_STRIP=5
               
               GCOV_PREFIX_STRIP 去掉原代码路径中的前几级，
               比如源代码路径为 /a/b/c/d.cpp, 如果 GCOV_PREFIX_STRIP = 2 则实际使用的路径是 c/d.cpp
               如果 GCOV_PREFIX = /run/gcov，则.gcda实际存放的路径是/run/gcov/c/d.gcda  (对应的文件夹会自动创建)
               
               具体例子:
               export GCOV_PREFIX="/tmp/gcov_tmp"  
               export GCOV_PREFIX_STRIP=8
               
    7. 参考资料:
            https://blog.csdn.net/yanxiangyfg/article/details/80989680
    8. 如果没有重新编译,即使程序重新运行,之前的测过的代码覆盖率也会保存.
       如果重新编译了,那么生成新的程序之前测试过的代码回零.

```

## gcov 运行流程
```shell
     1. 编译选项加入 -fprofile-arcs -ftest-coverage
     2. 运行可执行文件, 生成 .gcda 文件
     3. > gcov main.cpp.gcno   生成 main.cpp.gcov
```

##  gcov数据统计原理
```shell
    1. gcov是使用[基本块 BB] 和 [跳转 ARC] 计数，结合[程序流图]来实现代码覆盖率统计的.
    2. 基本块 BB
            如果一段程序的第一条语句被执行过一次，这段程序中的每一个都要执行一次，称为基本块。一个 BB 中的所有语句的执行次数一定是
            相同的。一般由多个顺序执行语句后边跟一个跳转语句组成。所以一般情况下 BB 的最后一条语句一定是一个跳转语句，
            跳转的目的地是另外一个 BB 的第一条语句，如果跳转时有条件的，就产生了分支，该 BB 就有两个 BB 作为目的地.
    3. 跳转 ARC
            要想知道程序中的每个语句和分支的执行次数，就必须知道每个 BB 和 ARC 的执行次数
    4. 程序流图
    5. 若用户进程并非调用 exit 正常退出，覆盖率统计数据就无法输出，也就无从生成报告了
```

## lcov 使用
```shell
    1. 初始化并创建基准数据文件
            > lcov -c -i -d ./ -o init.info
    2. 执行编译后的测试文件
            > ./test_cover
    3. 收集测试文件运行后产生的覆盖率文件
            > lcov -c -d ./ -o cover.info
    4. 合并基准数据和执行测试文件后生成的覆盖率数据
            > lcov -a init.info -a cover.info -o total.info  // -a : 合并文件
    5. 过滤不需要关注的源文件路径和信息( --remove 删除统计信息中如下的代码或文件，支持正则)
            > lcov -r total_output.info '*/usr/*' '*/3rd/*' 'common/*' 'frame/include' -o  simp.info
            
```

## 合并产生多个 info 文件

```shell
    1. 运行用例 1 产生 *.gcda , 根据 lcov 命令生成对应的 file1.info
    2. 运行用例 2 产生 *.gcda , 根据 lcov 命令生成对应的 file2.info
    3. 合并两个用例产生的info文件，输出同一个模块不同用例的总的统计数据
        > genhtml -o merge_html_dir file1.info file2.info // 合并后存放到 merge_html_dir 文件夹中
        > genhtml -o merge_html_dir *.info
```

## 问题处理
```shell
    1. ERROR: could not read source file /home/user/project/sub-dir1/subdir2/subdir1/subdir2/file.c
       解决方法: 在 home 目录下创建一个 ~/.lcovrc 文件,并加入一行 geninfo_auto_base = 1
       原因: 当编译工具链和源码不在同一个目录下时,会出现 ERROR: could not read source file 错误,这个 
             geninfo_auto_base = 1 选项指定 geninfo 需要自动确定基础目录来收集代码覆盖率数据
    2. 使用 lcov [srcfile]的命令生成.info文件的时候，提示如下错误, 无法生成info文件：
```

## CMakeList

```shell
    1. 
        OPTION(ENABLE_GCOV "OFF for disable  ON for Enable gcov debug, Linux builds only" OFF)
        IF (ENABLE_GCOV)
            SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -fprofile-arcs -ftest-coverage")
            SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -fprofile-arcs -ftest-coverage")
            SET(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} -fprofile-arcs -ftest-coverage -lgcov")
        
            set(LIBRARIES_GCOV
                    libgcov_preload.so
                    )
        
            message(STATUS "enable gcov")
        else(ENABLE_GCOV)
            set(LIBRARIES_GCOV
                    )
            message(STATUS "disnable gcov")
        
        ENDIF(ENABLE_GCOV)
```