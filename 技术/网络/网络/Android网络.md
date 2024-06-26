### 网络分层

**这里的网络分成模型采取的是5层结构** 

###### 物理层

- 负责比特流在节点间的传输，负责物理传输。通俗的说就是把计算机连接起来的物理手段。

###### 数据链路层

- 控制网络层与物理层之间的通信，主要功能就是如何在不可靠的物理线路上进行数据的可靠性传递。

###### 网络层

- 决定如何将数据从发送方路由到接收方。是建立主机到主机的通信。

###### 传输层

- 该层为两台主机上的应用程序提供端到端的通信。传输层有两个协议：TCP（传输控制协议）：是一个可靠的面向连接的协议，UDP（用户数据包协议）：不可靠的无连接协议。

###### 应用层

- 应用程序收到传输层的数据后，接下来就要进行解读，解读必须规定好格式，应用层就是规定应用程序的数据格式的。主要协议有：HTTP，FTP，SMTP，POP3。

### TCP的三次握手与四次挥手

#### TCP三次握手的过程如下（为了建立连接，来传输数据）

- 第一次握手：建立连接 客户端发送连接请求报文段，将SYN设置为1、seq为x，等待服务端的确认
- 第二次握手：服务端收到客户端的请求连接报文段SYN，对SYN报文段进行确认，然后服务端发送 SYN_ACK报文段
- 第三次握手：客户端收到服务端的SYN_ACK报文段，将ACK设置为y+1，然后向服务端发送ACK报文段，这个保本段发送完毕后，客户端和服务端就进入了连接成功的状态。

#### 四次挥手 （数据传输完毕，需要断开连接）

- 第一次挥手：客户端向服务端发送FIN报文段，告诉服务端没有数据要发送了。
- 第二次挥手：服务端收到客户端发送的FIN报文，向客户端回了一个ACK报文段。
- 第三次挥手：服务端向客户端发送报文段，请求关闭连接。
- 第四次挥手：客户端收到服务端的FIN报文段，向服务端发送ACK报文段。服务端收到客户端的ACK报文段后，就关闭连接了。客户端等到（最大报文段生存时间后）没有收到回复，说明服务端已经关闭，那么客户端就关闭。











































