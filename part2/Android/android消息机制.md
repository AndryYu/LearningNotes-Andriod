## Android消息机制(Handler的运行机制)
Android消息机制主要是指Handler的运行机制，Handler的运行需要底层的MessageQueue和Looper的支撑。**MessageQueue** 的内部存储了一组消息，以队列的形式对外提供插入和删除的工作。虽然叫消息队列，但是它的内部存储结构并不是真正的队列，而是采用单链表的数据机构来存储消息列表。**Looper** 会以无限循环的形式来查找是否有新消息，如果有的话就处理消息，否则就一直等待着。Looper中还有一个特殊的概念，那就是 **ThreadLocal**，ThreadLocal并不是线程，它的作用是可以在每个线程中存储数据。Handler创建的时候会采用当前线程的Looper来构造消息循环系统，那么Handler内部如何获取到当前线程的Looper呢？ 这就要使用ThreadLocal,ThreadLocal可以在不同的线程中互不干扰地存储并提供数据，通过ThreadLocal可以轻松获取每个线程的Looper。当然需要注意的是，线程是默认没有Looper的，如果需要使用Handler就必须为线程创建Looper。

### Android消息机制分析
#### 1.ThreadLocal的工作原理
ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程来说则无法获取到数据。

ThreadLocal的使用场景：
* 当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，可以考虑采用ThreadLocal;
* 复杂逻辑下的对象传递，比如监听器的传递，有些时候一个线程中的任务过于复杂，这可能表现为函数调用栈比较深以及代码入口的多样性，在这种情况下，我们有需要监听器能够贯穿整个线程的执行过程，可以采用ThreadLocal,采用ThreadLocal可以让监听器作为线程内的全局对象而存在，在线程内部只要通过get方法就可以获取到监听器。

```
  private ThreadLocal<Boolean> threadLocal = new ThreadLocal<Boolean>();
  ...
  threadLocal.set(true);  //主线程
  Log.d(TAG, "[Thread#Main]threadLocal="+threadLocal.get());

  new Thread("Thread#1"){ //线程1
    @override
    public void run(){
      threadLocal.set(false);  
      Log.d(TAG, "[Thread#Main]threadLocal="+threadLocal.get());
    }
  }.start();

  new Thread("Thread#2"){ //线程2
    @override
    public void run(){
      Log.d(TAG, "[Thread#Main]threadLocal="+threadLocal.get());
    }
  }.start();
```
运行日志如下：
```
   [Thread#main]threadLocal=true
   [Thread#1]threadLocal=false
   [Thread#2]threadLocal=null
```
虽然在不同线程中访问的是同一个ThreadLocal对象，但是它们通过ThreadLocal获取到的值却是不一样的。ThreadLocal之所以有这么奇妙的效果，是因为不同线程访问同一个ThreadLocal的get方法，ThreadLocal内部会从各自的线程中取出一个数组，然后再从数组中根据当前ThreadLocal的索引去查找出对应的value值。很显然，不同线程中的数组时不同的，这就是为什么通过ThreadLocal可以在不同的线程中维护一套数据的副本，并且彼此互不干扰。

#### 2.消息队列的工作原理
MessageQueue主要包含两个操作：插入和读取。读取操作本身会伴随着删除操作，插入和读取对应的方法分别为enqueueMessage和next，其中enqueueMessage的作用是往消息队列中插入一条消息，而next的作用是从消息队列中取出一条消息并将其从消息队列中移除。MessageQueue实际上是通过一个单链表的数据结构来维护消息列表，单链表在插入和删除上比较有优势。

#### 3.Looper的工作原理
Looper会不停地从MessageQueue中查看是否有新消息，如果有新消息就会立刻处理，否则就一直阻塞在那里。首先看一下它的构造方法，在构造方法中它会创建一个MessageQueue，然后将当前线程的对象保存起来
```
private Looper(boolean quitAllowed){
  mQueue = new MessageQueue(quitAllowed);
  mThread = Thread.currentThread();
}
```
通过Looper.prepare()即可为当前线程创建一个Looper，接着通过Looper.loop()来开启消息循环。<br/>
Looper的loop方法的工作过程也比较好理解，loop方法是一个死循环，唯一跳出循环的方式是MessageQueue的next方法返回了null。当Looper的quit方法被调用时，Looper就会调用MessageQueue的quit或者quitSafely方法来通知消息队列退出。当消息队列被标记为退出状态时，它的next方法就会返回null。也就是说，Looper必须退出，否则loop方法就会无限循环下去。当MessageQueue的next方法返回了新消息，Looper就会处理这条消息：msg.target.dispatchMessage(msg),这里的msg.target是发送这条消息的Handler对象，这样Handler发送的消息最终又交给它的dispatchmessage方法来处理了。但是这里不同的是，Handler的dispatchMessage方法是在创建Handler时所使用的Looper中执行的，这样就成功地将代码逻辑切换到指定的线程中去执行了。

#### 4.Handler的工作原理
Handler的工作主要包含消息的发送和接收过程。消息的发送可以通过post的一系列方法以及send的一系列方法来实现，post的一系列方法最终是通过send的一系列方法来实现的。最后，调用Handler的hanleMessage方法来处理消息。














### 主线程的消息循环
Android的主线程就是ActivityThread，主线程的入口方法为main，在main方法中系统会通过Looper.prepareMainLooper()来创建主线程的Looper以及MessageQueue，并通过Looper.loop来开启主线程的消息循环。
