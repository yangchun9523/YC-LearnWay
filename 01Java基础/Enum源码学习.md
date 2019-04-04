## Java Enum类
### Enum简介

enum 的全称为 enumeration， 是 JDK 1.5  中引入的新特性，存放在 java.lang 包中。

### enum关键词

```java
public enum Size{ SMALL, MEDIUM, LARGE };

# 反编译可以Class得到:
public final class Size extends java.lang.Enum{ 
    public static final Size SMALL; 
    public static final Size MEDIUM; 
    public static final Size LARGE; 
	
```
**实际上，这个声明定义的类型是一个类，它刚好有四个实例，在此尽量不要构造新对象。**  
比较两个枚举类型的值,不需要调用equals方法直接使用"=="就可以了。两者等价。  
Java Enum类型的语法结构尽管和java类的语法不一样，应该说差别比较大。
但是经过编译器编译之后产生的是一个class文件。
该class文件经过反编译可以看到实际上是生成了一个类，该类继承了java.lang.Enum<E>。

### enum用法

#### 1.作为常量
```java
public enum Color {  
  RED, GREEN, BLANK, YELLOW  
}
```

#### 2.switch及遍历
```java
# 遍历
for (Color e : Color.values()) {
	System.out.println(e.toString());
}

# switch
Color color = Color.RED;
switch(color){
	case RED:
	 System.out.println("这是");红色
	 break;
	 
	 // ... ...
}
```

#### 3.枚举中添加新方法

```java
# 如果打算自定义自己的方法，那么必须在enum实例序列的最后添加一个分号。而且 Java 要求必须先定义 enum 实例。

public enum Color {
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);
    // 成员变量
    private String name;
    private int index;

    // 构造方法
    private Color(String name, int index) {
        this.name = name;
        this.index = index;
    }

    // 普通方法
    public static String getName(int index) {
        for (Color c : Color.values()) {
			if (c.getIndex() == index) {
				return c.name;
			}
        }
        return null;
    }
    // get set 方法
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getIndex() {
        return index;
    }
    public void setIndex(int index) {
        this.index = index;
    }
 }
```
#### 4.Java不支持多继承，可以通过实现接口组织枚举类

```java
public interface Food {
	enum Coffee implements Food {
		BLACK_COFFEE, DECAF_COFFEE, LATTE, CAPPUCCINO
	}

	enum Dessert implements Food {
		FRUIT, CAKE, GELATO
	}
}
```

### EnumSet，EnumMap 的应用

```java 
# 使用EnumSet代替标志。enum要求其成员都是唯一的，但是enum中不能删除添加元素。可以使用allof()方法
	EnumSet<EnumTest> weekSet = EnumSet.allOf(EnumTest.class);
	for (EnumTest day : weekSet) {
		System.out.println(day);
	}
 
# EnumMap的key是enum，value是任何其他Object对象。
	EnumMap<EnumTest, String> weekMap = new EnumMap(EnumTest.class);
	weekMap.put(EnumTest.MON, "星期一");
	weekMap.put(EnumTest.TUE, "星期二");
	// ... ...
	for (Iterator<Entry<EnumTest, String>> iter = weekMap.entrySet().iterator(); iter.hasNext();) {
		Entry<EnumTest, String> entry = iter.next();
		System.out.println(entry.getKey().name() + ":" + entry.getValue());
	}
```





