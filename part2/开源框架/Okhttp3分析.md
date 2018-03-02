## OKHttp3分析
### 1.OkHttp的Okio实现原理
OkHttp中在Okio对IO的操作上进行了自己的封装，Okio拥有自己的缓存。Okio的读写都会经过自己的缓存，而Okio的缓存采用了池化的思想，也就是并不是用完了分配的内存就立即释放，在频繁读写时候，提高了一定的性能。
* BufferedSource接口主要用于读取
* BufferedSink接口主要用于写入

### 2.OkHttp怎么实现缓存

### 3.OkHttp Vs Volley
Volley在Android 2.3及以上版本使用的是HttpURLConnection，而在Android2.2及以下版本使用的是HttpClient。<br/>
OKHttp使用Okio进行数据传输。
