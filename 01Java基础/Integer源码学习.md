## Java Integer类

知识点:  

- 自动装箱、拆箱  
- 享元模式(设计模式)  

### Integer 简介
Integer类是基本数据类型int的包装器类，是抽象类Number的子类，位于java.lang包中。  
Integer类在对象中包装了一个基本类型int的值，也就是每个Integer对象包含一个int类型的字段。(private final int value;)

### 内部类

第一次使用时，缓存 -128 到 127(包含)的数据。(自动装箱数据)  
通过 -XX:AutoBoxCacheMax=<size> 设置最大的缓存范围。

```
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

### 方法

#### 1.parseInt(String s, int radix)
```
Examples:
 * parseInt("0", 10) returns 0
 * parseInt("473", 10) returns 473
 * parseInt("+42", 10) returns 42
 * parseInt("-0", 10) returns 0
 * parseInt("-FF", 16) returns -255
 * parseInt("1100110", 2) returns 102
 * parseInt("2147483647", 10) returns 2147483647
 * parseInt("-2147483648", 10) returns -2147483648
 * parseInt("2147483648", 10) throws a NumberFormatException
 * parseInt("99", 8) throws a NumberFormatException
 * parseInt("Kona", 10) throws a NumberFormatException
 * parseInt("Kona", 27) returns 411787
```

#### 2.toString(int i, int radix) 

#### 3.valueOf(int i)
```java
自动装箱的具体实现:
Integer num = 1;
Integer num = Integer. valueOf(1);
自动拆箱的具体实现:
Integer num = 1;
int x = num
int x = num.intValue()
```

#### 4.decode(String nm)

```text
不同进制的String文件转为Int
```

### 有意思的代码(享元模式的锅)
```java
public class IntegerSwap {
	public static void main(String[] args) throws Exception {
		Integer a = 1, b = 2;
		swapOne(a, b);
		System.out.println("a=" + a + ", b=" + b);
		swapTwo(a, b);
		System.out.println("a=" + a + ", b=" + b);
		Integer c = 1, d = 2;
		System.out.println("c=" + c + "; d=" + d);
	}

	// 常规swap
	static void swapOne(Integer a, Integer b) {
		Integer aTempValue = a;
		a = b;
		b = aTempValue;
	}

	// 使用反射swap
	static void swapTwo(Integer a1, Integer b1) throws Exception {
		Field valueField = Integer.class.getDeclaredField("value");
		valueField.setAccessible(true);
		int tempAValue = valueField.getInt(a1);
		valueField.setInt(a1, b1.intValue());
		valueField.setInt(b1, tempAValue);
	}
}
```