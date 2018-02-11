## Rxjava相关知识

### 1.Single
Single类似于Observable，不同的是，它总是只发射一个值，或者一个错误通知，而不是发射一系列的值。因此不同于Observable需要三个方法onNext、onError、onComplete，订阅Single只需要两个方法：
* onSuccess - Single发射单个的值到这个方法
* onError - 如果无法发射需要的值，Single发射一个Throwable对象到这个方法

##### Single的操作符


### 2.Subject


### 3.调度器Scheduler




### 4.操作符Operators
#### 4.1 创建操作
用于创建Observable的操作符
* create - 通过调用观察者的方法，从头创建一个Observable
* Defer - 在观察者订阅之前不创建这个Observable,为每一个观察者创建一个新的Observable
* Empty/Never/Throw - 创建行为受限的特殊Observable
* From - 将其它的对象或数据结构转换为Observable
* Interval - 创建一个定时发射证书序列的Observable
* Just - 将对象或者对象集合转换为一个会发射这些对象的Observable
* Range - 创建发射指定范围的证书序列的Observable
* Repeat - 创建重复发射特定的数据或主句序列的Observable
* start - 创建发射一个函数的返回值的Observable
* Timer - 创建在一个指定的延迟之后发射单个数据的Observable
