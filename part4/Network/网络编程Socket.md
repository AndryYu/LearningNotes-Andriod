## 网络编程UDP和TCP
### UDP
#### 1.简介

将数据、源和目的封装成数据包中，不需要建立连接；
每隔数据包的大小限制在64K内；
因无连接，是不可靠协议；
不需要建立连接，速度快；


#### 2.DatagramSocket类(java.net包)
1).此类表示用来发送和接收数据报包的套接字。
<p>数据报套接字是包投递服务的发送或接收点。每个在数据报套接字上发送或接收的包都是单独编址和路由的。从一台机器发送到另一台机器的多个包可能选择不同的路由，也可能按不同的顺序到达。在DatagramSocket上总是启用UDP广播发送。为了接收广播包，应该将DatagramSocket绑定到通配符地址。在某些实现中，将DatagramSocket绑定到一个更加具体的地址时，广播包也可以被接收!</p>

2）构造方法摘要
 DatagramSocket() <br/>
   构造数据报套接字并将其绑定到本地主机上任何可用的端口。 <br/>
 protected  DatagramSocket(DatagramSocketImpl impl) <br/>
    创建带有指定 DatagramSocketImpl 的未绑定数据报套接字。 <br/>
 DatagramSocket(int port) <br/>
    创建数据报套接字并将其绑定到本地主机上的指定端口。 <br/>
 DatagramSocket(int port, InetAddress laddr) <br/>
    创建数据报套接字，将其绑定到指定的本地地址。 <br/>
 DatagramSocket(SocketAddress bindaddr) <br/>
    创建数据报套接字，将其绑定到指定的本地套接字地址。  <br/>

  3）方法摘要
 void receive(DatagramPacket p) <br/>
    从此套接字接收数据报包。 <br/>
 void send(DatagramPacket p) <br/>
    从此套接字发送数据报包。 <br/>
 void close() <br/>
    关闭此数据报套接字。 <br/>
 void connect(InetAddress address, int port) <br/>
    将套接字连接到此套接字的远程地址。<br/>

#### 3.DatagramPacket类(java.net包)
1）此类表示数据报包

2）构造方法摘要 <br/>
DatagramPacket(byte[] buf, int length) <br/>
   构造 DatagramPacket，用来接收长度为 length 的数据包。 <br/>
DatagramPacket(byte[] buf, int length, InetAddress<br/> address, int port) <br/>
   构造数据报包，用来将长度为 length 的包发送到指定主机上的指定端口号。

3）方法摘要
InetAddress getAddress() <br/>
   返回某台机器的 IP 地址，此数据报将要发往该机器或者是从该机器接收到的。 <br/>
byte[] getData() <br/>
   返回数据缓冲区。 <br/>
void setData(byte[] buf) <br/>
   为此包设置数据缓冲区。<br/>
void setLength(int length) <br/>
   为此包设置长度。 <br/>
void setPort(int iport) <br/>
   设置要将此数据报发往的远程主机上的端口号。 <br/>
int getLength() <br/>
   返回将要发送或接收到的数据的长度。 <br/>
int getPort() <br/>
   返回某台远程主机的端口号，此数据报将要发往该主机或者是从该主机接收到的。 <br/>

### TCP
#### 1.简介
建立连接，形成传输数据的通道；
在连接中进行大量数据传输
通过三次握手完成连接，是可靠协议
必须建立连接，效率会稍低

#### 2.Socket类(java.net包)
1）此类实现客户端套接字（也可以就叫“套接字”）。套接字是两台机器间通信的端点

  2）构造方法摘要 <br/>
  Socket() <br/>
     通过系统默认类型的 SocketImpl 创建未连接套接字 <br/>
  Socket(InetAddress address, int port) <br/>
     创建一个流套接字并将其连接到指定 IP 地址的指定端口号。 <br/>
  Socket(InetAddress address, int port, InetAddress<br/> localAddr, int localPort)
     创建一个套接字并将其连接到指定远程地址上的指定远程端口。<br/>
  Socket(String host, int port) <br/>
     创建一个流套接字并将其连接到指定主机上的指定端口号。<br/>
  Socket(String host, int port, InetAddress localAddr, int localPort) <br/>
     创建一个套接字并将其连接到指定远程主机上的指定远程端口。

  3）方法摘要 <br/>
  void bind(SocketAddress bindpoint) <br/>
     将套接字绑定到本地地址。 <br/>
  void close() <br/>
     关闭此套接字。 <br/>
  void connect(SocketAddress endpoint) <br/>
     将此套接字连接到服务器。 <br/>
  void connect(SocketAddress endpoint, int timeout) <br/>
     将此套接字连接到服务器，并指定一个超时值。 <br/>
  InetAddress getInetAddress() <br/>
     返回套接字连接的地址。 <br/>
  InputStream getInputStream() <br/>
     返回此套接字的输入流。 <br/>
  OutputStream getOutputStream() <br/>
     返回此套接字的输出流。 <br/>
  int getPort() <br/>
     返回此套接字连接到的远程端口。 <br/>
  String toString() <br/>
     将此套接字转换为 String。 <br/>

#### 3.ServerSocket类(java.net包)
1）此类实现服务器套接字。服务器套接字等待请求通过网络传入。它基于该请求执行某些操作，然后可能向请求者返回结果。

2）构造方法摘要 <br/>
ServerSocket() <br/>
   创建非绑定服务器套接字。 <br/>
ServerSocket(int port) <br/>
   创建绑定到特定端口的服务器套接字。 <br/>
ServerSocket(int port, int backlog) <br/>
   利用指定的 backlog 创建服务器套接字并将其绑定到指定的本地端口号。 <br/>
ServerSocket(int port, int backlog, InetAddress bindAddr)<br/>
   使用指定的端口、侦听 backlog 和要绑定到的本地 IP 地址创建服务器。 <br/>

3）方法摘要 <br/>
Socket accept() <br/>
   侦听并接受到此套接字的连接。 <br/>
void close() <br/>
   关闭此套接字。<br/>
void bind(SocketAddress endpoint) <br/>
   将 ServerSocket 绑定到特定地址（IP 地址和端口号）。 <br/>
void bind(SocketAddress endpoint, int backlog) <br/>
   将 ServerSocket 绑定到特定地址（IP 地址和端口号）。 <br/>
String toString() <br/>
   作为 String 返回此套接字的实现地址和实现端口。 <br/>    
