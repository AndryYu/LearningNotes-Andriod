## 1. WifiManager



## 2.WifiInfo对象
WifiInfo对象可以通过WifiManager.getConnectionInfo()来获取。WifiInfo中包含了当前连接中的相关信息：
* getBSSID() 获取BSSID属性，也就是路由器的mac地址
* getDetailedStateOf() 获取客户端的连通性
* getHiddenSSID()  获取SSID是否被隐藏
* getIpAddress() 获取IP地址
* getLinkSpeed() 获取连接的速度
* getMacAddress() 获取Mac地址
* getRssi()  获取802.11n网络的信号
* getSSID()  获取SSID也就是wifi名称
* getSupplicanState()  获取具体客户端状态的信息



## 3.InetAddress类
InetAddress类的对象用于IP地址和域名，该类提供以下方法：
getByName(String s):获取一个InetAddress类的对象，该对象中含有主机的IP地址和域名，该对象用如下格式获取属性：
  * String getHostName():获取InetAddress对象的域名；
  * String getHostAddress():获取InetAddress对象的IP地址；
getLocalHost()：获取一个InetAddress对象，该对象含有本地机器的域名和IP地址。
