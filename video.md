# 视频

## 概要
```shell
  1. RTP（Real-time Transport Protocol）是实时流媒体传输协议，RTCP（Real-time Transport Control Protocol）是对 RTP 的控制协议，
     RTSP（Real Time Streaming Protocol）就是我们常说的 SDP（会话描述协议），是用来控制声音或影像的多媒体串流协议。
  2. RTP :(Real-time Transport Protocol)
        是用于 Internet 上针对多媒体数据流的一种传输层协议.RTP 协议和 RTCP(RTP 控制协议) 一起使用，
        而且它是建立在 UDP 协议上的.RTP 不像 http 和 ftp 可完整的下载整个影视文件，它是以固定的数据率在网络上发送数据，
        客户端也是按照这种速度观看影视文件，当影视画面播放过后，就不可以再重复播放，除非重新向服务器端要求数据。
  3. RTCP:（Real-time Transport Control Protocol）
        实时传输控制协议,是实时传输协议(RTP)的一个姐妹协议.
        RTP 协议和 RTCP(RTP控制协议) 一起使用，而且它是建立在 UDP 协议上的（一般用于视频会议）
  4. RTSP 与 RTP 最大的区别在于：RTSP 是一种双向实时数据传输协议，它允许客户端向服务器端发送请求，
     如回放、快进、倒退等操作。RTSP 可基于 RTP 来传送数据，还可以选择 TCP、UDP、组播 UDP 等通道来发送数据，\
     具有很好的扩展性。它时一种类似与 http 协议的网络应用层协议
  5. 回放只有在录像好的前提下进行的.

```

## 通信协议
```shell
  1. ISAPI 协议
        ISAPI（Intelligent Security API）协议是海康设备的新一代通信协议，对外开放，基于 HTTP 并且遵循 Restful 风格的文本协议。
        规定了设备和客户端之间如何通过 IP 进行通信的协议标准
```