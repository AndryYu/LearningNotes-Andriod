## Fiddler、Charles、WireShark三种抓包工具的比较

### Charles VS Fiddler
Fiddler只能运行在Windows平台，而Charles是基于Java实现的，基本上可以运行在所有主流的桌面系统，还有一个区别就是Fiddler开元免费、Charles是收费的。

### Fiddler VS WireShark
Fiddler是在windows上运行的程序，专门用来捕获Http和Https的<br/>
WireShark能获取Http，也能获取Https，但是不能解密Https，所以WireShark看不懂Https中的内容。<br/>
总结：如果是处理Http、Https还是用Fiddler，其他协议比如Tcp、Udp就用Wireshark
