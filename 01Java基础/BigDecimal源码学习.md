## Java BigDecimal类
### 简介
Java中提供了大数字(超过16位有效位)的操作类,即 java.math.BinInteger 类和 java.math.BigDecimal 类,用于高精度计算.  
其中 BigInteger 类是针对大整数的处理类,而 BigDecimal 类则是针对大小数的处理类.  
float和Double只能用来做科学计算或者是工程计算;在商业计算中,对数字精度要求较高.

### 方法
#### 1.构造方法
```java 
# 进制使用
BigDecimal BigDecimal(double d); 
# 常用
BigDecimal BigDecimal(String s); 
# 常用
static BigDecimal valueOf(double d); 

System.out.println(0.1 + 0.2);
System.out.println(new BigDecimal(0.1));
System.out.println(BigDecimal.valueOf(0.1));
# 输出：
# 0.30000000000000004
# 0.1000000000000000055511151231257827021181583404541015625
# 0.1
```
#### 2.BigDecimal setScale(int newScale, int roundingMode)
```text
作用：格式化小数点，默认四舍五入。
#ROUND_UP	2.35 -> 2.4
#ROUND_DOWN 2.35 -> 2.3
#ROUND_HALF_UP 2.35 -> 2.4 & 2.34 -> 2.4
```

#### 4.常用算数方法
```java
public BigDecimal add(BigDecimal value);                        //加法
public BigDecimal subtract(BigDecimal value);                   //减法 
public BigDecimal multiply(BigDecimal value);                   //乘法
public BigDecimal divide(BigDecimal value);                     //除法
public BigInteger remainder(BigInteger val)                     //取余
public BigInteger negate()                                      //相反数
```