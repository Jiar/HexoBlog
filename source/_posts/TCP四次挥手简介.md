---
title: TCP四次挥手简介
date: 2017-08-24 15:34:53
categories: 计算机网络
tags: 
- 计算机网络
- TCP
---

# TCP四次挥手简介

[简书阅读地址在这里](http://www.jianshu.com/p/a57a40163d4b)

阅读前提示：因为有部分知识会涉及到作者的另一篇博文《TCP三次握手简介》中的内容，建议先跳转阅读，[传送门在这里](http://blog.jiar.vip/2017/08/11/TCP%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B%E7%AE%80%E4%BB%8B)。

![TCP四次挥手（原创图片）](tcp_hand_wave_detail.png)

<!--more-->

### TCP首部简介

TCP四次挥手同样也涉及到TCP首部的一些知识，先前在《TCP三次握手简介》中介绍过TCP头部的内容了，这里就不列举所有了，挥手主要涉及到`ACK`和`FIN`这两个标志比特位

![TCP首部（图片来自网络）](tcp_head.png)

- ACK: 此标志表示应答域有效，就是说前面所说的TCP应答号将会包含在TCP数据包中。有两个取值: 0和1，为1的时候表示应答域有效，反之为0。
- FIN: 表示发送端已经达到数据末尾，也就是说双方的数据传送完成，没有数据可以传送了，发送FIN标志位的TCP数据包后，连接将被断开。这个标志的数据包也经常被用于进行端口扫描。


### TCP四次挥手过程

其实以下这张图片就能说明TCP四次挥手的过程以及握手两端状态的变化。

![TCP四次挥手（原创图片）](tcp_hand_wave_detail.png)

TCP协议是一种面向连接的、可靠的、基于字节流的运输层通信协议。TCP是全双工模式，这就意味着，当A发出FIN报文段时，只是表示A已经没有数据要发送了，A告诉B，它的数据已经全部发送完毕了；但是，这个时候A还是可以接受来自B的数据；当B返回ACK报文段时，表示B已经知道A没有数据发送了，但是B还是可以发送数据到A的；当B也发送了FIN报文段时，这个时候就表示B也没有数据要发送了，B就会告诉A自己也没有数据要发送了，之后彼此就会愉快的中断这次TCP连接。


#### 挥手过程

- 第一次挥手: 主动关闭方(可以使客户端，也可以是服务器端，这里标记为：A)，将FIN置为1，ACK置为1，Seq(Sequence Number)设置为X`为上一次对方传送过来的Ack(Acknowledgment Number)值`，Ack(Acknowledgment Number)设置为Y`为上一次对方传过来的Seq(Sequence Number)值+1`。设置好以上值后，将数据发送至被动关闭方(这里标记为：B)。然后A进入FIN_WAIT_1状态。

- 第二次挥手：B收到了A发送的FIN报文段，向A回复，将ACK置为1，Ack(Acknowledgment Number)设置为X`第一次挥手中的Seq(Sequence Number)值`+1，Seq(Sequence Number)设置为Y`第一次挥手中的Ack(Acknowledgment Number)值`。然后B进入CLOSE_WAIT状态，A收到B的回复后，进入FIN_WAIT_2状态。

- 第三次挥手：B再次向A发送报文，将FIN置为1，ACK置为1，Ack(Acknowledgment Number)设置为X+1`第二次挥手中的Ack(Acknowledgment Number)值`，Seq(Sequence Number)设置为Y`第二次挥手中的Seq(Sequence Number)值`。然后B进入LAST_ACK状态，A收到B的报文后，进入TIME_WAIT状态。

- 第四次挥手：A收到B发送的FIN报文段，像B回复，将ACK置为1，Ack(Acknowledgment Number)设置为Y`第三次挥手中的Seq(Sequence Number)值`+1，Seq(Sequence Number)设置为X+1`第三次挥手中的Ack(Acknowledgment Number)值`。然后A进入TIME_WAIT状态，B在收到报文后进入CLOSED状态，A在发送完报文等待了2MSL时间后进入CLOSED状态。


#### 状态变化


- ESTABLISHED：已建立连接

- FIN_WAIT_1和FIN_WAIT_2：FIN_WAIT_1和FIN_WAIT_2的意义在于等待B发送FIN报文（B在第三次挥手发送了FIN报文）。

- FIN_WAIT_1：A发送给B报文，请求关闭连接，然后A便进入这个状态，这个时候，表示A已经没有数据要发送了，不过A还能接收数据。

- FIN_WAIT_2：这个时候，SOCKET处于半连接状态，即A要求关闭连接，但是还要稍微等会，等到A收到B发送的FIN报文，并相应了这个报文，并过了2MSL后，才真正关闭，这里只是做个关闭标记。

- CLOSE_WAIT：这个过程是B在等待自己发送FIN报文。当A发送一个FIN报文给B后，B毫无疑问应该立刻回复ACK报文，此时B进入这个状态。接下来，B会观察自己是否还有数据没有发送给A，如果有，先把数据发送给A，再发送FIN报文；如果没有，那么B直接发送FIN报文给A。其实这个状态下，B是在等待自己做完剩余的工作，然后再准备结束关闭连接。

- LAST_ACK：这个状态是B在发送完FIN报文后，等待A的响应。如果接收到A的响应，则进入CLOSED状态。

- TIME_WAIT：A收到了B发送的FIN报文，用ACK报文进行回复。然后等待2MSL时长后，A进入CLOSED状态。如果A在FIN_WAIT_1状态下，同时收到了B的FIN标志和ACK标志的报文，则A可以直接进入到TIME_WAIT状态，而无须经过FIN_WAIT_2状态。


### 为什么 TIME_WAIT 状态要等待 2MSL 之后才关闭连接

- 2MSL表示两个MSL的时长，MSL全称为Maximum Segment Life，表示TCP 对TCP Segment 生存时间的限制。

- 为了保证A发送的最后一个ACK报文段能够到达B。这个ACK报文段有可能丢失，因而使处在LAST_ACK状态的B收不到对自己已发送的FIN+ACK报文段的确认。B会超时重传这个FIN+ACK报文段。而A就能在2MSL时间内收到这个重传的FIN+ACK报文段。接着A重传一次确认，重新启动2MSL计时器。最后A和B都正常进入到CLOSED状态。如果A在TIME_WAIT状态不等待一段时间，而是在发送完ACK报文段后立即释放连接，那么就无法收到B重传的FIN+ACK报文段，因而也不会在发送一次确认报文段。这样，B就无法按照正常步骤进入CLOSED状态。

- 防止已失效的连接请求报文段出现在本连接中。A在发送完最后一个ACK报文段后，在经过2MSL，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失。这样就可以使下一个新的连接中不会出现这种旧的连接请求报文段。


### 为什么要四次挥手

我在《TCP三次握手简介》得出过这样一个结论：`三次握手的本质是：将“四次握手”中的第二次、第三次握手合为一次，因为“四次握手”中的第二次、第三次握手都是由B向A传递报文，而且这两次发送报文的目的允许这两次报文合并为一次。`那么，TCP四次挥手中的第二次、第三次挥手，能否也能合为一次呢？

答案是否定的。将TCP四次挥手中的第二次、第三次挥手，合为一次。也就是将CLOSE_WAIT状态的停留时间变为0。然而，B之所以存在CLOSE_WAIT状态，是因为B可能还存在着需要发送给A但是未发送的数据，如果存在着这些数据，那么这个状态的时间，就是用来发送这些数据的，所以，TCP四次挥手中的第二次、第三次挥手无法合并为一次。所以，也就无法实现“TCP三次挥手”。


### 实践(抓包分析)

接下来我们通过网络抓包的方式来了解TCP的三次握手。我这里使用的抓包软件是`Wireshark`。

- 打开`Wireshark`，选择需要捕获的网络。
![wireshark_welcome](wireshark_welcome.png)

- 进入到主界面
![wireshark_main](wireshark_main.png)

- 找到TCP四次挥手
![wireshark_tcp_wave](wireshark_tcp_wave.png)
观察`Wireshark`上部已经捕获的网络数据包列表部分，看`Info`部分，能找到相对连续的四列(分别显示`A -> B [FIN, ACK]...`、`B -> A [ACK]...`、`B -> A [FIN, ACK]...`、`A -> B [ACK]...`)，便是TCP的四次挥手，在找的时候，注意`Source`栏和`Destination`栏中的ip地址的相对应，以及`Info`栏中的端口的对应。在`Wireshark`直接寻找四次挥手比寻找三次挥手要难得多。再下一篇文章中，我将会展示如何快速找到一组SOCKET的握手和挥手信息。

- 查看第一次挥手的详情
![wireshark_tcp_wave_first](wireshark_tcp_wave_first.png)

- 查看第二次挥手的详情
![wireshark_tcp_wave_second](wireshark_tcp_wave_second.png)

- 查看第三次挥手的详情
![wireshark_tcp_wave_third](wireshark_tcp_wave_third.png)

- 查看第四次挥手的详情
![wireshark_tcp_wave_fourth](wireshark_tcp_wave_fourth.png)

选中每次一的挥手数据包，点击下方的`Transmission Control Protocol(TCP)`，即可显示每次TCP握手的详情。在详情中，我们展开`Flags`，可以看到比特标志位是否有被设置的情况。
我们能发现，实践中的TCP状态情况，跟上面提到的理论是一致的。

- 第一次挥手: [FIN, ACK] Seq=161 Ack=734
- 第二次挥手: [ACK] Seq=734 Ack=162
- 第三次挥手: [FIN, ACK] Seq=734 Ack=162
- 第四次挥手: [ACK] Seq=162 Ack=735


### 参考

感谢下列作者的分享

> * [通俗大白话来理解TCP协议的三次握手和四次分手 - jawil](https://github.com/jawil/blog/issues/14)
> * [TCP的三次握手与四次挥手过程的每一步的具体状态变换](http://ab3813.blog.51cto.com/10538332/1773751)


### 结束语

以上便是我这次关于`TCP四次挥手`方面的分享，同时也感谢我在文章中提到的博文的博主的分享。如果您在阅读本文时发现有错误或不恰当之处，欢迎在评论中指出。


欢迎关注我的个人微信订阅号，我将不定期分享开发方面的干货。

![Jiar's 微信订阅号](/images/Dingyuehao.jpg)

