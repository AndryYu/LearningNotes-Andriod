## Java特性之注解
### 元注解
元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，但是它能够应用到其他的注解上面。<br/>
元注解有@Retention、@Documented、@Target、@Inherit、@Repeatable 5种。

**@Retention**<br/>
Retention的英文意为保留期的意思。当@Retention应用到一个注解上的时候，它解释说明了这个注解的存活时间。它的取值如下：
* RetentionPolicy.SOURCE 注解只在源码阶段逗留，在编译器进行编译时它将被丢弃忽视。
* RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到JVM中。
* RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到JVM中，所以在程序运行时可以获取到它们。

**@Documented**<br/>
顾名思义，这个元注解肯定是和文档有关。它的作用是能够将注解中的元素包含到Javadoc中去。

**@Target**<br/>
Target是目标的意思，@Target指定了注解运用的地方。@Target有下面的取值：
* ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
* ElementType.CONSTRUCTOR  可以给构造方法进行注解
* ElementType.FIELD 可以给属性进行注解
* ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
* ElementType.METHOD 可以给方法进行注解
* ElementType.PACKAGE 可以给一个包进行注解
* ElementType.PARAMETER 可以给一个方法内的参数进行注解
* ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举

**@Inherited**<br/>
Inherited是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被@Inherited注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。
```
  @Inherited
  @Retention(RetentionPolicy.RUNTIME)
  @interface Test {}


  @Test
  public class A {}


  public class B extends A {}
```
注解 Test 被 @Inherited 修饰，之后类 A 被 Test 注解，类 B 继承 A,类 B 也拥有 Test 这个注解。

**@Repeatable**<br/>
Repeatable自然是可重复的意思。@Repeatable是Java 1.8才加进来的，所以算是一个新的特性。什么样的注解会多次应用呢？通常是注解的值可以同时取多个。举个例子，一个人他既是程序员又是产品经理,同时他还是个画家。
```
  @interface Persons {
      Person[]  value();
  }


  @Repeatable(Persons.class)
  @interface Person{
      String role default "";
  }


  @Person(role="artist")
  @Person(role="coder")
  @Person(role="PM")
  public class SuperMan{

  }
```
注意上面的代码，@Repeatable 注解了 Person。而 @Repeatable 后面括号中的类相当于一个容器注解。

什么是容器注解呢？就是用来存放其它注解的地方。它本身也是一个注解。

### 注解的属性
注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以'无形参的方法'形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。<br/>
注解中属性可以由默认值，默认值需要用default关键值指定。比如：
```
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface TestAnnotation {

      public int id() default -1;

      public String msg() default "Hi";
  }
```
如果注解只有value这个属性，可以这样应用
```
  @Check("hi")
  int a;
```

### Java预置的注解
**@Deprecated**<br/>
这个元素是用来标记过时的元素。编译器在编译阶段遇到这个注解会发出提醒警告，告诉开发者正在调用一个过时的元素，比如过时的方法、过时的类、过时的成员变量。

**@Override**<br/>
提示子类要复写父类中被@Override修饰的方法。

**@SuppressWarnings**<br/>
阻止警告的意思，之前说过调用被@Deprecated注解的方法后，编译器会警告提醒，而有时候开发者会忽略这种警告，他们可以在调用的地方通过@SuppressWarnings达到目的。

**@SafeVarargs**<br/>
参数安全类型注解。它的目的是提醒开发者不要用参数做一些不安全的操作，它的存在会阻止编译器产生unchecked这样的警告。它是在Java 1.7的版本中加入的。

**@FunctionalInterface**<br/>
函数式接口注解，这个是Java 1.8版本引入的新特性。函数式接口(Functional Interface)就是一个具有一个方法的普通接口。
```
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

### 注解的提取
要想正确检阅注解，离不开一个手段，那就是反射。
#### 注解与反射
注解通过反射获取。首先可以通过 Class 对象的 isAnnotationPresent() 方法判断它是否应用了某个注解
```
  public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}
```
然后通过getAnnotation()方法来获取Annotation对象。
```
  public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}
```
或者是getAnnotations()方法。
```
  public Annotation[] getAnnotations() {}
```
前一种方法返回指定类型的注解，后一种方法返回注解到这个元素上的所有注解。如果获取到Annotation不为null，则就可以调用它们的属性方法。

### 注解的使用场景
当开发者使用了Annotation修饰了类、方法、Field等成员之后，这些Annotation不会自己生效，必须由开发者提供相应地代码来提取并处理Annotation信息。这些处理提取和处理Annotation的代码统称为APT(Annotation Processing Tool).

### 注解应用实例
**JUnit**<br/>
一个测试框架

**ButterKnife**<br/>
Android开发中的IOC框架

**Dagger2**<br/>
有名的依赖注入框架

**Retrofit**<br/>
http网络访问框架

### ButterKnife
运行时通过反射完成注入，会对性能有一定的损耗。ButterKnife用的是APT编译时解析技术。它在Java代码编译成Java字节码时就已经处理了@BindView、@OnClick这些注解。它的原理是读入Java源代码，解析注解，生成新的Java代码，新生成的Java代码最后被编译成Java字节码。
