RPC的性能提升问题：

1.序列化：Dubbo序列化成二进制消息流，速度最快。

2.网络通信：采用TCP协议的socket进行长连接，不需要三次握手与四次挥手。Spring cloud采用的http通信协议。