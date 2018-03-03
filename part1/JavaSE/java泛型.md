## Java泛型详解
### 泛型基础
#### 1.泛型类
我们首先顶一个简单的Box类:
```
  public class Box {
      private String object;
      public void set(String object) { this.object = object; }
      public String get() { return object; }
  }
```
这是最常见的做法，这样做的一个坏处是Box里面只能装入String类型的元素，今后如果我们需要装入Integer等其他类型的元素，还必须要另外重写一个Box，代码得不到复用，使用泛型可以很好的解决这个问题。
```
  public class Box<T>{
    //T stands for 'Type'
    private T t;
    public void set(T t){ this.t = t; }
    public T get(){return t;}
  }
```
这样我们的Box类便可以得到复用，我们可以将T替换成任何我们想要的类型:
```
  Box<Integer> intergerBox = new Box<Integer>();
  Box<Double> intergerBox = new Box<Double>();
  Box<String> intergerBox = new Box<String>();
```

#### 2.泛型方法
看完了泛型类，接下来我们来了解一下泛型方法。声明一个泛型方法很简单，只要在返回类型前面加一个类似<K,V>的形式就行了。
```
public class Util {
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
               p1.getValue().equals(p2.getValue());
    }
}
public class Pair<K, V> {
    private K key;
    private V value;
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
    public K getKey()   { return key; }
    public V getValue() { return value; }
}
```
我们可以像下面这样去调用泛型方法:
```
  Pair<Integer, String> p1 = new Pair<>(1, "apple");
  Pair<Integer, String> p2 = new Pair<>(2, "pear");
  boolean same = Util.<Integer, String>compare(p1, p2);
```
或者在Java 1.7/1.8利用type inference，让Java自动推导出相应地类型参数:
```
  Pair<Integer, String> p1 = new Pair<>(1, "apple");
  Pair<Integer, String> p2 = new Pair<>(2, "pear");
  boolean same = Util.compare(p1, p2);
```

#### 3.边界符
现在我们要实现这样一个功能，查找一个泛型数组中大于某个特定元素的个数，我们可以这样实现:
```
public static <T> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e > elem)  // compiler error
            ++count;
    return count;
}
```
但是这样很明显是错误的，因为除了short, int, double, long, float, byte, char等原始类型，其他的类并不一定能使用操作符>，所以编译器报错，那怎么解决这个问题呢？答案是使用边界符。
```
public interface Comparable<T> {
    public int compareTo(T o);
}
```
做一个类似于下面这样的声明，这样就等于告诉编译器类型参数T代表的都是实现了Comparable接口的类，这样等于告诉编译器它们都至少实现了compareTo方法。
```
public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e.compareTo(elem) > 0)
            ++count;
    return count;
}
```

### 通配符
在了解通配符之前，我们首先必须要澄清一个概念，还是借用我们上面定义的Box类，假设我们添加一个这样的方法：
```
  public void boxTest(Box<Number> n) { /* ... */ }
```
那么现在Box<Number> n允许接受什么类型的参数？我们是否能够传入Box<Integer>或者Box<Double>呢？答案是否定的，虽然Integer和Double是Number的子类，但是在泛型中Box<Integer>或者Box<Double>与Box<Number>之间并没有任何的关系。这一点非常重要，接下来我们通过一个完整的例子来加深一下理解。

首先我们先定义几个简单的类，下面我们将用到它：
```
    class Fruit {}
    class Apple extends Fruit {}
    class Orange extends Fruit {}
```
下面这个例子中，我们创建了一个泛型类Reader，然后在f1()中当我们尝试Fruit f = fruitReader.readExact(apples);编译器会报错，因为List<Fruit>与List<Apple>之间并没有任何的关系。
```
public class GenericReading {
    static List<Apple> apples = Arrays.asList(new Apple());
    static List<Fruit> fruit = Arrays.asList(new Fruit());
    static class Reader<T> {
        T readExact(List<T> list) {
            return list.get(0);
        }
    }
    static void f1() {
        Reader<Fruit> fruitReader = new Reader<Fruit>();
        // Errors: List<Fruit> cannot be applied to List<Apple>.
        // Fruit f = fruitReader.readExact(apples);
    }
    public static void main(String[] args) {
        f1();
    }
}
```
但是按照我们通常的思维习惯，Apple和Fruit之间肯定是存在联系，然而编译器却无法识别，那怎么在泛型代码中解决这个问题呢？我们可以通过使用通配符来解决这个问题：
```
static class CovariantReader<T> {
    T readCovariant(List<? extends T> list) {
        return list.get(0);
    }
}
static void f2() {
    CovariantReader<Fruit> fruitReader = new CovariantReader<Fruit>();
    Fruit f = fruitReader.readCovariant(fruit);
    Fruit a = fruitReader.readCovariant(apples);
}
public static void main(String[] args) {
    f2();
}
```
这样就相当与告诉编译器， fruitReader的readCovariant方法接受的参数只要是满足Fruit的子类就行(包括Fruit自身)，这样子类和父类之间的关系也就关联上了。

### PECS原则
* “Producer Extends” – 如果你需要一个只读List，用它来produce T，那么使用? extends T。
* “Consumer Super” – 如果你需要一个只写List，用它来consume T，那么使用? super T。
* 如果需要同时读取以及写入，那么我们就不能使用通配符了。

### 类型擦除
