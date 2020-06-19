---
order: 1
group:
  title: 网络协议
  order: 1
nav:
  title: 计算机基础
  order: 1
---

# UDP

**UDP** 的全称叫 [用户数据报协议](https://en.wikipedia.org/wiki/User_Datagram_Protocol)（ User Datagram Protocol ），是传输层的一个无连接协议。

我们知道 [OSI](https://en.wikipedia.org/wiki/OSI_model) 的传输层有两个主要的协议，互为补充。其中，面向连接的是 [TCP](https://en.wikipedia.org/wiki/TCP)，无连接的是 UDP。

## 如何理解无连接

通信技术其实大致可以分为三类：**面向连接的电路交换，面向连接的包交换，以及无连接的包交换**。

UDP 在这里使用的就是**无连接的包交换**，具体来说：

- 通信前发送端和接收端不需要建立和断开连接，也不需要维护连接状态
- 通信过程中，直接把每个带有目的地址的包（报文）送到线路上，由系统自主选定路线进行传输

这也就决定了 UDP 只是**报文的搬运工**，不会对报文进行任何拆分和拼接操作。

## UDP 的报文头

因为 UDP 是无连接的，不需要保证数据不丢失且有序到达，所以其头部开销非常小，只有 **8** 个字节，

![UDP.png](https://i.loli.net/2020/06/18/oNsMFtgzJGSrq9y.png)

其中，头部信息中包含了以下几个数据：

- **Source port & Destination port**：两个 16 位的端口号，分别为发送端口和接收端口
- **Length**：数据报文的长度
- **Checksum**：数据报文校验以及用于发现头部信息和数据中的错误的 IPV4 可选字段

## 特性

基于无连接的包交换，我们可以很好地理解 UDP 的以下特性，

### 不可靠性

UDP 的不可靠性体现在对数据报文的处理上，

- 协议收到什么就传递什么，也不会备份数据，对方能不能收到也不关心
- 没有拥塞控制，一直会以恒定的速度发送数据，所以在网络条件不好的情况下可能会导致丢包

### 高效性

由于不提供数据传送的保证机制，所以 UDP 的优点就是**高效**，适合某些实时性要求较高的场景，比如：

- 屏幕上报告股票市场，显示航空信息
- 电话会议，视频会议

## 总结

**UDP 是传输层一个面向报文，无连接的协议，它具有高效但不可靠的特性**。