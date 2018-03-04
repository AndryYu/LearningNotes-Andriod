## HashMap理解
### 1.HashMap的数据结构
在Java编程语言中，最基本的结构就是两种，一个是数组，另一个是模拟指针(引用)，所有的数据结构都可以用这两个基本结构来构造的。HashMap实际上是一个'链表散列'的数据结构，即数组和链表的结合体。HashMap底层就是一个数据结构，数组中的每一项又是一个链表。当新建一个HashMap的时候，就会初始化一个数组。
```
// The table, resized as necessary, length MUST always be a power of two
transient Entry[] table;

static class Entry<K,V> implements Map.Entry<K,V>{
  final K key;
  V value;
  Entry<K,V> next;
  final int hash;
  ...
}
```
可以看出，Entry就是数组中的元素，每个Map.Entry其实就是一个key-value对，它持有一个指向下一个元素的引用，这就构成了链表。

### 2.HashMap的存取实现
#### 存储
```
public V put(K key, V value) {
    // HashMap允许存放null键和null值。
    // 当key为null时，调用putForNullKey方法，将value放置在数组第一个位置。  
    if (key == null)
        return putForNullKey(value);
    // 根据key的keyCode重新计算hash值。
    int hash = hash(key.hashCode());
    // 搜索指定hash值在对应table中的索引。
    int i = indexFor(hash, table.length);
    // 如果 i 索引处的 Entry 不为 null，通过循环不断遍历 e 元素的下一个元素。
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    // 如果i索引处的Entry为null，表明此处还没有Entry。
    modCount++;
    // 将key、value添加到i索引处。
    addEntry(hash, key, value, i);
    return null;
}
```
从上面的源代码可以看出：当我们往HashMap中put元素的时候，先根据key的hashCode重新计算hash值，根据hash值得到这个元素在数组中的位置(即下标)。如果数组该位置上已经存放有其他元素了，那么在这个位置上的元素将以链表的形式存放，新加入的放在链头，最后加入的放在链尾。如果数组该位置上没有元素，就直接将该元素放到此数组中的该位置上。

addEntry(hash, key, value, i)方法根据计算出的hash值，将key-value对放在数组table的i索引处。addEntry是HashMap提供的一个包访问权限的方法，代码如下:
```
void addEntry(int hash, K key, V value, int bucketIndex) {
    // 获取指定 bucketIndex 索引处的 Entry  
    Entry<K,V> e = table[bucketIndex];
    // 将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry  
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    // 如果 Map 中的 key-value 对的数量超过了极限
    if (size++ >= threshold)
    // 把 table 对象的长度扩充到原来的2倍。
        resize(2 * table.length);
}
```
当系统决定存储HashMap中的key-value对时，完全没有考虑Entry中的value，仅仅只是根据key来计算并决定每个Entry的存储位置。我们完全可以把Map集合中的value当成key的附属，当系统决定了key的存储位置之后，value随之保存在那里即可。

根据上面put方法的源代码可以看出，当程序试图将一个key-value对放入HashMap中时，程序首先根据该key的hashCode返回值决定该Entry的存储位置：
* 如果两个Entry的key的hashCode()返回值相同，那它们的存储位置相同。如果这两个Entry的key通过equals比较返回true，新添加Entry的value将覆盖集合中原有Entry的value，但key不用覆盖。
* 如果两个Entry的key通过equals比较返回false，新添加的Entry将与集合中原有Entry形成Entry链，并且新添加的Entry位于Entry链的头部——具体说明继续看addEntry()方法的说明。

#### 读取
```
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
        e != null;
        e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))  
            return e.value;
    }
    return null;
}
```
从上面的源代码中可以看出：从HashMap中get元素时，首先计算key的hashCode，找到数组中对应位置的某一元素，然后通过key的equals方法在对应位置的链表中找到需要的元素。

### 3.HashMap的resize
HashMap数组扩容之后，最消耗性能的点就出现了:原数组中的数据必须重新计算其在新数组中的位置，并放进去，这就是resize.

那么HashMap什么时候进行扩容呢？当HashMap中的元素个数超过数组大小loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，这是一个折中的取值。也就是说，默认情况下，数组大小为16，那么当HashMap中元素个数超过16*0.75=12的时候，就把数组的大小扩展为2*16=32，即扩大一倍。然后重新计算每个元素在数组中的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数能够有效的提高HashMap的性能。

### 4.HashMap的性能参数
HashMap包含如下几个构造器:
1. HashMap()  构造一个初始容量为16，负载因子为0.75的HashMap.
2. HashMap(int initalCapacity)  构造一个初始容量为initalCapacity，负载因子为0.75的HashMap
3. HashMap(int initalCapacity, float loadFactor) 构造一个初始容量为initalCapacity，负载因子为loadFactor的HashMap

负载因子loadFactor衡量的是一个散列表的空间使用程度，负载因子越大表示散列表的装填程度越高，反之越小。对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。

HashMap的实现中，通过threshold字段来判断HashMap的最大容量。
```
threshold = (int)(capacity * loadFactor);
```
结合负载因子的定义公式可知，threshold就是在此loadFactor和capacity对应下允许的最大元素数目，超过这个数目就重新resize，以降低实际的负载因子。默认的负载因子0.75是对空间和时间效率的一个平衡选择。当容量超出此最大容量时，resize后的HashMap容量时原来容量的两倍。

### 5.Fail-Fast机制
我们知道java.util.HashMap不是线程安全的，因此如果在使用迭代器的过程中，有其他线程修改了map，那么将抛出ConcurrentModificationException,这就是所谓fail-Fast策略。

这一策略在源码中的实现是通过modCount域，modCount顾名思义就是修改次数。对HashMap内容的修改都将增加这个值，那么在迭代器初始化过程中会将这个值赋给迭代器的expectedModCount。
```
HashIterator() {
    expectedModCount = modCount;
    if (size > 0) { // advance to first entry
    Entry[] t = table;
    while (index < t.length && (next = t[index++]) == null)  
        ;
    }
}
```
在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了Map。注意到modCount声明为volatile，保证线程之间修改的可见性。
```
final Entry<K,V> nextEntry() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
```

### 6.HashMap的两种遍历方式
**第一种**
```
  Map map = new HashMap();
  Iterator iter = map.entrySet().Iterator();
  while(ite.hasNext()){
    Map.Entry entry = (Map.Entry)iter.next();
    Object key = entry.getKey();
    Object val = entry.getValue();
  }
```
效率高，以后一定要使用此种方式。

**第二种**
```
  Map map = new HashMap();
  Iterator iter = map.keySet().iterator();
  while(iter.hasNext()){
    Object key = iter.next();
    Object val = map.get(key);
  }
```
效率低，以后尽量少使用。
