<!-- TOC -->

- [数据类型](#数据类型)
    - [基本数据类型](#基本数据类型)
    - [包装类型](#包装类型)
    - [缓存池](#缓存池)
- [String](#string)
    - [概述](#概述)
    - [String不可变原因](#string不可变原因)
        - [1.缓存hash值](#1缓存hash值)
        - [2.String Pool需要](#2string-pool需要)
        - [3.安全性](#3安全性)
        - [4.线程安全](#4线程安全)
    - [String/StringBuffer/StringBuilder](#stringstringbufferstringbuilder)
        - [1.是否可变](#1是否可变)
        - [2.线程是否安全](#2线程是否安全)
    - [String Pool](#string-pool)
    - [new String("a")](#new-stringa)
- [运算](#运算)
    - [参数传递](#参数传递)
    - [float和double](#float和double)
    - [隐式类型转换](#隐式类型转换)
- [关键字](#关键字)
    - [final](#final)
        - [1. 数据](#1-数据)
        - [2. 方法](#2-方法)
        - [3. 类](#3-类)
    - [static](#static)
    - [finally](#finally)
    - [volatile](#volatile)
    - [sychronized](#sychronized)
    - [transient](#transient)
- [Object方法](#object方法)
    - [概述](#概述-1)
    - [equals](#equals)
        - [1. 等价关系](#1-等价关系)
        - [2. 等价与相等](#2-等价与相等)
        - [3. 实现](#3-实现)
    - [hashCode()](#hashcode)
    - [toString()](#tostring)
    - [clone()](#clone)
        - [1. cloneable](#1-cloneable)
        - [2. 浅拷贝](#2-浅拷贝)
        - [3. 深拷贝](#3-深拷贝)
        - [4. clone()替代方案](#4-clone替代方案)
- [继承](#继承)
    - [访问权限](#访问权限)
    - [抽象类与接口](#抽象类与接口)
        - [抽象类](#抽象类)
        - [接口](#接口)
    - [super](#super)
    - [重写与重载](#重写与重载)
        - [@Override](#override)
        - [Overload](#overload)
- [反射](#反射)
- [异常](#异常)
- [泛型](#泛型)
- [注解](#注解)

<!-- /TOC -->
# 数据类型
- byte/8
- char/16
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。

- [Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)
- [The Java® Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf)
## 基本数据类型
## 包装类型
## 缓存池
# String
## 概述
String被声明为final，不可被继承。String对象新建后不可变。
```
// Java 8中使用char数组存储数据
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```
```
// Java 9使用byte数组存储String，同时使用coder标识使用哪种编码
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```
## String不可变原因
### 1.缓存hash值
### 2.String Pool需要
String Pool存在Heap中，String对象创建后，相同的String创建时会从String Pool中直接使用引用，提升效率
### 3.安全性
String经常在网络传输、系统接口传输中作为参数，参数不可变保证网络传输数据的安全性
### 4.线程安全
[Program Creek : Why String is immutable in Java?](https://www.programcreek.com/2013/04/why-string-is-immutable-in-java/)
## String/StringBuffer/StringBuilder
### 1.是否可变
- String不可变
- StringBuffer/StringBuilder可变
### 2.线程是否安全
- String/StringBuffer线程安全
- StringBuilder线程不安全

[String/StringBuffer/StringBUilder](https://stackoverflow.com/questions/2971315/string-stringbuffer-and-stringbuilder)

[String/StringBuffer/StringBuilder实现原理](https://www.jianshu.com/p/64519f1b1137)
## String Pool
在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

[深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)

## new String("a")

# 运算
## 参数传递
Java 的参数是以值传递的形式传入方法中，而不是引用传递。

以下代码中 Dog dog 的 dog 是一个指针，存储的是对象的地址。在将一个参数传入一个方法时，本质上是将对象的地址以值的方式传递到形参中。
```
public class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        func(dog);
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        System.out.println(dog.getName());          // A
    }

    private static void func(Dog dog) {
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        dog = new Dog("B");
        System.out.println(dog.getObjectAddress()); // Dog@74a14482
        System.out.println(dog.getName());          // B
    }
}
```
[Is Java “pass-by-reference” or “pass-by-value”?
](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)

## float和double
Java 不能隐式执行向下转型，因为这会使得精度降低。

1.1 字面量属于 double 类型，不能直接将 1.1 直接赋值给 float 变量，因为这是向下转型。

```
// float f = 1.1;
```
1.1f 字面量才是 float 类型。

```
float f = 1.1f;
```

## 隐式类型转换

# 关键字
## final
### 1. 数据

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

对于基本类型，final 使数值不变；
对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。
```
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

### 2. 方法

声明方法不能被子类重写。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

### 3. 类

声明类不允许被继承。

## static
存在继承的情况下，初始化顺序为：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

## finally
- finally代码块是在try代码块中的return语句执行之后，返回之前执行的。
- finally代码块中的return语句覆盖try代码块中的return语句。
- try代码块中的return语句在异常的情况下不会被执行，这样具体返回哪个看情况；catch中的return执行情况与未发生异常时try中return的执行情况完全一样。

## volatile
防止指令重排序
[深度解析volatile禁止指令重排序](http://swiftlet.net/archives/3321)

[volatile原理](https://www.cnblogs.com/shan1393/p/8999683.html)
## sychronized

## transient

# Object方法
## 概述
```
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```
## equals
### 1. 等价关系

两个对象具有等价关系，需要满足以下五个条件：

- 自反性 x.equals(x); // true
- 对称性 x.equals(y) == y.equals(x); // true
- 传递性 if (x.equals(y) && y.equals(z))
    x.equals(z); // true;
- 一致性 多次调用 equals() 方法结果不变 x.equals(y) == x.equals(y); // true
- 与 null 的比较
 对任何不是 null 的对象 x 调用 x.equals(null) 结果都为 false, x.equals(null); // false;
### 2. 等价与相等

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。

### 3. 实现
- 检查是否为同一个对象的引用，如果是直接返回 true；
- 检查是否是同一个类型，如果不是，直接返回 false；
- 将 Object 对象进行转型；
- 判断每个关键域是否相等。

## hashCode()
等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价，这是因为计算哈希值具有随机性，两个值不同的对象可能计算出相同的哈希值。

在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象哈希值也相等

## toString()
默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

## clone()
### 1. cloneable
clone() 是 Object 的 protected 方法，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。

实现Cloneable接口后可用重写clone()方法
### 2. 浅拷贝
拷贝对象和原始对象的引用类型引用同一个对象。

### 3. 深拷贝

### 4. clone()替代方案
使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

# 继承
## 访问权限
- private
- public
- protected
## 抽象类与接口
- abstract
- interface
### 抽象类
抽象类不能实例化，只能被继承

类中有抽象方法，类必须是抽象类
### 接口

## super
- 访问父类的构造函数：可以使用 super() 函数访问父类的构造函数，从而委托父类完成一些初始化的工作。应该注意到，子类一定会调用父类的构造函数来完成初始化工作，一般是调用父类的默认构造函数，如果子类需要调用父类其它构造函数，那么就可以使用 super() 函数。
- 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现。

## 重写与重载
### @Override
构造方法调用级别
- this.func(this)
- super.func(this)
- this.func(super)
- super.func(super)
### Overload
# 反射
# 异常
- Throwable
    - Error
    - Exception

# 泛型
```
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```
[10道java泛型面试题](https://cloud.tencent.com/developer/article/1033693)
# 注解
Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。
[注解 Annotation 实现原理与自定义注解例子](https://www.cnblogs.com/acm-bingzi/p/javaAnnotation.html)
