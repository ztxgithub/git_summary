# c++ libcurl 

## 使用

```shell
(1) libcurl 是一个跨平台的网络协议库, 支持 HTTP POST, HTTP PUT, FTP 上传, HTTP 基本表单上传，代理，cookies,和用户认证.
(2) 在基于 LibCurl 的程序里，主要采用 callback function (回调函数) 的形式完成传输任务，在启动传输前设置好各类参数和回调函数，
    当满足条件时 libcurl 将调用用户的回调函数实现特定功能。流程： 
                 1. 调用 curl_global_init() 初始化 libcurl 
                 2. 调用 curl_easy_init() 函数得到 easy interface 型指针 
                 3. 调用 curl_easy_setopt() 设置传输选项 
                 4. 根据 curl_easy_setopt() 设置的传输选项，实现回调函数以完成用户特定任务 
                 5. 调用 curl_easy_perform() 函数完成传输任务 
                 6. 调用 curl_easy_cleanup() 释放内存 
            在整过过程中设置 curl_easy_setopt() 参数是最关键的，几乎所有的 libcurl 程序都要使用它
```

## 接口使用

```shell
    1. CURLcode curl_global_init(long flags);
       描述:
            这个函数只能用一次(其实在调用 curl_global_cleanup 函数后仍然可再用) 
            如果这个函数在 curl_easy_init 函数调用时还没调用，它将由 libcurl 库自动调用，在多线程下最好主动调用该函数以防止
            在线程中　curl_easy_init　时多次调用。 
       注意：虽然 libcurl 是线程安全的，但 curl_global_init 是不能保证线程安全的，所以不要在每个线程中都调用 curl_global_init,
            应该将该函数的调用放在主线程中
       参数:
            flags:
                CURL_GLOBAL_ALL/CURL_GLOBAL_DEFAULT //初始化所有的可能的调用
                CURL_GLOBAL_SSL //初始化支持安全套接字层 
                CURL_GLOBAL_WIN32 //初始化 win32 套接字库 
                CURL_GLOBAL_NOTHING //没有额外的初始化
                
    2. void curl_global_cleanup(void);
        描述:
            在结束 libcurl 使用的时候，用来对 curl_global_init 做的工作清理。类似于close的函数。
       注意：虽然 libcurl 是线程安全的，但 curl_global_cleanup 是不能保证线程安全的，所以不要在每个线程中都调用 curl_global_cleanup,
            应该将该函数的调用放在主线程中
       参数:
       
    3. char *curl_version( );
        描述:
            打印当前 libcurl 库的版本
            
    4. CURL *curl_easy_init( );
        描述:
            用来初始化一个 CURL 的指针(像返回 FILE 类型的指针). 相应的在调用结束时要用 curl_easy_cleanup 函数清理. 
            一般 curl_easy_init 意味着一个会话的开始. 它会返回一个 easy_handle(CURL*对象), 一般都用在easy系列的函数中
      
    5. void curl_easy_cleanup(CURL *handle);
        描述:
            结束一个会话.与 curl_easy_init 配合使用
            
    6. CURLcode curl_easy_setopt(CURL *handle, CURLoption option, parameter);
        描述:
            通知 curl 库程序将有如何的行为. 比如要查看一个网页的 html 代码等.(这个函数有些像 ioctl 函数)
        参数:
            handle: CURL 类型的指针
            option: CURLoption 类型的选项.(在 curl.h 库里有定义, man 也可以查看到) 
                                                              
            parameter: 这个参数既可以是个函数的指针,也可以是某个对象的指针,也可以是个 long 型的变量,这取决于第二个参数
            
    7. CURLcode curl_easy_perform(CURL *handle);
        描述:
            初始化 CURL 类型的指针以及 curl_easy_setopt 完成后调用
        返回值:
                CURLE_OK : 正常
                CURLE_UNSUPPORTED_PROTOCOL: 不支持的协议，由 URL 的头部指定
                CURLE_COULDNT_CONNECT: 不能连接到 remote 主机或者代理 
                CURLE_REMOTE_ACCESS_DENIED: 访问被拒绝 
                CURLE_HTTP_RETURNED_ERROR: Http 返回错误
                
                通过 const char *curl_easy_strerror(CURLcode errornum) 获取详细信息
                
    8. CURLcode curl_easy_getinfo(CURL *curl, CURLINFO info, ... ); 从响应头获取对应的参数
           info 参数:
                      a. CURLINFO_RESPONSE_CODE : 获取应答码
                           curl_easy_getinfo(pHttpHandle, CURLINFO_RESPONSE_CODE, &http_code);
                      b. CURLINFO_HEADER_SIZE : 头大小
                      c. CURLINFO_COOKIELIST : cookies 列表
                      d. CURLINFO_EFFECTIVE_URL: 取得本最终生效的 URL，如果有多次重定向，获取的值为最后一次 URL，
                                                要求第 3 个参数是个 char 型的指针
                      e. CURLINFO_SIZE_DOWNLOAD 获取下载字节数，要求第 3 个参数是个 double 型的指针。
                         这个字节数只能反映最近一次的下载
                      f. CURLINFO_SPEED_DOWNLOAD 获取平均下载数据，该选项要求传递一个 double 型参数指针，这个速度不是即时速度，
                         而是下载完成后的速度，单位是 字节/秒
                      g. CURLINFO_TOTAL_TIME 获取传输总耗时，要求传递一个 double 指针到函数中，这个总的时间里包括了域名解析，
                         以及 TCP 连接过程中所需要的时间
                      h. CURLINFO_CONTENT_TYPE 该选项获得 HTTP 中从服务器端收到的头部中的 Content-Type 信息
                      i. CURLINFO_CONTENT_LENGTH_DOWNLOAD 获取头部 content-length，要求第 3 个参数是个 double
                         型的指针。如果文件大小无法获取，那么函数返回值为 -1
            
```

## curl_easy_setopt 选项介绍

```shell
            1. CURLOPT_URL : 设置访问的 url, 这个一定要进行设置，
            2. CURLOPT_WRITEFUNCTION, CURLOPT_WRITEDATA 配合使用(用于 http 文件的下载,下载到本地)，也可以是 http 请求的应答
               body 的数据
                    CURLOPT_WRITEFUNCTION, 对应的　parameter　是 
                        size_t function( void *ptr, size_t size, size_t nmemb, void *stream); 函数将在 libcurl 
                        接收到数据后被调用, 因此函数多做数据保存的功能，如处理下载文件.
                    CURLOPT_WRITEDATA 传递指针给 libcurl，该指针表明 CURLOPT_WRITEFUNCTION 函数的 stream 指针的来源
                    如果没有通过 CURLOPT_WRITEFUNCTION 属性给 easy handle 设置回调函数，libcurl 
                    会提供一个默认的回调函数,只是简单的将接收到的数据打印到标准输出。也可以通过 CURLOPT_WRITEDATA
                    属性给默认回调函数传递一个已经打开的文件指针，用于将数据输出到文件里
                    
                例如:
                        curl_easy_setopt(curl, CURLOPT_URL, "sftp://user@server/home/user/file.txt");
                        
                        /* Define our callback to get called when there's data to be written */
                        curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, my_fwrite);
                        
                        /* Set a pointer to our struct to pass to the callback */
                        curl_easy_setopt(curl, CURLOPT_WRITEDATA, &ftpfile);
                        
            3. CURLOPT_HEADERFUNCTION，CURLOPT_HEADERDATA 配合使用
                    回调函数原型为 size_t function( void *ptr, size_t size,size_t nmemb, void *stream); 
                    libcurl 一旦接收到 http 头部数据后将调用该函数
                    
                 例如:
                        curl_easy_setopt(curl, CURLOPT_HEADERFUNCTION, write_response);
                        curl_easy_setopt(curl, CURLOPT_HEADERDATA, respfile);
                        
            4. CURLOPT_READFUNCTION, CURLOPT_READDATA (读取本地的数据传输给远程的主机)
            
            5. CURLOPT_NOPROGRESS, CURLOPT_PROGRESSFUNCTION, CURLOPT_PROGRESSDATA 跟数据传输进度相关的参数.
               CURLOPT_PROGRESSFUNCTION 指定的函数正常情况下每秒被 libcurl 调用一次, 为了使 CURLOPT_PROGRESSFUNCTION 被调用,
               CURLOPT_NOPROGRESS 必须被设置为 false, CURLOPT_PROGRESSDATA 指定的参数将作为 CURLOPT_PROGRESSFUNCTION
               指定函数的第一个参数
               
            6. 单位为秒
               CURLOPT_TIMEOUT 由于设置传输时间
               CURLOPT_CONNECTIONTIMEOUT 设置连接等待时间, 设置为0, 则无限等待。
               
            7. CURLOPT_FOLLOWLOCATION 设置重定位 URL, 设置为非0, 响应头信息 Location, 即 curl 会自己处理 302 等重定向
               CURLOPT_MAXREDIRS 指定 HTTP 重定向的最大次数
            
            8. 断点续传相关设置
                   (1) CURLOPT_RANGE 指定 char *参数传递给 libcurl, 用于指明 http 域的 RANGE 头域，例如： 
                            表示头 500 个字节：bytes=0-499 
                            表示第二个 500 字节：bytes=500-999 
                            表示最后 500 个字节：bytes=-500 
                            表示 500 字节以后的范围：bytes=500- 
                            第一个和最后一个字节：bytes=0-0,-1 
                            同时指定几个范围：bytes=500-600,601-999
                   (2) CURLOPT_RESUME_FROM 传递一个 long 参数给 libcurl，指定你希望开始传递的偏移量。CURLOPT_RESUME_FROM
                       大小限制为 2G，超过可以使用 CURLOPT_RESUME_FROM_LARGE
                   
            9. CURLOPT_USERAGENT : 该选项要求传递一个以 ‘\0’ 结尾的字符串指针，这个字符串用来在向服务器请求时发送 HTTP 头部中的 
                                   User-Agent 信息，有些服务器是需要检测这个信息的，如果没有设置 User-Agent，那么服务器拒绝请求。
                                   设置后，可以骗过服务器对此的检查
                                   
            10. CURLOPT_POSTFIELDS, CURLOPT_POSTFIELDSIZE
                (1) CURLOPT_POSTFIELDS 传递一个作为 HTTP “POST”操作的所有数据的字符串
                (2) CURLOPT_POSTFIELDSIZE 设置 POST 字节大小
                
            11. CURLOPT_NOBODY
                    设置该属性告诉 libcurl 想发起一个 HEAD 请求,想查询服务器某种资源的状态，比如某个文件的属性：修改时间，大小等等，
                    但是并不需要具体得到该文件，这时仅需要 HEAD 请求。
                    
            12. CURLOPT_MAX_RECV_SPEED_LARGE，CURLOPT_MAX_SEND_SPEED_LARGE
                    限速相关设置
                    1）CURLOPT_MAX_RECV_SPEED_LARGE，指定下载过程中最大速度，单位bytes/s。
                    2）CURLOPT_MAX_SEND_SPEED_LARG，指定上传过程中最大速度，单位bytes/s。
                    
            13. CURLOPT_FORBID_REUSE , CURLOPT_FRESH_CONNEC
                如果不使用长连接, 需要设置这两个属性
                1）CURLOPT_FORBID_REUSE 设置为1，在完成交互以后强迫断开连接，不重用。
                2）CURLOPT_FRESH_CONNECT设置为1，强制获取一个新的连接，替代缓存中的连接
                
            14. CURLOPT_NOSIGNAL
                    当多个线程都使用超时处理的时候，同时主线程中有 sleep 或是 wait 等操作。如果不设置这个选项，
                    libcurl 将会发信号打断这个 wait 从而可能导致程序 crash。 在多线程处理场景下使用超时选项时，
                    会忽略 signals 对应的处理函数
                    
            15. CURLOPT_IPRESOLVE
                指定 libcurl 域名解析模式。支持的选项有：
                1）CURL_IPRESOLVE_WHATEVER：默认值，相当于 PF_UNSPEC，支持IPv4/v6，具体以哪个优先需要看 libc 底层实现，
                                           Android 中默认以 IPv6 优先，当 IPv6 栈无法使用时，libcurl 会用 IPv4。
                2）CURL_IPRESOLVE_V4：.仅请求 A 记录，即只解析为 IPv4 地址。
                3）CURL_IPRESOLVE_V6： 仅请求 AAAA 记录，即只解析为 IPv6 地址。
                注意：该功能生效的前提是 libcurl 支持 IPv6，需要在 curl/lib/curl_config.h配置 #define ENABLE_IPV6 1
                
            16. CURLOPT_DNS_CACHE_TIMEOUT
                设置 libcurl DNS 缓存超时时间，默认为60秒，即每 60 秒清一次 libcurl 自身保存的 DNS 缓存。
                如果设置为 0，则不适用 DNS 缓存，设置为 -1，则永远不清缓存。
                
            17. CURLOPT_DNS_USE_GLOBAL_CACHE
                让 libcurl 使用系统 DNS 缓存，默认情况下，libcurl 使用本身的 DNS 缓存。
                
            HTTPS 参数
            18. CURLOPT_SSL_VERIFYPEER
                验证 HTTPS 请求对象的合法性，就是用第三方证书机构颁发的 CA 数字证书来解密服务端返回的证书，来验证其合法性。
                可在编译时就将 CA 数字证书编译进去，也可以通过参数 CURLOPT_CAINFO 或者 CURLOPT_CAPATH 设置根证书
                如果要使用默认值为 1
            19. CURLOPT_SSL_VERIFYHOST
                该参数主要用于 https 请求时返回的证书是否与请求的域名相符合，避免被中间着篡改证书文件。默认值为2
                
                
            20. CURLOPT_FOLLOWLOCATION
                
              
              
                
```

## 使用经验

```shell
    1. 打印日志
        /* Switch on full protocol/debug output */
       curl_easy_setopt(curl, CURLOPT_VERBOSE, 1L);
       
    2. 不应该在线程之间共享同一个 libcurl handle(CURL *对象)，不管是 easy handle还是 multi handle
        一个线程每次只能使用一个 handle
        
    3. 当 libcurl 无法正常工作时, 处理这些问题可以将 CURLOPT_VERBOSE 属性设置为 1, libcurl 会输出通信过程中的一些细节。
       如果使用的是 http 协议，请求头/响应头也会被输出。将 CURLOPT_HEADER 设为1，这些头信息将出现在消息的内容中
       
    4. curl_easy_perform 是阻塞的方式进行下载的, curl_easy_perform 执行后,程序会在这里阻塞等待下载结束(成功结束或者失败结束).
       此时若正常下载一段时间后,进行网络中断, curl_easy_perform 并不会返回失败,而是阻塞整个程序卡在这里,此时即使网络连接重新恢复,
       curl_easy_perform 也无法恢复继续下载,导致整个程序出现”死机”状态.
       
       解决方案:
                下载过程中,设置超时时间为 30 秒, 30 秒后若下载未完成就重新连接进行下载(这个可解决卡死问题)
                curl_easy_setopt( curl, CURLOPT_TIMEOUT, 10 );//接收数据时超时设置，如果 10 秒内数据未接收完，直接退出
```

## http 处理
```shell
    1. 自定义请求方式(CustomRequest)
            可以设置是通过 get, post 请求, 通过 CURLOPT_CUSTOMREQUEST 来设置自定义的请求方式, libcurl 默认以 GET 方式提交请求
            curl_easy_setopt(easy_handle, CURLOPT_CUSTOMREQUEST, “POST”);
            
    2. 修改消息头
            HTTP 协议提供了消息头, 请求消息头用于告诉服务器如何处理请求；响应消息头则告诉浏览器如何处理接收到的数据。在 libcurl 中，
            你可以自由的添加 这些消息头：
            例子:
                struct curl_slist headers=NULL; / init to NULL is important */ 
                headers = curl_slist_append(headers, “Hey-server-hey: how are you?”); 
                headers = curl_slist_append(headers, “X-silly-content: yes”); 
                
                /* pass our list of custom made headers */ 
                curl_easy_setopt(easyhandle, CURLOPT_HTTPHEADER, headers); 
                
                curl_easy_perform(easyhandle); /* transfer http */ 
                curl_slist_free_all(headers); /* free the header list */ 
                
            对于已经存在的消息头，可以重新设置它的值
                headers = curl_slist_append(headers, “Accept: Agent-007”);  // 这些默认就有
                headers = curl_slist_append(headers, “Host: munged.host.line”); 　// 这些默认就有
                
            删除消息头，　对于一个已经存在的消息头，设置它的内容为空就是删除
                headers = curl_slist_append(headers, “Accept:”);
                
    3. http 应答信息
            (1) 发出 http 请求后，服务器会返回应答头信息和应答数据，如果仅仅是打印[应答头]的所有内容，则直接可以通过
                curl_easy_setopt(curl, CURLOPT_HEADERFUNCTION, 打印函数) 的方式来完成
            (2) 如果需要获取应答头中特定的信息，比如应答码、cookies列表等, 则通过 
                CURLcode curl_easy_getinfo(CURL *curl, CURLINFO info, ... );
                info 参数:
                           a. CURLINFO_RESPONSE_CODE : 获取应答码
                                curl_easy_getinfo(pHttpHandle, CURLINFO_RESPONSE_CODE, &http_code);
                           b. CURLINFO_HEADER_SIZE : 头大小
                           c. CURLINFO_COOKIELIST : cookies 列表
                           
    4. 例子:
            (1) http post 请求
            int HttpsClient::httpPost(const std::string & strUrl, std::vector<std::string>& vectorHeaders, const std::string& strBody,
             std::string & strRsp, int timeout)
            {
             CURL* pHttpHandle = curl_easy_init()
             struct curl_slist *headers = NULL;
             for (size_t i = 0; i < vectorHeaders.size(); i++)
             {
             headers = curl_slist_append(headers, vectorHeaders[i].c_str());  // 加入响应头
             }
            
                curl_easy_setopt(pHttpHandle, CURLOPT_HTTPHEADER, headers);
    
            
             if (CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_URL, strUrl.c_str()) ||  // 指定 url
             CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_POST, 1) ||    // 设置为 post 模式
             CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_POSTFIELDS, strBody.c_str()) ||  // 输入 post 内容
             CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_READFUNCTION, NULL) ||
             CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_WRITEFUNCTION, OnWriteData) ||  // 调用收到的响应内容
             CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_WRITEDATA, &strRsp) ||  // 与 OnWriteData 传入参数一一对应
             CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_NOSIGNAL, 1)) // 屏蔽任何信号值
             {
             FDRV_ERROR("[0x%08x] - HttpsClient setopt main failed!", 0x01);
             //return HPR_ERROR;
             iRetVal = HPR_PASS;
             }
            
             if (CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_CONNECTTIMEOUT, timeout) ||
             CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_TIMEOUT, timeout))
             {
             FDRV_ERROR("[0x%08x] - HttpsClient setopt timeout failed!", 0x01);
             //return HPR_ERROR;
             iRetVal = HPR_PASS;
             }
            
             int nRet = curl_easy_perform(pHttpHandle);
             if (CURLE_OK != nRet)
             {
             FDRV_ERROR("[0x%08x] - HttpsClient perform failed!", 0x01);
             //return HPR_ERROR;
             iRetVal = HPR_PASS;
             }
            
             int http_code = -1;
             if (CURLE_OK != curl_easy_getinfo(pHttpHandle, CURLINFO_RESPONSE_CODE, &http_code))  // 获取响应头的返回码
             {
             FDRV_ERROR("[0x%08x] - HttpsClient getinfo failed!", 0x01);
             //return HPR_ERROR;
             iRetVal = HPR_PASS;
             }
            
             if (NULL != headers)
             {
             curl_slist_free_all(headers);
             }
             if (NULL != pHttpHandle)
             {
             curl_easy_cleanup(pHttpHandle);
             }
           
            }
            
            (2) https post
                     if (!m_sslCaiInfo.empty())//bidirection cert
                     {
                     if (CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_SSL_VERIFYPEER, 1) ||
                     CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_SSL_VERIFYHOST, 2) ||
                     CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_CAINFO, m_sslCaiInfo.c_str()))
                     {
                     FDRV_ERROR("[0x%08x] - HttpsClient setopt host ssl caiinfo failed!", 0x01);
                     //return HPR_ERROR;
                     iRetVal = HPR_PASS;
                     }
                     }
                     else
                     {
                     if (CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_SSL_VERIFYPEER, 0) ||  // 跳过 CA 证书认证
                     CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_SSL_VERIFYHOST, 0))
                     {
                     FDRV_ERROR("[0x%08x] - HttpsClient setopt host ssl verify failed!", 0x01);
                     //return HPR_ERROR;
                     iRetVal = HPR_PASS;
                     }
                     }
                    
                     if (!m_sslCrt.empty())
                     {
                     if (CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_SSLCERT, m_sslCrt.c_str()) ||
                     CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_SSLCERTTYPE, m_sslCrtType.c_str()))
                     {
                     FDRV_ERROR("[0x%08x] - HttpsClient setopt cert failed!", 0x01);
                     //return HPR_ERROR;
                     iRetVal = HPR_PASS;
                     }
                     }
                    
                     if (!m_sslkey.empty())
                     {
                     if (CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_SSLKEY, m_sslkey.c_str()) ||
                     CURLE_OK != curl_easy_setopt(pHttpHandle, CURLOPT_SSLKEYTYPE, m_sslKeyType.c_str()))
                     {
                     FDRV_ERROR("[0x%08x] - HttpsClient setopt key failed!", 0x01);
                     //return HPR_ERROR;
                     iRetVal = HPR_PASS;
                     }
                     }
                     
            (3) ssl 证书使用
                    
                    if(pCaPath)
                    {
                    
                        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 1L);//openssl　编译时使用　curl官网或者　firefox导出的第三方根证书文件 
                
                        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 2L);
                
                        curl_easy_setopt(curl, CURLOPT_CAINFO, pCaPath);/*pCaPath为证书路径 */
                    }
                    else
                    { 　　　　
                    
                　　　　 curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 1L);
                
                　　　　 curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 2L);
                
                　　　　 curl_easy_setopt(curl, CURLOPT_CAINFO, "cacert.pem");//cacert.pem为curl官网下载的根证书文件
                    
                    }
            
         
            
            
```

## 密码的设置

```shell
    1. 一些协议支持在 URL中 直接指定用户名和密码，类似于： protocol://user:password@example.com/path/。libcurl 能正确的识别这种
      URL 中的用户名与密码并执行相应的操作。如果你提供的用户名和密码中有特殊字符，首先应该对其进行 URL 编码
    2. 也可以通过 CURLOPT_USERPWD 属性来设置用户名与密码。参数是格式如 “user:password ”的字符串：
            curl_easy_setopt(easy_handle, CURLOPT_USERPWD, "user_name:password"); 
    3. 有时候在访问代理服务器的时候，可能每次都要求提供用户名和密码进行用户身份验证。这种情况下，libcurl提供了另 一个属性
       CURLOPT_PROXYUSERPWD：
       curl_easy_setopt(easy_handle, CURLOPT_PROXYUSERPWD, "user_name:password");
    4. 在使用 SSL 时, 可能需要提供一个私钥用于数据安全传输，通过 CURLOPT_KEYPASSWD 来设置私钥：
        curl_easy_setopt(easy_handle, CURLOPT_KEYPASSWD, "keypassword");
```

## 基础知识

```shell
    1. Post 请求的两种编码格式：application/x-www-form-urlencoded 和 multipart/form-data
       在 http 请求头
       　Content-Type：application/x-www-form-urlencoded
       
       (1) application/x-www-form-urlencoded(默认)
            如果是　post , 在 postman 中　Body -> x-www-form-urlencoded -> key 赋值为 appKey, Value 赋值为 123456
            则在实际的报文中
                POST /api/lapp/token/get HTTP/1.1
                Content-Type: application/x-www-form-urlencoded
                cache-control: no-cache
                Postman-Token: 18285c07-0734-493d-a604-cb75769075e5
                User-Agent: PostmanRuntime/7.1.5
                Accept: */*
                Host: open.ys7.com
                accept-encoding: gzip, deflate
                content-length: 39
                Connection: keep-alive
                
                appKey=123456
                
            也可以在 postman 中右上部分的 "code"
            
    2.
        POST 请求的两种编码格式：application/x-www-urlencoded 是浏览器默认的编码格式，用于键值对参数，参数之间用 & 间隔；
                              multipart/form-data 常用于文件等二进制，也可用于键值对参数，
                              最后连接成一串字符传输(参考Java OK HTTP)。除了这两个编码格式，
                              还有 application/json 也经常使用。
     
             POST /api/lapp/token/get HTTP/1.1
                             Content-Type: application/json
                             cache-control: no-cache
                             Postman-Token: 18285c07-0734-493d-a604-cb75769075e5
                             User-Agent: PostmanRuntime/7.1.5
                             Accept: */*
                             Host: open.ys7.com
                             accept-encoding: gzip, deflate
                             content-length: 39
                             Connection: keep-alive
                             
                             {
                                "xxxx", "xxxx"
                             }
                             
    3. https
            (1) http 传输过程都是明文传输，不安全， https 则会对报文进行加密．
            (2) https 传输流程:
                    第一步：进行非对称加密过程
                           即双方通过公钥，私钥和 CA 证书来获得会话秘钥．例如刚开始客户端先尝试与服务器
                           建立安全连接，此时服务器将服务器的公钥和 CA 证书传给客户端，这个时候客户端先验证 CA 证书的合法性，
                           如果合法则随机生成一对客户端的公钥,私钥以及客户端会话密钥，然后用服务器公钥加密客户端公钥和客户端会话密钥，
                           再传给服务端，服务端用自己的私钥可以成功解密出客户端的公钥和客户端的会话密钥．这个时候服务端再用得到的
                           客户端的公钥加密服务器的会话密钥, 最后客户端再用自己的私钥解密出服务器的会话密钥．
                    第二步: 对称加密过程(正常的数据传输)
                            客户端将数据用自己的密钥加密传给服务端，然后在服务端用客户端的密钥解密得出数据．
                            服务端用自己的会话密钥加密传给客户端，然后客户端用服务端的会话密钥解密．
```

## 参考资料
```shell
    1. https://blog.csdn.net/c1194758555/article/details/81027225
```