## 四大组件之Activity、Fragment
### 1.Activity的启动模式
#### standard
默认的启动模式，这种模式每次都会创建新的实例，每次点击standard模式会创建Activity后，都会创建新的Activity覆盖在原Activity上。

#### singleTo
如果指定启动Activity为singleTop模式，那么在启动时，系统会判断当前栈顶Activity是不是要启动的Activity，如果是则不创建新的Activity而直接引用这个Activity；如果不是则创建新的Activity。这种启动模式**通常用于接收到消息后显示的界面，例如QQ接收到消息后弹出Activity**。这种启动模式虽然不会创建新的实例，但是系统仍然会在Activity启动时调用**onNewIntent()** 方法。

#### singleTask
singleTask模式与singleTop模式类似，只不过singleTop是检测栈顶元素是否是需要启动的Activity，而singleTask是检测整个Activity栈中是否存在当前需要启动的Activity。如果存在，则将该Activity置于栈顶，并将该Activity以上的Activity都销毁。不过这里是指在同一个App中启动这个singleTask的Activity，如果是其他程序以singleTask模式来启动这个Activity，那么它将创建一个新的任务栈。不过这里有一点需要注意的是，如果启动的模式为singleTask的Activity已经在后台一个任务栈中了，那么启动后，后台的这个任务栈将一起被切换到前台。<br/>
这种启动模式通常可以用来退出整个应用：将主Activity设为singleTask模式，然后在要退出的Activity中转到主Activity，从而将主Activity之上的Activity都清除，然后重写主Activity的onNewIntent()方法，在方法中加上一句finish()，将最后一个Activity结束掉。

#### SingleInstance
声明为singleInstance的Activity会出现在一个新的任务栈中，而且该任务栈中只存在这一个Activity。举个例子来说，如果应用A的任务栈中创建了MainActivity实例，且启动模式为singleInstance，如果应用B也要激活MainActivity，则不需要创建，两个应用共享该Activity实例。这种启动模式常用于需要与程序分离的界面，如在SetupWizard中调用紧急呼叫，就是使用这种启动模式。



### 2.Fragment add和replace的区别
首先获取FragmentTransaction对象:
```
   FragmentTransaction transaction = fm.beginTransaction();
```
**添加**
```
  transaction.add(R.id.fragment_container, oneFragment).commit();
```
添加进来的fragment都是可见的(visible)，后添加的fragment会展示在先添加的fragment上面，在绘制界面的时候会绘制所有可见的view，所以大多数add都是和hide或者remove同时使用的，这样可以节省绘制界面时间和内存消耗。

**替换**
```
  transaction.replace(R.id.fragment_container, oneFragment).commit();
```
替换会把容器中的所有内容全部替换掉，有一些app会使用这样的做法，保持只有一个fragment在显示，减少了界面的层级关系。
