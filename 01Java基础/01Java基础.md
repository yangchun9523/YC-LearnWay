## Java基础
### 1.常用类源码学习(基于JDK1.8)
[String源码学习](String源码学习.md)  
[Integer源码学习](Integer源码学习.md)  
[Enum源码学习](Enum源码学习.md)  
[BigDecimal源码学习](BigDecimal源码学习.md)  
[ThreadLocal源码学习](ThreadLocal源码学习.md)  
[ClassLoader源码学习](ClassLoader源码学习.md)

#### Java数据结构源码学习

- list  
	[ArrayList源码学习](ArrayList源码学习.md)  
	[LinkedList源码学习](LinkedList源码学习.md)  
	[CopyOnWriteArrayList源码学习](CopyOnWriteArrayList源码学习.md)
- map  
	[HashMap源码学习](HashMap源码学习.md)  
	[LinkedHashMap源码学习](LinkedHashMap源码学习.md)  
	[ConcurrentHashMap源码学习](ConcurrentHashMap源码学习.md)  
- set  
	HashSet源码学习,内部使用HashMap<E,Object>  

### 2.变量类型

- 局部变量
	- 没有默认值,声明后必须初始化才能使用。
	- 局部变量在方法、构造方法、或者语句块被执行的时候创建,当它们执行完成后,变量将会被销毁。
- 实例变量 (成员变量)
	- 有默认值,数值型变量的默认值是0,布尔型变量的默认值是false,引用类型变量的默认值是null。
	- 实例变量在对象创建的时候创建,在对象被销毁的时候销毁。
- 静态变量 (类变量)
	- 有默认值与实例变量一致。
	- 静态变量在程序开始时创建,在程序结束时销毁。
	- 类变量被声明为public static final类型时，类变量名称必须使用大写字母。

### 3.String各种函数
### 3.1 substring的原理
jdk1.7之后，substring方法通过原字符串创建了一个新的String对象。
### 3.2 replaceFirst、replaceAll、replace的区别
replaceFirst 匹配并替换第一个命中的，参数是正则表达式  
replaceAll 匹配并替换所有命中的，参数是正则表达式  
replace 参数为字符时，是循环替换字符。参数为字符集时和replaceAll相似,具体实现使用正则。  
### 3.3 String对“+”的重载
反编译可以看出，String对“+”的支持其实就是使用了StringBuilder以及他的append、toString两个方法。
### 3.4 String.valueOf和Integer.toString的区别
`Integer.toString(int i)`该方法返回指定整数的有符号位的String对象，以10进制字符串形式返回   
`String.valueOf()`有很多重载方法,其中以`int`为参数时，实际调用了`Integer.toString`方法。  
### 3.5 String的不可变性
源码上`private final char value[];`String内部实际存储数据的属性 char数组 不可变。

### 4.自动拆装箱
Java是一种强类型语言，第一次声明变量必须说明数据类型，第一次变量赋值称为变量的初始化。 
Java提供了基本数据类型，这种数据的变量不需要使用new创建，他们不会在堆上创建，而是直接在栈内存中存储，因此会更加高效。   
Java基本类型共有八种，基本类型可以分为三类：  
 
- 字符类型char
- 布尔类型boolean
- 整数类型byte、short、int、long
- 浮点数类型float、double

Java语言是一个面向对象的语言，但是Java中的基本数据类型却是不面向对象的。这在实际使用时存在很多的不便(不具有对象的性质<属性、方法>)，为了解决这个不足，在设计类时为每个基本数据类型设计了一个对应的类进行代表，这样八个和基本数据类型对应的类统称为包装类(Wrapper Class)。

基本数据类型|包装类
:---:|:---:
byte|Byte
boolean|Boolean
short|Short
char|Character
int|Integer
long|Long
float|Float
double|Double

#### 4.1自动拆箱与自动装箱
自动装箱: 就是将基本数据类型自动转换成对应的包装类。  
自动拆箱：就是将包装类自动转换成对应的基本数据类型。  

```java
Integer i =10;  //自动装箱
int b= i;     //自动拆箱
```
原理

```java 
Integer integer=Integer.valueOf(10);  //自动装箱
int i=integer.intValue(); //自动拆箱
```
#### 4.2Integer的缓存机制
缓存机制作用: 节省内存、提升性能。
源码部分:

```java
	public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

### 5.熟悉Java中各种关键字
- transient 让被修饰的成员变量不被序列化
- instanceof 用来在运行时指出对象是否是特定类的一个实例
- volatile 被修饰的成员变量在每次被线程访问时，都强迫从主内存中重读该成员变量的值。而当成员变量发生变化时，强迫线程将变化值回写到主内存。
- synchronized 确保线程互斥的访问同步代码
- final 被修饰的 变量不可改变|方法不可覆盖|类不可继承
- static 被修饰为变量静态变量|方法为静态方法

### 集合类
常用集合类的使用

ArrayList和LinkedList和Vector的区别 

SynchronizedList和Vector的区别

HashMap、HashTable、ConcurrentHashMap区别

Java 8中stream相关用法

apache集合处理工具类的使用

不同版本的JDK中HashMap的实现的区别以及原因

### 枚举
枚举的用法、枚举与单例、Enum类

### Java IO&Java NIO，并学会使用
bio、nio和aio的区别、三种IO的用法与原理、netty

### Java反射与javassist
反射与工厂模式、 java.lang.reflect.*

### Java序列化
什么是序列化与反序列化、为什么序列化

序列化底层原理

序列化与单例模式

protobuf

为什么说序列化并不安全

### 注解
元注解、自定义注解、Java中常用注解使用、注解与反射的结合

### JMS
什么是Java消息服务、JMS消息传送模型


### 泛型
泛型与继承

类型擦除

泛型中K T V E  

object等的含义、泛型各种用法

### 单元测试
junit、mock、mockito、内存数据库（h2）

### 正则表达式
java.lang.util.regex.*

### 常用的Java工具库
commons.lang, commons.*... guava-libraries netty

### 什么是API&SPI
### 异常
异常类型、正确处理异常、自定义异常

### 时间处理
时区、时令、Java中时间API

### 编码方式
解决乱码问题、常用编码方式

### 语法糖
Java中语法糖原理、解语法糖
