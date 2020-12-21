# htrace 使用
```shell
    1.　基本的追踪埋点操作分为 3 种, 开始结束业务, 开始结束本地过程, 开始结束远程过程
    2.  开始结束业务
            (1) 成对出现
                htrace_trace_start();  // 开始一个业务, 调用 htrace_trace_start 后，会生成全局唯一的 traceid 用于标识此业务
                
                .......
                
                
                htrace_trace_end();    // 结束一个业务, 调用 htrace_trace_end 函数会导致此 traceid 与 spanid 被清理并释放线程存储中的追踪上下文

            (2) 由于 htrace_trace_start 与 htrace_trace_end 之间也被当做一个过程对待，因此这里同时也会生成一个全局唯一的
                spanid 用于标识此过程（一个 spanid 对应一个过程）
                
            (3) 这里会新生成一个追踪上下文（物理位置在线程存储空间中）用于存储 traceid 与 spanid
            
    3.  开始结束远程过程
            (1) 开始结束远程过程的接口
                   a. 能产生打印输出的 4 个接口
                        主动发起的是客户端
                        htrace_client_send
                        htrace_client_receive
                        
                        
                        htrace_server_receive
                        htrace_server_send
                   b. 用于提取、存入追踪信息至上下文的 4 个接口(这 4 个接口会改变上下文状态(spanid, traceid))
                        htrace_client_convert_contextintls_to_msgstring
                        htrace_client_convert_msgstring_to_contextintls
                        htrace_server_convert_msgstring_to_contextintls
                        htrace_server_convert_contextintls_to_msgstring
                        
            (2) 实例代码
                   (1) 作为客户端主动发起的
                   
                            htrace_trace_start();  // 开始一个业务
                      
                            htrace_client_send("获取短URL");  // 打印调用链日志
                            // 将当前的上下文保存到 strTraceId_msg 和 strSpanId_msg
                            htrace_client_convert_contextintls_to_msgstring(strTraceId_msg, strSpanId_msg);
                            // 将上下文状态信息 strTraceId_msg 和 strSpanId_msg 传给对端
                            sendToVnsc(strTraceId_msg, strSpanId_msg, "payload"); 
                            
                
                            struResault = recvFromVnsc(); // （伪代码）
                            // 将 strTraceId_msg 和 strSpanId_msg 重新赋值到上下文
                            htrace_client_convert_msgstring_to_contextintls(strTraceId_msg, strSpanId_msg);
                            htrace_client_receive(struResault.bSucc, struResault.strErrorCode, struResault.strNotation);
                            
                            htrace_client_receive();  // 打印调用链日志
                            
                   (2) 作为服务端被动接受的
                   
                            // 处理收到客户端的信息, 拿到 rev_strTraceId_msg 和 rev_strSpanId_msg
                            struResponse = recvFromClient();
                            // 将收到的 rev_strTraceId_msg 和 rev_strSpanId_msg 赋值到服务端上下文
                            htrace_server_convert_msgstring_to_contextintls(rev_strTraceId_msg, rev_strSpanId_msg);
                            htrace_server_receive ("获取短URL");  // 打印调用链日志
                            
                            // 执行短URL处理业务,并成功
                            
                            htrace_server_send(true, NULL, NULL);  // 打印调用链日志
                             // 将当前的服务器上下文保存到 strTraceId_msg 和 strSpanId_msg
                            htrace_server_convert_contextintls_to_msgstring(strTraceId_msg, strSpanId_msg);
                            // 将 strTraceId_msg 和 strSpanId_msg 传回客户端
                            sendToClient(strTraceId, strSpanId, "the short url is rtsp://xxx");

                            



            
```

## 调用步骤

```shell
    1.  先调用 htrace_init 或 htrace_init2 进行初始化操作
```

## 接口
```shell
    1. 获取当前追踪信息(追踪上下文) 的信息(主要是 traceid 与 spanid)
        htrace_get_traceinfo(htrace_trace_id_t * strOutTraceId, htrace_span_id_t * strOutSpanId, htrace_span_id_t * strOutParentSpanId);
        char traceId[33] = {0};
        char spanId[33] = {0};
        htrace_get_traceinfo(&traceId, &spanId, NULL);
        
       注意:
            htrace_get_traceinfo 接口可以多次调用，不会影响追踪上下文状态，其他接口不能保证不会修改追踪上下文状态。
            因此查看追踪信息请使用此接口，请勿使用 htrace_client_convert_contextintls_to_msgstring 或
            htrace_server_convert_contextintls_to_msgstring
    
```
