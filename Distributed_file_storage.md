# 分布式文件存储

## HDFS 

``` shell
    1. 架构组件
            (1)  一个 HDFS 集群包含一个 NameNode(主服务器), 用来管理文件系统名称空间和管理客户端对文件的访问
                 NameNode 执行文件系统命名空间操作，如打开，关闭和重命名文件和目录, 确定块到 DataNode 的映射. 
                 NameNode 是所有 HDFS 元数据的仲裁者和存储库. 该系统的设计方式是用户数据永远不会流经 NameNode
                 NameNode 控制着关于 blocks 复制的所有决定, NameNode 定时得接收集群中 DataNode 发送的心跳和块报告,
                 收到心跳意味着 DataNode 在正常地运行着,一个块报告包含着 DataNode 上所有块信息的集合。
            (2) 
                
                
			
```
