## JAVA 反射机制
### 一、反射机制简介
反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；  
对于任意一个对象，都能够调用它的任意一个方法和属性；  
这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

### 二、反射机制使用场景
- 在运行时判断任意一个对象所属的类；  
- 在运行时构造任意一个类的对象；  
- 在运行时判断任意一个类所具有的成员变量和方法；  
- 在运行时调用任意一个对象的方法；  
- 生成动态代理。  

### 三、反射机制主要属性
- java.lang.Class;  // 类  
- java.lang.reflect.Constructor;  //构造方法  
- java.lang.reflect.Field;   // 类字段  
- java.lang.reflect.Method;   // 类方法

#### 3.1 获取类对象主要方法

```java
	// 创建Class类对象
	Class class1 = String.class;
	Class class2 = (new String()).getClass() ;
	Class class3 = Class.forName("java.lang.String");
	// 创建类实例对象
	class1..newInstance();
```

#### 3.2 获取类所有属性

```java
	Field[] valueOfStrings = String.class.getDeclaredFields();
		for (Field field : valueOfStrings) {
			 System.out.println(field.getName());
            		 // 获取修饰权限符
            		 int mo = field.getModifiers();
			 System.out.println("mo: "+mo);
           		 String priv = Modifier.toString(mo);
            		 // 属性类型
            		 Class type = field.getType();
            		 System.out.println(priv + " " + type.getName() + " " + field.getName());
		}
```

#### 3.3 获取类所有方法并执行

```java
	public static void main(String[] args) throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException {
		String x = "hello world";
		System.out.println(x);
		//获取String类中的value字段
 		Field valueOfString = String.class.getDeclaredField("value");
		//改变value属性的访问权限(private)
		valueOfString.setAccessible(true);
		//获取x对象上的value属性的值
		char[] value = (char[]) valueOfString.get(x);
		value[5] = '_';
		System.out.println(x);
	}
```
