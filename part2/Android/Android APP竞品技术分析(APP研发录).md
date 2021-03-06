## App研发录 —— 第九章 App竞品技术分析
### 1.竞品分析概述
我们通常将同行业内竞争对手的产品定义为竞品，所以竞品分析通常就是分析竞争对手的产品。<br/>
从技术层面而言，同行业内竞争对手的APP产品一定要经常研究，而对于整个APP应用领域，各个行业都有其优势，我们要学习他们各自的优点，用到自己的App中，这才是竞品分析的意义所在。

### 2.App安装包的结构
![image.png](http://upload-images.jianshu.io/upload_images/5361549-053b81006ad5ceb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
简单介绍一下这些目录和文件的用途：
* resources.arscz 这个文件是编译后的二进制文件的所以，也就是apk文件的资源表(索引)。
* lib目录下的子目录armeabi存放的是一些so文件。
* META-INF 目录下存放的是签名信息，用来保证apk包的完整性和系统的安全。但这个目录下的文件却不会被签名，从而给了我们无限的想象空间。
* assets 目录下main可以看到很多基础数据，以及一些本地会使用到的HTML、CSS和JavaScript文件。
* res 目录下面的anim子目录很值得研究，这个目录存放APP所有的动画效果。

注意，res目录中的很多xml文件打开后是乱码，AndroidManifest.xml也是如此，那是打包的时候对xml文件进行了压缩，所以看到的往往是全角的字符和乱码，不便于查找我们想要看的内容。有一款神器用于看到apk包中正常的内容，AXMLPrinter2.jar，它可以将apk中已经处理过的xml还原为刻度格式。命令如下所示：
```
  java -jar AXMLPrinter2.jar AndroidManifest.xml
```
### 3.竞品技术一瞥：开机速度
![image.png](http://upload-images.jianshu.io/upload_images/5361549-5471eaeafd35f6e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们仔细研究一下每一步都做了哪些事情：
1. Splash广告的逻辑时，首次加载App包中的图片，同时调用MobileAPI的一个接口，获取下一次打开的图片URL，把这张图片存放在本地。那么下次再打开这个App时，就加载这张新图片，同时，仍然调用MobileAPI的那个接口，看是否有新的Splash图片要下载。为了确保首页打开速度，MobileAPI的这个接口一定是异步调用的。
2. 引导页，不要超过4页，甚至4页我都认为多。
3. 进入首页之前，很多App会要求用户选择所在城市，有的App是默认选一个城市进入，有的App则是异步定位当前城市，同时给用户选择所在城市的机会。
4. App首页的设计，现在通用的做法是，尽可能多地把所有产品都显示在首页，会有轮播广告，会有搜索框，会有滚动条。

### 4.竞品技术二瞥：HTML5页面的打开速度
#### 4.1 把HTML5页面嵌入到Zip包中
App每次启动的时候，会启动一个线程，异步把Zip包解压到本地的某个目录下，然后每次从本地读取HTML5页面，这样就不用每次从服务器加载HTML5页面了。<br/>
Zip包里的内容发生变化怎么处理？需要有个版本控制机制。每次加载HTML5页面之前，先问一下服务器，当前HTML5页面的版本是什么，如果与本地保存的版本号相同，就直接加载本地HTML5；否则，就从服务器重新下载一个新的Zip包，仍然解压到本地相同的目录下。

#### 4.2 Zip包的增量更新机制
即使如此，每次有新版本的HTML5，都要下载一个最新的Zip包，还是很慢。为此我们要减小Zip的体积。我们知道，Zip包中包括HTML5页面、图片、CSS和JS文件，但并不是每次升级每个文件都要更新，我们要把那些不随版本升级而变化的文件挑出来，压缩成common.zip，放入App包中，仍然是第一次启动App后解压缩到本地。这样每次HTML5页面的版本更新，确保要下载的Zip包只包括新增的和修改的文件就可以了，从而确保了Zip包的体积最小，可以快速下载到App，仍然解压到相同的目录下，如果有相同的文件则将其覆盖。我们称这种机制为'增量更新'。<br/>
我说的这种增量包，只包括新增的和修改的文件，对于删除的文件，我们不用去管它，就把它扔在手机的本地目录下好了。就算是增量更新，也要控制增量包的大小在100KB以内。

#### 4.3 制作Zip增量包
通过后台生成增量压缩包，虽然逻辑很复杂，但是App的业务逻辑却非常简单，只要知道增量压缩包的下载地址就够了。我们要控制增量包中图片的数量，只要保证App首屏显示的图片在增量压缩包中即可，至于屏幕外的图片，还是使用url加载，同时在App的WebView控件上建立图片缓存。<br/>
有的Html5页面会在Html页面中使用Ajax调用网络接口获取数据，Ajax因为存在跨域的问题而不能访问到网络接口数据，这时我们就要同时从App的代码和Html的代码进行配置，才能解决这个问题。

#### 4.4 使用WebView预先加载HTML5并缓存到本地

### 5.竞品技术三瞥：安装包的大小
**png和jpg的区别及使用场景**<br/>
众所周知，png有透明通道，而jpg没有，此外png是无损压缩的，而jpg是有损压缩的，所以png中存储的信息会很多，体积自然就大了。但是手机却偏偏对png情有独钟，会对其进行硬件加速，所以我们会发现，同样一张北京图，png虽然体积比jpg大但是加载速度却要快一些。<br/>
综上所述，对于App包中的图片，我们都使用png格式的，而对于要从网上加载的图片，考虑到流量以及下载速度，则使用jpg格式的，因为它有较高的压缩率，体积很小。

**Splash、引导图和背景图**<br/>


### 6.竞品技术四瞥：性能优化
**App自动选取最佳服务器的策略**<br/>

**使用TCP+Protobuf**<br/>

### 7.竞品技术五瞥：数据采集工具
**页面跳转器**<br/>

**打点统计**<br/>

**ABTest**<br/>

### 8.竞品技术六瞥：热修复




### 9.竞品技术七瞥：曲径通幽
**Android包中META-INF目录的妙用**




**classes.dex的拆与合**



### 10.竞品技术八瞥：模块化拆分








### 11.竞品技术九瞥：第三方SDK





### 12.竞品技术十瞥：版本策略和App彩蛋
