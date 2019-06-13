## Java8中的Streams API详解
### 一、Stream简介
Java8中的 Stream 是对**集合**（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的**聚合操作**（aggregate operation），或者**大批量数据操作** (bulk data operation)。  
Stream API借助于同样新出现的 **Lambda** 表达式，极大的提高编程效率和程序可读性。同时它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用 fork/join 并行方式来拆分任务和加速处理过程。  
Java 8中首次出现的 java.util.stream 是一个**函数式语言+多核**时代综合影响的产物。

#### 1.1、聚合操作
在传统的 J2EE 应用中，Java 代码经常不得不依赖于关系型数据库的聚合操作来完成诸如：   

- 客户每月平均消费金额  
- 最昂贵的在售商品  
- 本周完成的有效订单（排除了无效的）  
- 取十个数据样本作为首页推荐  

而脱离数据库时，Java的集合 API 中，仅仅有极少量的辅助型方法，更多的时候是程序员需要用 Iterator 来遍历集合，完成相关的聚合应用逻辑。这是一种远不够高效、笨拙的方法。  
使用Java8 中的 Stream，代码更加简洁易读；而且使用并发模式，程序执行速度更快。

#### 1.2、Stream特点
Stream 就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。  

流的操作类型分为**转换操作**、**终结操作**两种：

- Intermediate：一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。
- Terminal：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。
- short-circuiting:对于一个 terminal 操作，如果它接受的是一个无限大的 Stream，但能在有限的时间计算出结果。(用于无限大流)

在对于一个 Stream 进行多次转换操作 (Intermediate 操作)，每次都对 Stream 的每个元素进行转换，而且是执行多次，这样时间复杂度就是 N（转换次数）个 for 循环里把所有操作都做掉的总和吗？其实不是这样的，转换操作都是 **lazy** 的，多个转换操作只会在 Terminal 操作的时候融合起来，一次循环完成。  
我们可以这样简单的理解，Stream 里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在 Terminal 操作的时候循环 Stream 对应的集合，然后对每个元素执行所有的函数。

### 二、Stream使用
#### 2.1、Stream的创建
```java
	// 1.通过数组创建
	int[] arr = new int[] { 1, 2, 3, 4, 5 };
	IntStream stream0 = Arrays.stream(arr);
	// 2.自有方法
	Stream<Integer> stream1 = Stream.of(1, 2, 3, 4, 5);
	// 3.通过集合
	List<String> strs = Arrays.asList("1", "2", "3", "4");
	Stream<String> stream2 = strs.stream();
	// 4.无限流
	Stream<String> stream3 = Stream.generate(() -> "test" + (int)(Math.random() * 10));
	// 5.无限流
	Stream<Integer> stream4 = Stream.iterate(0, x -> x + 1);
```

#### 2.2、Stream的操作
Stream的操作总览：  

- Intermediate：  
map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

- Terminal：  
forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

- Short-circuiting：  
anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit

#### 2.2.1、map
它的作用就是把 input Stream 的每一个元素，映射成 output Stream 的另外一个元素。

```java
	// 将小写String转为大写
	Stream<String> stream = Stream.of("a", "b", "c", "d");
	stream.map(str -> str.toUpperCase()).forEach(System.out::print);
	// 输出:ABCD
```

#### 2.2.2、flatMap
flatMap的作用是将多个Stream连接成一个Stream，与map有所区别，这是重新生成一个Stream对象取而代之。

```java
	// 将多个数据集 合并成一个处理
	List<String> strs1 = Arrays.asList("a", "b", "c", "d");
	List<String> strs2 = Arrays.asList("e", "f", "g", "h");
	Stream.of(strs1, strs2).flatMap(str -> str.stream()).forEach(System.out::print);
	// 输出:abcdefgh
```

#### 2.2.3、filter
filter 对原始 Stream 进行某项过滤，通过过滤的元素被留下来生成一个新 Stream。

```java
	// 过滤 >= 3的元素
	Stream.of(1, 2, 3, 4, 5).filter(i -> i >= 3).forEach(System.out::print);
	// 输出:345
```

#### 2.2.4、forEach
forEach 方法接收一个 Lambda 表达式，然后在 Stream 的每一个元素上执行该表达式。

```java
	// 过滤 >= 3的元素
	Stream.of(1, 2, 3, 4, 5).forEach(System.out::print);
	// 输出:12345
```
#### 2.2.5、peek
peek 作用是在 Stream 处理流程中，处理元素

```
Stream.of("a", "b", "c", "d").peek(e -> System.out.println("origin value: " + e))
		.map(str -> str.toUpperCase()).peek(e -> System.out.println("maped value: " + e))
		.collect(Collectors.toList());
// 输出：
origin value: a
maped value: A
origin value: b
maped value: B
origin value: c
maped value: C
origin value: d
maped value: D
```
#### 2.2.5、findFirst
findFirst是一个 termimal 兼 short-circuiting 操作，它总是返回 Stream 的第一个元素，或者空。

```java
Optional<String> x = Stream.of("a", "b", "c", "d").findFirst();
System.out.println(x.orElse("A"));
// 输出:a
```

#### 2.2.6、reduce
主要作用是把 Stream 元素组合起来。它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面 Stream 的第一个、第二个、第 n 个元素组合。从这个意义上说，字符串拼接、数值的 sum、min、max、average 都是特殊的 reduce。

```java
	String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat); 
	System.out.println(concat);
```

#### 2.2.7、limit/skip
limit 返回 Stream 的前面 n 个元素；skip 则是扔掉前 n 个元素

```java
	Stream.of("A", "B", "C", "D").limit(3).skip(1).forEach(System.out::print);
	\\输出:BC
```

#### 2.2.8、sorted
对 Stream 的排序通过 sorted 进行，它比数组的排序更强之处在于你可以首先对 Stream 进行各类 map、filter、limit、skip 甚至 distinct 来减少元素数量后，再排序，这能帮助程序明显缩短执行时间。

```java
Stream.of("A", "B", "C", "D").limit(3).skip(1).sorted((x,y) -> y.compareTo(x)).forEach(System.out::print);
\\ 输出:CB
```

#### 2.2.9、min/max/distinct
min 和 max 的功能也可以通过对 Stream 元素先排序，再 findFirst 来实现，但前者的性能会更好，为 O(n)。

```java
	// 去重
	Stream.of("A", "B", "C", "D", "A").distinct().forEach(System.out::print);
	System.out.println();
	// 最大
	System.out.println(Stream.of("A", "B", "C", "D", "A").max((x,y) -> x.compareTo(y)).get());
	// 最小
	System.out.println(Stream.of("A", "B", "C", "D", "A").min((x,y) -> x.compareTo(y)).get());
```

#### 2.2.10、Match
Stream 有三个 match 方法，它们都不是要遍历全部元素才能返回结果。从语义上说：

- allMatch：Stream 中全部元素符合传入的 predicate，返回 true
- anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true
- noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true

### 三、Stream进阶
#### 3.1 groupingBy/partitioningBy

```java
public class GroupAndPart {
	static class User{
		String name;
		int age;
		char sex;
		public User(String name, int age, char sex) {
			this.name = name;
			this.age = age;
			this.sex = sex;
		}
	}
	
	public static void main(String[] args) {
		// create data
		User user1 = new User("yang", 11, 'Y');
		User user2 = new User("yang1", 12, 'M');
		User user3 = new User("yang2", 11, 'Y');
		User user4 = new User("yang3", 13, 'Y');
		User user5 = new User("yang4", 11, 'M');
		User user6 = new User("yang5", 13, 'Y');
		List<User> list = new ArrayList<>();
		list.add(user1);
		list.add(user2);
		list.add(user3);
		list.add(user4);
		list.add(user5);
		list.add(user6);
		list.add(user2);
		list.add(user4);
		
		// 排序之后，按 年龄 分组
		LinkedHashMap<Integer, List<User>> ageGroup = list.stream().sorted((x,y) -> x.age -y.age).collect(Collectors.groupingBy(user -> user.age, LinkedHashMap::new, Collectors.toList()));
		ageGroup.forEach((k,v) -> System.out.println(k + ":" + v.size()));
		
		// 按照 年龄大小分区
		Map<Boolean, List<User>> agepart = list.stream().collect(Collectors.partitioningBy(user -> user.age > 12, Collectors.toList()));
		agepart.forEach((k,v) -> System.out.println(k + ":" + v.size()));
	}
}
```

### 四、总结
总之，Stream 的特性可以归纳为：

1. 不是数据结构
2. 它没有内部存储，它只是用操作管道从 source（数据结构、数组、generator function、IO channel）抓取数据。
3. 它也绝不修改自己所封装的底层数据结构的数据。例如 Stream 的 filter 操作会产生一个不包含被过滤元素的新 Stream，而不是从 source 删除那些元素。
4. 所有 Stream 的操作必须以 lambda 表达式为参数
5. 不支持索引访问
6. 很容易生成数组或者 List
7. 惰性化
8. 并行能力
9. 当一个 Stream 是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的。
10. 可以是无限的