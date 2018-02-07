# HTTP 概要

``` shell
    1.URI概念(Uniform Resource Identifier)：统一资源标识符,是一个用于标识某一互联网资源名称的字符串,URI的最常见的形式是
    统一资源定位符（URL）,经常指定为非正式的网址. URI可被视为定位符（URL）,名称（URN）或两者兼备.
    统一资源名（URN）如同一个人的名称,而统一资源定位符（URL）代表一个人的住址。换言之,URN定义某事物的身份,而URL提供查找该事物的方法.
    在类Unix操作系统中,一个典型的URL地址可能是一个文件目录,例如file:///home/username/RomeoAndJuliet.pdf。该URL标识出存储于
    本地硬盘中的电子书文件。
    
    通用URI的格式如下：
        scheme:[//[user[:password]@]host[:port]][/path][?query][#fragment]
        
                        hierarchical part
            ┌───────────────────┴─────────────────────┐
                        authority               path
            ┌───────────────┴───────────────┐┌───┴────┐
      abc://username:password@example.com:123/path/data?key=value&key2=value2#fragid1
      └┬┘   └───────┬───────┘ └────┬────┘ └┬┘           └─────────┬─────────┘ └──┬──┘
    scheme  user information     host     port                  query         fragment
    
    scheme:协议（或称为服务方式）web服务器中通常用的就是HTTP和HTTPS，但它还可以是gopher、wais、ftp、mailto
    
    path?query_string#anchor就是我们上面说的第三部分,path指定资源在服务器上的路径（注意：像这种archive/2010/05/18/1738301.html，
    不一定就是web服务器上的绝对路径，而是经过URL重写之后的路径，但不管怎么样说，它还是唯一标识了资源在服务器上的路径）
    后面的query_string包含传递给web应用程序（如CGI）的数据.查询字符串以键/值对的形式,并且每个键值对之间用&隔开,
    如userId=skynet&password=123456；最后当使用HTTP，#anchor表示web页面的某一个部分。
    
    2.web服务器
        web服务器的主要功能就是传送web页面给clients.这意味着,传送HTML文档和其它包含在文档中的内容,
        诸如images、style sheets、JavaScripts.client通常是一个web浏览器或web爬虫,使用HTTP发起一个指定资源的请求,
        web服务器用指定的内容响应请求,或当不能做指定请求时返回一个错误消息.请求的资源通常是web服务器的辅助存储器上的一个实际文件，
        但是这不是必须，取决于web服务器的实现.
        虽然web服务器的主要功能是提供内容，但一个完整的HTTP实现还包括接收来自client的内容。这个功能用于提交web表单,包括上载文件.
        
    3.web服务器怎样提供服务
        当用户通过点击一个超链接或在浏览器的地址栏中输入一个URL浏览一个web站点。但是同一个站点如何同时在网络上的不同计算机上显示的呢？
        以我博客的主页为例,当你在浏览器的地址栏中输入http://home.cnblogs.com/skynet/时,通过一个Internet连接,通过将域名转换为ip地址,
        然后定位到博客园服务器,你的浏览器初始化一个与博客园web服务器的连接.web服务器上存储了我的博客里所有的资源，
        如我写的每篇文章、文章中用到的图片、还有博客模板中用到的css、脚本等等。
        一旦连接建立，浏览器使用HTTP从web服务器请求数据,服务器传输数据给你的浏览器。浏览器接着转换和格式化数据显示到你的浏览器中。
        类似的，web服务器可以同时发生文件到多个client，允许多个client同时浏览同一页面.
        
    4.客户端如何唯一标识web服务器的资源
        
        
```