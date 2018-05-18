# gsoap

## gsoap 概要

``` shell

    gSOAP编译工具提供了一个基于SOAP/XML 的C/C++ 语言实现,从而让C/C++语言开发WebService客户或服务端的程序工作变得轻松了很多.
    gSOAP利用编译器技术提供了一组透明化的SOAP API,并将与开发无关的SOAP实现细节相关的内容对用户隐藏起来.
    gSOAP的编译器能够自动的将用户定义的本地化的C或C++数据类型转变为符合XML语法的数据结构,反之亦然。
    只用一组简单的API就将用户从SOAP细节实现工作中解脱了出来，从则可以专注于应用程序逻辑的实现工作
    
```
## 生成gsoap客户端基本流程

``` shell

    1.从Web服务提供者处获取Web Service的WSDL文件，通常是一个URL,
          如：http://www.cs.fsu.edu/~engelen/calc.wsdl
          
    2． 使用gSoap工具wsdl2h,根据WSDL生成一个C/C++语法结构的头文件,这一步将会得到一个头文件,如：calc.h,
        该步的目的：实现WSDL文件到.h文件的数据映射。
        
    　　　如：wsdl2h -s -o calc.h http://www.cs.fsu.edu/~engelen/calc.wsdl
    
            选项
                -o 文件名，指定输出头文件
                -n 名空间前缀 代替默认的ns
                -c 产生纯C代码，否则是C++代码
                -s 不要使用STL代码(如果要使用stl则将gsoap/import/stlvector.h文件复制到当前目录)
                -t 文件名，指定type map文件，默认为typemap.dat
                -e 禁止为enum成员加上名空间前缀
    
    3． 使用gSoap的预编译器soapcpp2，根据上一步得到的头文件来生成存根文件（soapStub.h）和客户端代码框架, 这一步将会得到几个
        如：calc.nsmap、soapC.cpp、soapH.h、soapStub.h、soapcalcProxy.cpp、soapcalcProxy.h这一步的目的：生成相应的底层通信代码
        
      　　如：soapcpp2 -i -x -C -L calc.h
      
                常用参数：
                    -C 仅生成客户端代码
                    -S 仅生成服务器端代码
                    -L 不要产生soapClientLib.c和soapServerLib.c文件
                    -c 产生纯C代码，否则是C++代码(与头文件有关)
                    -I 指定import路径（见上文）
                    -x 不要产生XML示例文件
                    -i 生成C++封装(代理)，客户端为xxxxProxy.h(.cpp)，服务器端为xxxxService.h(.cpp)
                    -d < path >: 将代码生成在 < path > 下
                    -I < path >: 包含其他文件时使用,指明 < path > (多个的话,用`:'分割),相当于#import ,
                                 该路径一般是gSOAP目录下的import目录,该目录下有一堆文件供soapcpp2生成代码时使用
                    
          各个文件代表的含义：
            soapStub.h：根据输入的.h文件生成的数据定义文件，一般我们不直接引用它
            soapH.h和soapC.cpp：客户端和服务器端应包含该头文件,它包含了soapStub.h.针对soapStub.h中的数据类型,
                               cpp文件实现了序列化、反序列化方法
                               
            soapXXXProxy.h和soapXXXProxy.cpp:这两个文件用于客户端,是客户端调用webservice的框架文件,
                                             我们的代码主要在此实现或从它继承.
                                             
            soapXXXService.h和soapXXXService.cpp:这两个文件用于服务器端,是服务器端实现webservice的框架文件,
                                                 我们的代码主要在此实现或从它继承
                                                 
      
    4.还需要
        gsoap目录下的stdsoap2.h和stdsoap2.cpp文件
```

## gsoap 的应用
    
``` shell

    1.soap_destroy(struct soap *soap):释放所有动态分配的C++类(soap_new"分配的类)，必须在soap_end()之前调用。
    2.soap_end(struct soap *soap):释放所有存储临时数据(临时数据是指那些在序列化/反序列化过程中创建的例如hash表等用来帮助解析、
                                   跟踪xml的数据)和反序列化数据(反序列化数据是指在接收soap过程中产生的用
                                   malloc和new分配空间存储的数据)中除类之外的空间（soap_malloc的数据也属于反序列化数据）
    3.soap_done(struct soap *soap):Detach soap结构（即初始化化soap结构）
    4.soap_free(struct soap *soap):Detach 且释放soap结构
    
    前两个函数常用于每个调用完成后，后两个函数常用于不需要这个service前。
    
```
    

## soap(Simple Object Access Protocol)

``` shell

    如果我们要调用远程对象的方法,就必定要告诉对方,我们要调用的是一个什么方法,以及这个方法的参数的值等等,然后对方把数据返回给我们.
    这其中就涉及到两个问题：1、数据如何在网络上传输。2、如何表示数据？用什么格式去表示函数以及它的参数等等.
    
    SOAP提供“请求”的规范：你想让人家办事，总得告诉人家你想干什么吧，SOAP就是定义这个“请求”的格式的，
    按照SOAP定义的“请求”格式“书写”请求就可以保证Web Service能够正确的解读你想让它干什么以及你为它提供了什么参数。
    在这个请求中，你需要描述的主要问题有：向哪个Web Service发送请求，请求的参数类型、参数值、返回值类型。
    这些都“填写”完毕，也就完成了符合SOAP规范的SOAP消息
    
    1.SOAP的传输协议
        SOAP的传输协议使用的就是HTTP协议.只不过HTTP传输的内容是HTML文本，而SOAP协议传输的是SOAP的数据
        
        POST /WebServices/WeatherWebService.asmx HTTP/1.1
        
        User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; MS Web Services Client Protocol 2.0.50727.3603)
        Content-Type: text/xml; charset=utf-8
        SOAPAction: "http://WebXml.com.cn/getSupportCity"
        Host: www.webxml.com.cn
        Content-Length: 348
        Expect: 100-continue
        Connection: Keep-Alive
        
        <?xml version="1.0" encoding="utf-8"?>
        <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" 
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
                xmlns:xsd="http://www.w3.org/2001/XMLSchema">
            <soap:Body>
                <getSupportCity xmlns="http://WebXml.com.cn/">
                <byProvinceName>广东</byProvinceName>
                </getSupportCity>
            </soap:Body>
        </soap:Envelope>
     
     一个SOAP请求其实就是一个HTTP请求,如果请求头中有SOAPAction这一段，那么请求会被当作SOAP的内容来处理而不会当作HTML来解析.
     可以用上面指定SOAPAction头来表示内容是SOAP的内容,也可以指定 Content-Type: application/soap+xml 来表示内容是SOAP的内容.
     SOAP请求中最后的那段XML数据，这个就是请求的具体内容,
     
     这个就是SOAP规定的请求的数据格式:
        a) Envelope
                SOAP的请求内容必须以Envelope做为根节点.
                (1)xmlns:soap: Envelope的schema的相关定义
                    xmlns:soap="http://www.w3.org/2001/12/soap-envelope",不能修改,否则会出错.
                    http://www.w3.org/2001/12/soap-envelope里面有Envelope的schema的相关定义  
                (2)soap:encodingStyle:指定了数据元素的类型
                        soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding"
                        
        b) Header
        
            这个是可选的,如果需要添加Header元素,那么它必须是Envelope的第一个元素。
            Header的内容并没有严格的限制,我们可以自己添加一些和应用程序相关的内容，但是客户端一定要记得处理这些Header元素,
            可以加上mustUnderstand强制进行处理.
            
        c) Body
            这个就是请求的主题内容了,请求什么函数,参数是什么类型等等都在这里面指定. 
            用标签表示一个函数,然后用子元素表示它的参数.
```

## WSDL(Web Services Description Language)

## 生成gsoap客户端基本流程

``` shell
    WSDL是用来描述WebService的,它用XML的格式描述了WebService有哪些方法、参数类型、访问路径等等。
    我们要使用一个WebService肯定首先要获取它的WSDL(可以是url),对于C/C++来讲,可以用gsoap库
    
    WSDL提供“能办的事的说明”：我想帮你的忙，但是我要告诉你我都能干什么，以及干这些事情需要的参数类型
    
    一个WSDL文档由四部分组成：
            1.types
                指定了WebService用到的所有数据类型,作为下面message中数据包引用的类型；
                
            2.message
                message定义了通讯的数据包（通讯一般是双向的，因此，一个调用过程有两个message定义，
                一般情况下用<methodname>in和<methodname>out分别表示进入和发出服务器的数据包）,
                该数据包引用了上面types中的最终的复杂类型；
                
            3.portType
                包含了若干operation（对应于代码实现中的method）,定义了该operation进出的message；
                指出了这个WebService所有支持的操作，就是说有哪些方法可供调用。
                
            4.binding
                binding是对上面operation的网络描述,也就是将operation与某一可以用soap形式进行网络访问的方式“绑定”在一起,
                binding指明了每个method的访问地址和方式
                
            　　soap12:binding元素的transport指明传输协议，这里是http协议。
            
            　　operation 指明要暴露给外界调用的操作。
            
            　　use属性指定输入输出的编码方式，这里没有指定编码。
            
            5.services
            　指定服务的一些信息,主要是指定服务的访问路径。
```
