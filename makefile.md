# makefile 

## 使用
```shell
    1. 当在命令行使用 "make" 命令时，不带任何目标(target)， 往往只会执行第一条命令。所以一般 "all" target 放在第一条，
      all 也只是一个通用的称呼，把所有目标包含进来， 这样就可以编译所有依赖的文件
    2. .PHONY 的用法是用于防止 make clean , makefile 中 clean target ,与当前目录下有文件名为 "clean" 文件相冲突。
        加了 .PHONY 之后只会运行 makefile 中的 clean target。
        
        实例:
        .PHONY: clean
        clean:
          rm -rf *.o
```

## 语法
```shell
    1. makefile 内容1:
                  hellomake: hellomake.c hellofunc.c
                      gcc -o hellomake hellomake.c hellofunc.c -I.     
                      
                  // 注意 gcc 前面一定要加 【tab】键 
    2. makefile 内容2:
                CC=gcc
                CFLAGS=-I.

                hellomake: hellomake.o hellofunc.o
                    $(CC) -o hellomake hellomake.o hellofunc.o
                    
                首先， CC 和 CFLAGS 是 make 中的宏定义, CFLAGS 是给 C 编译器的选项, make 需要编译 hellomake 可执行文件，需要
                单独编译 hellomake.o 和 hellofunc.o 文件。这个 makefile 有一个缺点，就是当 hellomake.h 内容发生变化时, make 
                不会重新编译 .c file.
                
    3. makefile 内容3:更通用的做法
            CC=gcc
            CFLAGS=-I.
            DEPS = hellomake.h
            OBJ = hellomake.o hellofunc.o 

            %.o: %.c $(DEPS)
              $(CC) -c -o $@ $< $(CFLAGS)

            hellomake: $(OBJ)
              $(CC) -o $@ $^ $(CFLAGS)
              
    4. makefile 内容4: 更项目化的做法
          IDIR =../include
          CC=gcc
          CFLAGS=-I$(IDIR)

          ODIR=obj
          LDIR =../lib

          LIBS=-lm

          _DEPS = hellomake.h
          DEPS = $(patsubst %,$(IDIR)/%,$(_DEPS))

          _OBJ = hellomake.o hellofunc.o 
          OBJ = $(patsubst %,$(ODIR)/%,$(_OBJ))


          $(ODIR)/%.o: %.c $(DEPS)
            $(CC) -c -o $@ $< $(CFLAGS)

          hellomake: $(OBJ)
            $(CC) -o $@ $^ $(CFLAGS) $(LIBS)

          .PHONY: clean

          clean:
            rm -f $(ODIR)/*.o *~ core $(INCDIR)/*~ 
```
## 基本信息
```shell
  1. "=" 赋值操作符
        make 会将整个 makefile 展开后，再决定变量的值。变量的值将会是整个 makefile 最后被指定的值。

              x = 123
              y = $(x) xxx
              x = xyz

        在上例中，y 的值将会是 xyz xxx ，而不是 123 xxx 
        
  2. ":=" 表示变量的值决定于它在 makefile 中的位置，而不是整个 makefile 展开后的最终值。
              x = 123
              y = $(x) xxx
              x = xyz
              
              y 的值将会是 123 xxx 
              
  3. makefile 的宏变量
        (1) CFLAGS: for the C compiler, 用于c 编译器的选项
        (2) CXXFLAGS for C++
        (3) CPPFLAGS for both
        
  4. wildcard : 全路径通配符
        makefile 内容:
          src=$(wildcard *.c ./sub/*.c)
          @echo $(src)
        输出:
          a.c b.c ./sub/sa.c ./sub/sb.c   wildcard 把 指定目录 ./ 和 ./sub/ 下的所有后缀是c的文件全部展开。
          
        实际例子：
          得到指定目录的所有的源文件（得到 makefile 当前目录下和 inc 目录下所有的源文件）
          SRC = $(wildcard *.c) $(wildcard inc/*.c)  
          
  
  5. notdir : 去除路径
        makefile 内容:
          src=$(wildcard *.c ./sub/*.c)
          dir=$(notdir $(src))
          @echo $(dir)
          
      输出:
          a.c b.c sa.c sb.c   notdir 把展开的文件去除掉路径信息
          
  6. patsubst ：替换通配符
        makefile 内容:
          src=$(wildcard *.c ./sub/*.c)
          dir=$(notdir $(src))
          obj=$(patsubst %.c,%.o,$(dir) )
          @echo $(obj)
          
      输出:
          a.o b.o sa.o sb.o   patsubst 把 $(dir) 中的变量符合后缀是 .c 的全部替换成 .o
          实际的原理是首先在 $(dir) 中匹配出所有的 %.c ，之后所有满足条件的目标集合和 %, 
          之后再用 % 替换掉 %.o ，% 替换其他(.o 保留)
          
      实例一:
        PROJ_LIBS =  hpr hlog pcre 
        LDLIBS := $(LDLIBS) $(patsubst %, -l%, $(PROJ_LIBS))
        @echo $(LDLIBS)
        
        输出:
          -lhpr -lhlog -lpcre -lpcrecpp
      
  7. $@  表示目标文件
     $^  表示所有的依赖文件
     $<  表示第一个依赖文件
     $?  表示比目标还要新的依赖文件列表
     
      main: main.o hello.o hi.o
        gcc -o $@ $^          ($^ 表示所有依赖的文件，main.o hello.o hi.o)
      main.o: main.c
              cc -c $<        ($< 代表第一个依赖的文件, main.c )
      hello.o: hello.c
              cc -c $<
      hi.o: hi.c
              cc -c $<
      clean:
              rm *.o
              rm main
              
  8. 替换引用规则
        obj=$(dir:%.c=%.o) 或则 obj = ${dir:%.c=%.o}
        将 $dir 中的每一个匹配 %.c 都替换为 %.o
        
  9. 应用 shell 方法
        获取绝对路径: absolute_dir = $(shell pwd) : 这个条件是在 makefile 目录下运行 make
        提取出文件夹路径: PROJ_TARGET_DIR := $(dir /bin/centso/x64/libxxx.so)  PROJ_TARGET_DIR 的值为 /bin/centso/x64/
        创建文件夹: MAKE_PROJ_TARGET := $(shell if [ ! -d $(PROJ_TARGET_DIR) ]; then mkdir -p $(PROJ_TARGET_DIR) ; fi )    
        
  10. foreach 函数
        $(foreach <var>,<list>,<text> ) : 把参数 <list> 中的单词逐一取出放到参数 <var> 所指定的变量中，然后再执行 <text> 所包含的表达式。
                                          其中 <list> 以空格为分隔，而最终输出的内容也以空格分隔输出
        实例:
        names := a b c d
        files := $(foreach n,$(names),$(n).o)
        那么 files 的值为 a.o b.o c.o d.o
  11. filter 过滤函数
        $(filter pattern…,text) : 过滤掉字串 “text” 中所有不符合模式 “pattern” 的单词，保留所
                                  有符合此模式的单词。可以使用多个模式。模式中一般需要包含模式字
                                  符“%”。存在多个模式时，模式表达式之间使用空格分割。 
        实例:
            C_SRCS  := $(filter %.c, $(SOURCES))
        
```