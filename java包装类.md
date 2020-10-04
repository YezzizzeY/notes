---
title: java包装类
date: 2019-10-22 18:34:26
tags: java
---
<details>
<summary>详细</summary>
java的基本数据类型都有包装类，所谓包装类，就是为了能将这些基本数据类型当做对象操作，从而有更多对它们的常用操作方法，并有自动装箱/拆箱机制使得二者可以相互转换。包装类继承自Number类。

其中Number类继承自java.io.Serializable序列化接口，有实现各类值的函数，源码定义如下：

```java
public abstract class Number implements java.io.Serializable {

public abstract int intValue();

public abstract long longValue();

public abstract float floatValue();

public abstract double doubleValue();

public byte byteValue() {
    return (byte)intValue();
}

public short shortValue() {
    return (short)intValue();
}

private static final long serialVersionUID = -8742448824652078965L;
}

```
所有基本类型的包装类型都继承了该抽象类，并且是final声明不可继承改变；



原始类型：boolean，char，byte，short，int，long，float，double

包装类型：Boolean，Character，Byte，Short，Integer，Long，Float，Double



除了实现Number类的方法，每个包装类又实现了许多详细的方法，以Int包装类型Integer为例：

 Integer.ParseInt()

```java
String hash = "02138975";
System.out.println(Integer.parseInt(hash));  //2138975
```

 任何类型+""变成字符串

```java
Integer i = 128;
String hash = "02138975";
System.out.println(i+hash);  //12802138975
```

toString()方法

```java
Integer i = 128;
System.out.println(i.toString());  //128
```

其他方法，

```java
toStringUTF16(int i, int radix)
toUnsignedString(int i, int radix)
toHexString(int i)
compare(int x, int y)
```

等等，就不一一列举了。源码在java.long中。



###### 自动拆箱与装箱

在使用Integer类型当作int类型使用时，会感觉和正常的int没有多大区别，自动装箱和拆箱的好处就在这。

```java
Integer src = 100; //自动装箱的过程，相当于Integer src = new Integer(100);
int dest = src + 5; //src本身是引用数据类型，不能直接跟基本数据类型运算，首先它会自动进行拆箱操作，相当于：int dest = src.intValue() + 5 ;
```

不过需要注意的是，当Integer类型的值在-128-127之间时，会读取缓存而不创建新的对象：比如：

```java
Integer a = 500;

Integer b = 500;

Integer c = 100;

Integer d = 100;
```

这时a和b的值相等，但是是两个对象；c和d的值相等，而且是一个对象。

```java
a==b : false

a.equals(b) : true

c==d : true

c.equals(d) : true
```

原因是源码的valueOf()函数，

```java
public static Integer valueOf(int i) {    
  if (i >= IntegerCache.low && i <= IntegerCache.high)   return IntegerCache.cache[i + (-  IntegerCache.low)];  
  return new Integer(i);
}
private static class IntegerCache {
  static final int low = -128;
  static final int high;
```

这个函数对于-128到127之外的数，会新建一个Integer对象并返回 ， 写a==b时，两个对象不一样，而对再这个范围之间的数用的一个对象，所以相等。 

另外需要注意的是：

- int和Integer(无论new否)比，都为true，因为会把Integer自动拆箱为int再去比 。 

- Integer与new Integer不会相等。不会经历拆箱过程，new出来的对象存放在堆，而非new的Integer常量则在常量池（在方法区），他们的内存地址不一样 

- Integer初值为null，int初值为0

  </details>