## Android 8.0（Oreo）新特性
Android O新版本更新和优化主要集中在两个方面：Fluid Experiences和Vitals。Fluid Experience主要包含了四个显著特性：Notification Dots, Picture in Picture, Autofill Framework和Smart Text Selection;而Vitals主要在电池续航、安全、启动时间以及稳定性这几个方面做优化。

##第一部分

### 一：通知渠道 - Notification Channels
通知渠道是由应用自行定义的通知内容类别，借助渠道，开发者可以让用户对不同种类的通知进行精细控制，用户可以单独拦截或更改每个渠道的行为，而不是统一管理应用的所有通知。


### 二：画中画模式 - PIP
Android O现已支持Activity的画中画模式。PIP是一种多窗口模式，多用于视频播放，即你可以一边发微信一边看视频。


### 三：自适应图标 - Adaptive Icons
Android的屏幕适配一直以来都折磨着不少的开发者。为了帮助开发者更好的与设备UI集成，Android O支持创建自适应图标，系统可以基于设备选择的蒙版将这些图标显示为不同形状。系统还将实现与图标的自动交互，并在启动器、快捷方式、设置、共享对话框以及概览屏幕中使用它们。


### 四：自动填充框架
Android O 还引入了自动填充框架，简化了用户在账号创建、登录和信用卡表单之类的填写工作，在用户选择自动填充框架之后，新老用户都可以使用自动填充框架，我们使用 Chrome 的时候已经体验过了自动填充用户名和密码的功能，只不过这次是在系统层面提供了这样的一种功能，可以快速的填充用户名，地址甚至密码等，而且用户也不需要去担心安全问题。


### 五：xml字体和可下载字体
Android O 推出了 xml 字体，可以在资源文件中建立 font 字体资源文件夹，放入相应的字体 ttf 文件，然后建立自己的字体 xml 文件，在 R 文件中编译，最终作为一种资源供 TextView 等使用。


### 六：固定快捷方式和小部件 - Pinning shortcuts
Pinning shortcuts 是一个比 APP shortcuts 更小的快捷方式，放置于桌面上，用于更快速的打开某一 APP 的某单一任务。Pinning shortcuts 在桌面上可呈现不同的图标显示。


### 七：TextView字体自动适配
　Android O 版本允许设置 TextView 的字体大小根据设置的初始大小自动放大或者缩小，这样就可以让字体的显示在不同的屏幕和不同的显示内容上达到最优的效果，而且使用 Android support library 26 中的 android.support.v4.widget.TextViewCompat 可以让该特性支持到 14 版本，Android O 版本的 TextView 已经可以支持 autosize 了，设置 autosize 特性也非常简单，在 O 版本上，只需要使用 setAutoSizeTextTypeWithDefaults(@AutoSizeTextType int autoSizeTextType) 或者
```
<TextView
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:autoSizeTextType="uniform"/>
```

### 八：媒体增强
Android O 版本新增 VolumeShaper 类，用来为应用提供声音的淡入淡出等音效；新增AudioFocusRequest 类用来提供检测音频焦点的新功能；新增了以下的方法 getMetrics 方法用来返回一个包含配置和性能信息的 Bundle 对象：
* MediaPlayer.getMetrics()
* MediaRecorder.getMetrics()
* MediaCodec.getMetrics()
* MediaExtractor.getMetrics()

  MediaPlayer 新增了一些新的方法，这些方法可以用来增强应用处理媒体播放的能力：

## 第二部分：行为变更
### 一：后台执行限制
Android O 在当进程进入已缓存状态时，如果没有活动的组件，系统将解除应用具有的所有唤醒锁（已缓存状态指的是没有前台 Activity 或者正在执行的前台 Service）。同时 Android O 上运行在后台的应用将会有限制的使用后台的 Service，并且应用也不能在 Manifest 中注册一些不必要的隐式广播用来进行自启等操作：
* 在后台运行的应用对后台服务的访问受到限制;
* 应用无法使用其清单注册大部分隐式广播（即并非专门针对此应用的广播，比如 ACTION_PACKAGE_REPLACED 针对所有应用是一个隐式广播，而ACTION_MY_PACKAGE_REPLACED只针对本应用就不是一个隐式广播）。

### 二：安全性
Android O 包含以下与安全性有关的变更：
* 不再支持 SSLv3；
* 应用的 WebView 对象将在多进程模式下运行。网页内容在独立的进程中处理，此进程与包含应用的进程相隔离，以提高安全性；
* 在与未正确实现 TLS 协议版本协商的服务器建立 HTTPS 连接时，HttpsURLConnection 不再尝试回退到之前的 TLS 协议版本并重试的权宜方法；
* Android O 将使用安全计算 (SECCOMP) 过滤器来过滤所有应用。允许的系统调用列表仅限于通过 bionic 公开的系统调用。此外，还提供了其他几个后向兼容的系统调用，但我们不建议使用这些系统调用。

### 三：网络连接和HTTP(S)连接
Android O 对网络连接和 HTTP(S) 连接行为做出了不少变更，其中包括无正文的 OPTIONS 请求现在有 Content-Length: 0 标头；HttpURLConnection 在包含斜线的主机或颁发机构名称后面附加一条斜线，将 http://example.com 转化为 http://example.com/ ；通过 ProxySelector.setDefault() 设置的自定义代理选择器的范围变化；URI 不能包含空白标签；如果之前执行的 connect() 方法失败，send(java.net.DatagramPacket) 方法将会引发 SocketException；在回退到 TCP Echo 协议之前，InetAddress.isReachable() 会尝试执行 ICMP；隧道 HTTP(S) 连接处理进行了一些变更。

### 四：权限
在 Android O 之前，如果应用在运行时请求权限并且被授予该权限，系统会错误地将属于同一权限组并且在清单中注册的其他权限也一起授予应用。对于针对 Android O 的应用，此行为已被纠正。系统只会授予应用明确请求的权限。然而，一旦用户为应用授予某个权限，则所有后续对该权限组中权限的请求都将被自动批准。

### 五：媒体变更
* 使用 AudioTrack 时，如果应用请求了足够大的音频缓冲区，则框架将尝试使用深度缓冲区输出（如果可用）；
* 音频流类型应仅用于音量控制；所有其他流类型的使用（例如 AudioTrack 构造函数）仍有效，但系统会将其作为错误记录下来；
* 当用户打电话时，活动的媒体流将在通话期间静音；
* 所有与音频相关的 API 均使用 AudioAttributes 来描述音频播放用例；
* 框架会执行音频闪避，进行 AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK 时，应用不会失去焦点。新的 API 适用于需要暂停而不是闪避的应用。不过，Android O 中未提供此行为。

### 六：Native libraries
在针对 Android O 的应用中，如果 Native libraries 包含任何可写且可执行的代码段，则不会再加载 Native libraries，可写和可执行必须是在新版本必须是互斥的，倘若某些应用的 Native libraries 包含不正确的加载代码段，则此变更可能会导致这些应用停止工作。


### 七：其他
1. ContentProvider 支持分页，即获取内容的选中区域的子集；
2. ContentProvider 和 ContentResolver 增加 refresh 方法，用来让客户端更容易的知道数据是不是最新；
3. JobScheduler 更新，让应用更容易遵从后台执行限制；
4. 集合的处理的变化，AbstractCollection.removeAll() 和 AbstractCollection.retainAll() 始终引发 NullPointerException
5. 语言区域和国际化变化；
6. 联系人提供程序使用情况统计方法的变更；
7. 蓝牙 ScanRecord.getBytes() 方法检索的数据长度变更；
8. 输入和导航；


## 第三部分：API变更
### 一：WebView 新API
Android O 预览版本提供了几个新的 API 用来管理 WebView：

* Version API
  第一个是提供获取 WebView 版本信息的 API:
  ```
    PackageInfo webViewPackageInfo = WebView.getCurrentWebViewPackage();
    Log.d("MY_APP_TAG", "WebView version: " + webViewPackageInfo.versionName);
  ```
* Google Safe Browsing API
  可以再 Manifest 中配置 enable，然后在 WebView 打开未知不安全 url 的时候提示用户：
  ```
  <manifest>
    <meta-data android:name="android.webkit.WebView.EnableSafeBrowsing"
               android:value="true" />
    ...
    <application> ... </application>
  </manifest>
 ```
* Termination Handle API
  WebView 绘制进程被杀或者 Crash 的回调；
* Renderer Importance API
  用来设置 WebView 绘制进程的优先级别，为了提供应用的稳定性，一般情况下应用不需要去修改绘制进程优先级，如果需要使用请和 Termination Handle API 一起搭配使用；

### 二：findViewById
findViewById 函数现在返回的是 <T extends View>，所以以后 findViewById 就不需要强转了。

### 三：统一的 margins 和 padding
　Android 引入了几个新的 xml 属性：<br/>
`layout_marginVertical`，同时设置 `layout_marginTop` 和 `layout_marginTop` 属性；<br/>
`layout_marginHorizontal`，同时设置 `layout_marginLeft` 和 `layout_marginRight`属性；<br/>
`paddingVertical`，同时设置 `paddingTop` 和 `paddingBottom`属性；<br/>
`paddingHorizontal`，同时设置 `paddingLeft` 和 `paddingRight`属性；<br/>

### 四：AnimationSet
Android O 中，AnimationSet API 现在支持了动画的 seek 和动画倒转播放，seek 操作可以设置 AnimationSet 从指定的点开始播放，倒转播放则将以前需要重复定义两个相反的动画操作简化成只需要定义一个动画即可。

### 五：提醒窗口
