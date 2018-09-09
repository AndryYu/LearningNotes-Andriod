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
* 本地数据存储
* service保活机制
* Xmpp 文字聊天部分
* Xmpp 联系人部分
* 图片和文件处理部分
* 一些注意点

### Notification通知模块
这部分功能主要是当有未读消息推送过来的时候，显示通知栏，点击通知栏进入App内部。主要在Notification类和SINotification类中。显示未读条数包括各个房间未读条数和系统未读条数。

### 本地数据存储
这部分主要包括本地数据库创建和维护，每个表的意义。数据库表总共有如下
1. SIRoomShow(聊天窗口)
房间类型：0.表示系统信息；1.表示单人聊天；2.表示群组聊天
2. SISystemMessage(系统信息表)<br/>
消息类型几种对应关系：0.初始值；1.好友邀请；2.群聊邀请；3.被群踢出的信息；4.群解散信息；5.系统消息；6.升级通知
3. SIUser(用户信息表)

4. SIChatMessage
聊天消息，每一条聊天消息都是一个SIChatMsg对象。
5. SIChateMultiRoom(多人群聊表)

6. SIChatRoomMember(群聊成员关联表)
群聊中包含的成员关联表，取出时最好按照ID从小到大排序，可以因此判断成员进群的先后顺序。
7. SIChatSingleRoom

8. SIFile(用户发送的文件对象表)
包括file文件、image、video、audio/aar、状态(下载/上传)。
9. SIGroup(好友分组表)
该表包含组名、相关描述、联系人列表。
10. SIGroupMembers

11. SIContact(联系人表)
从Server获得的联系人Item，smack中是一个Roster Item对象。每一个具体的联系人对应一个SIContact对象。

12. SIVCard(个人交换信息)

13. SIChatSystemRoom(系统通知栏显示信息)

14. SIMessageEffectiveRange(有效离线信息范围)
该表中记录着关联用户信息、消息类型、起始时间和结束时间。

### Service保活机制
通过broadcastReceiver广播监听应用启动，然后在启动Service时，做一个AlarmManager定时(1分钟)检测操作。



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
