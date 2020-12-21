# WEB 服务

## RESTful
```shell
  1. 客户端发出的数据操作指令都是 "动词 + 宾语" 的结构。比如，GET /articles这个命令，GET是动词，/articles是宾语。
     动词通常就是五种 HTTP 方法(根据 HTTP 规范，动词一律大写)
          GET：读取（Read）
          POST：新建（Create）
          PUT：更新（Update）
          PATCH：更新（Update），通常是部分更新
          DELETE：删除（Delete）
   2. 动词的覆盖
        有些客户端只能使用 GET 和 POST 这两种方法。服务器必须接受 POST 模拟其他三个方法（PUT、PATCH、DELETE）。
        通常客户端发出的 HTTP 请求，要加上 X-HTTP-Method-Override 属性，告诉服务器应该使用哪一个动词，覆盖 POST 方法
        例如:
            POST /api/Person/4 HTTP/1.1  
            X-HTTP-Method-Override: PUT
            X-HTTP-Method-Override 指定本次请求的方法是 PUT，而不是 POST。
   3. 宾语必须是名词
        宾语就是 API 的 URL，是 HTTP 动词作用的对象。它应该是名词，不能是动词。比如，/articles这个 URL 就是正确的,
        /getAllCars 是错误的
   4. 都使用复数 URL
          GET /articles/2 要好于 GET /article/2
   5.避免多级 URL
        因为多级 url(/authors/12/categories/2) 不利于扩展，语义也不明确，除了第一级，其他级别都用查询字符串表达
        例如 GET /authors/12?categories=2
   6. 状态码
          HTTP 状态码就是一个三位数，分成五个类别
          1xx：相关信息
          2xx：操作成功， POST 返回 201 状态码，表示生成了新的资源；DELETE 返回 204 状态码，表示资源已经不存在
                         202 Accepted状态码表示服务器已经收到请求，但还未进行处理，会在未来再处理，通常用于异步操作
          3xx：重定向  302 和 307 用于 GET 请求，而 303 用于 POST、PUT 和 DELETE 请求。收到 303 以后，
                      浏览器不会自动跳转，而会让用户自己决定下一步怎么办
          4xx：客户端错误
                400 Bad Request：服务器不理解客户端的请求，未做任何处理。
                401 Unauthorized：用户未提供身份验证凭据，或者没有通过身份验证。
                403 Forbidden：用户通过了身份验证，但是不具有访问资源所需的权限。
                404 Not Found：所请求的资源不存在，或不可用。
                405 Method Not Allowed：用户已经通过身份验证，但是所用的 HTTP 方法不在他的权限之内。
                410 Gone：所请求的资源已从这个地址转移，不再可用。
                415 Unsupported Media Type：客户端要求的返回格式不支持。比如，API 只能返回 JSON 格式，
                                            但是客户端要求返回 XML 格式。
                422 Unprocessable Entity ：客户端上传的附件无法处理，导致请求失败。
                429 Too Many Requests：客户端的请求次数超过限额
          5xx：服务器错误
                500 Internal Server Error：客户端请求有效，服务器处理时发生了意外。
                503 Service Unavailable：服务器无法处理请求，一般用于网站维护状态。
   7. 服务器的回应
        (1) API 返回的数据格式应该是 json 对象, 请求的 HTTP 头的ACCEPT属性也要设成 application/json
        (2) 业务上的错误, 最好也返回 400 的错误码.
        (3) 可提供链接(在内容中)
                {
                  ...
                  "feeds_url": "https://api.github.com/feeds",
                  "followers_url": "https://api.github.com/user/followers"
                  ...
                }
```

## 基础知识

```shell
    1. 443 端口, 主要是用于 HTTPS 服务，是提供加密和通过安全端口传输的另一种 HTTP
    2. 80 端口：为 HTTP（HyperText Transport Protocol)即超文本传输协议开放的
    3. HTTP 协议代理服务器常用端口号：80/8080/3128/8081/9098
```