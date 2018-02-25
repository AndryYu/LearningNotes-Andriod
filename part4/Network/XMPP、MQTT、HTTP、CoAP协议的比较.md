## XMPP协议、MQTT协议、HTTP协议、CoAP协议的比较
#### 1.国外相关专业数据比较
Protocol协议  |  Transport | Messaging | 2G,3G,4G Suitability(1000s nodes)|LLN Suitability(1000s nodes)|Compute Resources |Success Storied
:----|:---- |:---- |:---- |:---- |:---- |:----
Coap | UDP | Request/Response | Excellent | Excellent | 10Ks RAM/Flash | Utility Field Area Networks
XMPP | TCP | Publish/Subscribe Request/Response | Excellent |Fair | 10Ks RAM/Flash |Remote management of consumer white goods
RESTful HTTP| TCP | Request/Response | Excellent | Fair | 10Ks RAM/Flash | Smart Energy Profile 2 (premise energy management/home services)
MQTT| TCP | Publish/Subscribe Request/Response | Excellent |Fair | 10Ks RAM/Flash | Extending enterprise messaging into IoT application

XML的解析对应嵌入式设备来说是比较痛苦，所以在嵌入式设备做开发的时候，最好不要选择基于XML的协议。另外HTTP协议对于嵌入式设备来说是属于重量级也不是很合适。

#### 2.协议基本介绍
* 物联网协议XMPP
<p>XMPP是一种基于标准通用标记语言的子集XML的协议，它继承了在XML环境中灵活的发展性。因此，基于XMPP的应用具有超强的可扩展性。经过扩展以后的XMPP可以通过发送扩展的信息来处理用户的需求，以及在XMPP的顶端建立如内容发布系统和基于地址的服务等应用程序。而且，XMPP包含了针对服务端的软件协议，使之能与另一个进行通话，这使得开发者更容易建立客户应用程序或给一个已经配置好XMPP协议的系统添加功能。</p>
* 物联网协议MQTT
<p>MQTT(Message Queuing Telemetry Transport, 消息队列遥测传输)是IBM开发的一个即时通讯协议。该协议支持所有平台，几乎可以把所有联网物品和外部连接起来，被用来当做传感器和制动器的通信协议</p>
* 物联网协议CoAP
<p>CoAP是受限制的应用协议的代名词。在最近几年的时间中，专家们预测会有更多的设备相互连接，而这些设备的数量将远超人类的数量。在这种大背景下，物联网和M2M技术应运而生。虽然对人而言，连接入互联网显得方便容易，但是对于那些微型设备而言接入互联网非常困难。在当前由PC机组成的世界，信息交换是通过TCP和应用层协议HTTP实现的。但是对于小型设备而言，实现TCP和HTTP协议显然是一个过分的要求。为了让小设备可以接入互联网，CoAP协议被设计出来。CoAP是一种应用层协议，它运行于UDP协议之上而不是像HTTP那样运行于TCP之上。CoAP协议非常的小巧，最小的数据包仅为4字节</p>
* 物联网协议RESTful HTTP
<p>REST 指的是一组 架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful。</p>
<p>Web 应用程序最重要的 REST 原则是，客户端和服务器之间的交互在请求之间是无状态的。从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外，无状态请求可以由任何可用服务器回答，这十分适合 云计算之类的环境。客户端可以缓存数据以改进性能.</p>

#### 3.现有的移动端（Android）消息推送方案
* 方案1、 使用XMPP协议（Openfire + Spark + Smack）  
简介：基于XML协议的通讯协议，前身是Jabber，目前已由IETF国际标准化组织完成了标准化工作。  
优点：协议成熟、强大、可扩展性强、目前主要应用于许多聊天系统中，且已有开源的Java版的开发实例androidpn。  
缺点：协议较复杂、冗余（基于XML）、费流量、费电，部署硬件成本高。  

* 方案2、 使用MQTT协议（更多信息见：  http://mqtt.org/ ）  
简介：轻量级的、基于代理的“发布/订阅”模式的消息传输协议。  
优点：协议简洁、小巧、可扩展性强、省流量、省电，目前已经应用到企业领域（参考：  http://mqtt.org/software ），且已有C++版的服务端组件rsmb。  
缺点：不够成熟、实现较复杂、服务端组件rsmb不开源，部署硬件成本较高。  
