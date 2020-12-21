# python 

## 注意事项

```shell
1. 注释 """ xxxx """ 必须是两行以上的代码, 一行的代码用 #
```

## Python 语法

```shell
    1. super() 函数用于调用父类的一个方法,调用的方法一般是 super(type[, object-or-type]), 例如: super(type, self).xxx
       是调用父类的方法.
       
    2.     try:
               print('try块的代码，没有异常')
           except:
               print('程序出现异常')
           else:
               # 将else_test放在else块中
               else_test()
               
           当 try 块没有出现异常时，程序会执行 else 块, 如果希望某段代码的异常能被后面的 except 块捕获，那么就将这段代码
           放在 try 块的代码之后；如果希望某段代码的异常能向外传播（不被 except 块捕获），那么就将这段代码放在 else 块中
           
    3. range(start, stop[, step])
        start: 计数从 start 开始。默认是从 0 开始。例如 range(5) 等价于 range(0, 5);
        stop: 计数到 stop 结束，但不包括 stop。例如：range(0, 5) 是 [0, 1, 2, 3, 4] 没有 5
        step：步长，默认为 1。例如：range(0, 5) 等价于 range(0, 5, 1)
        
    4. 将 string 转成 byte 
        str.to_bytes(byte_num, byteorder='big)
        其中 byte_num 是用多少个字节来存, byteorder 代表大小段
        
    5. 
        (1) r/R : 其前缀表示非转义的原始字符串
            r'input\n', 通过 r' 关键字, ’\n’ 变成了 ’\‘ 和 ’n’。也就是 \n 表示的是两个字符，而不是换行
        (2) b: 以 byte 的格式保存, 在 bytes 中,优先显示 ASCII 字符,无法显示为 ASCII 字符的字节，用 \x## 显示
             b'input'
        (3) u: 表示 unicode 字符串(python3 默认)
         
       
```
## Python Socket

```shell
    1. socket 创建
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    2. connect 连接操作
            HOST = '127.0.0.1'  # The server's hostname or IP address
            PORT = 65432        # The port used by the server
            sock.connect((HOST, PORT))

```

## Python time
```shell
    import time
    1. time.time() 返回当前时间戳(秒数)
```

## Python binascii(二进制和 ASCII 的转换)
```shell
    1. binascii.b2a_hex(str) : 将 binary 转 ascii 并用十六进制表示
        >> str1 = b"h"  
        >> binascii.b2a_hex(str1)
        输出:
            b'68'
    2. binascii.a2b_hex(十六进制数据) : 将十六进制数据对应 ascii , 再转化为 byte 的字符串
       >>> binascii.a2b_hex(b'68')
       b'h'
```

## PyQt5
```shell
    0. 资料 https://www.cnblogs.com/linyfeng/tag/Python/
       (1) 打包; pyinstaller -F -n DeviceV2.1 --noconsole --distpath G:\Document\tool-install\智慧消防设备报警模拟工具V2 navigation_bar.py 
       (2) 
            国内源：
                    https://pypi.mirrors.ustc.edu.cn/simple/   中国科技大学源
                    https://pypi.tuna.tsinghua.edu.cn/simple　　清华大学源
    1. PyQt5 安装
        (1) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyqt5
        (2) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyqt5-tools
        (3) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyinstaller   // 将 python 打包成 exe
        (4) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple requests
        (5) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pycryptodome 修改\Python\Python37\Lib\site-packages下crypto文件夹改为Crypto
        (6) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple xlrd
        (7) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple paho-mqtt
        (8) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple stomp.py
        (9) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple paramiko
        (10) pip install -i https://pypi.tuna.tsinghua.edu.cn/simple psycopg2
    2. 配置
        (1) 配置 python.exe 的路劲
                setting -> project Interpreter 选择 pip install 的路劲
        (2) 配置 Qt Designer 和 PyUIC 的信息
                setting -> Tools -> External Tools
                Qt Designer:
                     新增 Name：可自己定义(Qt Designer)
                     Program：指向上述安装 PyQt5-tools 里面的 designer.exe (D:\install_soft\Lib\site-packages\pyqt5_tools\Qt\bin\designer.exe)
                     work directory：使用变量 $FileDir$ (D:\install_soft\Lib\site-packages\pyqt5_tools\Qt\bin)
                     
                
                PyUIC(将 Qt 界面转换成 py 代码):
                    新增 Name：可自己定义(Qt Designer)
                    Program：指向 python.exe (D:\install_soft\python.exe)
                    Arguments: -m PyQt5.uic.pyuic $FileName$ -o $FileNameWithoutExtension$.py
                    work directory：使用项目工程路劲(E:\项目\工具\python模拟软件\device_03_19\device)
                    
    3. Qt Designer基本控件介绍
        (1) 显示控件(Display Widgets)
                a. Lable：文本标签,显示文本，可以用来标记控件
                b. Text Browser： 显示文本控件, 用于后台命令执行结果显示, self.user_textBrowser.setText("xxxx")
        
        (2) 输入控件(Input Widgets), 提供与用户输入交互
                a. Line Edit：单行文本框，输入单行字符串。控件对象常用函数为Text() 返回文本框内容，用于获取输入。setText() 用于设置文本框显示。
                b. Text Edit：多行文本框，输入多行字符串。控件对象常用函数同Line Edit控件。
                c. Combo Box：下拉框列表。用于输入指定枚举值。
                
        (3) 控件按钮(Button)，供用户选择与执行 
                a.Push Button：命令按钮, 常见的确认、取消、关闭等按钮就是这个控件。clicked 信号一定要记住。 
                              clicked 信号就是指鼠标左键按下然后释放时会发送信号，从而触发相应操作。
                b.Radio Button：单选框按钮。
                c.Check Box：多选框按钮。
                
        (4) 其他
                a. 可以提前查看界面效果, [Form] -> [Prview]
                
    4. 布局管理(Layouts)
        (1) Horizontally layout(水平布局) : 被选中的控件在水平方向上从左到右排列
        (2) Vertically layout(垂直布局) : 被选中的控件在垂直方向上依次排列
        (3) Form layout(表单布局): 以 2 列的形式布局在表单中. 左列包含标签(label), 右列包含输入控件
        (4) Grid layout(网格布局): 将窗口分隔成行和列的网格来进行排列, 被选中组合的控件以网格的形式排列
        
    5.内置信号和槽
        (1) 按钮是常用的触发动作请求的方式, 用来与用户进行交互操作. 常见的按钮包括 QPushButton、QRadioButton 和 QCheckBox.
            这些按钮都继承自 QAbstractButton 类, QAbstractButton提供的信号包括：
                Clicked：鼠标左键点击按钮并释放触发该信号(最常用)
                Pressed：鼠标左键按下时触发该信号
                Released：鼠标左键释放时触发该信号
                Toggled：控件标记状态发生改变时触发该信号。
                
    . 界面与业务逻辑分离实现
            (1) 通过创建主程序调用界面文件方式实现
            (2) 控件与代码函数的映射关系是通过 Qt Designer 中 [Signal/Slot Editor] 进行添加.
            
    . 其他
        (1) 弹出消息提示框, QMessageBox.information(self, "信息提示框", "OK,内置信号与自定义槽函数！")
        
```
