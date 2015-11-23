netty是异步的事件驱动框架，用于快速开发高性能协议的服务器和客户端。netty是使用NIO的客户端服务器框架，可以快速容易的开发网络程序，大大的简化了网络数据流编程，例如TCP UDP服务。
# features
![components](./images/components.png)
## design
- 为各种传输类型提供统一的API，阻塞或非阻塞的套接字
- 基于弹性的和可扩展的事件模型，允许清晰的关注点分离
- 可定制化的线程模型
- True connectionless datagram socket support (since 3.1)

## Ease of use
- 完善的文档，用户向导，例子
- 没有附加的依赖JDK 5 (Netty 3.x) or 6 (Netty 4.x) is enough

## Performance
- 更好的吞吐量，更低的延迟
- 更少的资源消费
- 最小的没必要的内存复制

## Security
- Complete SSL/TLS and StartTLS support