## Java中的==、hashCode和equals的区别
### 1.==
java中的数据类型，可分为两类：
1. 基本数据类型，如byte,short,char,int,long,float,double,boolean。他们之间的比较，使用双等号(==)，比较的是他们的**值**。
2. 引用类型(类、接口、数组)<br/>
  当他们用(==)进行比较的时候，**比较的是他们在内存中的存放地址**，所以除非是同一个new出来的对象，他们的比较后结果为true，否则比较后结果为false。

对象是放在堆中的，栈中存放的是对象的引用(地址)，由此可见'=='是对栈中的值进行比较的。如果要比较堆中对象的内容是否相同，那么就要重写equals方法了。

### 2.equals
1. 默认情况下(没有覆盖equals方法)equals方法都是调用Object类的equals方法，而Object的equals方法主要用于判断 **对象的内存地址引用是否同一个地址(是不是同一个对象)。** 下面是Object类中equals方法:
```
public boolean equals(Object obj){
  return (this==obj);
}
```
定义的equals与==等效的。
2. 要是类中覆盖了equals方法，那么就要根据具体的代码来确定equals方法的作用了，**覆盖后一般都是通过对象的内容是否相等来判断对象是否相等。** 下面是String类对equals进行了重写：
```
public boolean equals(Object anObject) {  
    if (this == anObject) {  
        return true;  
    }  
    if (anObject instanceof String) {  
        String anotherString = (String)anObject;  
        int n = count;  
        if (n == anotherString.count) {  
        char v1[] = value;  
        char v2[] = anotherString.value;  
        int i = offset;  
        int j = anotherString.offset;  
        while (n-- != 0) {  
            if (v1[i++] != v2[j++])  
            return false;  
        }  
        return true;  
        }  
    }  
    return false;  
    }  
```
即String中equals方法判断相等的步骤是：
  1. 若A==B 即是同一个String对象，返回true
  2. 若对比对象是String类型则继续，否则返回false
  3. 判断A、B长度是否一样，不一样的话返回false
  4. 逐个字符比较，若有不相等字符，返回false

这里对equals重新需要注意五点
1. 自反性：对任意引用值x，x.equals(x)的返回值一定为true
2. 对称性：对于任何引用值x、y,当且仅当y.equals(x)返回值为true时，x.equals(y)的返回值一定为true；
3. 传递性：如果x.equals(y)=true,y.equals(z)=true,则x.equals(z)=返回true
4. 一致性：如果参与比较的对象没任何改变，则对象比较的结果也不应该有任何改变
5. 非空性：任何非空的引用值x，x.equals(null)的返回值一定为false

实现高质量equals方法的诀窍：
1. **使用==符号检查"参数是否为这个对象的引用"**。如果是，则返回true，这只不过是一种性能优化，如果比较操作有可能很昂贵，就值得这么做。
2. **使用instanceof操作符检查"参数是否为正确的类型"**。如果不是，则返回false。一般来说，所谓"正确的类型"是指equals方法所在的那个类。
3. 把参数转换成正确的类型。因此转换之前进行过instanceof测试，所以确保会成功。
4. 对于该类中的每个'关键'域，检查参数中的域是否与该对象中对应的域相匹配。如果这些测试全部成功，则返回true；否则返回false。
5. 当编写完成了equals方法之后，检查'对称性'、'传递性'、'一致性'。

### 3.hashCode
hashCode()方法返回的就是一个数值，从方法的名称上就可以看出，其目的是生成一个hash码。hash码的主要用途就是在对对象进行散列的时候作为key输入，据此很容易推断出，我们需要每个对象的hash码尽可能不同，这样才能保证散列的存取性能。事实上，Object类提供的默认实现确定保证每个对象的hash码不同(在对象的内存基础上经过特定算法返回一个hash码)。java采用了哈希表的原理。哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。

散列函数将任意长度的二进制值映射为较短的固定长度的二进制值，这个小的二进制值称为哈希值。好的散列函数在输入域中很少出现散列冲突。

#### hashCode的作用
当集合要添加新的元素时，先调用这个元素的hashCode方法，就一下子能定位到它应该放置的物理位置上。如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了;如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存。不同就散列到其它的地址。这里存在一个冲突解决的问题。这样一来实际调用equals方法的次数就大大降低了，几乎只需要一二次。

### 4.equals方法和hashCode方法关系
java对于equals方法和hashCode方法是这样规定的：
1. 同一对象上多次调用hashCode()方法，总是返回相同的整形值。
2. 如果a.equals(b)，则一定有a.hashCode()==b.hashCode()。
3. 如果!a.equals(b)，则a.hashCode不一定等于b.hashCode().
4. a.hashCode()==b.hashCode() 则 a.equals(b)可真可假
5. a.hashCode()！= b.hashCode() 则 a.equals(b)为假。

上面结论简记
1. 如果两个对象equals，Java运行时环境会认为他们的hashcode一定相等。
2. 如果两个对象不equals，他们的hashcode有可能相等。
3. 如果两个对象hashcode相等，他们不一定equals。
4. 如果两个对象hashcode不相等，他们一定不equals。

关于这两个方法的重要规范：
**规范一：** 若重写equals(Object obj)方法，有必要重写hashcode()方法，确保通过equals(Object obj)方法判断结果为true的两个对象具备相等的hashcode()返回值。说得简单点就是：“如果两个对象相同，那么他们的hashcode应该相等”。

**规范二：** 如果equals(Object obj)返回false，即两个对象“不相同”，并不要求对这两个对象调用hashcode()方法得到两个不相同的数。说的简单点就是：“如果两个对象不相同，他们的hashcode可能相同”。

### 5.为什么覆盖equals时总要覆盖hashCode
一个很常见的错误根源在于没有覆盖hashCode方法。在每个覆盖了equals方法的类中，也必须覆盖hashCode方法。如果不这样做的话，就会违反Object.hashCode的通用约定，从而导致该类无法结合所有基于散列的集合一起正常运作，这样的集合包括HashMap、HashSet和Hashtable。
1. 在应用程序的执行期间，只要对象的equals方法的比较操作所用到的信息没有被修改，那么对这同一个对象调用多次，hashCode方法都必须始终如一地返回同一个整数。在同一个应用程序的多次执行过程中，每次执行所返回的整数可以不一致。
2. 如果两个对象根据equals()方法比较是相等的，那么调用这两个对象中任意一个对象的hashCode方法都必须产生同样的整数结果。
3. 如果两个对象根据equals()方法比较是不相等的，那么调用这两个对象中任意一个对象的hashCode方法，则不一定要产生相同的整数结果。但是程序员应该知道，给不相等的对象产生截然不同的整数结果，有可能提高散列表的性能。
