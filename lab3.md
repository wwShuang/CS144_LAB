> https://kangyupl.gitee.io/cs144.github.io/assignments/lab3.pdf
> 

# Lab 3: the TCP sender
先看一遍：[关于TCP传输协议看这一篇就够了](https://zhuanlan.zhihu.com/p/61987654)
> sender负责的工作说起来很简单，就是从发送方的outbound ByteStream中读取数据，将它们切分并包装成各个数据段，尽可能快地发送到网络中，在发送的过程中要进行超时重传

It will be your TCPSender’s responsibility to: 
- Keep track of the receiver’s window (processing incoming acknos and window sizes) 
- Fill the window when possible, by reading from the ByteStream, creating new TCP segments (including SYN and FIN flags if needed), and sending them
- Keep track of which segments have been sent but not yet acknowledged by the receiver— we call these “outstanding” segments
- Re-send outstanding segments if enough time passes since they were sent, and they haven’t been acknowledged yet

## 3.1 When should the TCPSender conclude that a segment was lost and send it again?


## 3.2 Implementing the TCP sender

# reference
1. [LAB3#](https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html#806187688)
2. [CS144计算机网络lab3：TCP sender](https://zhuanlan.zhihu.com/p/299536254)
3. [关于TCP传输协议看这一篇就够了](https://zhuanlan.zhihu.com/p/61987654)
4. 
