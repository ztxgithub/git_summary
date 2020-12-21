# Qt 使用

## 基本背景
```shell
    1. MinGW: 为了在 Windows 系统里可以使用 GNU 工具, 诞生了 MinGW( Minimalist GNU for Windows)项目, 利用 MinGW 就可以生成
       Windows 里面的 exe 程序和 dll 链接库
    2. GNU 工具
            (1) gcc GNU : C 语言编译器。
            (2) g++ GNU : C++ 语言编译器。
            (3) ld     : GNU 链接器, 将目标文件和库文件链接起来, 创建可执行程序和动态链接库。
            (4) ar     :生成静态库 .a ，可以编辑和管理静态链接库。
            (5) make :生成器, 可以根据 makefile 文件自动编译链接生成可执行程序或库文件。
            (6) gdb     :调试器，用于调试可执行程序。
            (7) ldd 查  :看可执行文件依赖的共享库（扩展名 .so，也叫动态链接库）
    3. MinGW 与 Linux/Unix 系统里 GNU 工具集的有些区别：
            (1) MinGW 里面工具带有扩展名 .exe, Linux/Unix 系统里工具通常都是没有扩展名的
            (2) MinGW 里面的生成器文件名为 mingw32-make.exe, Linux/Unix 系统里就叫 make。
            (3) MinGW 在链接时是链接到 *.a 库引用文件, 生成的可执行程序运行时依赖 *.dll,
                而 Linux/Unix 系统里链接时和运行时都是使用 *.so 。
            (4) MinGW 里也没有 ldd 工具，因为 Windows 不使用 .so 共享库文件。如果要查看 Windows 里可执行文件的依赖库,
               需要使用微软自家的 Dependency Walker 工具。Windows 里面动态库扩展名为 .dll, 
               MinGW 可以通过 dlltool 来生成用于创建和使用动态链接库需要的文件，如 .def 和 .lib
    4. Qt 工具集
            (1) qmake 
                    核心的项目构建工具，可以生成跨平台的 .pro 项目文件，并能依据不同操作系统和编译工具生成相应的 Makefile，
                    用于构建可执行程序或链接库。
            (2) uic 
                    User Interface Compiler，用户界面编译器，Qt 使用 XML 语法格式的 .ui 文件定义用户界面, uic 根据 .ui 文件
                    生成用于创建用户界面的 C++ 代码头文件，比如 ui_*****.h 。
            (3) moc
                 Meta-Object Compiler，元对象编译器，moc 处理 C++ 头文件的类定义里面的 Q_OBJECT 宏, 它会生成源代码文件，
                 比如 moc_*****.cpp , 其中包含相应类的元对象代码，元对象代码主要用于实现 Qt 信号/槽机制、运行时类型定义、
                 动态属性系统。
            (4) rcc
                 Resource Compiler，资源文件编译器，负责在项目构建过程中编译 .qrc 资源文件，将资源嵌入到最终的 Qt 程序里。
            (5) qtcreator 
                    集成开发环境，包含项目生成管理、代码编辑、图形界面可视化编辑、 编译生成、程序调试、上下文帮助、
                    版本控制系统集成等众多功能， 还支持手机和嵌入式设备的程序生成部署。
            (6) assistant 
                    Qt 助手，帮助文档浏览查询工具，Qt 库所有模块和开发工具的帮助文档、示例代码等都可以检索到，是 Qt 开发必备神器，
                    也可用于自学 Qt。
            (7) designer 
                    Qt 设计师，专门用于可视化编辑图形用户界面（所见即所得），生成 .ui 文件用于 Qt 项目。
            (8) linguist 
                    Qt 语言家，代码里用 tr() 宏包裹的就是可翻译的字符串，开发人员可用 lupdate 命令生成项目的待翻译字符串文件 .ts，
                    用 linguist 翻译多国语言 .ts ，翻译完成后用 lrelease 命令生成 .qm 文件，然后就可用于多国语言界面显示。
            (9) qmlscene 
                    在 Qt 4.x 里是用 qmlviewer 进行 QML 程序的原型设计和测试，Qt 5 用 qmlscene 取代了旧的 qmlviewer。
                    新的 qmlscene 另外还支持 Qt 5 中的新特性 scenegraph 
    5. c++11 标准
            (1) 最新的 C++11 标准是 2011 年 8 月 12 日公布的，在公布之前该标准原名为 C++0x
            (2) 编译器里面开始支持 C++11 的版本是 MSVC 2010、GCC 4.5、Clang 3.1，这之后版本的编译器都在逐步完善对 C++11 的支持
            (3) MSVC 编译器默认开启 C++11 特性，GCC（g++命令）则需要自己添加选项 -std=c++0x ,
                上面 CXXFLAGS 就是为 GCC 编译器（g++命令）添加 -std=c++0x 选项
    6. Dynamic Link 和 Static Link
            (1) Linux/Unix 系统里静态库扩展名一般是 .a，动态库扩展名一般是 .so 
                Windows 系统里 VC 编译器用的静态库扩展名一般是 .lib，动态库扩展名一般是 .dll
    7. Explicit Linking 和 Implicit Linking
            (1) Explicit Linking : 显式链接, 在运行时使用主动加载动态库的函数，Linux 里比如用 dlopen 函数打开并加载动态库，
                                   Windows 里一般用 LoadLibrary 打开并加载动态库，只有当程序代码执行到这些函数时，
                                   其参数里的动态库才会被加载. 显式链接方式是在运行时加载动态库，其程序启动时并不检查这些动态库是否存在
            (2) Implicit Linking: 隐式链接, 所有的编译环境默认都是采用隐式链接的方式使用动态库。
                                  隐式链接会在链接生成可执行程序时就确立依赖关系，在该程序启动时，操作系统自动会检查它依赖的动态库，
                                  并一一加载到该程序的内存空间，程序员就不需要操心什么时候加载动态库了
                                  
                                  VC 编译环境, 链接时使用动态库对应的 .lib 文件(包含动态库的导出函数声明，但没有实际功能代码), 
                                  在 .exe 程序运行前系统会检查依赖的 .dll
    8. Qt Creator 可以创建多种项目
            (1) Qt Widgets Application, 支持桌面平台的有图形用户界面(Graphic User Interface，GUI) 界面的应用程序.
                GUI 的设计完全基于 C++ 语言，采用 Qt 提供的一套 C++ 类库。
            (2) Qt Console Application，控制台应用程序，无 GUI 界面，一般用于学习 C/C++ 语言，
                只需要简单的输入输出操作时可创建此类项目。
            (3) Qt Quick Application，创建可部署的 Qt Quick 2 应用程序。Qt Quick 是 Qt 支持的一套 GUI 开发架构，
                其界面设计采用 QML 语言，程序架构采用 C++ 语言。利用 Qt Quick 可以设计非常炫的用户界面，
                一般用于移动设备或嵌入式设备上无边框的应用程序的设计。
            (4) Qt Quick Controls 2 Application，创建基于 Qt Quick Controls 2 组件的可部署的 Qt Quick 2 应用程序。
                Qt Quick Controls 2 组件只有 Qt 5.7 及以后版本才有。
            (5) Qt Canvas 3D Application，创建 Qt Canvas 3D QML 项目，也是基于 QML 语言的界面设计，支持 3D 画布。
```

## 经验

```shell
    1. 如果要用 Qt Createor 进行项目开发则项目的路劲需要是英文.
```

## Qt 知识
```shell
    1. 界面的基类(base class)
            (1) QMainWindow 是主窗口类，主窗口具有主菜单栏、工具栏和状态栏，类似于一般的应用程序的主窗口；
            (2) QWidget 是所有具有可视界面类的基类，选择 QWidget 创建的界面对各种界面组件都可以 支持；
            (3) QDialog 是对话框类，可建立一个基于对话框的界面；
    2. 一个 Label 标签组件的继承关系是 QObject -> QWidget -> QFrame -> QLabel
    3. Qt 项目目录结构分布
            (1) 窗体类的头文件(widget.h)
                    在创建项目时, 选择窗体基类是 QWidget, 在 widget.h 中定义了一个继承自 QWidget 的类 Widget
                    #ifndef WIDGET_H
                    #define WIDGET_H
                    
                    #include <QWidget>
                    
                    namespace Ui {
                     // 这是声明了一个名称为 Ui 的命名空间（namespace），包含一个类 Widget。但是这个类 Widget 
                     // 并不是本文件里定义的类 Widget，而是 ui_widget.h 文件里定义的类，用于描述界面组件的。
                     // 这个声明相当于一个外部类型声明
                    class Widget;  
                    }
                    
                    class Widget : public QWidget
                    {
                    // 使用 Qt 的信号与槽（signal 和 slot）机制的类都必须加入的一个宏
                        Q_OBJECT
                    
                    public:
                        explicit Widget(QWidget *parent = 0);
                        ~Widget();
                    
                    private:
                    // 这个指针是用前面声明的 namespace Ui 里的 Widget 类定义的，指针 ui 是指向可视化设计的界面，
                    // 后面会看到要访问界面上的组件，都需要通过这个指针 ui。
                        Ui::Widget *ui;
                    };
                    
                    #endif // WIDGET_H
                    
            (2) 类 Widget 的实现代码(widget.cpp)
                    #include "widget.h"
                    #include "ui_widget.h"
                    
                    Widget::Widget(QWidget *parent) :
                        QWidget(parent),
                        ui(new Ui::Widget)   // 对父类进行 parent 的初始化
                    {
                        ui->setupUi(this);  // 实现窗口的生成与各种属性的设置、信号与槽的关联
                    }
                    
                    Widget::~Widget()
                    {
                        delete ui;
                    }
                    
            (3) widget.ui 文件
                    widget.ui 是窗体界面定义文件，是一个 XML 文件，定义了窗口上的所有组件的属性设置、布局，及其信号与槽函数的关联等。
                    用 UI 设计器可视化设计的界面都由 Qt 自动解析，并以 XML 文件的形式保存下来。在设计界面时,
                    只需在 UI 设计器里进行可视化设计即可，而不用管 widget.ui 文件是怎么生成的
            (4) ui_widget.h 文件
                    ui_widget.h 是在对 widget.ui 文件编译后生成的一个文件, ui_widget.h 会出现在编译后的目录下, 
                    或与 widget.ui 同目录(与项目的 shadow build 编译设置有关)
                    a. 在　ui_widget.h　中　setupUi() 调用了函数 retranslateUi(Widget)，用来设置界面各组件的文字内容属性，
                       如标签的文字、按键的文字、窗体的标题等
                    b. 在 setupUi() 中设置信号与槽的关联, 
                            QObject::connect(btnClose, SIGNAL(clicked()), Widget, SLOT(close()));
                            调用 connect() 函数, 将在 UI 设计器里设置的信号与槽的关联转换为语句。这里是将 btnClose 按键的 
                            clicked() 信号与窗体 Widget 的 close() 槽函数关联起来
                            
                            QMetaObject::connectSlotsByName(Widget);
                            设置槽函数的关联方式，用于将 UI 设计器自动生成的组件信号的槽函数与组件信号相关联
            (5) main 主函数
                    int main(int argc, char *argv[])
                    {
                        QApplication a(argc, argv); //定义并创建应用程序
                        Widget w; //定义并创建窗口
                        w.show(); //显示窗口
                        return a.exec(); //应用程序运行
                    }
                    
    4. 界面组件布局
        (1) 界面组件的层次关系: 将界面上的各个组件的分布设计得更加美观
                使用一些容器类，如 QgoupBox、QtabWidget、QFrame 等。
        (2) 布局管理
                a. Vertical Layout: 垂直方向布局，组件自动在垂直方向上分布
                b. Horizontal Layout: 水平方向布局，组件自动在水平方向上分布
                c. Grid Layout: 网格状布局，网状布局大小改变时，每个网格的大小都改变
                d. Form Layout: 窗体布局，与网格状布局类似，但是只有最右侧的一列网格会改变大小
                e. Horizontal Spacer: 一个用于水平分隔的空格
                f. Vertical Spacer 一个用于垂直分隔的空格
        (3) 程序流程
                a. 信号 clicked(bool) 会将 CheckBox 组件当前的选择状态作为一个参数传递，在响应代码里可以直接利用这个传递的参数.
                   而如果用信号 clicked(), 则需要在代码里读取 CheckBox 组件的选中状态。为了简化代码，选择 clicked(bool) 信号
                
```
