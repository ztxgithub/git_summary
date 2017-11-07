# WebSocket 概要

``` shell
    1.Websocket是一个持久化的协议,相对于HTTP这种非持久的协议来说.内容和HTTP差不多
        GET /chat HTTP/1.1
        Host: server.example.com
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
        Sec-WebSocket-Protocol: chat, superchat
        Sec-WebSocket-Version: 13
        Origin: http://example.com

    2.Websocket的作用
        在讲Websocket之前,基本上是long poll 和 ajax轮询 的原理,这些都是客户端主动连接服务端,服务端不能主动联系客户端，
        ajax轮询是非阻塞的方式向服务端查看,是否有对应的信息,没有也返回Response,而long poll 是阻塞方式查询服务器,
        等待服务器有消息.
        而Websocket是一次持久化连接,只需要一次HTTP握手,所以说整个通讯过程是建立在一次连接/状态中,在这过程中服务器可以主动给
        客户端发消息.
```