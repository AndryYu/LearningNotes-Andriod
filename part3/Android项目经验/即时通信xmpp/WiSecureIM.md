## WiSecureIM
这是一款使用Xmpp协议实现的即时聊天的应用App，由于不是自己从头开发的，现对整个App做一个简单的分析。App架构
* app (application)主要处理界面展示相关逻辑
* sdklibrary
  * badges
  * ptr
  * wifileselector
  * wipicturecrop
  * wipictureselector
* smacklibrary
  * smack-core

功能模块分析:
* Notification 通知模块
* Xmpp 文字聊天部分
* Xmpp 联系人部分
* 图片和文件处理部分
* 一些注意点

### Notification通知模块
这部分功能主要是当有未读消息推送过来的时候，显示通知栏，点击通知栏进入App内部。主要在Notification类和SINotification类中。显示未读条数包括各个房间未读条数和系统未读条数。


### 一些注意点
#### 1.使用TaskStackBuilder
当应用处于后台时，默认情况下，从通知启动一个Activity，按返回键会回到主菜单界面。但遇到这样的需求，按返回键仍然要留在当前应用。类似于微信、QQ等点击通知栏，从Chat界面，点击返回键会回到主Activity。<br/>
在这种情况下，就需要使用TaskStackBuilder方法。需要两步集成
1. 在AndroidManifest.xml配置Activity关系
```
<activity android:name=".MessageActivity"
          android:parentActivityName=".MainActivity"/>
```
2. 在showNotifaction方法里面添加如下代码
```
private void 在showNotifaction(){
  ...
  TaskStackBuilder stackBuilder = TaskStackBuilder.create(mContext);
            stackBuilder.addParentStack(intent.getComponent());
            stackBuilder.addNextIntent(intent);
  ...
}
```
