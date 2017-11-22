# gsoap

## 生成gsoap客户端基本流程

``` shell

    1.从Web服务提供者处获取Web Service的WSDL文件，通常是一个URL，
          如：http://www.cs.fsu.edu/~engelen/calc.wsdl
          
    2． 使用gSoap工具wsdl2h，根据WSDL生成一个C/C++语法结构的头文件，这一步将会得到一个头文件，如：calc.h，
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
      
    4.还需要
        gsoap目录下的stdsoap2.h和stdsoap2.cpp文件
     
     
```

## soap(Simple Object Access Protocol)

``` shell

    如果我们要调用远程对象的方法,就必定要告诉对方,我们要调用的是一个什么方法,以及这个方法的参数的值等等,然后对方把数据返回给我们.
    这其中就涉及到两个问题：1、数据如何在网络上传输。2、如何表示数据？用什么格式去表示函数以及它的参数等等.
    
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
     
```


