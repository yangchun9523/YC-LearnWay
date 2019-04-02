## String源码学习
学习String源码之前先了解变量的声明及赋值

```java
Person per = null;  //声明
per = new Person();  //实例化操作，通过关键字new来实例化
Person per = new Person();//声明并实例化
```

了解Jvm虚拟机内存块及对象空间存储分配(JVM篇详细学习)

```text
 1、栈：存放基本数据类型及对象变量的引用，对象本身不存放于栈中而是存放于堆中
	1）、基础类型 byte (8位)、boolean (1位)、char (16位)、int (32位)、short (16位)、float (32位)、double (64位)、long (64位)
	2）、java代码作用域中定义一个变量时，则java就在栈中为这个变量分配内存空间，当该变量退出该作用域时，java会自动释放该变量所占的空间
2、堆：new操作符的对象
	1）、new创建的对象和数组
	2）、在堆中分配的内存，由Java虚拟机的自动垃圾回收器来管理
3、静态域：static定义的静态成员变量
4、常量池：存放常量

# 你真的知道Java中boolean类型占用多少个字节吗？
case1: 1个bit
理由: boolean类型的值只有true和false两种逻辑值，在编译后会使用1和0来表示，这两个数在内存中只需要1位（bit）即可存储，位是计算机最小的存储单位。
case2: 1个字节
理由: 虽然编译后1和0只需占用1位空间，但计算机处理数据的最小单位是1个字节，1个字节等于8位，实际存储的空间是：用1个字节的最低位存储，其他7位用0填补，如果值是true的话则存储的二进制为：0000 0001，如果是false的话则存储的二进制为：0000 0000。
case3: 4个字节
理由: 《Java虚拟机规范》一书中的描述：“虽然定义了boolean这种数据类型，但是只对它提供了非常有限的支持。在Java虚拟机中没有任何供boolean值专用的字节码指令，Java语言表达式所操作的boolean值，在编译之后都使用Java虚拟机中的int数据类型来代替，而boolean数组将会被编码成Java虚拟机的byte数组，每个元素boolean元素占8位”。这样我们可以得出boolean类型占了单独使用是4个字节，在数组中又是1个字节。

总结: 显然第三条是更准确的说法，那虚拟机为什么要用int来代替boolean呢？为什么不用byte或short，这样不是更节省内存空间吗。大多数人都会很自然的这样去想，我同样也有这个疑问，经过查阅资料发现，使用int的原因是，对于当下32位的处理器（CPU）来说，一次处理数据是32位（这里不是指的是32/64位系统，而是指CPU硬件层面），具有高效存取的特点。这其实是运算效率和存储空间之间的博弈，两者都非常的重要。

```

### String类型
Java中String不是基本数据类型，而是一种特殊的类。String代表的是不可变的字符序列，为不可变对象，一旦被创建,就不能修改它的值。对于已经存在的String对象的修改都是重新创建一个新的对象,然后把新的值保存进去。

### 接口实现

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
```

- final 不可继承
- java.io.Serializable 序列化接口
- Comparable<String> 实例化对象可比较大小
- CharSequence 只读的字符序列接口，提供 length()\charAt()等有用方法

### 主要变量

```
/** The value is used for character storage. */
	private final char value[];
/** Cache the hash code for the string */
    private int hash; // Default to 0
```

value[]是存储String的内容的，即当使用String str = "abc";的时候，本质上，"abc"是存储在一个char类型的数组中的。 final 修饰说明String不可修改，只能创建新对象。
而hash是String实例化的hashcode的一个缓存。因为String经常被用于比较，比如在HashMap中。如果每次进行比较都重新计算hashcode的值的话，那无疑是比较麻烦的，而保存一个hashcode的缓存无疑能优化这样的操作。

### 内部类
`private static class CaseInsensitiveComparator`  
提供忽略大小写比较大小实现。

### 方法
1. 构造方法很多，设置编码可能要到。注意`@Deprecated`过时方法。
	
	```text
	# String 对象两种创建方式
	# 1.String 特殊的创建方式  a -> 常量池
	String a = "abc";
	# 2.对象常规的创建方式 b -> 堆内存
	String b = new String("abc");
	```
2. getChar方法,都调用了System.arraycopy()函数(许多数组的合并截取都会使用到)。

	```java
	void getChars(char dst[], int dstBegin) {
        System.arraycopy(value, 0, dst, dstBegin, value.length);
    }
    public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) 	{
        if (srcBegin < 0) {
            throw new StringIndexOutOfBoundsException(srcBegin);
        }
        if (srcEnd > value.length) {
            throw new StringIndexOutOfBoundsException(srcEnd);
        }
        if (srcBegin > srcEnd) {
            throw new StringIndexOutOfBoundsException(srcEnd - srcBegin);
        }
        System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
    }
	```
	
3. hashCode(),&equals() String经常为Map<key,V>的因素

```java 
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

4. substring()&concat() 进一步说明String不可又该的特点

```java
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
    return new String(buf, true);
}
```

5. replace()&replaceAll 都是替换字符，replaceAll前一个参数是正则功能更强大。
6. split() 第一个参数也可以是正则，注意特殊字符转义(比如 . )
7. JDK 8 新方法 join

```java
/* <pre>{@code
 *     String message = String.join("-", "Java", "is", "cool");
 *     // message returned is: "Java-is-cool"
 * }</pre></blockquote>
 */
public static String join(CharSequence delimiter, CharSequence... elements) {
        Objects.requireNonNull(delimiter);
        Objects.requireNonNull(elements);
        // Number of elements not likely worth Arrays.stream overhead.
        StringJoiner joiner = new StringJoiner(delimiter);
        for (CharSequence cs: elements) {
            joiner.add(cs);
        }
        return joiner.toString();
    }
```
8. format()方法，有性能需求慎用。
9. native方法intern()

```
由于jdk1.7中将字符串常量池改为存放在堆中，因此intern()方法的实现原理相对以前的版本也有所改变。
　jdk1.6中字符串常量池存放在永久代中：jdk1.6中只能查询或创建在字符串常量池；
　jdk1.7中字符串常量池存放在堆中：jdk1.7中会先查询字符串常量池，若没有又会到堆中再去查询并存储堆的引用，然后返回。
总结:
jdk1.6的环境下使用intern()方法后，String对象只会引用或创建在字符串常量池中的对象。
jdk1,7的环境下使用intern()方法后，String对象需要注意所引用的是字符串常量池中的还是堆中的对象。
然后intern()方法的作用上，用一句话概括的话就是：intern()方法设计的初衷就是为了重用String对象，以节省内存消耗。
```
10. StringBuilder对String运算符 +和+= 重载了。

```text
String是不可变的，即是说String a = "a" ； a + "b" 之后，a还是”a“，这个”ab“实际上又生成了一个新的String对象。
将这个赋值语句剖析一下：
	"a" + "b" + "c"：先计算前两个  "a" + "b"：这时内存中其实有四个字符串，”a“,"b","c","ab"，然后再+”c“，这是内存中有五个String：”a“,"b","c","ab"，”abc“。
通过javap来反编译上面的赋值语句，会发现，在编译本条语句时，编译器会自作主张的引入了StringBuilder，并调用了StringBuilder.append方法，这样就不用再生成多余的字符串了。
```

11. String对象正的不可变？(java反射机制)

final:如果修饰的成员变量是基本类型，则表示这个变量的值不能改变。
final:如果修饰的成员变量是一个引用类型，则是说这个引用的地址的值不能修改，但是这个引用所指向的对象里面的内容还是可以改变的。

从上文可知String的成员变量是private final 的，也就是初始化之后不可改变。那么在这几个成员中， value比较特殊，因为他是一个引用变量，而不是真正的对象。value是final修饰的，也就是说final不能再指向其他数组对象，那么我能改变value指向的数组吗？ 比如将数组中的某个位置上的字符变为下划线“_”。 至少在我们自己写的普通代码中不能够做到，因为我们根本不能够访问到这个value引用，更不能通过这个引用去修改数组。 那么用什么方式可以访问私有成员呢？ 没错，用反射， 可以反射出String对象中的value属性， 进而改变通过获得的value引用改变数组的结构。下面是实例代码： 

```java
//创建字符串"Hello World"， 并赋给引用s
String s = "Hello World"; 
 
System.out.println("s = " + s); //Hello World
 
//获取String类中的value字段
Field valueFieldOfString = String.class.getDeclaredField("value");
 
//改变value属性的访问权限
valueFieldOfString.setAccessible(true);
 
//获取s对象上的value属性的值
char[] value = (char[]) valueFieldOfString.get(s);
 
//改变value所引用的数组中的第5个字符
value[5] = '_';
 
System.out.println("s = " + s);  //Hello_World
}
```